# ARES-01 🚀

> **Autonomous & Teleoperated Mission Rover with a 5-DOF Robotic Arm**

ARES-01 is a custom-built teleoperated rover designed for remote inspection, manipulation, and navigation. It features a complete software and hardware stack, bringing together an ESP32-S3 microcontroller, low-latency WebRTC video streaming, and a React-based mission control dashboard. 

What makes ARES-01 unique is its **Bilingual Voice Control System** (Bengali & English) built with a custom zero-latency fast-path execution engine, allowing instant physical hardware response to voice commands without hitting external APIs for basic navigation.

## ✨ Key Features

* **5-DOF Robotic Arm:** Fully custom robotic arm driven by DC motors via H-Bridges (L298N/TB6612FNG), with joint angle calculations mapped to absolute PWM ranges. Includes Base, Shoulder, Elbow, Wrist, and Gripper control.
* **Low-Latency Video Stream:** Concurrent HTTP MJPEG streaming directly from the ESP32-S3 OV2640 camera to the modern React dashboard.
* **Bilingual Voice Command:** Native support for English and Bengali (e.g., "সামনে যাও", "ডানে ঘুরাও"). Features a built-in VAD (Voice Activity Detection) mechanism for ultra-fast (1.2s) response times.
* **Mission Control Dashboard:** A modern, responsive React/TypeScript UI featuring an interactive 5-axis visual slider system, live telemetry (Ping, Battery, Wi-Fi signal), and manual/autonomous mode toggles.
* **Zero Data Loss Architecture:** Pure Web/WebSocket architecture ensures no intermediary cloud database bottlenecks.

## 🧠 System Architecture

* **Firmware:** Written in C++ using the Arduino Core on PlatformIO. Runs FreeRTOS tasks to handle motor PWM generation, WebSocket telemetry, and camera streaming concurrently.
* **Web App:** Built with React, TypeScript, and TailwindCSS. Uses the SpeechRecognition API for voice capture and WebSockets for real-time duplex communication with the rover.
* **Hardware:** 
  * **Brain:** ESP32-S3 (WROOM-1)
  * **Drive & Arm Actuation:** Standard DC Motors + L298N Motor Drivers
  * **Vision:** OV2640 Camera Module
  * **Power:** 12.6V Li-ion Battery pack stepped down via 5V Buck Converters.

## 📂 Repository Structure

```text
ARES-01/
├── esp32_firmware/       # PlatformIO project containing the C++ ESP32 source code
├── web_app/              # React frontend for the Mission Control Dashboard
├── tools/                # Python scripts for network scanning, mocking, and WS testing
└── ARES_01_Project_Documentation.md  # Detailed hardware & API specs
```

## 🛠️ Getting Started

### 1. Flash the Firmware
1. Open the `esp32_firmware` directory in VS Code (with the PlatformIO extension installed).
2. Connect your ESP32-S3 via USB.
3. Build and upload:
   ```bash
   cd esp32_firmware
   pio run -t upload
   ```

### 2. Run the Dashboard
1. Ensure Node.js (v18+) and `pnpm` are installed.
2. Install dependencies and start the dev server:
   ```bash
   cd web_app
   pnpm install
   pnpm run dev
   ```
3. Open `http://localhost:5173` in your browser. Enter the Rover's IP address (broadcasted over serial) to connect.

## 🤝 Contributors

*(Contributors will be added here...)*

---
*Developed with ❤️ by the ARES-01 Team.*
