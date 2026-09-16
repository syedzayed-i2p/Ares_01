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
  * 2.3 Comprehensive Pin Mapping & Wiring Diagram
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
| 2 | **PCA9685 PWM Driver** | 2 | Since the ESP32 doesn't have enough PWM pins for 9 motors, these drivers expand the capability. They convert simple I2C signals into 16 channels of perfect PWM signals. |
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
**WARNING:** Never connect the 11.1V LiPo battery directly to the ESP32.
1. **LiPo Battery (11.1V)** connects to the **Main Switch**.
2. **From the Switch:**
   * **Path A (High Power):** 11.1V goes directly into the `12V Input` of the **L298N Motor Drivers**.
   * **Path B (Logic Power):** 11.1V goes into the `IN+` of the **Buck Converter**.
3. **From the Buck Converter:**
   * Outputs a stable **5V**.
   * 5V goes to the **ESP32-S3 `5V Pin`**.
   * 5V goes to the **PCA9685 `VCC/V+ Pin`**.
   * 5V goes to the **Logic Level Converter `HV Pin`**.

### 2.3 Comprehensive Pin Mapping & Wiring Guide (A-Z)
This section details the exact pin-to-pin wiring for every component in the system to ensure correct power delivery and logic control.

#### A. Power Distribution Network
**WARNING:** Never connect the 11.1V LiPo battery directly to the ESP32 or Logic components.
* **LiPo Battery (11.1V 3S)** ➔ `XT60 Connector` ➔ `Main Power Switch`.
* **From the Main Switch (11.1V High-Power Line):**
  * ➔ `12V Input` of **L298N Motor Driver 1** (Front Wheels)
  * ➔ `12V Input` of **L298N Motor Driver 2** (Rear Wheels)
  * ➔ `12V Input` of **L298N Motor Driver 3** (Arm Base & Shoulder)
  * ➔ `VMOT` of **TB6612FNG Driver 1** (Arm Elbow & Wrist)
  * ➔ `VMOT` of **TB6612FNG Driver 2** (Arm Gripper)
  * ➔ `IN+` of the **Buck Converter**
* **From the Buck Converter (5V Stable Logic Line):**
  * ➔ `5V Pin` of **ESP32-S3**
  * ➔ `VCC` and `V+` of **PCA9685 Driver 1 & 2**
  * ➔ `HV (High Voltage)` of **I2C Logic Level Converter**
  * ➔ `VCC` of **TB6612FNG Driver 1 & 2**

#### B. I2C Bus & Logic Level Translation
Because the ESP32-S3 operates at 3.3V and the PCA9685 operates best at 5V logic, the signals must pass through the Logic Converter.
* **ESP32-S3 `3.3V Pin`** ➔ Logic Converter `LV (Low Voltage)`
* **ESP32-S3 `GND`** ➔ Logic Converter `GND`
* **ESP32-S3 `SDA Pin`** ➔ Logic Converter `LV1` ➔ Logic Converter `HV1` ➔ **PCA9685 (#1 & #2) `SDA`**
* **ESP32-S3 `SCL Pin`** ➔ Logic Converter `LV2` ➔ Logic Converter `HV2` ➔ **PCA9685 (#1 & #2) `SCL`**

#### C. PCA9685 I2C Addressing (CRITICAL)
Since we are using **two** PCA9685 boards on the same I2C bus to get 32 PWM channels, they MUST have different addresses to avoid conflicts.
* **PCA9685 Board 1 (Rover Wheels & Heavy Arm Joints):** 
  * Keep exactly as it comes from the factory.
  * **Default Address:** `0x40`.
* **PCA9685 Board 2 (Precision Arm Joints):** 
  * You must change its address to `0x41`. 
  * **How to do it:** Look at the top right of the second PCA9685 module. You will see solder pads labeled A0, A1, A2, etc. Use a soldering iron to put a drop of solder across the two halves of the **A0** pad, bridging them together. This changes the hardware address to `0x41`.

#### D. PCA9685 to Motor Drivers Control Wiring (Full Mapping)
*The PCA9685 modules send PWM signals (Speed) and Logic High/Low (Direction) to the motor drivers.*

**PCA9685 Board 1 (Address `0x40`) ➔ L298N Drivers (Drive Wheels & Arm Base):**
* **L298N #1 (Front Wheels):**
  * Channel 0 ➔ `ENA` (Left Front Speed)
  * Channel 1, 2 ➔ `IN1`, `IN2` (Left Front Direction)
  * Channel 3 ➔ `ENB` (Right Front Speed)
  * Channel 4, 5 ➔ `IN3`, `IN4` (Right Front Direction)
* **L298N #2 (Rear Wheels):**
  * Channel 6 ➔ `ENA` (Left Rear Speed)
  * Channel 7, 8 ➔ `IN1`, `IN2` (Left Rear Direction)
  * Channel 9 ➔ `ENB` (Right Rear Speed)
  * Channel 10, 11 ➔ `IN3`, `IN4` (Right Rear Direction)
* **L298N #3 (Arm Base & Shoulder - Partial):**
  * Channel 12 ➔ `ENA` (Arm Base Speed)
  * Channel 13, 14 ➔ `IN1`, `IN2` (Arm Base Direction)
  * Channel 15 ➔ `ENB` (Arm Shoulder Speed)

**PCA9685 Board 2 (Address `0x41`) ➔ Overflow & TB6612FNG Drivers (Arm Precision Joints):**
* **L298N #3 (Arm Shoulder - Continued):**
  * Channel 0, 1 ➔ `IN3`, `IN4` (Arm Shoulder Direction)
* **TB6612FNG #1 (Arm Elbow & Wrist):**
  * Channel 2 ➔ `PWMA` (Elbow Speed)
  * Channel 3, 4 ➔ `AIN1`, `AIN2` (Elbow Direction)
  * Channel 5 ➔ `PWMB` (Wrist Speed)
  * Channel 6, 7 ➔ `BIN1`, `BIN2` (Wrist Direction)
* **TB6612FNG #2 (Arm Gripper):**
  * Channel 8 ➔ `PWMA` (Gripper Speed)
  * Channel 9, 10 ➔ `AIN1`, `AIN2` (Gripper Direction)

#### E. Motor Drivers to Physical Motors
* **L298N #1 `OUT1` / `OUT2`** ➔ Front Left DC Wheel Motor
* **L298N #1 `OUT3` / `OUT4`** ➔ Front Right DC Wheel Motor
* **L298N #2 `OUT1` / `OUT2`** ➔ Rear Left DC Wheel Motor
* **L298N #2 `OUT3` / `OUT4`** ➔ Rear Right DC Wheel Motor
* **L298N #3 `OUT1` / `OUT2`** ➔ Arm Base Rotation Gear Motor
* **L298N #3 `OUT3` / `OUT4`** ➔ Arm Shoulder Elevation Gear Motor
* **TB6612FNG #1 `AOUT1` / `AOUT2`** ➔ Arm Elbow Gear Motor
* **TB6612FNG #1 `BOUT1` / `BOUT2`** ➔ Arm Wrist Gear Motor
* **TB6612FNG #2 `AOUT1` / `AOUT2`** ➔ Arm Gripper Actuator Motor

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
  "joint": "SHOULDER",
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
