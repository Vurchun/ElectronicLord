# ElectronicLord  
**Smart Electronic Load**

---

## 📘 Device Description

The **Smart Electronic Load** is a high-precision testing instrument designed for evaluating the performance of:

- Power supplies  
- Rechargeable batteries  
- Wiring  
- Other electronic components  

By simulating various load scenarios, the device enables accurate measurements and assessment of real-world operating conditions.

### Supported Modes

- **Constant Current (CC)**  
- **Constant Voltage (CV)**  
- **Constant Resistance (CR)**  
- **Constant Power (CP)**  

Integration with a microcontroller enables:

- Local control via display and buttons  
- Remote monitoring and configuration via computer or mobile device  

---

## 🛠 Architecture & Implementation Details

### Key Components

- Dedicated control board (with display and buttons; optional touchscreen in future versions)  
- Modular load system for scalable power handling  
- Four-wire voltage measurement for increased accuracy  
- Interfaces for data communication: OTG/UART (Wi-Fi support planned)  

### Software Features

- Power limit control for safe operation  
- Expandable load module support (no firmware changes required)  
- Visualization of statistical data in graphical format *(planned)*  
- Data transmission via OTG/UART *(Wi-Fi support planned)*  
- Supported modes: constant power, resistance, voltage, and current  

---

## 📦 Module Reference Table

| Module Code     | Description                          |
|-----------------|--------------------------------------|
| EL.T01.DFLT     | Default module                       |
| EL.T01.MBRD     | Main board module                    |
| EL.T01.PLAM     | Power load adjustable module         |
| EL.T01.PLSM     | Power load stable module             |
| EL.T01.MSCH     | Shunt module                         |
| EL.T01.KVMM     | Kelvin voltage measure module        |
| EL.T01.CVCM     | Cell voltage control module          |
| EL.T01.UIM      | User interfacer module               |
| EL.T01.UPSM     | Uninterruptible power supply module  |
| EL.T01.CHGM     | Battary charge module                |
| EL.T01.MSCH     | Montage scheme                       |

---

## ⚙️ Feature Note

In addition to setting the maximum power to the load module, the following should be assessed:

- **MOSFET transistor health**  
- **Voltage required to open transistors to a given level**  

> A transistor showing signs of thermal damage ("burnt") will have increased resistance, requiring a higher voltage to fully open to the same level.

---

