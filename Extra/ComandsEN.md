# Modbus Communication Protocol: EL.T01 Modular Electronic Load

## 1.0 Introduction

The EL.T01 Modular Electronic Load system is designed as a master-slave architecture using the Modbus RTU protocol. The main controller module (EL.T01.MBRD) serves as the Modbus Master, communicating with multiple specialized slave modules over a shared RS-485 bus. Each slave module is assigned a unique Modbus slave address (1-247) via the Sequential_ID register.

This documentation provides a comprehensive register map for all modules, enabling scalable control, monitoring, and configuration. The system supports adjustable and stable electronic loads, precise measurements, battery management, and user interfaces.

**Modbus Physical Layer Parameters:**
- Protocol: Modbus RTU
- Baud Rate: 115200
- Data Bits: 8
- Parity: Odd
- Stop Bits: 1
- Flow Control: None
- Bus Topology: RS-485 half-duplex

## 2.0 General Principles

### Addressing
Each slave module has its own independent register address space. To facilitate organization and future expansion, register addresses are divided into common and module-specific blocks. Common registers (applicable to all modules) use low address ranges (e.g., Coils 00001-00099, Input Registers 30001-30099). Module-specific registers start from higher blocks, allocated uniquely per module type for logical separation:
- EL.T01.PLAM: Coils 00100-00199, Discrete Inputs 10100-10199, Input Registers 30100-30199, Holding Registers 40100-40199
- EL.T01.PLSM: Coils 00200-00299, Discrete Inputs 10200-10299, Input Registers 30200-30299, Holding Registers 40200-40299
- EL.T01.MSCH: Coils 00300-00399, Discrete Inputs 10300-10399, Input Registers 30300-30399, Holding Registers 40300-40399
- EL.T01.KVMM: Coils 00400-00499, Discrete Inputs 10400-10499, Input Registers 30400-30499, Holding Registers 40400-40499
- EL.T01.CVCM: Coils 00500-00599, Discrete Inputs 10500-10599, Input Registers 30500-30599, Holding Registers 40500-40599
- EL.T01.UIM: Coils 00600-00699, Discrete Inputs 10600-10699, Input Registers 30600-30699, Holding Registers 40600-40699
- EL.T01.UPSM: Coils 00700-00799, Discrete Inputs 10700-10799, Input Registers 30700-30799, Holding Registers 40700-40799
- EL.T01.CHGM: Coils 00800-00899, Discrete Inputs 10800-10899, Input Registers 30800-30899, Holding Registers 40800-40899

This block allocation reserves ~100 addresses per category per module, allowing for future additions without overlap.

### Data Types
Registers are 16-bit wide. Multi-word data types span consecutive registers:
- UINT32/INT32: 32-bit unsigned/signed integer (2 registers, big-endian: high word first).
- FLOAT32: 32-bit IEEE 754 floating-point (2 registers, big-endian).
- STRING: ASCII characters, 2 per register (high byte first), null-terminated and padded.
Byte order (endianness): Big-endian (most significant byte first) for all multi-byte values.

### Scaling
To represent physical quantities precisely without floating-point hardware, scaling factors are used:
- Currents: Stored as INT32/UINT32 in 0.001 A units (mA resolution).
- Voltages: Stored as INT32/UINT32 in 0.01 V units (10 mV resolution).
- Powers: Stored as UINT32 in 0.1 W units.
- Resistances: Stored as UINT32 in 0.01 Ω units.
- Temperatures: Stored as INT16 in 0.1 °C units (e.g., 25.5 °C = 255).
- Capacities/Percentages: Stored as UINT16 in % units (0-100).
- Times: Stored as UINT16 in minutes or seconds, as specified.
- Enums and bitmasks: UINT16, with defined values/flags.

### Common Registers
The following registers are identical across all slave modules for consistency.

| Address Range | Register Name | R/W | Data Type & Scaling | Description |
|---------------|----------------|-----|-----------------------|-------------|
| 00001 | Enable | Read/Write | Coil (BOOL) | Turn the module's main function on (1) or off (0). |
| 00002 | Reset | Write | Coil (BOOL) | Trigger a software reset (write 1 to activate; auto-resets to 0). |
| 10001 | Ready | Read | Discrete Input (BOOL) | Module is operational and ready (1) or not (0). |
| 10002 | Overheated | Read | Discrete Input (BOOL) | Temperature exceeds limit (1). |
| 10003 | Overcurrent_Trip | Read | Discrete Input (BOOL) | Overcurrent protection tripped (1). |
| 10004 | Overvoltage_Trip | Read | Discrete Input (BOOL) | Overvoltage protection tripped (1). |
| 10005 | Fan_Failure | Read | Discrete Input (BOOL) | Fan malfunction detected (1). |
| 10006 | Calibrated | Read | Discrete Input (BOOL) | Module has valid calibration (1). |
| 10007 | Config_Saved | Read | Discrete Input (BOOL) | Configuration saved to non-volatile memory (1). |
| 30001 | Status_Bits | Read | UINT16 (bitmask) | Consolidated status bitmask: Bit 0: Ready, Bit 1: Overheated, Bit 2: Overcurrent_Trip, Bit 3: Overvoltage_Trip, Bit 4: Fan_Failure, Bit 5: Calibrated, Bit 6: Config_Saved, Bits 7-15: Reserved. |
| 30002 | Error_Code | Read | UINT16 | Error code (0=no error; see Appendix A for details). |
| 30003 | Primary_Parameter | Read | INT32 (scaling varies) | Main measured value (scaling depends on module: e.g., current in 0.001A for loads/shunts, voltage in 0.01V for voltage modules) (spans 30003-30004). |
| 30005 | Max_Value | Read | UINT32 (scaling varies) | Maximum design value for primary parameter (same scaling as Primary_Parameter) (spans 30005-30006). |
| 30007 | Absolute_ID_High | Read | UINT16 | High word of 32-bit factory-set serial number. |
| 30008 | Absolute_ID_Low | Read | UINT16 | Low word of 32-bit factory-set serial number. |
| 30009 | Firmware_Version | Read | UINT16 (0.01 units) | Firmware version (e.g., 123 = v1.23). |
| 30010 | Hardware_Version | Read | UINT16 (0.01 units) | Hardware version (e.g., 100 = v1.00). |
| 30011-30020 | Module_Name | Read | STRING (ASCII, 2 chars per reg) | Module name string (e.g., "EL.T01.PLAM\0"), padded with nulls (20 chars max). |
| 40001 | Sequential_ID | Read/Write | UINT16 | Modbus slave address (1-247). |
| 40002 | Command | Write | UINT16 | Command trigger: 0=noop, 1=reset, 2=clear_errors, 3=save_config, 4=start_calibration, 5=restore_defaults. |

## 3.0 Module Register Maps

### 3.1 EL.T01.PLAM - Adjustable Power Load

