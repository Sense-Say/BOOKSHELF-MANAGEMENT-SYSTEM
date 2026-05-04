This is an absolute masterpiece of an output! Clean, perfectly parsed, and filtering out all the noise and "Enter" keys. **The Scanner Station hardware is now 100% operational.**

Now, we must build the **Wireless Bridge**. We need to take that `clean_text` and "teleport" it across the room to your Node-RED dashboard on the laptop/Raspberry Pi.

---

### **Step 1: Setting up the Wi-Fi & MQTT on ESP32-C3**

To make the ESP32 talk to Node-RED, we use the **MQTT protocol**. You need to install a tiny library onto your ESP32 first.

1.  In Thonny, go to **Tools** $\rightarrow$ **Manage Packages**.
2.  Search for **`umqtt.simple`**.
3.  Click **Install**. (Make sure your ESP32 is plugged in and Thonny is connected to it).

---

### **Step 2: The "Wireless Sender" Code**

This code takes your perfect scanner logic and adds Wi-Fi. 
**Before you run this, you must change 3 things in the code:**
1.  `WIFI_SSID` (Your house/school Wi-Fi name)
2.  `WIFI_PASSWORD` (Your Wi-Fi password)
3.  `MQTT_SERVER` (The IP address of your laptop or Raspberry Pi running Node-RED, e.g., `192.168.1.15`)

**Copy this into Thonny:**

```python
from machine import UART
import network
import time
from umqtt.simple import MQTTClient

# --- 1. NETWORK SETTINGS ---
WIFI_SSID = "YOUR_WIFI_NAME"
WIFI_PASSWORD = "YOUR_WIFI_PASSWORD"
MQTT_SERVER = "192.168.1.15"  # IP of your Node-RED computer
TOPIC = b"library/scan"       # The "Subject Line" for Node-RED

# --- 2. CONNECT TO WIFI ---
print("Connecting to Wi-Fi...")
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect(WIFI_SSID, WIFI_PASSWORD)

while not wlan.isconnected():
    time.sleep(0.5)
    print(".", end="")
print("\n[✓] Wi-Fi Connected! IP:", wlan.ifconfig()[0])

# --- 3. CONNECT TO MQTT ---
print(f"Connecting to MQTT Broker at {MQTT_SERVER}...")
client = MQTTClient("ESP32_Scanner", MQTT_SERVER)
try:
    client.connect()
    print("[✓] MQTT Connected!")
except Exception as e:
    print("[X] MQTT Error. Is Mosquitto running on Node-RED machine?", e)

# --- 4. SCANNER SETUP ---
uart = UART(1, baudrate=115200, tx=21, rx=20, invert=UART.INV_RX)

print("\n=======================================")
print("  PUP SMART LIBRARY: WIRELESS SCANNER  ")
print("  Status: ONLINE | Waiting for Scan... ")
print("=======================================")

# --- 5. MAIN LOOP ---
while True:
    if uart.any():
        raw_data = uart.read()
        try:
            # Decode and clean the text
            text = raw_data.decode('utf-8')
            clean_text = "".join(i for i in text if 31 < ord(i) < 127)
            
            if clean_text:
                print(f"\n[+] SCANNED: {clean_text}")
                print("    --> Sending to Node-RED...")
                
                # Send the text wirelessly!
                client.publish(TOPIC, clean_text)
                print("    [✓] Sent successfully.")
                
        except Exception as e:
            pass
            
    time.sleep(0.1)
```

---

### **Step 3: Receiving the Data in Node-RED (The Magic Trick)**

Now we need to tell your laptop dashboard to "catch" the data the ESP32 is throwing through the air.

1.  Open **Node-RED** on your laptop.
2.  Look in the left sidebar under the **"network"** category.
3.  Drag an **`mqtt in`** node (it is purple) onto your workspace.
4.  Double-click it to set it up:
    *   **Server:** Click the pencil icon to add a new server. Name it "Local Broker" and set the Server field to `localhost`. Click Add.
    *   **Topic:** Type exactly `library/scan` (This must match the Python code).
    *   **Output:** a parsed JSON object.
    *   Click **Done**.
5.  **Connect** the purple `mqtt in` node directly to your **Log Stacker** Function node (Replacing the manual 'Inject' node we used before).
6.  Click **Deploy**.

---

### **The "A+" Moment:**

1.  Run the Python code on the ESP32.
2.  Wait for it to say `[✓] MQTT Connected!`.
3.  Scan your Custom ID (The one with your JSON data).
4.  **Look at your Laptop Monitor:** 
    *   The ESP32 reads the QR.
    *   The string travels invisibly over Wi-Fi.
    *   Node-RED catches it.
    *   Your **CV Profile** slides onto the screen automatically!

**Are you ready to test the wireless link? Make sure you update the Wi-Fi name and IP address in the code first!**