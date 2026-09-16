# ARES-01: Autonomous & Teleoperated Mission Rover
**Comprehensive Project Manual**

---
**Submitted By:**
Department of Electronics Technology
Diploma in Engineering
Session: 2022-23

**Supervised By:**
Gazi Saiful Islam
Chief Instructor, Electronics Department
Barishal Polytechnic Institute, Barishal

**Institute:**
Barishal Polytechnic Institute
---

## 📑 Table of Contents
* **Part 1: User / Operation Manual**
  * 1.1 Project Overview
  * 1.2 Operating the Dashboard & Voice Control
  * 1.3 Operation Guidelines & Hints
* **Part 2: Build / Assembly Manual**
  * 2.1 Complete Parts List
  * 2.2 Hardware Assembly & Wiring Overview
* **Part 3: Technical / Developer Manual**
  * 3.1 Software & System Architecture
  * 3.2 Code Architecture
  * 3.3 System Workflow & Data Flow
  * 3.4 Version Control & Troubleshooting Matrix
  * 3.5 GitHub Repository

---

## Part 1: User / Operation Manual

### 1.1 Project Overview
**ARES-01** is a custom-designed, teleoperated mission rover built for remote inspection, manipulation, and navigation. The system combines a robust hardware chassis with a **5-DOF (Degree of Freedom) Robotic Arm**, all controlled wirelessly through a modern web-based dashboard. 
The defining feature is its **Bilingual Voice Control System** (Bengali and English). It processes voice commands locally within the browser using Voice Activity Detection (VAD) and instantly transmits execution directives via WebSockets, ensuring real-time response.

### 1.2 Operating the Dashboard & Voice Control
* **Dashboard Access:** Open the React web application in your browser and enter the ESP32-S3's local IP address.
* **Manual Control:** Use the on-screen interactive sliders to control the 5-DOF robotic arm (Base, Shoulder, Elbow, Wrist, Gripper). The sliders map 0-180 degree angles to absolute PWM ranges.
* **Voice Control:** Click the microphone icon to activate. Speak clearly in either Bengali (e.g., "সামনে যাও", "হাত ঘুরাও") or English. The VAD will auto-detect the end of your sentence and execute the command instantly (~1.2s response time).

### 1.3 Operation Guidelines & Hints
* **Battery Safety:** Always monitor the battery telemetry on the dashboard. Do not let the 3S LiPo pack drop below 10.5V to prevent cell damage.
* **Voice Command Best Practices:** Speak clearly and pause for 1 second after finishing a command to trigger immediate execution.

---

## Part 2: Build / Assembly Manual

### 2.1 Complete Parts List
Below is the comprehensive list of all hardware components used to assemble ARES-01:

| Sl. No. | Component Name | Quantity | Description / Application |
| :---: | :--- | :---: | :--- |
| 1 | **ESP32-S3 CAM Dev Board (with OV2640 & Antenna)** | 1 | The main dual-core microcontroller processing video, WebSockets, and logic. |
| 2 | **PCA9685 PWM Driver** | 2 | Expands PWM outputs via I2C to control multiple motor drivers simultaneously. |
| 3 | **I2C Logic Level Converter** | 1 | Safely translates the 3.3V I2C logic of the ESP32 to 5V logic for external modules. |
| 4 | **TB6612FNG Motor Driver** | 2 | Efficient motor drivers used for high-precision motor control (wheels/arm). |
| 5 | **L298N Motor Driver** | 3 | High-power H-Bridge drivers for handling the main drive chassis and heavy arm joints. |
| 6 | **Voltage Regulator Buck Converter** | 1 | Steps down the 11.1V battery voltage to a stable 5V for logic boards. |
| 7 | **LiPo Battery, 3300mAh, 11.1V, 3S** | 1 | The primary high-voltage, high-discharge power source for the entire rover. |
| 8 | **T-Connector (XT60)** | 1 | Secure power connector for the LiPo battery. |
| 9 | **Switch** | 1 | Main power toggle switch for the system. |
| 10 | **Robotic Arm Edge Kit (5 DOF Gear Motor)** | 1 | The physical mechanical arm structure driven by gear motors. |
| 11 | **DC Motor (Rover Wheel)** | 4 | Geared DC motors driving the four wheels of the rover base. |

