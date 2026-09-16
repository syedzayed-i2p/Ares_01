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

### 2.3 Comprehensive Pin Mapping & Wiring Diagram
*(Note: Digital mapping based on I2C routing and standard H-Bridge control)*

**Data Communication (I2C Bus):**
* **ESP32-S3 `SDA Pin`** ➔ Logic Converter `LV1` ➔ Logic Converter `HV1` ➔ **PCA9685 `SDA`**
* **ESP32-S3 `SCL Pin`** ➔ Logic Converter `LV2` ➔ Logic Converter `HV2` ➔ **PCA9685 `SCL`**

**PCA9685 to Motor Driver (Example for 1 Wheel):**
* PCA9685 `PWM Channel 0` ➔ L298N `ENA` (Speed Control)
* PCA9685 `PWM Channel 1` ➔ L298N `IN1` (Direction Forward)
* PCA9685 `PWM Channel 2` ➔ L298N `IN2` (Direction Reverse)
* L298N `OUT1` & `OUT2` ➔ Positive & Negative terminals of the DC Wheel Motor.
*(This exact 3-pin logic is repeated across all PCA channels for the 4 wheels and 5 arm joints).*

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