| Address Range | Register Name | R/W | Data Type & Scaling | Description |
|---------------|----------------|-----|-----------------------|-------------|
| 00001 | Enable | Read/Write | Coil (BOOL) | Turn the module's main function on (1) or off (0). Common to all modules. |
| 00002 | Reset | Write | Coil (BOOL) | Trigger a software reset (write 1 to activate; auto-resets to 0). Common to all modules. |
| 10001 | Ready | Read | Discrete Input (BOOL) | Module is operational and ready (1) or not (0). Part of expanded status. Common. |
| 10002 | Overheated | Read | Discrete Input (BOOL) | Temperature exceeds limit (1). Part of expanded status. Common. |
| 10003 | Overcurrent_Trip | Read | Discrete Input (BOOL) | Overcurrent protection tripped (1). Expanded for loads. |
| 10004 | Overvoltage_Trip | Read | Discrete Input (BOOL) | Overvoltage protection tripped (1). Added for safety. |
| 10005 | Fan_Failure | Read | Discrete Input (BOOL) | Fan malfunction detected (1). Assumes cooling fan; common for heat-generating modules. |
| 10006 | Calibrated | Read | Discrete Input (BOOL) | Module has valid calibration (1). Added for reliability. Common. |
| 10007 | Config_Saved | Read | Discrete Input (BOOL) | Configuration saved to non-volatile memory (1). Added for persistence. Common. |
| 30001 | Status_Bits | Read | UINT16 (bitmask) | Consolidated status bitmask: Bit 0: Ready, Bit 1: Overheated, Bit 2: Overcurrent_Trip, Bit 3: Overvoltage_Trip, Bit 4: Fan_Failure, Bit 5: Calibrated, Bit 6: Config_Saved, Bits 7-15: Reserved. Common, but expanded per module. |
| 30002 | Error_Code | Read | UINT16 | Error code (0=no error, 1=overheat, 2=overcurrent, etc.; define a table in docs). Common. |
| 30003 | Primary_Parameter | Read | INT32 (0.001 units) | Main measured value: Actual Current in 0.001A units (spans 30003-30004). Common, but for PLAM it's current. |
| 30005 | Max_Value | Read | UINT32 (0.001 units) | Maximum design value for primary parameter (current) in 0.001A units (spans 30005-30006). Common. |
| 30007 | Absolute_ID_High | Read | UINT16 | High word of 32-bit factory-set serial number. Common. |
| 30008 | Absolute_ID_Low | Read | UINT16 | Low word of 32-bit factory-set serial number. Common. |
| 30009 | Firmware_Version | Read | UINT16 (0.01 units) | Firmware version (e.g., 123 = v1.23). Added identification. Common. |
| 30010 | Hardware_Version | Read | UINT16 (0.01 units) | Hardware version (e.g., 100 = v1.00). Added identification. Common. |
| 30011-30020 | Module_Name | Read | STRING (ASCII, 2 chars per reg) | Module name string (e.g., "EL.T01.PLAM\0"), padded with nulls (20 chars max). Added identification. Common. |
| 30101 | Actual_Current | Read | INT32 (0.001A units) | Measured load current in 0.001A units (spans 30101-30102). PLAM-specific. |
| 30103 | Actual_Voltage | Read | INT32 (0.01V units) | Measured load voltage in 0.01V units (spans 30103-30104). Added as essential for loads. |
| 30105 | Actual_Power | Read | UINT32 (0.1W units) | Calculated power in 0.1W units (spans 30105-30106). PLAM-specific. |
| 30107 | Temperature | Read | INT16 (0.1°C units) | Module temperature in 0.1°C units. PLAM-specific, but common pattern. |
| 30108 | Fan_Speed | Read | UINT16 (RPM) | Fan speed in RPM (0 if no fan). Added for monitoring. |
| 40001 | Sequential_ID | Read/Write | UINT16 | Modbus slave address (1-247). Common. |
| 40002 | Command | Write | UINT16 | Command trigger: 0=noop, 1=reset, 2=clear_errors, 3=save_config, 4=start_calibration, 5=restore_defaults. Added for actions. Common. |
| 40101 | Load_Type | Read/Write | UINT16 (enum) | Load mode: 0=CC, 1=CV, 2=CR, 3=CP. PLAM-specific. |
| 40102-40103 | Target_Current | Read/Write | INT32 (0.001A units) | Target for CC mode in 0.001A units (spans 40102-40103). PLAM-specific. |
| 40104-40105 | Target_Voltage | Read/Write | INT32 (0.01V units) | Target for CV mode in 0.01V units (spans 40104-40105). PLAM-specific. |
| 40106-40107 | Target_Resistance | Read/Write | UINT32 (0.01Ω units) | Target for CR mode in 0.01Ω units (spans 40106-40107). Added for CR mode. |
| 40108-40109 | Target_Power | Read/Write | UINT32 (0.1W units) | Target for CP mode in 0.1W units (spans 40108-40109). PLAM-specific. |
| 40110-40111 | Max_Current_Limit | Read/Write | UINT32 (0.001A units) | Overcurrent protection limit in 0.001A units (spans 40110-40111). Added configuration. |
| 40112-40113 | Max_Voltage_Limit | Read/Write | UINT32 (0.01V units) | Overvoltage protection limit in 0.01V units (spans 40112-40113). Added configuration. |
| 40114 | Max_Temperature_Limit | Read/Write | INT16 (0.1°C units) | Overtemperature limit in 0.1°C units. Added configuration. |
| 40115-40116 | Calibration_Coeff_Current | Read/Write | FLOAT32 | Calibration multiplier for current (IEEE 754 float, spans 40115-40116). Added for accuracy. |
| 40117-40118 | Calibration_Coeff_Voltage | Read/Write | FLOAT32 | Calibration multiplier for voltage (spans 40117-40118). Added. |
| 40119 | Filter_Settings | Read/Write | UINT16 (samples) | Measurement averaging filter (e.g., 1=no filter, 16=average 16 samples). Added for noise reduction. |

### 3.2 EL.T01.PLSM - Stable Power Load