### 2.2 Hardware Assembly & Wiring Overview
* **Power Distribution:** The 11.1V 3S LiPo battery connects via the XT60 T-Connector through the main Switch. Raw 11.1V power is routed to the L298N and TB6612FNG motor drivers for maximum torque. The Voltage Regulator Buck Converter steps this down to 5V to power the ESP32-S3 and PCA9685 drivers.
* **Logic Routing:** The ESP32-S3 communicates with the PCA9685 PWM drivers via the I2C bus. Because the ESP32 operates at 3.3V, an I2C Logic Level Converter bridges the connection to ensure stable signals.
* **Motor Actuation:** The PCA9685 outputs precise PWM signals to the L298N and TB6612FNG motor drivers, which in turn drive the 4 DC Rover Wheels and the 5 DOF Robotic Arm Edge Kit.

---

## Part 3: Technical / Developer Manual

### 3.1 Software & System Architecture
The system architecture is divided into a **Frontend Client** and a **Backend Firmware**.
* **Frontend (React Web App):** Built using React, TypeScript, and TailwindCSS. It utilizes the browser's native SpeechRecognition API for VAD and establishes a pure WebSocket connection for real-time duplex communication.
* **Backend (ESP32 Firmware):** Built on the Arduino Core (PlatformIO). It leverages FreeRTOS for true concurrency. Core 0 handles the intensive OV2640 MJPEG camera stream, while Core 1 manages the WebSocket server, parses incoming JSON control packets, and updates hardware PWM registers.

### 3.2 Code Architecture
```text
ARES-01/
├── esp32_firmware/            # Firmware Backend
│   ├── src/main.cpp           # Entry point and FreeRTOS task definitions
│   ├── include/config.h       # Pin assignments and hardware constraints
│
├── web_app/                   # React Frontend
│   ├── src/pages/dashboard.tsx # Main UI, 5-DOF sliders, and telemetry
│   ├── src/hooks/             # Custom React hooks for WebSocket and VAD Voice logic
```

### 3.3 System Workflow & Data Flow
1. **Input Generation:** The user moves a slider or speaks a voice command.
2. **Translation & JSON Packing:** The React app translates the input into a structured JSON payload (e.g., `{"type": "drive", "dir": "F", "speed": 255}`).
3. **Transmission:** The payload is sent via WebSockets with near-zero latency.
4. **Parsing & Execution:** The ESP32 decodes the JSON, determines the target motor, and updates the PCA9685 PWM duty cycle via I2C. The motor drivers then actuate the motors.

### 3.4 Version Control & Troubleshooting Matrix

| Issue / Fault Symptom | Probable Cause | Diagnostic Check | Corrective Action |
| :--- | :--- | :--- | :--- |
| **Motors not responding** | PWM Jumper still present | Inspect L298N ENA/ENB headers | Remove black jumper caps |
| **ESP32 reboots continuously** | Power conflict on 5V pin | Check L298N 5V-EN jumper | Remove 5V-EN jumper cap |
| **Camera stream failure** | GPIO pin conflict | Check connection to GPIO 4 / 5 | Disconnect peripherals on pins 4/5 |
| **Incorrect battery reading**| Resistor divider mismatch | Measure 33k and 10k resistors | Recalibrate ADC telemetry scalar |
| **Erratic motor movement** | Common ground loop/potentials | Check GND continuity bus | Tie all module grounds together |

### 3.5 GitHub Repository
* **Repository Link:** [https://github.com/syedzayed-i2p/Ares_01](https://github.com/syedzayed-i2p/Ares_01)
* **Status:** Open Source (MIT License)

---
*Developed by the ARES-01 Team.*
