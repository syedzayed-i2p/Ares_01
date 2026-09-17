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
  * 2.1 3D Printed Chassis (Rover Body)
  * 2.2 Detailed Component List & Usage Justification
  * 2.3 System Power Distribution (11.1V & 5V Logic)
  * 2.4 I2C Logic Level Translation
  * 2.5 Comprehensive Motor Driver Wiring (Pin-to-Pin)
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
   **`http://localhost:1420`**

### 1.4 Mission Control Dashboard Guide (How to Use)
Once the web app is open, follow these steps to connect and control the rover:

**1. System Settings & Connecting to the Rover (⚙️ Icon)**
*   Look at the top-right corner of the dashboard and click the **Settings (⚙️ Gear)** icon.
*   A modal titled **SYSTEM SETTINGS** will appear. Under "Hardware Connections", find the **Rover IP Address** field.
*   Enter your Rover's local IP Address (e.g., `192.168.4.1` for Offline AP) and click **Connect**.
*   The "ROVER OFFLINE" red badge at the top-left will turn into a green "ROVER ONLINE" badge, and telemetry (Ping and RSSI) will activate.

**2. Manual Control Tab (Arm & Drive)**
*   **Drive Controls (Directional Pad):** Located at the bottom right. Use the Up, Down, Left, and Right arrow buttons to move the rover. The red square in the middle acts as a quick STOP button.
*   **5-DOF Arm Control:** Use the sliders or the increment/decrement arrows to rotate the Base, Shoulder, Elbow, Wrist, and Gripper. You can also manually type an exact coordinate angle (e.g., `90°`) and click **EXECUTE**.

**3. Offline Macros & AI Control Tab**
*   **Offline Text Command (AI Directive):** Click the "OFFLINE MACROS" or "AI Directive" tab. Type commands manually like `"Initiate pick up sequence"` in the text box and hit send. The local AI engine will process the text offline without needing the cloud.
*   **Voice Control:** Switch to the **Voice Command** tab. Click the language dropdown and select your language (e.g., **বাংলা**). Click the large **Mic (🎙️)** button and speak (e.g., "সামনে যাও"). The transcript and system response will appear in the Voice Commands Log.

**4. ESP Studio & Serial Monitor (Terminal `>_` Icon)**
*   Click the **Terminal (`>_`)** icon at the top right to open the built-in **ESP Studio**.
*   Here you can browse the embedded C++ firmware (like `main.cpp`) directly from the browser.
*   To debug the rover, connect the ESP32 via USB and click the **Connect Serial** button on the right panel (Baud Rate: 115200) to view real-time hardware logs.

**5. UI Theme Customization (🌙/☀️ Icon)**
*   Click the **Moon/Sun** icon at the top-right corner to toggle the dashboard's visual theme between **Dark Mode**, **Light Mode**, or System Default for comfortable viewing in any environment.

---

## Part 2: Build / Assembly Manual & Hardware Details

### 2.1 3D Printed Chassis (Rover Body)
The physical structure of ARES-01 is based on a highly robust open-source 3D printable model. If anyone wishes to replicate this rover entirely from scratch, they can easily download the chassis components and 3D print them (PLA or PETG recommended).