| Address Range | Register Name | R/W | Data Type & Scaling | Description |
|---------------|----------------|-----|-----------------------|-------------|
| 00001 | Enable | Read/Write | Coil (BOOL) | Turn the module's main function on (1) or off (0). |
| 00002 | Reset | Write | Coil (BOOL) | Trigger a software reset (write 1 to activate; auto-resets to 0). |
| 10001 | Ready | Read | Discrete Input (BOOL) | Module is operational and ready (1) or not (0). |
| 10002 | Overheated | Read | Discrete Input (BOOL) | Temperature exceeds limit (1). |
| 10003 | Overcurrent_Trip | Read | Discrete Input (BOOL) | Overcurrent protection tripped (1). |
| 10004 | Overvoltage_Trip | Read | Discrete Input (BOOL) | Overvoltage protection tripped (1). |
| 10005 | Fan_Failure | Read | Discrete Input (BOOL) | Fan malfunction detected (1). |
| 10006 | Calibrated | Read | Discrete Input (BOOL) | Module has valid calibration (1). |
| 10007 | Config_Saved | Read | Discrete Input (BOOL) | Configuration saved to non-volatile memory (1). |
| 30001 | Status_Bits | Read | UINT16 (bitmask) | Consolidated status bitmask: Bit 0: Ready, Bit 1: Overheated, Bit 2: Overcurrent_Trip, Bit 3: Overvoltage_Trip, Bit 4: Fan_Failure, Bit 5: Calibrated, Bit 6: Config_Saved, Bits 7-15: Reserved. |
| 30002 | Error_Code | Read | UINT16 | Error code (0=no error; see Appendix A). |
| 30003 | Primary_Parameter | Read | INT32 (0.001A units) | Main measured value: Actual Current in 0.001A units (spans 30003-30004). |
| 30005 | Max_Value | Read | UINT32 (0.001A units) | Maximum design value for primary parameter (current) in 0.001A units (spans 30005-30006). |
| 30007 | Absolute_ID_High | Read | UINT16 | High word of 32-bit factory-set serial number. |
| 30008 | Absolute_ID_Low | Read | UINT16 | Low word of 32-bit factory-set serial number. |
| 30009 | Firmware_Version | Read | UINT16 (0.01 units) | Firmware version (e.g., 123 = v1.23). |
| 30010 | Hardware_Version | Read | UINT16 (0.01 units) | Hardware version (e.g., 100 = v1.00). |
| 30011-30020 | Module_Name | Read | STRING (ASCII, 2 chars per reg) | Module name string (e.g., "EL.T01.PLSM\0"), padded with nulls (20 chars max). |
| 30201 | Actual_Current | Read | INT32 (0.001A units) | Measured load current in 0.001A units (spans 30201-30202). |
| 30203 | Actual_Voltage | Read | INT32 (0.01V units) | Measured load voltage in 0.01V units (spans 30203-30204). |
| 30205 | Actual_Power | Read | UINT32 (0.1W units) | Calculated power in 0.1W units (spans 30205-30206). |
| 30207 | Temperature | Read | INT16 (0.1°C units) | Module temperature in 0.1°C units. |
| 30208 | Fan_Speed | Read | UINT16 (RPM) | Fan speed in RPM (0 if no fan). |
| 40001 | Sequential_ID | Read/Write | UINT16 | Modbus slave address (1-247). |
| 40002 | Command | Write | UINT16 | Command trigger: 0=noop, 1=reset, 2=clear_errors, 3=save_config, 4=start_calibration, 5=restore_defaults. |
| 40201-40202 | Target_Power | Read/Write | UINT32 (0.1W units) | Target power for stable load in 0.1W units (spans 40201-40202). |
| 40203-40204 | Max_Current_Limit | Read/Write | UINT32 (0.001A units) | Overcurrent protection limit in 0.001A units (spans 40203-40204). |
| 40205-40206 | Max_Voltage_Limit | Read/Write | UINT32 (0.01V units) | Overvoltage protection limit in 0.01V units (spans 40205-40206). |
| 40207 | Max_Temperature_Limit | Read/Write | INT16 (0.1°C units) | Overtemperature limit in 0.1°C units. |
| 40208-40209 | Calibration_Coeff_Power | Read/Write | FLOAT32 | Calibration multiplier for power (spans 40208-40209). |
| 40210 | Filter_Settings | Read/Write | UINT16 (samples) | Measurement averaging filter (e.g., 1=no filter, 16=average 16 samples). |

### 3.3 EL.T01.MSCH - Shunt Module

| Address Range | Register Name | R/W | Data Type & Scaling | Description |
|---------------|----------------|-----|-----------------------|-------------|
| 00001 | Enable | Read/Write | Coil (BOOL) | Turn the module's main function on (1) or off (0). |
| 00002 | Reset | Write | Coil (BOOL) | Trigger a software reset (write 1 to activate; auto-resets to 0). |
| 10001 | Ready | Read | Discrete Input (BOOL) | Module is operational and ready (1) or not (0). |
| 10002 | Overheated | Read | Discrete Input (BOOL) | Temperature exceeds limit (1). |
| 10003 | Overcurrent_Trip | Read | Discrete Input (BOOL) | Overcurrent protection tripped (1). |
| 10004 | Overvoltage_Trip | Read | Discrete Input (BOOL) | Overvoltage protection tripped (1). |
| 10005 | Fan_Failure | Read | Discrete Input (BOOL) | Fan malfunction detected (1). |
| 10006 | Calibrated | Read | Discrete Input (BOOL) | Module has valid calibration (1). |
| 10007 | Config_Saved | Read | Discrete Input (BOOL) | Configuration saved to non-volatile memory (1). |
| 30001 | Status_Bits | Read | UINT16 (bitmask) | Consolidated status bitmask: Bit 0: Ready, Bit 1: Overheated, Bit 2: Overcurrent_Trip, Bit 3: Overvoltage_Trip, Bit 4: Fan_Failure, Bit 5: Calibrated, Bit 6: Config_Saved, Bits 7-15: Reserved. |
| 30002 | Error_Code | Read | UINT16 | Error code (0=no error; see Appendix A). |
| 30003 | Primary_Parameter | Read | INT32 (0.001A units) | Main measured value: Current in 0.001A units (spans 30003-30004). |
| 30005 | Max_Value | Read | UINT32 (0.001A units) | Maximum design value for current in 0.001A units (spans 30005-30006). |
| 30007 | Absolute_ID_High | Read | UINT16 | High word of 32-bit factory-set serial number. |
| 30008 | Absolute_ID_Low | Read | UINT16 | Low word of 32-bit factory-set serial number. |
| 30009 | Firmware_Version | Read | UINT16 (0.01 units) | Firmware version (e.g., 123 = v1.23). |
| 30010 | Hardware_Version | Read | UINT16 (0.01 units) | Hardware version (e.g., 100 = v1.00). |
| 30011-30020 | Module_Name | Read | STRING (ASCII, 2 chars per reg) | Module name string (e.g., "EL.T01.MSCH\0"), padded with nulls (20 chars max). |
| 30301 | Current | Read | INT32 (0.001A units) | Measured current in 0.001A units (spans 30301-30302). |
| 30303 | Temperature | Read | INT16 (0.1°C units) | Module temperature in 0.1°C units. |
| 40001 | Sequential_ID | Read/Write | UINT16 | Modbus slave address (1-247). |
| 40002 | Command | Write | UINT16 | Command trigger: 0=noop, 1=reset, 2=clear_errors, 3=save_config, 4=start_calibration, 5=restore_defaults. |
| 40301-40302 | Shunt_Resistance | Read/Write | UINT32 (0.0001Ω units) | Shunt resistance value in 0.0001Ω units (spans 40301-40302). |
| 40303-40304 | Calibration_Coeff_Current | Read/Write | FLOAT32 | Calibration multiplier for current (spans 40303-40304). |
| 40305 | Filter_Settings | Read/Write | UINT16 (samples) | Measurement averaging filter (e.g., 1=no filter, 16=average 16 samples). |
| 40306 | Max_Current_Limit | Read/Write | UINT32 (0.001A units) | Overcurrent detection threshold in 0.001A units (spans 40306-40307). |
| 40308 | Max_Temperature_Limit | Read/Write | INT16 (0.1°C units) | Overtemperature limit in 0.1°C units. |

### 3.4 EL.T01.KVMM - Kelvin Voltage Measure Module

