# ROVER PROJECT DOCUMENTATION PACKAGE

## 1. Project Summary
* **Project Name**: ARES-01 Autonomous Mars Rover Project
* **Development Status**: Advanced Prototype / Active
* **Main Purpose**: Remote-controlled and AI-assisted robotic rover platform with an integrated 5-DOF robotic arm and camera streaming.
* **Intended Applications**: Remote inspection, AI-based autonomous control, object manipulation, robotic education.
* **Development Environment**: Antigravity 2.0
* **Programming Languages**: C++ (Firmware), TypeScript/JavaScript (Web App)
* **Frameworks/Libraries**: React, Vite, Tailwind CSS, Shadcn/Radix UI (Web), Arduino Core for ESP32, FreeRTOS (Firmware).
* **Deployment Method**: Web application runs locally (or hosted) and connects via HTTP/WebSocket to the ESP32 acting as a network server/AP.

## 2. System Architecture
**Dual-Brain Serverless Architecture**:
The system is divided into two primary logical domains:
1. **Edge Firmware (ESP32-S3-CAM)**: Handles hard real-time tasks, camera MJPEG streaming, hardware I2C PWM generation, and motor driving. Operates completely locally.
2. **Control Client (React Web App)**: Handles UI, AI command parsing (via Web Speech API and Groq LLM API), and teleoperation (virtual joystick/sliders).

