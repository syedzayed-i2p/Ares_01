# ARES-01

> **Autonomous & Teleoperated Mission Rover with a 5-DOF Robotic Arm**

ARES-01 is an advanced, custom-built teleoperated rover designed for remote inspection, manipulation, and navigation in complex environments. It features a fully integrated software and hardware stack, bridging an ESP32-S3 microcontroller with a modern React-based mission control dashboard.

What fundamentally distinguishes ARES-01 is its **Bilingual Voice Control System** (supporting both Bengali and English). It utilizes a custom zero-latency fast-path execution engine with built-in Voice Activity Detection (VAD). This allows for instantaneous physical hardware response to voice commands without relying on external cloud APIs for core navigation.

## Core Features

* **5-DOF Robotic Arm Integration**
  Fully custom robotic arm actuated by standard DC motors via H-Bridge drivers (L298N/TB6612FNG). Joint angle calculations from the UI are precisely mapped to absolute PWM ranges, offering smooth control over the Base, Shoulder, Elbow, Wrist, and Gripper.
* **Low-Latency Video Streaming**
  Utilizes the ESP32-S3 OV2640 camera module to serve a concurrent HTTP MJPEG stream. By isolating the camera stream on a dedicated FreeRTOS task, the rover maintains high framerates without blocking motor control or telemetry loops.
* **Bilingual Voice Command Engine**
  Native Natural Language support for English and Bengali (e.g., "সামনে যাও", "ডানে ঘুরাও"). The custom VAD mechanism ensures ultra-fast (~1.2s) response times by executing localized keywords instantly.
* **Mission Control Dashboard**
  A highly responsive React and TypeScript user interface. It features an interactive 5-axis visual slider system for the arm, live real-time telemetry (Ping, Battery Voltage, Wi-Fi Signal Strength), and seamless toggling between manual and autonomous modes.
* **Zero Data Loss Architecture**
  Built on a pure WebSocket and HTTP architecture. Direct peer-to-peer communication ensures no intermediary cloud database bottlenecks, resulting in minimal latency and high reliability.

## System Architecture

The project is divided into two primary environments:

### 1. Firmware (C++ / FreeRTOS)
Developed using the Arduino Core on PlatformIO. The firmware heavily utilizes FreeRTOS to achieve true hardware concurrency:
* **Task 1 (Core 0):** Handles the intensive MJPEG camera stream and HTTP server.
* **Task 2 (Core 1):** Manages the WebSocket server for real-time telemetry, parses incoming JSON movement directives, and drives the hardware PWM channels.

### 2. Hardware Stack
* **Microcontroller:** ESP32-S3 (WROOM-1)
* **Motor Drivers:** L298N (Drive & Arm Actuation)
* **Vision System:** OV2640 Camera Module
* **Power Delivery:** 12.6V Li-ion Battery pack regulated via 5V High-Current Buck Converters.

## Repository Structure

```text
ARES-01/
├── esp32_firmware/       # PlatformIO project containing the C++ ESP32 source code
├── web_app/              # React frontend for the Mission Control Dashboard
├── tools/                # Python scripts for network scanning, mocking, and WS testing
└── ARES_01_Project_Documentation.md  # Detailed hardware & API specifications
```

## Getting Started

**Prerequisites:**
- Node.js (v18 or higher) and pnpm for the web dashboard.
- VS Code with the PlatformIO extension for firmware compilation.

### 1. Flash the Firmware:
1. Navigate to the esp32_firmware directory in VS Code.
2. Connect your ESP32-S3 board via USB.
3. Build and upload the firmware:
   ```bash
   cd esp32_firmware
   pio run -t upload
   ```

### 2. Run the Mission Control Dashboard:
1. Open a new terminal and navigate to the web application directory.
2. Install the required dependencies and start the Vite development server:
   ```bash
   cd web_app
   pnpm install
   pnpm run dev
   ```
3. Open `http://localhost:5173` in your browser. Enter the Rover's local IP address (broadcasted over the serial monitor) to establish the WebSocket connection.

## Contributors

*(Contributors will be added here...)*

---
*Developed by the ARES-01 Team.*
