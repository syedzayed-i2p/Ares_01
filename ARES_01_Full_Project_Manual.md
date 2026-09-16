# ARES-01: Autonomous & Teleoperated Mission Rover
**Comprehensive Project Manual & Technical Research Report**

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
1. [Project Overview](#1-project-overview)
2. [Parts List & Hardware Description](#2-parts-list--hardware-description)
3. [Software & System Architecture](#3-software--system-architecture)
4. [Code Architecture](#4-code-architecture)
5. [Project Workflow & Data Flow](#5-project-workflow--data-flow)
6. [GitHub Repository](#6-github-repository)
7. [Helpful Hints](#7-helpful-hints)
8. [Troubleshooting Matrix](#8-troubleshooting-matrix)

---

## 1. Project Overview
**ARES-01** is a custom-designed, teleoperated mission rover built for remote inspection, manipulation, and navigation. The system combines a robust hardware chassis with a **5-DOF (Degree of Freedom) Robotic Arm**, all controlled wirelessly through a modern web-based dashboard. 

The most defining feature of ARES-01 is its **Bilingual Voice Control System** (Bengali and English). It processes voice commands locally within the browser using Voice Activity Detection (VAD) and instantly transmits execution directives via WebSockets, bypassing the latency of cloud-based AI APIs to ensure real-time hardware response.

---

## 2. Parts List & Hardware Description

| Component | Qty | Description & Application |
| :--- | :---: | :--- |
| **ESP32-S3 WROOM-1** | 1 | The main dual-core brain of the rover. It generates PWM signals, hosts the WebSocket server for telemetry, and processes the HTTP video stream concurrently. |
| **OV2640 Camera Module** | 1 | Attached to the ESP32-S3 to capture real-time video. It streams MJPEG video directly to the web dashboard for remote navigation. |
| **L298N / TB6612 Motor Drivers** | 3+ | High-power H-Bridge drivers. Used to actuate the standard DC motors for the rover's wheels and the 5-DOF robotic arm joints (Base, Shoulder, Elbow, Wrist, Gripper). |
| **Standard DC Motors** | 9 | Used for the rover's drive mechanism (4x) and the robotic arm articulation (5x). |
| **12.6V Li-ion Battery Pack** | 1 | The primary high-voltage power source for the entire system, providing sufficient current to drive multiple motors simultaneously. |
| **5V Buck Converter** | 1 | A step-down voltage regulator used to safely convert the 12.6V battery voltage into a stable 5V supply for the ESP32-S3 and logic components. |
| **Resistor Network (33kΩ, 10kΩ)** | 1 | A voltage divider circuit used to safely step down the 12.6V battery voltage to a readable <3.3V range for the ESP32 Analog-to-Digital Converter (ADC) for live battery monitoring. |

---

## 3. Software & System Architecture
The system architecture is divided into a **Frontend Client** (Mission Control Dashboard) and a **Backend Firmware** (Rover Hardware).

### The Frontend (React Web Application)
* Built using **React, TypeScript, and TailwindCSS**.
* **Voice Recognition Engine:** Uses the browser's native SpeechRecognition API configured for `bn-BD` and `en-US`. 
* **Control UI:** Features interactive sliders that map 0-180 degree angles to absolute PWM ranges for precise arm control.
* **Communication:** Establishes a pure WebSocket connection (`ws://<ESP32_IP>/ws`) to send JSON payloads and receive live telemetry (Ping, Battery %).

### The Backend (ESP32 Firmware)
* Built on the **Arduino Core using PlatformIO**.
* **FreeRTOS Concurrency:** 
  * **Core 0:** Exclusively handles the heavy lifting of the OV2640 camera buffer and HTTP MJPEG streaming.
  * **Core 1:** Maintains the WebSocket server, parses incoming JSON control packets, and instantly updates the hardware PWM registers to move the motors.

---

## 4. Code Architecture
The project is strictly modular to maintain clean code and easy debugging:

```text
ARES-01/
├── esp32_firmware/            # Firmware Backend
│   ├── src/main.cpp           # Entry point and FreeRTOS task definitions
│   ├── include/config.h       # Pin assignments and hardware constraints
│   ├── platformio.ini         # Build configurations and dependency management
│
├── web_app/                   # React Frontend
│   ├── src/pages/dashboard.tsx # Main UI, 5-DOF sliders, and telemetry
│   ├── src/hooks/             # Custom React hooks for WebSocket and VAD Voice logic
│
└── tools/                     # Utility Python scripts for offline testing
```

---

## 5. Project Workflow & Data Flow
How a command travels from the user to the physical rover:

1. **Input Generation:** The user moves a slider on the dashboard or speaks a voice command (e.g., "সামনে যাও").
2. **Translation & JSON Packing:** The React app translates the input into a structured JSON payload: `{"type": "drive", "dir": "F", "speed": 255}`.
3. **WebSocket Transmission:** The JSON packet is sent over the local Wi-Fi network with near-zero latency.
4. **Firmware Parsing:** The ESP32 receives the JSON on Core 1, decodes the parameters using the `ArduinoJson` library, and determines the target motor.
5. **Hardware Execution:** The ESP32 updates the specific GPIO pin's PWM duty cycle. The L298N driver receives this signal and pushes voltage to the DC motor, resulting in physical movement.

---

## 6. GitHub Repository
The complete source code, version history, and release notes can be found at the official repository:
* **Link:** [github.com/syedratul-i2p/Ares_01](https://github.com/syedratul-i2p/Ares_01)
* **Status:** Open Source (MIT License)

---

## 7. Helpful Hints
* **Battery Safety:** Always monitor the battery telemetry on the dashboard. Do not let the Li-ion pack drop below 10.5V to prevent permanent cell damage.
* **Voice Command Best Practices:** For the best Voice Activity Detection (VAD) performance, speak clearly and pause for 1 second after finishing a command to trigger immediate execution.
* **Firmware Uploading:** Always disconnect the ESP32 from the L298N motor drivers when flashing firmware via USB to prevent power backflow or serial port conflicts.

---

## 8. VERSION CONTROL AND TROUBLESHOOTING MATRIX

| Issue / Fault Symptom | Probable Cause | Diagnostic Check | Corrective Action |
| :--- | :--- | :--- | :--- |
| **Motors not responding** | PWM Jumper still present | Inspect L298N ENA/ENB headers | Remove black jumper caps |
| **ESP32 reboots continuously** | Power conflict on 5V pin | Check L298N 5V-EN jumper | Remove 5V-EN jumper cap |
| **Camera stream failure** | GPIO pin conflict | Check connection to GPIO 4 / 5 | Disconnect peripherals on pins 4/5 |
| **Incorrect battery reading**| Resistor divider mismatch | Measure 33k and 10k resistors | Recalibrate ADC telemetry scalar |
| **Erratic motor movement** | Common ground loop/potentials | Check GND continuity bus | Tie all module grounds together |