| Address Range | Register Name | R/W | Data Type & Scaling | Description |
|---------------|----------------|-----|-----------------------|-------------|
| 00001 | Enable | Read/Write | Coil (BOOL) | Turn the module's main function on (1) or off (0). |
| 00002 | Reset | Write | Coil (BOOL) | Trigger a software reset (write 1 to activate; auto-resets to 0). |
| 10001 | Ready | Read | Discrete Input (BOOL) | Module is operational and ready (1) or not (0). |
| 10002 | Overheated | Read | Discrete Input (BOOL) | Temperature exceeds limit (1). |
| 10003 | Overcurrent_Trip | Read | Discrete Input (BOOL) | Overcurrent protection tripped (1). |
| 10004 | Overvoltage_Trip | Read | Discrete Input (BOOL) | Overvoltage protection tripped (1). |
| 10005 | Fan_Failure | Read | Discrete Input (BOOL) | Fan malfunction detected (1). |
| 10006 | Calibrated | Read | Discrete Input (BOOL) | Module has valid calibration (1). |
| 10007 | Config_Saved | Read | Discrete Input (BOOL) | Configuration saved to non-volatile memory (1). |
| 30001 | Status_Bits | Read | UINT16 (bitmask) | Consolidated status bitmask: Bit 0: Ready, Bit 1: Overheated, Bit 2: Overcurrent_Trip, Bit 3: Overvoltage_Trip, Bit 4: Fan_Failure, Bit 5: Calibrated, Bit 6: Config_Saved, Bits 7-15: Reserved. |
| 30002 | Error_Code | Read | UINT16 | Error code (0=no error; see Appendix A). |
| 30003 | Primary_Parameter | Read | INT32 (0.01V units) | Main measured value: Voltage in 0.01V units (spans 30003-30004). |
| 30005 | Max_Value | Read | UINT32 (0.01V units) | Maximum design value for voltage in 0.01V units (spans 30005-30006). |
| 30007 | Absolute_ID_High | Read | UINT16 | High word of 32-bit factory-set serial number. |
| 30008 | Absolute_ID_Low | Read | UINT16 | Low word of 32-bit factory-set serial number. |
| 30009 | Firmware_Version | Read | UINT16 (0.01 units) | Firmware version (e.g., 123 = v1.23). |
| 30010 | Hardware_Version | Read | UINT16 (0.01 units) | Hardware version (e.g., 100 = v1.00). |
| 30011-30020 | Module_Name | Read | STRING (ASCII, 2 chars per reg) | Module name string (e.g., "EL.T01.KVMM\0"), padded with nulls (20 chars max). |
| 30401 | Voltage | Read | INT32 (0.01V units) | Measured Kelvin voltage in 0.01V units (spans 30401-30402). |
| 30403 | Temperature | Read | INT16 (0.1°C units) | Module temperature in 0.1°C units. |
| 40001 | Sequential_ID | Read/Write | UINT16 | Modbus slave address (1-247). |
| 40002 | Command | Write | UINT16 | Command trigger: 0=noop, 1=reset, 2=clear_errors, 3=save_config, 4=start_calibration, 5=restore_defaults. |
| 40401 | Voltage_Range | Read/Write | UINT16 (enum) | Measurement range: 0=low (e.g., 0-10V), 1=high (e.g., 0-100V). |
| 40402-40403 | Calibration_Coeff_Voltage | Read/Write | FLOAT32 | Calibration multiplier for voltage (spans 40402-40403). |
| 40404 | Filter_Settings | Read/Write | UINT16 (samples) | Measurement averaging filter (e.g., 1=no filter, 16=average 16 samples). |
| 40405-40406 | Max_Voltage_Limit | Read/Write | UINT32 (0.01V units) | Overvoltage detection threshold in 0.01V units (spans 40405-40406). |
| 40407 | Max_Temperature_Limit | Read/Write | INT16 (0.1°C units) | Overtemperature limit in 0.1°C units. |

### 3.5 EL.T01.CVCM - Cell Voltage Control Module

| Address Range | Register Name | R/W | Data Type & Scaling | Description |
|---------------|----------------|-----|-----------------------|-------------|
| 00001 | Enable | Read/Write | Coil (BOOL) | Turn the module's main function on (1) or off (0). |
| 00002 | Reset | Write | Coil (BOOL) | Trigger a software reset (write 1 to activate; auto-resets to 0). |
| 10001 | Ready | Read | Discrete Input (BOOL) | Module is operational and ready (1) or not (0). |
| 10002 | Overheated | Read | Discrete Input (BOOL) | Temperature exceeds limit (1). |
| 10003 | Overcurrent_Trip | Read | Discrete Input (BOOL) | Overcurrent protection tripped (1). |
| 10004 | Overvoltage_Trip | Read | Discrete Input (BOOL) | Overvoltage protection tripped (1). |
| 10005 | Fan_Failure | Read | Discrete Input (BOOL) | Fan malfunction detected (1). |
| 10006 | Calibrated | Read | Discrete Input (BOOL) | Module has valid calibration (1). |
| 10007 | Config_Saved | Read | Discrete Input (BOOL) | Configuration saved to non-volatile memory (1). |
| 30001 | Status_Bits | Read | UINT16 (bitmask) | Consolidated status bitmask: Bit 0: Ready, Bit 1: Overheated, Bit 2: Overcurrent_Trip, Bit 3: Overvoltage_Trip, Bit 4: Fan_Failure, Bit 5: Calibrated, Bit 6: Config_Saved, Bits 7-15: Reserved. |
| 30002 | Error_Code | Read | UINT16 | Error code (0=no error; see Appendix A). |
| 30003 | Primary_Parameter | Read | INT32 (0.01V units) | Main measured value: Average cell voltage in 0.01V units (spans 30003-30004). |
| 30005 | Max_Value | Read | UINT32 (0.01V units) | Maximum design value for cell voltage in 0.01V units (spans 30005-30006). |
| 30007 | Absolute_ID_High | Read | UINT16 | High word of 32-bit factory-set serial number. |
| 30008 | Absolute_ID_Low | Read | UINT16 | Low word of 32-bit factory-set serial number. |
| 30009 | Firmware_Version | Read | UINT16 (0.01 units) | Firmware version (e.g., 123 = v1.23). |
| 30010 | Hardware_Version | Read | UINT16 (0.01 units) | Hardware version (e.g., 100 = v1.00). |
| 30011-30020 | Module_Name | Read | STRING (ASCII, 2 chars per reg) | Module name string (e.g., "EL.T01.CVCM\0"), padded with nulls (20 chars max). |
| 30501 | Cell_Count | Read | UINT16 | Number of cells monitored (e.g., 1-16). |
| 30502 | Temperature | Read | INT16 (0.1°C units) | Module temperature in 0.1°C units. |
| 30503-30534 | Cell_Voltages | Read | INT32[] (0.01V units) | Array of cell voltages, each INT32 spanning 2 registers (e.g., Cell 1: 30503-30504, Cell 2: 30505-30506, up to 16 cells). Unused cells read 0. |
| 40001 | Sequential_ID | Read/Write | UINT16 | Modbus slave address (1-247). |
| 40002 | Command | Write | UINT16 | Command trigger: 0=noop, 1=reset, 2=clear_errors, 3=save_config, 4=start_calibration, 5=restore_defaults. |
| 40501-40502 | Calibration_Coeff_Voltage | Read/Write | FLOAT32 | Global calibration multiplier for voltages (spans 40501-40502). |
| 40503 | Filter_Settings | Read/Write | UINT16 (samples) | Measurement averaging filter (e.g., 1=no filter, 16=average 16 samples). |
| 40504-40505 | Max_Voltage_Limit_Per_Cell | Read/Write | UINT32 (0.01V units) | Overvoltage limit per cell in 0.01V units (spans 40504-40505). |
| 40506 | Max_Temperature_Limit | Read/Write | INT16 (0.1°C units) | Overtemperature limit in 0.1°C units. |

