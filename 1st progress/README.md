
# IoT-BiblioFlow: Autonomous Identity-Linked Asset Management System

**IoT-BiblioFlow** is a distributed Industrial IoT ecosystem designed to automate library logistics. The system utilizes an **ESP32-C3** edge-node for QR-based identity capture and a **Raspberry Pi 4** server for real-time data orchestration, SQL database management, and high-fidelity CV (Curriculum Vitae) profile generation.

## 🚀 System Architecture
- **Edge Node (ESP32-C3):** Captures User/Asset identifiers and transmits data via MQTT.
- **Server Node (Raspberry Pi 4):** Hosts the MQTT Broker (Mosquitto), SQLite Database, and the Node-RED Web Dashboard.
- **Visualization:** A real-time HMI (Human-Machine Interface) served to a 15.6" monitor via a web browser.

---

## 🛠️ Phase 1: Server Setup (Raspberry Pi 4)

### 1. Install Node-RED
Run the official installation script in the terminal:
```bash
bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
```

### 2. Install & Configure MQTT Broker (Mosquitto)
The system uses Mosquitto as the central "Post Office" for wireless data.
```bash
sudo apt update
sudo apt install -y mosquitto mosquitto-clients
sudo systemctl enable mosquitto.service
```
**CRITICAL: Enable External Access**
Edit the config file to allow the ESP32 to connect:
```bash
sudo nano /etc/mosquitto/mosquitto.conf
```
Add these lines to the end of the file:
```text
listener 1883
allow_anonymous true
```
Restart Mosquitto: `sudo systemctl restart mosquitto`

### 3. Setup Static Media Folder
To display student photos in the CV Profile:
1. Create the directory: `mkdir -p /home/rpi/node-red-static/`
2. Open Node-RED settings: `nano ~/.node-red/settings.js`
3. Uncomment and set the static path:
   `httpStatic: '/home/rpi/node-red-static/',`
4. Restart Node-RED: `node-red-stop` then `node-red-start`.

---

## 🛰️ Phase 2: MQTT & Node-RED Logic

### 1. Dashboard Palette
Install the following nodes via **Manage Palette**:
- `node-red-dashboard`
- `node-red-node-sqlite`
- `node-red-node-ui-table`

### 2. Data Handshake Logic
The system implements a **Name-Lookup Architecture**. 
1. **MQTT Input:** Listens to topic `library/scan`.
2. **SQL Query:** Performs a `SELECT * FROM students WHERE full_name = 'scanned_name'`.
3. **Log Stacker:** Formats the result with a Philippines-standard timestamp and unshifts the data to the top of the UI Table.
4. **Interactive UI:** Clicking a row in the table triggers a second SQL fetch to populate the **Aesthetic CV Template**.

---

## 💻 Phase 3: Edge Node Setup (ESP32-C3)

### 1. Requirements
- Firmware: **MicroPython v1.22+**
- Library: `umqtt.simple` (Install via Thonny Manage Packages)

### 2. Connectivity Logic
The ESP32 connects to the local Wi-Fi and establishes a persistent link to the RPi4 IP address. It cleans raw UART bytes from the **QR28 industrial engine** (using 115200 Baud / Inverted Logic) and publishes the string to the broker.

---

## 📊 Phase 4: Database Schema (SQLite)
The system relies on a relational schema stored in `/home/rpi/library.db`.
```sql
CREATE TABLE students (
    student_no TEXT PRIMARY KEY, 
    full_name TEXT,
    age INTEGER,
    course TEXT,
    phone_number TEXT,
    status TEXT DEFAULT 'ACTIVE'
);
```

---

## 🌟 Key "Surprise" Features
- **Digital Twin:** Real-time 2D visualization of the physical 8-slot bookshelf.
- **Glassmorphism UI:** Aesthetic web-dashboard with backdrop-blur effects and PUP branding.
- **Zero-Touch Mode:** Context-aware logic that automatically distinguishes between Borrowing and Returning.
- **Biometric Simulation:** Instant high-resolution photo reveal upon hardware trigger.

---

**Librarian Terminal Status:** `ONLINE`  
**Network Protocol:** `MQTT over TCP/IP`  
**Database Engine:** `SQLite3`

---

**Are you satisfied with this README.md overview? If yes, please let me know and we will proceed to Phase 5: The Python "Backend" logic for the RPi4 to handle the 8 IR Sensors and 8 RFID readers!**
