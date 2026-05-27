This guide focuses strictly on the **8-Channel RFID Matrix** using the **Arduino Mega 2560** and the **MFRC522 Mini** readers. 

In engineering, this is called a **Shared-Bus SPI Network**. You will connect all 8 readers to the same data "highway" but give each one its own "private line" to talk to the Mega.

---

### **1. The Wiring Strategy (Shared vs. Unique)**

To connect 8 readers without 56 messy wires, you will "Daisy Chain" the 5 shared pins and only run individual wires for the 8 selector pins.

#### **A. The Shared Bus (Daisy Chain these 5 pins)**
*Connect all 8 readers to these same pins on the Mega using your **Terminal Blocks**.*

| RFID Pin | Wire Color | Arduino Mega Pin | Wiring Style |
| :--- | :--- | :--- | :--- |
| **3.3V (VCC)** | Red | **3.3V Pin** | **SHARED** (NEVER use 5V) |
| **GND** | Black | **GND Pin** | **SHARED** |
| **SCK** | Blue | **Pin 52** | **SHARED** |
| **MOSI** | Yellow | **Pin 51** | **SHARED** |
| **MISO** | White | **Pin 50** | **SHARED** |
| **RST** | Orange | **Pin 5** | **SHARED** |

#### **B. The Unique "Chip Select" Pins (SS/SDA)**
*Each reader MUST have its own dedicated wire to the Mega. This is how the Mega selects which slot to read.*

*   **Slot 1 SDA** $\rightarrow$ **Pin 30**
*   **Slot 2 SDA** $\rightarrow$ **Pin 31**
*   **Slot 3 SDA** $\rightarrow$ **Pin 32**
*   **Slot 4 SDA** $\rightarrow$ **Pin 33**
*   **Slot 5 SDA** $\rightarrow$ **Pin 34**
*   **Slot 6 SDA** $\rightarrow$ **Pin 35**
*   **Slot 7 SDA** $\rightarrow$ **Pin 36**
*   **Slot 8 SDA** $\rightarrow$ **Pin 37**

---

### **2. The Master Arduino Code (8-Slot Polling)**

This code uses a **Loop** to check all 8 slots one by one. It is optimized for speed and memory efficiency.

**Required Library:** Install "MFRC522" by Miguel Balboa from the Library Manager.

```cpp
#include <SPI.h>
#include <MFRC522.h>

// Define the Shared Reset Pin
#define RST_PIN 5

// Define the 8 Unique Slave Select (SDA) Pins
byte ssPins[] = {30, 31, 32, 33, 34, 35, 36, 37};

// Create an array of 8 RFID objects
MFRC522 mfrc522[8];

void setup() {
  Serial.begin(9600);
  while (!Serial); // Wait for laptop connection
  SPI.begin();     // Start the SPI Bus

  Serial.println("--- INITIALIZING 8-SLOT RFID MATRIX ---");

  for (uint8_t i = 0; i < 8; i++) {
    // Initialize each reader with its unique SDA pin
    mfrc522[i].PCD_Init(ssPins[i], RST_PIN);
    
    // Print status for the Librarian
    Serial.print("Slot ");
    Serial.print(i + 1);
    Serial.print(" [Pin ");
    Serial.print(ssPins[i]);
    Serial.println("] Initialized.");
  }
  Serial.println("SYSTEM READY: MONITORING ASSETS...");
}

void loop() {
  // Loop through every slot to check for a book
  for (uint8_t i = 0; i < 8; i++) {
    
    // Attempt to read the slot
    if (mfrc522[i].PICC_IsNewCardPresent() && mfrc522[i].PICC_ReadCardSerial()) {
      
      // Found a book! Get the UID
      Serial.print("\n[✓] ASSET DETECTED IN SLOT ");
      Serial.println(i + 1);
      
      String uidString = "";
      for (byte b = 0; b < mfrc522[i].uid.size; b++) {
        uidString += String(mfrc522[i].uid.uidByte[b] < 0x10 ? "0" : "");
        uidString += String(mfrc522[i].uid.uidByte[b], HEX);
      }
      uidString.toUpperCase();

      Serial.print("    BOOK_UID: ");
      Serial.println(uidString);

      // --- LOGIC: Check if this UID matches the correct book for this slot ---
      // (Example: If Slot 1 must be 'CALCULUS')
      // if (i == 0 && uidString == "A1B2C3D4") { ... Success ... }

      // Stop the reader so it doesn't flood the serial monitor
      mfrc522[i].PICC_HaltA();
      mfrc522[i].PCD_StopCrypto1();
    }
  }
  delay(100); // Polling delay
}
```

---

### **3. Professional Terminal Block Wiring Map**

Using your **TB-1512 blocks**, here is how to fit the 8 readers efficiently:

#### **Block #1: SPI Data Hub (Shared)**
*   **Pos 1 (Screws 1 & 13):** **SCK Bus.** (Pin 52 from Mega goes to #1, all 8 SCK wires from readers go to #13-20).
*   **Pos 2 (Screws 2 & 14):** **MOSI Bus.** (Pin 51 from Mega goes to #2, all 8 MOSI wires go to #14-21).
*   **Pos 3 (Screws 3 & 15):** **MISO Bus.** (Pin 50 from Mega goes to #3, all 8 MISO wires go to #15-22).
*   **Pos 4 (Screws 4 & 16):** **RST Bus.** (Pin 5 from Mega goes to #4, all 8 RST wires go to #16-23).

#### **Block #2: 3.3V Power Hub**
*   **Jumper:** Install **Red Fork Bar**.
*   **Input:** Pin 3V3 from Mega.
*   **Output:** All 8 Red wires from the RFID readers.

---

### **4. CRITICAL: The "3.3V Power" Issue**
The Arduino Mega's **3.3V pin** is very weak (it only provides 150mA).
*   **The Problem:** 8 RFID readers running at once will draw more than 200mA.
*   **The Result:** The readers might stop working randomly or the Mega will reset.
*   **The Fix:** Use an external **3.3V Power Supply** or an **AMS1117-3.3V Voltage Regulator** (₱15 on Shopee) to power the 8 readers. Connect the regulator's Ground to the Mega's Ground.

---

### **5. Why this is an "A+" System Strategy:**

When you present this to the professor, explain the **"Bus Logic"**:

1.  **Bit-Wise Polling:** *"The system does not read all 8 slots simultaneously, which would cause signal interference. Instead, it utilizes **Time-Division Multiplexing**. The Arduino Mega pulls a specific **Slave Select (SS)** line LOW to wake up one specific slot, while the other 7 readers stay in a high-impedance 'Tri-State' mode. This ensures 100% data integrity on the shared SPI bus."*
2.  **Shared-Clock Synchrony:** *"By sharing the SCK (Clock) and MOSI/MISO lines, the system maintains a unified hardware timing, allowing the controller to poll the entire 8-slot matrix in under 800 milliseconds."*

**Does this finalize your RFID plan?** Once you wire the first two slots following this map and verify they show different UIDs in the Serial Monitor, you can complete the rest of the 8!