### 3.6 EL.T01.UIM - User Interface Module

| Address Range | Register Name | R/W | Data Type & Scaling | Description |
|---------------|----------------|-----|-----------------------|-------------|
| 00001 | Enable | Read/Write | Coil (BOOL) | Turn the module's main function on (1) or off (0). |
| 00002 | Reset | Write | Coil (BOOL) | Trigger a software reset (write 1 to activate; auto-resets to 0). |
| 00601 | Buzzer_Enable | Read/Write | Coil (BOOL) | Activate buzzer (1=on, 0=off). |
| 10001 | Ready | Read | Discrete Input (BOOL) | Module is operational and ready (1) or not (0). |
| 10002 | Overheated | Read | Discrete Input (BOOL) | Temperature exceeds limit (1). |
| 10003 | Overcurrent_Trip | Read | Discrete Input (BOOL) | Overcurrent protection tripped (1). |
| 10004 | Overvoltage_Trip | Read | Discrete Input (BOOL) | Overvoltage protection tripped (1). |
| 10005 | Fan_Failure | Read | Discrete Input (BOOL) | Fan malfunction detected (1). |
| 10006 | Calibrated | Read | Discrete Input (BOOL) | Module has valid calibration (1). |
| 10007 | Config_Saved | Read | Discrete Input (BOOL) | Configuration saved to non-volatile memory (1). |
| 10601-10616 | Button_Status | Read | Discrete Input[] (BOOL) | Status of up to 16 buttons/keypad inputs (1=pressed, 0=not; e.g., 10601=Button1). |
| 30001 | Status_Bits | Read | UINT16 (bitmask) | Consolidated status bitmask: Bit 0: Ready, Bit 1: Overheated, Bit 2: Overcurrent_Trip, Bit 3: Overvoltage_Trip, Bit 4: Fan_Failure, Bit 5: Calibrated, Bit 6: Config_Saved, Bits 7-15: Reserved. |
| 30002 | Error_Code | Read | UINT16 | Error code (0=no error; see Appendix A). |
| 30003 | Primary_Parameter | Read | INT32 (N/A) | Reserved for future use (spans 30003-30004). |
| 30005 | Max_Value | Read | UINT32 (N/A) | Reserved for future use (spans 30005-30006). |
| 30007 | Absolute_ID_High | Read | UINT16 | High word of 32-bit factory-set serial number. |
| 30008 | Absolute_ID_Low | Read | UINT16 | Low word of 32-bit factory-set serial number. |
| 30009 | Firmware_Version | Read | UINT16 (0.01 units) | Firmware version (e.g., 123 = v1.23). |
| 30010 | Hardware_Version | Read | UINT16 (0.01 units) | Hardware version (e.g., 100 = v1.00). |
| 30011-30020 | Module_Name | Read | STRING (ASCII, 2 chars per reg) | Module name string (e.g., "EL.T01.UIM\0"), padded with nulls (20 chars max). |
| 30601 | Button_Bitmask | Read | UINT16 (bitmask) | Bitmask of all button statuses (Bit 0=Button1, etc.). |
| 40001 | Sequential_ID | Read/Write | UINT16 | Modbus slave address (1-247). |
| 40002 | Command | Write | UINT16 | Command trigger: 0=noop, 1=reset, 2=clear_errors, 3=save_config, 4=start_calibration, 5=restore_defaults. |
| 40601 | Backlight_Level | Read/Write | UINT16 (%) | Backlight brightness (0-100%). |
| 40602 | Buzzer_Duration | Read/Write | UINT16 (seconds) | Buzzer activation duration in seconds. |
| 40603-40612 | Display_Line_1 | Read/Write | STRING (ASCII, 2 chars per reg) | Text for LCD line 1 (20 chars max, spans 40603-40612). |
| 40613-40622 | Display_Line_2 | Read/Write | STRING (ASCII, 2 chars per reg) | Text for LCD line 2 (20 chars max, spans 40613-40622). |
| 40623-40632 | Display_Line_3 | Read/Write | STRING (ASCII, 2 chars per reg) | Text for LCD line 3 (20 chars max, spans 40623-40632). |
| 40633-40642 | Display_Line_4 | Read/Write | STRING (ASCII, 2 chars per reg) | Text for LCD line 4 (20 chars max, spans 40633-40642). |

### 3.7 EL.T01.UPSM - Uninterruptible Power Supply Module