* **Download Link:** [Mini Mars Rover on Printables](https://www.printables.com/model/791157-mini-mars-rover)
* **Integration:** This chassis perfectly houses the 4 DC Rover Wheels and provides a stable mounting platform for the 5-DOF Robotic Arm Edge Kit on top.

### 2.2 Detailed Component List & Usage Justification
Every component in ARES-01 serves a specific purpose in the architecture.

| Sl | Component Name | Qty | Specific Usage in Project |
| :---: | :--- | :---: | :--- |
| 1 | **ESP32-S3 CAM Dev Board (with OV2640 & Antenna)** | 1 | The absolute "Brain" of the rover. It processes the camera video, hosts the Wi-Fi WebSocket server, and issues I2C commands to move motors. |
| 2 | **PCA9685 PWM Driver** | 2 | Since the ESP32 doesn't have enough PWM pins for 9 motors, these drivers expand the capability. They convert simple I2C signals into 32 channels of perfect PWM signals. |
| 3 | **I2C Logic Level Converter** | 1 | The ESP32 runs at 3.3V, but the PCA9685 prefers 5V logic. This chip bridges them, ensuring signals aren't corrupted or hardware fried. |
| 4 | **TB6612FNG Motor Driver** | 2 | Used for the smaller precision gear motors of the robotic arm (Shoulder, Elbow, Wrist, Gripper). Highly efficient for precise 4-joint control. |
| 5 | **L298N Motor Driver** | 3 | Used for the heavy-duty rover base wheels (2 drivers) and the main base rotation joint of the robotic arm (1 driver) which require high current. |
| 6 | **Voltage Regulator (Buck Converter)** | 1 | Takes the dangerous 11.1V from the LiPo battery and converts it into a perfectly safe, stable 5.0V to power the delicate ESP32 and logic boards. |
| 7 | **LiPo Battery (3300mAh, 11.1V, 3S)** | 1 | The main fuel tank. 11.1 Volts provides immense torque to the motors, while 3300mAh ensures long mission duration. |
| 8 | **T-Connector (XT60)** | 1 | A secure, spark-proof connector ensuring the high-current battery doesn't accidentally disconnect during rover movement. |
| 9 | **Switch** | 1 | The main kill-switch to power the entire system on or off instantly. |
| 10 | **Robotic Arm Edge Kit (5 DOF)** | 1 | The mechanical chassis of the arm, allowing complex object manipulation. |
| 11 | **DC Motor (Rover Wheel)** | 4 | The 4-wheel drive system enabling all-terrain navigation. |

### 2.3 System Power Distribution (11.1V & 5V Logic)
**WARNING:** Never connect the 11.1V LiPo battery directly to the ESP32 or Logic components.

| Power Source | Component | Input Voltage | Output | Target Connection |
| :--- | :--- | :---: | :---: | :--- |
| **LiPo Battery (3S)** | Main Switch | 11.1V | 11.1V | ➔ XT60 Connector |
| **Main Switch** | L298N Drivers (x3) | 11.1V | - | ➔ `12V Input` Pin |
| **Main Switch** | TB6612 Drivers (x2) | 11.1V | - | ➔ `VMOT` Pin |
| **Main Switch** | Buck Converter | 11.1V | 5.0V | ➔ `IN+` Pin |
| **Buck Converter** | ESP32-S3 | 5.0V | - | ➔ `5V` Pin |
| **Buck Converter** | PCA9685 (x2) | 5.0V | - | ➔ `VCC` & `V+` Pins |
| **Buck Converter** | Level Converter | 5.0V | - | ➔ `HV` (High Voltage Ref) |

### 2.4 I2C Logic Level Translation
Because the ESP32-S3 operates at 3.3V and the PCA9685 operates best at 5V, data must pass through the Logic Converter safely.

| ESP32-S3 (3.3V Logic) | Level Converter Bridge | PCA9685 (5V Logic) |
| :--- | :--- | :--- |
| `GPIO 1 (SDA)` | ➔ `LV1` ➔ `HV1` | ➔ `SDA` (Board 1 & 2) |
| `GPIO 2 (SCL)` | ➔ `LV2` ➔ `HV2` | ➔ `SCL` (Board 1 & 2) |
| `3.3V Pin` | ➔ `LV` (Reference) | - |
| `GND` | ➔ `GND` | ➔ `GND` (Common) |

### 2.5 Comprehensive Motor Driver Wiring (Pin-to-Pin)
*This section details the exact pin-to-pin wiring extracted directly from the ESP32-S3 C++ firmware source code (`HardwareController.cpp`).*

#### A. PCA9685 Addressing & Daisy-Chain Setup
To control 9 motors simultaneously, two PCA9685 boards must be physically connected together (Daisy-Chained) and share the same I2C bus using different hardware addresses.

**1. Hardware Addressing:**
*   **Board 1 (Drive Wheels):** Factory Default **`0x40`**.
*   **Board 2 (Robotic Arm):** Change to **`0x41`**. *(To do this: Solder the two halves of the `A0` pad together on the board).*

**2. Daisy-Chain Physical Wiring (Linking the Boards):**
Instead of connecting both boards directly to the ESP32, you connect Board 1 to the logic converter, and then plug Board 2 directly into the side header pins of Board 1.
*   `SCL` (Board 1) ➔ `SCL` (Board 2)
*   `SDA` (Board 1) ➔ `SDA` (Board 2)
*   `VCC` (Board 1) ➔ `VCC` (Board 2) *(5V Logic Power)*
*   `GND` (Board 1) ➔ `GND` (Board 2)
*   `V+` (Board 1) ➔ `V+` (Board 2) *(Motor Power)*

#### B. PCA Board 1 (0x40) ➔ 2x L298N (Drive Wheels)
*Used exclusively for the 4-wheel drive system.*

| Motor Driver | Target Motor | PWM (Speed) | Direction 1 | Direction 2 |
| :--- | :--- | :--- | :--- | :--- |
| **L298N #1 (Left)** | Front Left Wheel | PCA1 `Ch 0` ➔ `ENA` | PCA1 `Ch 2` ➔ `IN1` | PCA1 `Ch 1` ➔ `IN2` |
| | Back Left Wheel | PCA1 `Ch 5` ➔ `ENB` | PCA1 `Ch 4` ➔ `IN3` | PCA1 `Ch 3` ➔ `IN4` |
| **L298N #2 (Right)**| Front Right Wheel| PCA1 `Ch 6` ➔ `ENA` | PCA1 `Ch 8` ➔ `IN1` | PCA1 `Ch 7` ➔ `IN2` |
| | Back Right Wheel | PCA1 `Ch 11` ➔ `ENB`| PCA1 `Ch 10` ➔ `IN3`| PCA1 `Ch 9` ➔ `IN4` |
*(Note: PCA1 Channel 15 is pulled HIGH in firmware to optionally enable TB6612 STBY pins if shared).*

#### C. PCA Board 2 (0x41) ➔ 1x L298N & 2x TB6612FNG (Robotic Arm)
*Used exclusively for the 5-DOF Robotic Arm.*

| Motor Driver | Target Arm Joint | PWM (Speed) | Direction 1 | Direction 2 |
| :--- | :--- | :--- | :--- | :--- |
| **L298N #3 (Heavy)**| Base Rotation | PCA2 `Ch 12` ➔ `ENA` | PCA2 `Ch 13` ➔ `IN1` | PCA2 `Ch 14` ➔ `IN2` |
| **TB6612 #1** | Arm Wrist | PCA2 `Ch 0` ➔ `PWMA` | PCA2 `Ch 1` ➔ `AIN1` | PCA2 `Ch 2` ➔ `AIN2` |
| | Arm Elbow | PCA2 `Ch 3` ➔ `PWMB` | PCA2 `Ch 4` ➔ `BIN1` | PCA2 `Ch 5` ➔ `BIN2` |
| **TB6612 #2** | Arm Shoulder | PCA2 `Ch 6` ➔ `PWMA` | PCA2 `Ch 7` ➔ `AIN1` | PCA2 `Ch 8` ➔ `AIN2` |
| | Arm Gripper | PCA2 `Ch 9` ➔ `PWMB` | PCA2 `Ch 15` ➔ `BIN1`| PCA2 `Ch 11` ➔ `BIN2` |

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
