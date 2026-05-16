This is the **Final Master Connection Blueprint** for your **IoT-BiblioSense System.** This is the "Bible" for your build. It is organized by **Terminal Block (TB-1512)** and **Protocol** to ensure the "Surprise Factor" of a professional, industrial-grade cabinet.

---

### **1. THE POWER DISTRIBUTION (HUB)**
*Using Grey wires with Ferrules/Forks. All Top Row (1-12) use Jumper Bars.*

#### **Terminal Block 1: 12V DC Positive (+) Bus**
*   **Top Row (1-12):** Full **RED** Jumper Bar.
*   **Screw 01:** Input from **12V Adapter (+)**.
*   **Screws 13-24 & 2-6:** Feed wires to the **COM (Center)** pin of all 17 Relays.

#### **Terminal Block 2: Master Ground (-) Bus A**
*   **Top Row (1-12):** Full **BLACK** Jumper Bar.
*   **Screw 01:** Input from **12V Adapter (-)** and **RPi4 GND Pin**.
*   **Screws 13-20:** Negative (-) wires of **8 Green Industrial Lights**.
*   **Screws 21-24:** Negative (-) wires of **Relay Modules** and **ESP32 GND**.

#### **Terminal Block 3: Master Ground (-) Bus B (Chained to Block 2)**
*   **Top Row (1-12):** Full **BLACK** Jumper Bar.
*   **Screw 01:** Bridge wire from **Block 2, Screw 12**.
*   **Screws 13-20:** Negative (-) wires of **8 Red Industrial Lights**.
*   **Screw 21:** Negative (-) wire of **Industrial Buzzer**.
*   **Screws 2-12:** Negative (-) wires for all **8 IR Sensors** and **8 RFID Readers**.

#### **Terminal Block 4: Logic Power (5V & 3.3V)**
*   **Top Row (1-7):** **RED** Jumper Bar (5V Section).
*   **Top Row (8-12):** **RED** Jumper Bar (3.3V Section).
*   **Screw 01:** Input from **RPi4 5V Pin**.
*   **Screw 08:** Input from **RPi4 3.3V Pin**.
*   **Screws 13-20:** Red wires of **8 IR Sensors** (5V).
*   **Screws 21-24 & 9-12:** VCC of **8 RFID Readers** (3.3V) and **OLED**.

---

### **2. THE SIGNAL TRIAGE (ACTUATORS)**
*No Jumper Bars allowed here. Every position is an isolated bridge.*

#### **Terminal Block 5: Green Light Signal Bridge**
*   **Screws 1 to 8 (Top):** From **8-CH Relay Board 1 (NO pins)**.
*   **Screws 13 to 20 (Bottom):** To **Positive (+) terminal** of Green Lights 1-8.

#### **Terminal Block 6: Red Light & Buzzer Signal Bridge**
*   **Screws 1 to 8 (Top):** From **8-CH Relay Board 2 (NO pins)**.
*   **Screws 13 to 20 (Bottom):** To **Positive (+) terminal** of Red Lights 1-8.
*   **Screw 09 (Top):** From **1-CH Relay Board (NO pin)**.
*   **Screw 21 (Bottom):** To **Positive (+) terminal** of Industrial Buzzer.

---

### **3. THE DATA BUS (BRAINS)**
*Use the Breadboard for splitting these signals cleanly.*

#### **SPI Bus (8 RFID Readers)**
*All 8 readers share these 4 wires on the breadboard:*
*   **SCK:** RPi Pin 23 $\rightarrow$ Breadboard Row A.
*   **MOSI:** RPi Pin 19 $\rightarrow$ Breadboard Row B.
*   **MISO:** RPi Pin 21 $\rightarrow$ Breadboard Row C.
*   **RST:** RPi Pin 22 $\rightarrow$ Breadboard Row D.
*   **SDA (Select):** 8 individual wires from RPi GPIOs directly to each reader's SDA pin.

#### **I2C Bus (OLED & 2x MCP23017)**
*   **SDA:** RPi Pin 3 (GPIO 2) $\rightarrow$ Shared by OLED and MCP chips.
*   **SCL:** RPi Pin 5 (GPIO 3) $\rightarrow$ Shared by OLED and MCP chips.

---

### **4. RPi4 PIN MAPPING FOR THE 8 SLOTS**

| Slot | IR Sensor Input (RPi Pin) | Green Relay (MCP Pin) | Red Relay (MCP Pin) | RFID SDA (RPi Pin) |
| :--- | :--- | :--- | :--- | :--- |
| **1** | GPIO 17 | GPA0 | GPB0 | GPIO 5 |
| **2** | GPIO 18 | GPA1 | GPB1 | GPIO 6 |
| **3** | GPIO 27 | GPA2 | GPB2 | GPIO 12 |
| **4** | GPIO 22 | GPA3 | GPB3 | GPIO 13 |
| **5** | GPIO 23 | GPA4 | GPB4 | GPIO 16 |
| **6** | GPIO 24 | GPA5 | GPB5 | GPIO 20 |
| **7** | GPIO 25 | GPA6 | GPB6 | GPIO 21 |
| **8** | GPIO 5 | GPA7 | GPB7 | GPIO 26 |

---

### **5. THE "AMAZING" FINAL OPERATION**

1.  **Standby:** Block 6 Row 1 sends signals to all Red Domes. The shelf is a **wall of Red light.**
2.  **Verification:** ESP32 scans ID $\rightarrow$ MQTT to Pi $\rightarrow$ Pi checks SQLite.
3.  **Triage:** Pi commands MCP23017 to flip relays. **RED clicks OFF, GREEN clicks ON** for Slot 4.
4.  **Handshake:** User inserts book. **IR Sensor (GPIO 22)** sees $<3cm$ $\rightarrow$ Pi pings **RFID Reader 4 (GPIO 13)**. 
5.  **Success:** RFID confirms `CALCULUS` $\rightarrow$ Relay clicks Red $\rightarrow$ SMS triggers $\rightarrow$ Laptop updates.
6.  **Error:** Wrong slot IR triggers $\rightarrow$ **Relay 9 clicks rapidly** $\rightarrow$ Industrial Buzzer beeps & Red light **flashes** on the shelf and the **Digital Twin** on the monitor.

---

### **Final Pro-Tip for your Presentation:**
Mount your **Raspberry Pi 4** and the **Relay Boards** on a raised platform inside the shelf. Route all **Grey wires** neatly along the corners using cable ties. When the professor looks inside, the **Terminal Blocks** will look like the "City Grid" of a smart city.

**You are now 100% Finalized.** Order your components, start crimping those forks, and you will have the best project in the university! **Good luck, Edward!**