**Hardware Block Diagram**:
[Battery (12.6V)] → [5V Buck Converter] → [ESP32-S3]
[ESP32-S3 (I2C)] → [PCA9685 #1 (0x40)] → [Chassis Motor Drivers] → [4x DC Wheels]
[ESP32-S3 (I2C)] → [PCA9685 #2 (0x41)] → [Arm Motor Drivers] → [5x Arm DC Motors]

**Software Architecture Diagram**:
[User Speech/UI] → [React Dashboard] → [HTTP POST/WebSocket] → [ESP32 WebServer] → [HardwareController.cpp] → [I2C] → [PCA9685] → [Actuators]

## 3. Hardware Inventory
| ID | Component | Manufacturer | Model | Quantity | Purpose | Interface |
|----|-----------|--------------|-------|----------|---------|-----------|
| U1 | Microcontroller | Espressif | ESP32-S3-CAM V1.3 | 1 | Main Controller | Wi-Fi, I2C |
| U2 | Camera Sensor | OmniVision | OV3660 | 1 | Video Streaming | DVP |
| U3 | PWM Controller | Adafruit | PCA9685 | 2 | PWM generation for motors | I2C (0x40, 0x41) |
| U4 | Motor Driver | Various | L298N / TB6612FNG | Multiple | DC Motor Control | PWM/GPIO |
| M1-4| Chassis Motors | N/A | DC Gear Motors | 4 | Rover Locomotion | DC Power |
| M5-9| Arm Motors | N/A | DC Gear Motors | 5 | Robotic Arm Joints | DC Power |
| P1 | Step-down Conv | N/A | 5V Buck Converter | 1 | Logic Power Supply | Power |
| B1 | Battery | N/A | 3S LiPo (12.6V) | 1 | Main Power | Power |
*(Note: Exact models for battery and chassis motors [NOT VERIFIED — REQUIRES USER CONFIRMATION])*

## 4. Mechanical Design
* **Chassis Structure**: 4-wheel drive skid-steer / differential drive configuration.
* **Arm Mounting**: Mounted on the rover base.
* **Camera Mounting**: Integrated into the ESP32-S3-CAM module on the rover chassis.
*(Physical dimensions, weight distribution, and 3D printing files: [NOT VERIFIED — REQUIRES USER CONFIRMATION])*

## 5. Robotic Arm Integration
* **DOF**: 5 Degrees of Freedom (Base, Shoulder, Elbow, Wrist, Gripper).
* **Actuation Method**: Surprisingly, the arm does NOT use standard 3-wire RC servos. It utilizes standard DC motors driven by H-Bridges (like L298N).
* **Control Method**: The web UI sends "absolute angles" (0-180), which the firmware intercepts. 90 is treated as STOP (deadband 89-91). 91-180 maps to variable forward speed (PWM 3800-4095). 89-0 maps to variable reverse speed. 

## 6. Power Architecture
**POWER TREE**:
Battery (Assumed 12.6V 3S LiPo)
↓
5V Buck Converter (Provides safe unified 5V power)
↓
ESP32-S3, PCA9685 Controllers, Logic side of Motor Drivers
*(Motor VIN power path: [NOT VERIFIED — REQUIRES USER CONFIRMATION]. Motor drivers may be powered directly from 12.6V or a separate buck converter depending on motor rating.)*

## 7. Electrical & Wiring
**MASTER PINOUT TABLE**:
| Source | Pin | Destination | Pin | Function |
|--------|-----|-------------|-----|----------|
| ESP32 | GPIO 1 | PCA9685 #1 & #2 | SDA | I2C Data |
| ESP32 | GPIO 2 | PCA9685 #1 & #2 | SCL | I2C Clock |
| PCA9685 #1 (0x40) | 0, 2, 1 | Motor Drv FL | PWM, IN1, IN2 | Front Left Wheel |
| PCA9685 #1 (0x40) | 5, 4, 3 | Motor Drv BL | PWM, IN1, IN2 | Back Left Wheel |
| PCA9685 #1 (0x40) | 6, 8, 7 | Motor Drv FR | PWM, IN1, IN2 | Front Right Wheel |
| PCA9685 #1 (0x40) | 11, 10, 9 | Motor Drv BR | PWM, IN1, IN2 | Back Right Wheel |
| PCA9685 #1 (0x40) | 15 | TB6612FNG (if used)| STBY | Motor Standby HIGH |
| PCA9685 #2 (0x41) | 6, 7, 8 | Motor Drv Arm | PWM, IN1, IN2 | Shoulder Joint |
| PCA9685 #2 (0x41) | 3, 4, 5 | Motor Drv Arm | PWM, IN1, IN2 | Elbow Joint |
| PCA9685 #2 (0x41) | 0, 1, 2 | Motor Drv Arm | PWM, IN1, IN2 | Wrist Joint |
| PCA9685 #2 (0x41) | 9, 15, 11 | Motor Drv Arm | PWM, IN1, IN2 | Gripper |
| PCA9685 #2 (0x41) | 12, 13, 14| Motor Drv Arm | PWM, IN1, IN2 | Base Joint (Inverted in code) |

## 8. Controller & Firmware
* **Microcontroller**: ESP32-S3 running Arduino Core (PlatformIO).
* **Firmware Structure**: `main.cpp` handles Web Server, WebSocket Server, and Camera DMA buffers. `HardwareController.cpp` abstracts all I2C PCA9685 manipulation.
* **Multithreading**: FreeRTOS Dual-Core Architecture is initialized.
* **Safety Logic**: Deadband implemented between values 89-91 to prevent motor jitter. Minimum PWM floor set to 3800 for torque preservation.

## 9. Motor & Drive System
* **Drive Configuration**: 4-wheel independent PWM control acting as skid-steer. 
* **Drive Logic**: Left wheels use IN2/IN4 as forward. Right wheels use IN2/IN4 as forward.
* **Speed Scaling**: UI joystick sends -255 to 255. Firmware maps this to 0-180 UI value, which then maps to 3800-4095 PWM.

## 10. Sensors
* **Primary Sensor**: OmniVision OV3660 Camera Sensor (20MHz XCLK, PIXFORMAT_JPEG, FRAMESIZE_VGA/SVGA).
* **Other Sensors**: Distance/Obstacle sensors have been physically removed from the codebase to prevent FreeRTOS blocking/CPU starvation on the ESP32.

## 11. Web Application
* **Frontend**: React (Vite), TypeScript.
* **UI Layout**:
  * **Video Feed**: Centered canvas rendering MJPEG stream.
  * **Arm Control Panel (5DOF)**: Absolute sliders mimicking servo angles.
  * **Drive Controls**: Virtual Joystick or D-Pad for chassis motion.
  * **Voice Link**: Speech-to-text integration with dual-language support (bn-BD / en-US).
* **Offline Media Grouping**: Uses File System Access API to permanently save captures to an 'ARES01' local folder.

## 12. Network & Connectivity
* **Topology**: Dual-Mode Wi-Fi.
* **AP Mode**: ESP32 broadcasts SSID: `ARES_01` (Password: `Admin123`).
* **STA Mode**: Dynamically connects to local network if available.
* **Ports**: HTTP (Port 80), WebSocket (Port 81).

## 13. API & Communication
| Endpoint | Method | Input (JSON) | Output | Purpose |
|----------|--------|--------------|--------|---------|
| `/command` | POST | `{mode:"manual", action:"drive", direction:"FORWARD", speed:255}` | `200 OK` | Chassis Control |
| `/command` | POST | `{mode:"arm", action:"arm_control", joint:"base", angle:90}` | `200 OK` | Arm Joint Control |
| `/command` | POST | `{mode:"manual", action:"arm_macro", direction:"PICKUP", speed:255}` | `200 OK` | Pre-defined Arm Macro |
| `/capture` | GET | N/A | JPEG Image | Still frame capture |
| `ws://<ip>:81` | WS | N/A | Telemetry JSON | Real-time status |

## 14. Operating Modes
1. **Manual Mode**: Direct slider and joystick control from the UI.
2. **Macro Mode**: Execution of pre-programmed arm sequences (e.g., Home, Pickup, Drop).
3. **Voice Mode (Local Fast-Path)**: Real-time STT parses distinct Bengali/English keywords for instantaneous hardware response.
4. **Autonomous Mode (Groq AI)**: Complex queries fall back to Groq API to generate autonomous payload directives.

## 15. Calibration & Homing
* **Motor Calibration**: Motors do not have absolute encoders. They are treated as continuous rotation.
* **Arm Homing**: Clicking "Home" sends an angle of `90` to all joints, which instantly cuts PWM to all H-bridges (software brake), centering the "virtual" position on the UI.

## 16. Startup
1. Power on 12.6V Battery switch.
2. Buck converter powers ESP32.
3. ESP32 boots FreeRTOS, initializes I2C (Pins 1,2) and PCA9685.
4. Motors default to STOP (PWM 0).
5. Wi-Fi AP `ARES_01` begins broadcasting.
6. User connects device to Wi-Fi.
7. User runs `pnpm run dev` (or accesses hosted app).
8. Web app authenticates camera feed and WebSocket telemetry.

## 17. Operation
* **Rover Control**: Drag joystick or press D-Pad. Release to auto-brake.
* **Arm Control**: Slide the 5DOF UI sliders. Releasing the slider sends angle 90 (STOP) to the DC motors.
* **Voice Control**: Click the microphone icon. Say "সামনে যাও" (Go forward) or "রিস্ট নিচে" (Wrist down). STT translates this to JSON API payloads.

## 18. Shutdown
1. Center all arm sliders to ensure PWM is 0.
2. Ensure Rover joystick is centered.
3. Close the Web Application.
4. Turn off main physical power switch to kill 12.6V battery line.

## 19. Safety
* **CAUTION**: DC Motors acting as arm joints do not possess physical limit switches in the firmware logic. Operating them beyond mechanical limits may result in motor stall and high current draw. 
* **WARNING**: 12.6V LiPo batteries must be disconnected if the rover is left unattended.
* **NOTICE**: STT Voice commands implement strict isolation logic. Arm joint commands ("Wrist down") will explicitly block Chassis translation ("Drive backward") to prevent collision overrides.

## 20. Troubleshooting
| Problem | Possible Cause | Solution |
|---------|----------------|----------|
| STT executes wrong command | Phonetic misinterpretation | Voice parser includes extensive Bengali synonyms (e.g., 'বেস', 'দেশ', 'ফেস') to catch errors. Use clear pronunciation. |
| Motors jitter but don't move | Voltage drop on 12.6V line | Charge battery. Ensure buck converter is rated for sufficient amperage. |
| Arm doesn't stop | Web app disconnected during slide | Press physical E-Stop or refresh page and click "Home" macro to send 0 PWM. |

## 21. Maintenance
* Periodically check PCA9685 I2C jumper wires, as they are daisy-chained.
* Verify motor driver heatsinks, as continuous stall current on arm joints will generate extreme heat.
*(Mechanical inspection intervals: [NOT VERIFIED — REQUIRES USER CONFIRMATION])*

## 22. Software Setup
**Firmware**:
1. Install VS Code + PlatformIO.
2. Open `esp32_firmware`.
3. Select environment `esp32s3dev`.
4. Run `pio run --target upload`.

**Web Application**:
1. Install Node.js & pnpm.
2. Run `pnpm install`.
3. Run `pnpm run dev`.

## 23. Project File Structure
```text
Project/
├── esp32_firmware/
│   ├── platformio.ini       (Build config: ESP32-S3, PSRAM flags)
│   └── src/
│       ├── main.cpp         (Web server, camera DMA, HTTP routing)
│       └── HardwareController.cpp/.h (I2C logic, PCA9685 mapping, Motor matrix)
├── web_app/
│   ├── package.json         (Vite, React, Tailwind, Radix dependencies)
│   └── src/
│       └── pages/
│           └── dashboard.tsx (Main UI, STT parser, HTTP API client, Camera Canvas)
```

## 24. Technical Specifications
* **Controller**: ESP32-S3-CAM V1.3
* **Camera**: OV3660 (VGA/SVGA, 20MHz XCLK)
* **Wi-Fi**: 802.11 b/g/n (AP `ARES_01` + STA)
* **Actuators**: 4x Drive DC Motors, 5x Arm DC Motors
* **PWM Controllers**: 2x Adafruit PCA9685 (I2C 0x40, 0x41)
* **AI Processing**: Groq LLM API
* *(Dimensions, Weight, Battery Runtime: [NOT VERIFIED — REQUIRES USER CONFIRMATION])*

## 25. Limitations
* **Pseudo-Servos**: The robotic arm uses DC motors mapped to 0-180 UI scales. It relies on human-in-the-loop visual feedback. There is no closed-loop PID positioning in firmware.
* **Sensor Deprecation**: Environmental sensors were removed to prevent FreeRTOS blocking, meaning the rover has no autonomous physical collision avoidance.

## 26. Required Images & Diagrams
* [FIGURE 01 — Complete Rover Overview]
* [FIGURE 02 — Software Architecture Diagram]
* [FIGURE 03 — PCA9685 I2C Daisy-Chain Diagram]
* [FIGURE 04 — Motor Driver Wiring (Chassis & Arm)]
* [FIGURE 05 — Web App Interface Dashboard]

## 27. Final Manual Mapping
*(To be utilized by OpenAI Prism)*
1. Overview & Safety (Sections 1, 19, 25)
2. Hardware & Architecture (Sections 2, 3, 4, 6, 24)
3. Electrical & Wiring (Sections 7, 9)
4. Software & Network (Sections 8, 11, 12, 13, 22, 23)
5. Operation & Control (Sections 5, 14, 15, 16, 17, 18)
6. Troubleshooting & Maintenance (Sections 20, 21)

## 28. Information Gaps
| ID | Missing Information | Why Required | User Must Provide |
|----|---------------------|--------------|-------------------|
| 01 | Physical Dimensions | For Technical Specs | Yes |
| 02 | Battery Capacity | For Technical Specs | Yes |
| 03 | Motor Driver Models | For accurate wiring diagrams (L298N vs TB6612FNG) | Yes |
| 04 | Motor Current Ratings| For Power Architecture tree | Yes |
| 05 | Mechanical Assembly | For Build Manual instructions | Yes |

## 29. Verification Checklist
- [x] Pin numbers verified against `HardwareController.cpp`
- [x] API endpoints verified against `main.cpp`
- [x] Motor logic (DC vs Servo) verified via PCA9685 implementation
- [x] Web client logic verified against `dashboard.tsx`
- [x] Uncertain hardware parameters flagged as unverified