| Address Range | Register Name | R/W | Data Type & Scaling | Description |
|---------------|----------------|-----|-----------------------|-------------|
| 00001 | Enable | Read/Write | Coil (BOOL) | Turn the module's main function on (1) or off (0). |
| 00002 | Reset | Write | Coil (BOOL) | Trigger a software reset (write 1 to activate; auto-resets to 0). |
| 10001 | Ready | Read | Discrete Input (BOOL) | Module is operational and ready (1) or not (0). |
| 10002 | Overheated | Read | Discrete Input (BOOL) | Temperature exceeds limit (1). |
| 10003 | Overcurrent_Trip | Read | Discrete Input (BOOL) | Overcurrent protection tripped (1). |
| 10004 | Overvoltage_Trip | Read | Discrete Input (BOOL) | Overvoltage protection tripped (1). |
| 10005 | Fan_Failure | Read | Discrete Input (BOOL) | Fan malfunction detected (1). |
| 10006 | Calibrated | Read | Discrete Input (BOOL) | Module has valid calibration (1). |
| 10007 | Config_Saved | Read | Discrete Input (BOOL) | Configuration saved to non-volatile memory (1). |
| 10701 | On_Battery | Read | Discrete Input (BOOL) | UPS is running on battery (1) or mains (0). |
| 10702 | Battery_Low | Read | Discrete Input (BOOL) | Battery capacity below threshold (1). |
| 10703 | Battery_Fault | Read | Discrete Input (BOOL) | Battery fault detected (1). |
| 30001 | Status_Bits | Read | UINT16 (bitmask) | Consolidated status bitmask: Bit 0: Ready, Bit 1: Overheated, Bit 2: Overcurrent_Trip, Bit 3: Overvoltage_Trip, Bit 4: Fan_Failure, Bit 5: Calibrated, Bit 6: Config_Saved, Bit 7: On_Battery, Bit 8: Battery_Low, Bit 9: Battery_Fault, Bits 10-15: Reserved. |
| 30002 | Error_Code | Read | UINT16 | Error code (0=no error; see Appendix A). |
| 30003 | Primary_Parameter | Read | INT32 (0.01V units) | Main measured value: Output Voltage in 0.01V units (spans 30003-30004). |
| 30005 | Max_Value | Read | UINT32 (0.01V units) | Maximum design value for output voltage in 0.01V units (spans 30005-30006). |
| 30007 | Absolute_ID_High | Read | UINT16 | High word of 32-bit factory-set serial number. |
| 30008 | Absolute_ID_Low | Read | UINT16 | Low word of 32-bit factory-set serial number. |
| 30009 | Firmware_Version | Read | UINT16 (0.01 units) | Firmware version (e.g., 123 = v1.23). |
| 30010 | Hardware_Version | Read | UINT16 (0.01 units) | Hardware version (e.g., 100 = v1.00). |
| 30011-30020 | Module_Name | Read | STRING (ASCII, 2 chars per reg) | Module name string (e.g., "EL.T01.UPSM\0"), padded with nulls (20 chars max). |
| 30701 | Remaining_Capacity | Read | UINT16 (%) | Battery remaining capacity (0-100%). |
| 30702 | Output_Voltage | Read | INT32 (0.01V units) | Output voltage in 0.01V units (spans 30702-30703). |
| 30704 | Input_Voltage | Read | INT32 (0.01V units) | Input (mains) voltage in 0.01V units (spans 30704-30705). |
| 30706 | Load_Current | Read | INT32 (0.001A units) | Load current in 0.001A units (spans 30706-30707). |
| 30708 | Temperature | Read | INT16 (0.1°C units) | Module temperature in 0.1°C units. |
| 30709 | Runtime_Remaining | Read | UINT16 (minutes) | Estimated runtime on battery in minutes. |
| 40001 | Sequential_ID | Read/Write | UINT16 | Modbus slave address (1-247). |
| 40002 | Command | Write | UINT16 | Command trigger: 0=noop, 1=reset, 2=clear_errors, 3=save_config, 4=start_calibration, 5=restore_defaults. |
| 40701 | Low_Battery_Threshold | Read/Write | UINT16 (%) | Low battery alarm threshold (0-100%). |
| 40702-40703 | Calibration_Coeff_Voltage | Read/Write | FLOAT32 | Calibration multiplier for voltages (spans 40702-40703). |
| 40704-40705 | Calibration_Coeff_Current | Read/Write | FLOAT32 | Calibration multiplier for current (spans 40704-40705). |
| 40706 | Filter_Settings | Read/Write | UINT16 (samples) | Measurement averaging filter (e.g., 1=no filter, 16=average 16 samples). |
| 40707 | Max_Temperature_Limit | Read/Write | INT16 (0.1°C units) | Overtemperature limit in 0.1°C units. |

### 3.8 EL.T01.CHGM - Battery Charger Module

| Address Range | Register Name | R/W | Data Type & Scaling | Description |
|---------------|----------------|-----|-----------------------|-------------|
| 00001 | Enable | Read/Write | Coil (BOOL) | Turn the module's main function on (1) or off (0). |
| 00002 | Reset | Write | Coil (BOOL) | Trigger a software reset (write 1 to activate; auto-resets to 0). |
| 10001 | Ready | Read | Discrete Input (BOOL) | Module is operational and ready (1) or not (0). |
| 10002 | Overheated | Read | Discrete Input (BOOL) | Temperature exceeds limit (1). |
| 10003 | Overcurrent_Trip | Read | Discrete Input (BOOL) | Overcurrent protection tripped (1). |
| 10004 | Overvoltage_Trip | Read | Discrete Input (BOOL) | Overvoltage protection tripped (1). |
| 10005 | Fan_Failure | Read | Discrete Input (BOOL) | Fan malfunction detected (1). |
| 10006 | Calibrated | Read | Discrete Input (BOOL) | Module has valid calibration (1). |
| 10007 | Config_Saved | Read | Discrete Input (BOOL) | Configuration saved to non-volatile memory (1). |
| 10801 | Charging | Read | Discrete Input (BOOL) | Charging in progress (1). |
| 10802 | Charge_Complete | Read | Discrete Input (BOOL) | Charge cycle complete (1). |
| 10803 | Battery_Fault | Read | Discrete Input (BOOL) | Battery fault detected (1). |
| 30001 | Status_Bits | Read | UINT16 (bitmask) | Consolidated status bitmask: Bit 0: Ready, Bit 1: Overheated, Bit 2: Overcurrent_Trip, Bit 3: Overvoltage_Trip, Bit 4: Fan_Failure, Bit 5: Calibrated, Bit 6: Config_Saved, Bit 7: Charging, Bit 8: Charge_Complete, Bit 9: Battery_Fault, Bits 10-15: Reserved. |
| 30002 | Error_Code | Read | UINT16 | Error code (0=no error; see Appendix A). |
| 30003 | Primary_Parameter | Read | INT32 (0.001A units) | Main measured value: Actual Charge Current in 0.001A units (spans 30003-30004). |
| 30005 | Max_Value | Read | UINT32 (0.001A units) | Maximum design value for charge current in 0.001A units (spans 30005-30006). |
| 30007 | Absolute_ID_High | Read | UINT16 | High word of 32-bit factory-set serial number. |
| 30008 | Absolute_ID_Low | Read | UINT16 | Low word of 32-bit factory-set serial number. |
| 30009 | Firmware_Version | Read | UINT16 (0.01 units) | Firmware version (e.g., 123 = v1.23). |
| 30010 | Hardware_Version | Read | UINT16 (0.01 units) | Hardware version (e.g., 100 = v1.00). |
| 30011-30020 | Module_Name | Read | STRING (ASCII, 2 chars per reg) | Module name string (e.g., "EL.T01.CHGM\0"), padded with nulls (20 chars max). |
| 30801 | Actual_Charge_Current | Read | INT32 (0.001A units) | Measured charge current in 0.001A units (spans 30801-30802). |
| 30803 | Actual_Charge_Voltage | Read | INT32 (0.01V units) | Measured charge voltage in 0.01V units (spans 30803-30804). |
| 30805 | Temperature | Read | INT16 (0.1°C units) | Module temperature in 0.1°C units. |
| 30806 | Charge_Stage | Read | UINT16 (enum) | Current charge stage: 0=Idle, 1=Bulk, 2=Absorption, 3=Float, 4=Equalize. |
| 30807 | Time_In_Stage | Read | UINT16 (minutes) | Time elapsed in current stage in minutes. |
| 40001 | Sequential_ID | Read/Write | UINT16 | Modbus slave address (1-247). |
| 40002 | Command | Write | UINT16 | Command trigger: 0=noop, 1=reset, 2=clear_errors, 3=save_config, 4=start_calibration, 5=restore_defaults, 6=start_charge, 7=stop_charge. |
| 40801 | Charge_Profile_ID | Read/Write | UINT16 (enum) | Charge profile: 0=Li-ion, 1=NiMH, 2=Lead-Acid, 3=Custom. |
| 40802-40803 | Charge_Current | Read/Write | INT32 (0.001A units) | Target charge current in 0.001A units (spans 40802-40803). |
| 40804-40805 | Charge_Voltage | Read/Write | INT32 (0.01V units) | Target charge voltage in 0.01V units (spans 40804-40805). |
| 40806 | Max_Charge_Time | Read/Write | UINT16 (minutes) | Maximum charge time limit in minutes. |
| 40807-40808 | Termination_Current | Read/Write | INT32 (0.001A units) | Termination current for absorption stage in 0.001A units (spans 40807-40808). |
| 40809-40810 | Calibration_Coeff_Current | Read/Write | FLOAT32 | Calibration multiplier for current (spans 40809-40810). |
| 40811-40812 | Calibration_Coeff_Voltage | Read/Write | FLOAT32 | Calibration multiplier for voltage (spans 40811-40812). |
| 40813 | Filter_Settings | Read/Write | UINT16 (samples) | Measurement averaging filter (e.g., 1=no filter, 16=average 16 samples). |
| 40814 | Max_Temperature_Limit | Read/Write | INT16 (0.1°C units) | Overtemperature limit in 0.1°C units. |

