# ARES-01: Autonomous & Teleoperated Mission Rover
**Comprehensive Project Manual, Build Guide & Technical Architecture**

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
* **Part 1: User / Operation Manual (Zero-to-Hero Guide)**
  * 1.1 Downloading the Project
  * 1.2 Setting Up the Environment (Node.js & pnpm)
  * 1.3 Running the Web Dashboard
  * 1.4 Operating the Rover (Voice & Sliders)
* **Part 2: Build / Assembly Manual & Hardware Details**
  * 2.1 Detailed Component List & Usage Justification
  * 2.2 System Power Distribution
  * 2.3 Comprehensive Pin Mapping & Wiring Guide (A-Z)
* **Part 3: Technical / Developer Manual**
  * 3.1 Software & System Architecture
  * 3.2 WebSocket Communication JSON Protocol
  * 3.3 Version Control & Troubleshooting Matrix
  * 3.4 Official GitHub Repository

---

## Part 1: User / Operation Manual (Zero-to-Hero Guide)
*This section is designed for absolute beginners. It explains exactly how to run the ARES-01 dashboard on your computer from scratch.*

### 1.1 Downloading the Project
1. Open your web browser and go to the official GitHub repository: [https://github.com/syedzayed-i2p/Ares_01](https://github.com/syedzayed-i2p/Ares_01)
2. Click on the green **"<> Code"** button located at the top right of the file list.
3. Select **"Download ZIP"**.
4. Once downloaded, locate the `.zip` file in your Downloads folder. Right-click on it and select **"Extract All..."** (or use WinRAR/7-Zip) to extract the files into a normal folder.

### 1.2 Setting Up the Environment (Node.js & pnpm)
Before running the dashboard, your computer needs Node.js.
1. Go to [nodejs.org](https://nodejs.org/) and download the **LTS (Long Term Support)** version. Install it like any normal software.
2. Once Node.js is installed, we need to install `pnpm` (a fast package manager).
3. Press `Windows Key`, type **PowerShell**, right-click on it, and select **"Run as Administrator"**.
4. Type the following command and press Enter:
   ```bash
   npm install -g pnpm
   ```
5. *Wait for the installation to finish.* You are now ready to run the project.

### 1.3 Running the Web Dashboard
1. Go to the extracted `Ares_01` folder. Double click and go inside the `web_app` folder.
2. Inside the `web_app` folder, click on the **address bar** at the top of the folder window, type `powershell`, and hit **Enter**. This will open a blue PowerShell window directly in that folder.
3. First, download all the required project files by typing:
   ```bash
   pnpm install
   ```
4. Once it finishes downloading, start the local server by typing:
   ```bash
   pnpm run dev
   ```
5. You will see a message saying a local server has started. Open Google Chrome (or any modern browser) and type this address in the URL bar:
   **`http://localhost:5173`**

### 1.4 Operating the Rover (Voice & Sliders)
* **Connecting:** Turn on the physical rover. Wait 10 seconds. Enter the Rover's local IP address in the dashboard box and click "Connect". The live camera feed will appear.
* **Manual Control:** Use the interactive sliders on the screen to control the 5-DOF robotic arm (Base, Shoulder, Elbow, Wrist, Gripper). Sliding it updates the physical arm in real time.
* **Voice Control:** Click the Microphone icon. Speak clearly in Bengali (e.g., "সামনে যাও", "ডানে ঘুরাও"). Wait 1 second after speaking; the system's Voice Activity Detection (VAD) will automatically process the command and move the rover.

---

## Part 2: Build / Assembly Manual & Hardware Details

### 2.1 Detailed Component List & Usage Justification
Every component in ARES-01 serves a specific purpose in the architecture.

| Sl | Component Name | Qty | Specific Usage in Project |
| :---: | :--- | :---: | :--- |
| 1 | **ESP32-S3 CAM Dev Board (with OV2640 & Antenna)** | 1 | The absolute "Brain" of the rover. It processes the camera video, hosts the Wi-Fi WebSocket server, and issues I2C commands to move motors. |
| 2 | **PCA9685 PWM Driver** | 2 | Since the ESP32 doesn't have enough PWM pins for 9 motors, these drivers expand the capability. They convert simple I2C signals into 32 channels of perfect PWM signals. |
| 3 | **I2C Logic Level Converter** | 1 | The ESP32 runs at 3.3V, but the PCA9685 prefers 5V logic. This chip bridges them, ensuring signals aren't corrupted or hardware fried. |
| 4 | **TB6612FNG Motor Driver** | 2 | Used for the smaller precision gear motors of the robotic arm. It is highly efficient and runs cooler than older drivers. |
| 5 | **L298N Motor Driver** | 3 | Used for the heavy-duty rover base wheels and the main base/shoulder joint of the robotic arm which require high current. |
| 6 | **Voltage Regulator (Buck Converter)** | 1 | Takes the dangerous 11.1V from the LiPo battery and converts it into a perfectly safe, stable 5.0V to power the delicate ESP32 and logic boards. |
| 7 | **LiPo Battery (3300mAh, 11.1V, 3S)** | 1 | The main fuel tank. 11.1 Volts provides immense torque to the motors, while 3300mAh ensures long mission duration. |
| 8 | **T-Connector (XT60)** | 1 | A secure, spark-proof connector ensuring the high-current battery doesn't accidentally disconnect during rover movement. |
| 9 | **Switch** | 1 | The main kill-switch to power the entire system on or off instantly. |
| 10 | **Robotic Arm Edge Kit (5 DOF)** | 1 | The mechanical chassis of the arm, allowing complex object manipulation. |
| 11 | **DC Motor (Rover Wheel)** | 4 | The 4-wheel drive system enabling all-terrain navigation. |

### 2.2 System Power Distribution
**WARNING:** Never connect the 11.1V LiPo battery directly to the ESP32 or Logic components.
1. **LiPo Battery (11.1V 3S)** -> `XT60 Connector` -> `Main Power Switch`.
2. **From the Main Switch (11.1V High-Power Line):**
   * -> `12V Input` of **L298N Motor Driver 1** (Front Wheels)
   * -> `12V Input` of **L298N Motor Driver 2** (Rear Wheels)
   * -> `12V Input` of **L298N Motor Driver 3** (Arm Base)
   * -> `VMOT` of **TB6612FNG Driver 1** (Arm Elbow & Wrist)
   * -> `VMOT` of **TB6612FNG Driver 2** (Arm Gripper & Shoulder)
   * -> `IN+` of the **Buck Converter**
3. **From the Buck Converter (5V Stable Logic Line):**
   * -> `5V Pin` of **ESP32-S3**
   * -> `VCC` and `V+` of **PCA9685 Driver 1 & 2**
   * -> `HV (High Voltage)` of **I2C Logic Level Converter**
   * -> `VCC` of **TB6612FNG Driver 1 & 2**

### 2.3 Comprehensive Pin Mapping & Wiring Guide (A-Z)
*This section details the exact pin-to-pin wiring extracted directly from the ESP32-S3 C++ firmware source code.*

#### A. I2C Bus & Logic Level Translation
* **ESP32-S3 `3.3V Pin`** -> Logic Converter `LV (Low Voltage)`
* **ESP32-S3 `GND`** -> Logic Converter `GND`
* **ESP32-S3 `GPIO 1 (SDA)`** -> Logic Converter `LV1` -> Logic Converter `HV1` -> **PCA9685 (#1 & #2) `SDA`**
* **ESP32-S3 `GPIO 2 (SCL)`** -> Logic Converter `LV2` -> Logic Converter `HV2` -> **PCA9685 (#1 & #2) `SCL`**

#### B. PCA9685 I2C Addressing (CRITICAL)
Since we are using **two** PCA9685 boards on the same I2C bus, they MUST have different addresses.
* **PCA9685 Board 1 (Chassis Wheels):** 
  * Keep exactly as it comes from the factory.
  * **Default Address:** `0x40`.
* **PCA9685 Board 2 (Robotic Arm):** 
  * You must change its address to `0x41`. 
  * **How to do it:** Look at the top right of the second PCA9685 module. You will see solder pads labeled A0, A1, A2, etc. Use a soldering iron to put a drop of solder across the two halves of the **A0** pad, bridging them together. This changes the hardware address to `0x41`.

#### C. PCA9685 to Motor Drivers Control Wiring (Full Mapping)
*The PCA9685 sends PWM signals (Speed) and Logic High/Low (Direction) to the motor drivers based on the `HardwareController.cpp` definitions.*

**PCA9685 Board 1 (Address 0x40) -> Drive Wheels:**
* **Front Left Wheel:**
  * Channel 0 -> `ENA` (Speed)
  * Channel 2 -> `IN1` (Direction Forward)
  * Channel 1 -> `IN2` (Direction Reverse)
* **Back Left Wheel:**
  * Channel 5 -> `ENB` (Speed)
  * Channel 4 -> `IN3` (Direction Forward)
  * Channel 3 -> `IN4` (Direction Reverse)
* **Front Right Wheel:**
  * Channel 6 -> `ENA` (Speed)
  * Channel 8 -> `IN1` (Direction Forward)
  * Channel 7 -> `IN2` (Direction Reverse)
* **Back Right Wheel:**
  * Channel 11 -> `ENB` (Speed)
  * Channel 10 -> `IN3` (Direction Forward)
  * Channel 9 -> `IN4` (Direction Reverse)
* **TB6612FNG Standby Control:**
  * Channel 15 -> Pulled HIGH to enable TB6612FNG `STBY` pins.

**PCA9685 Board 2 (Address 0x41) -> Robotic Arm Joints:**
* **Arm Wrist:**
  * Channel 0 -> `PWMA` (Speed)
  * Channel 1 -> `AIN1` (Direction)
  * Channel 2 -> `AIN2` (Direction)
* **Arm Elbow:**
  * Channel 3 -> `PWMB` (Speed)
  * Channel 4 -> `BIN1` (Direction)
  * Channel 5 -> `BIN2` (Direction)
* **Arm Shoulder:**
  * Channel 6 -> `ENA` (Speed)
  * Channel 7 -> `IN1` (Direction)
  * Channel 8 -> `IN2` (Direction)
* **Arm Gripper:**
  * Channel 9 -> `PWMA` (Speed)
  * Channel 15 -> `AIN1` (Direction)
  * Channel 11 -> `AIN2` (Direction)
* **Arm Base:**
  * Channel 12 -> `ENB` (Speed)
  * Channel 13 -> `IN3` (Direction)
  * Channel 14 -> `IN4` (Direction)

---

## Part 3: Technical / Developer Manual

### 3.1 Software & System Architecture
* **Frontend (React/Vite):** Utilizes functional components and custom React hooks (`useVoiceCommand`, `useWebSocket`). State management controls the 5-DOF sliders.
* **Backend Firmware (FreeRTOS):** 
  * `Core 0`: Pinned for the OV2640 camera capture and HTTP chunked MJPEG response. This ensures video encoding never interrupts physical movement.
  * `Core 1`: Handles WebSocket asynchronous events. Parses JSON strings via `ArduinoJson` library.

### 3.2 WebSocket Communication JSON Protocol
When a user clicks a button or speaks, the frontend sends a highly structured JSON payload. 

**Example Payload for Rover Movement:**
```json
{
  "command_type": "drive",
  "direction": "FORWARD",
  "speed_pwm": 255
}
```

**Example Payload for Arm Movement (Absolute Angle):**
```json
{
  "command_type": "arm",
  "joint": "shoulder",
  "angle": 120
}
```

### 3.3 Version Control & Troubleshooting Matrix

| Issue / Fault Symptom | Probable Cause | Diagnostic Check | Corrective Action |
| :--- | :--- | :--- | :--- |
| **Motors not responding** | PWM Jumper still present | Inspect L298N ENA/ENB headers | Remove black jumper caps from L298N |
| **ESP32 reboots continuously**| Power conflict on 5V pin | Check L298N 5V-EN jumper | Remove 5V-EN jumper cap on all drivers |
| **Camera stream failure** | GPIO pin conflict | Check connection to GPIO 4 / 5 | Disconnect any peripherals on pins 4/5 |
| **Erratic motor movement** | Common ground loop/potentials | Check GND continuity bus | Tie all module (Driver, ESP32, Buck) GNDs together |
| **`pnpm` command not found**| Node.js path error | Open PowerShell, type `node -v` | Re-install Node.js and check "Add to PATH" |

### 3.4 Official GitHub Repository
All source code is maintained openly.
* **Repository:** [https://github.com/syedzayed-i2p/Ares_01](https://github.com/syedzayed-i2p/Ares_01)
* **License:** MIT Open Source

---
*End of Documentation. Developed by the ARES-01 Project Team.*