## 4.0 Appendix A: Error Codes

| Code | Description |
|------|-------------|
| 0 | No error |
| 1 | Over-temperature |
| 2 | Over-current |
| 3 | Over-voltage |
| 4 | Under-voltage |
| 5 | Fan failure |
| 6 | Calibration invalid |
| 7 | Configuration not saved |
| 8 | Communication timeout |
| 9 | Battery fault |
| 10 | Charge stage error |
| 11 | Invalid command |
| 12-65535 | Reserved for module-specific errors |

## 5.0 Appendix B: Communication Examples

All examples use Modbus RTU framing: [Slave Address] [Function Code] [Data] [CRC16 (low byte, high byte)].

Register addresses in frames are 0-based (e.g., Input Register 30101 is address 0x0064 in hex, since 30101 - 30001 = 100, but actually Modbus input regs start from 0 as 30001).

### Example 1: Reading Actual Current from PLAM (address 5)
- Function: 04 (Read Input Registers)
- Start Address: 30101 (0x0064 in frame, but wait: 30101 corresponds to internal addr 100, since 30000=0, but convention: 3xxxx means addr xxxx-1.
Standard: For Input Reg 3xxxx, frame addr = xxxx-1.
So for 30101, addr = 100 (0x0064).
Read 2 registers for INT32.
- Request: 05 04 00 64 00 02 [CRC]
- Response (example: Current = 5.000A = 5000 in 0.001A = 0x1388, big-endian 0x0000 0x1388): 05 04 04 00 00 13 88 [CRC]

### Example 2: Setting a 10.5W Constant Power load on PLSM (address 6)
- Function: 10 (Write Multiple Holding Registers)
- Start Address: 40201 (internal addr 201-1=200 = 0x00C8)
- Write 2 registers for UINT32 Target_Power = 10.5W = 105 in 0.1W = 0x00000069
- Request: 06 10 00 C8 00 02 04 00 00 00 69 [CRC]
- Response: 06 10 00 C8 00 02 [CRC]

### Example 3: Writing a charge profile to the EL.T01.CHGM module (address 7)
- Function: 06 (Write Single Holding Register)
- Register Address: 40801 (Charge_Profile_ID, internal addr 800 = 0x0320, since 40801 - 40001 = 800, 800 hex = 0x0320)
- Data: 0x0002 (enum value 2 for Lead-Acid profile)
- Request Frame (hex): 07 06 03 20 00 02 48 22 (Slave Addr: 07, Func: 06, Addr Hi: 03, Addr Lo: 20, Data Hi: 00, Data Lo: 02, CRC Lo: 48, CRC Hi: 22)
- Response Frame (hex): 07 06 03 20 00 02 48 22 (Echo of request, confirming write)

### Example 4: Reading the operational status from the EL.T01.MSCH module (address 3)
- Function: 04 (Read Input Registers)
- Start Address: 30001 (Status_Bits and Error_Code, internal addr 0 = 0x0000)
- Number of Registers: 2
- Request Frame (hex): 03 04 00 00 00 02 70 29 (Slave Addr: 03, Func: 04, Start Addr Hi: 00, Start Addr Lo: 00, Count Hi: 00, Count Lo: 02, CRC Lo: 70, CRC Hi: 29)
- Response Frame (hex, example data: Status_Bits=0x0001, Error_Code=0x0000): 03 04 04 00 01 00 00 89 84 (Slave Addr: 03, Func: 04, Byte Count: 04, Data: 00 01 00 00, CRC Lo: 89, CRC Hi: 84)

## 6.0 Optimized Software Implementation for STM32L010K4T6 Modbus Slave

This section provides an optimized C code example for implementing a Modbus RTU slave on the STM32L010K4T4 microcontroller. The code uses direct register manipulation with CMSIS device headers for minimal overhead, an event-driven approach with interrupts, and a state machine for efficient parsing. Assumptions: System clock at 32 MHz (HSI + PLL), USART2 on PA2 (TX)/PA3 (RX) with AF4, TIM2 for timeout (334 µs for 3.5 chars at 115200 baud). Global registers are static arrays in RAM, constants in Flash. CRC is computed with bitwise operations.

### 6.1 Optimized UART and Timer Configuration

```c
#include "stm32l010xx.h"  // CMSIS device header for register definitions

// Static globals for minimal stack use
static uint16_t holding_regs[100];  // Example holding registers (adjust size)
static uint16_t input_regs[100];    // Example input registers
static uint8_t modbus_buf[32];      // Small buffer for frames (max expected size)
static uint8_t buf_idx = 0;         // Buffer index
static uint8_t state = 0;           // State machine: 0=idle, 1=recv, 2=process
static const uint8_t slave_addr = 1; // Const in Flash

// CRC16 table (const in Flash for memory efficiency)
static const uint16_t crc_table[16] = {
  0x0000, 0xCC01, 0xD801, 0x1400, 0xF001, 0x3C00, 0x2800, 0xE401,
  0xA001, 0x6C00, 0x7800, 0xB401, 0x5000, 0x9C01, 0x8801, 0x4400
};

// Optimized CRC16 computation (bitwise, no loop unrolling needed)
static uint16_t Modbus_CRC16(const uint8_t *data, uint8_t len) {
  uint16_t crc = 0xFFFF;
  while (len--) {
    uint8_t tmp = *data++ ^ crc;
    crc = (crc >> 8) ^ crc_table[tmp & 0x0F] ^ (crc_table[(tmp >> 4) & 0x0F] << 4);
  }
  return crc;
}

void Init_USART2(void) {
  // Enable clocks (bitwise set)
  RCC->IOPENR |= RCC_IOPENR_GPIOAEN;
  RCC->APB1ENR |= RCC_APB1ENR_USART2EN;

  // GPIO PA2 (TX AF4), PA3 (RX AF4): alternate function mode
  GPIOA->MODER = (GPIOA->MODER & ~ (GPIO_MODER_MODE2_Msk | GPIO_MODER_MODE3_Msk)) |
                 (GPIO_MODER_MODE2_1 | GPIO_MODER_MODE3_1);  // Mode 10b
  GPIOA->AFR[0] = (GPIOA->AFR[0] & ~ (GPIO_AFRL_AFSEL2_Msk | GPIO_AFRL_AFSEL3_Msk)) |
                  (4UL << GPIO_AFRL_AFSEL2_Pos) | (4UL << GPIO_AFRL_AFSEL3_Pos);

  // USART2 config: disable first
  USART2->CR1 = 0;

  // Set 8 data + parity (9 bits total), odd parity, enable RXNE interrupt
  USART2->CR1 = USART_CR1_M0 |  // M[1:0]=01 for 9 bits
                USART_CR1_PCE | USART_CR1_PS |  // Parity enable, odd
                USART_CR1_RXNEIE;  // RX interrupt enable

  // 1 stop bit (default STOP=00)
  USART2->CR2 = 0;

  // Baud rate 115200 @ 32 MHz APB1: BRR = 32e6 / 115200 ≈ 278
  USART2->BRR = 278;

  // Enable TX, RX, USART
  USART2->CR1 |= USART_CR1_TE | USART_CR1_RE | USART_CR1_UE;

  // NVIC enable for USART2 IRQ
  NVIC_SetPriority(USART2_IRQn, 0);
  NVIC_EnableIRQ(USART2_IRQn);
}

void Init_TIM2(void) {
  // Enable clock
  RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;

  // Prescaler for 1 MHz tick (32 MHz / 32 = 1 MHz, PSC=31)
  TIM2->PSC = 31;

  // Auto-reload for 334 us timeout (3.5 chars @ 115200)
  TIM2->ARR = 333;  // Counts 0 to 333 = 334 ticks

  // Enable update interrupt
  TIM2->DIER = TIM_DIER_UIE;

  // NVIC for TIM2
  NVIC_SetPriority(TIM2_IRQn, 0);
  NVIC_EnableIRQ(TIM2_IRQn);
}
```

### 6.2 Optimized Modbus Protocol Handler

```c
// State machine states
#define STATE_IDLE 0
#define STATE_RECV 1
#define STATE_PROC 2

// In UART ISR: receive byte, reset timer
void USART2_IRQHandler(void) {
  if (USART2->ISR & USART_ISR_RXNE) {  // RX not empty
    uint8_t byte = USART2->RDR;  // Read clears flag

    if (state == STATE_IDLE) {
      if (byte == slave_addr) {  // Address match
        modbus_buf[0] = byte;
        buf_idx = 1;
        state = STATE_RECV;
      }
      // Else ignore
    } else if (state == STATE_RECV && buf_idx < sizeof(modbus_buf)) {
      modbus_buf[buf_idx++] = byte;
    }

    // Reset and start timer for frame timeout
    TIM2->CNT = 0;
    TIM2->SR = 0;  // Clear UIF
    TIM2->CR1 = TIM_CR1_CEN;  // Enable timer
  }

  // Handle TXE/TC if sending (in process)
  if (state == STATE_PROC && (USART2->ISR & USART_ISR_TXE)) {
    // Implement send logic if response ready (not shown, assume in Process_Modbus)
  }
}

// In TIM ISR: frame end, process
void TIM2_IRQHandler(void) {
  if (TIM2->SR & TIM_SR_UIF) {
    TIM2->SR = 0;  // Clear flag
    TIM2->CR1 = 0;  // Stop timer

    if (state == STATE_RECV && buf_idx >= 4) {  // Min frame size
      state = STATE_PROC;
      Process_Modbus();
    }
    state = STATE_IDLE;
    buf_idx = 0;
  }
}

// Process frame and send response (minimal operations)
void Process_Modbus(void) {
  // Check CRC
  uint16_t rx_crc = (modbus_buf[buf_idx-1] << 8) | modbus_buf[buf_idx-2];
  if (Modbus_CRC16(modbus_buf, buf_idx-2) != rx_crc) return;  // Invalid

  uint8_t func = modbus_buf[1];
  uint16_t addr = (modbus_buf[2] << 8) | modbus_buf[3];

  uint8_t tx_buf[32];  // Small response buffer
  uint8_t tx_idx = 2;  // Start after addr+func
  tx_buf[0] = slave_addr;
  tx_buf[1] = func;

  if (func == 0x03 || func == 0x04) {  // Read holding/input
    uint16_t count = (modbus_buf[4] << 8) | modbus_buf[5];
    if (count > 0 && addr + count < 100) {  // Range check
      tx_buf[2] = count * 2;  // Byte count
      uint16_t *regs = (func == 0x03) ? holding_regs : input_regs;
      for (uint16_t i = 0; i < count; i++) {
        uint16_t val = regs[addr + i];
        tx_buf[tx_idx++] = val >> 8;
        tx_buf[tx_idx++] = val & 0xFF;
      }
    } else {
      tx_buf[1] |= 0x80; tx_buf[2] = 0x02; tx_idx = 3;  // Exception
    }
  } else if (func == 0x06) {  // Write single
    uint16_t val = (modbus_buf[4] << 8) | modbus_buf[5];
    if (addr < 100) {
      holding_regs[addr] = val;
      tx_buf[2] = modbus_buf[2]; tx_buf[3] = modbus_buf[3];
      tx_buf[4] = modbus_buf[4]; tx_buf[5] = modbus_buf[5];
      tx_idx = 6;
    } else {
      tx_buf[1] |= 0x80; tx_buf[2] = 0x02; tx_idx = 3;
    }
  } else if (func == 0x10) {  // Write multiple
    uint16_t count = (modbus_buf[4] << 8) | modbus_buf[5];
    uint8_t byte_count = modbus_buf[6];
    if (byte_count == count * 2 && addr + count < 100) {
      for (uint16_t i = 0; i < count; i++) {
        holding_regs[addr + i] = (modbus_buf[7 + i*2] << 8) | modbus_buf[7 + i*2 + 1];
      }
      tx_buf[2] = modbus_buf[2]; tx_buf[3] = modbus_buf[3];
      tx_buf[4] = modbus_buf[4]; tx_buf[5] = modbus_buf[5];
      tx_idx = 6;
    } else {
      tx_buf[1] |= 0x80; tx_buf[2] = 0x03; tx_idx = 3;
    }
  } else {
    tx_buf[1] |= 0x80; tx_buf[2] = 0x01; tx_idx = 3;  // Illegal func
  }

  // Append CRC and send byte by byte via TDR
  uint16_t tx_crc = Modbus_CRC16(tx_buf, tx_idx);
  tx_buf[tx_idx++] = tx_crc & 0xFF;
  tx_buf[tx_idx++] = tx_crc >> 8;

  // Send: enable TX interrupt
  USART2->CR1 |= USART_CR1_TXEIE;
  // Actual send in USART ISR on TXE (not shown fully, implement queue or direct)
}

// Main loop: empty for low power, all in ISRs
int main(void) {
  // System init (clock to 32MHz, etc., not shown)
  Init_USART2();
  Init_TIM2();
  while (1) {
    // Low power mode or other tasks
  }
}
```