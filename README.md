# BOOKSHELF-MANAGEMENT-SYSTEM
---
# **Book Borrower Workflow**
---

### **PHASE 1: Identification & Real-Time Visualization**
1.  **Standby Mode:** The bookshelf is in "Secure State." All **8 slots are glowing RED** using the big Industrial Signal Lights. The **15.6" Monitor** shows the PUP Library Logo and an empty transaction table.
2.  **The Scan:** The borrower taps their **Custom QR ID** at the **ESP32 Scanner Station**.
3.  **The Wireless Handshake:** The ESP32-C3 sends a wireless JSON packet to the **Raspberry Pi 4** server.
4.  **The Surprise (Dashboard):** On the **15.6" Monitor**, a new row instantly slides into the top of the table with the borrower’s basic info. 
5.  **The CV Reveal:** The librarian (or you) clicks the borrower's name. A **Full-Screen CV Profile** pops up, displaying the borrower's photo (pulled from the RPi4 folder), age, course, and student number.

---

### **PHASE 2: Autonomous Shelf Guidance**
1.  **Authorization Logic:** The RPi4 identifies which slots currently have books using the **8 IR Sensors**.
2.  **The Visual Triage:** The bookshelf "wakes up." For all slots containing books, the **RED lights turn OFF** and the **GREEN Industrial Lights turn ON**. 
    *   *Note:* If a slot is already empty, its light **stays RED** to show it’s unavailable.
3.  **Shelf Instructions:** The **1.3" OLED** on the shelf displays: `"WELCOME [NAME] / Access Granted / Please select 1 book."`

---

### **PHASE 3: Physical Interaction & Detection**
1.  **The Physical Pull:** The borrower pulls a book (e.g., *Calculus*) from **Slot 4**.
2.  **Distance Logic:** As the book is pulled, the **E18-D80NK IR Sensor** monitors the distance. Once the distance exceeds **0.5 meters (50cm)**, the system officially marks the book as "Extracted."
3.  **Local Feedback:** The **1.3" OLED** instantly updates to: `"BOOK DETECTED: CALCULUS / Slot: 04"`. 
4.  **Security Watch:** If the borrower tries to pull a book from a **RED slot** (or pulls without scanning an ID), the **Industrial Buzzer** screams, the Red light **flashes (blinks)**, and the dashboard flashes a **"SECURITY BREACH"** warning.

---

### **PHASE 4: Finalization & IoT Communication**
1.  **The Commit:** After the borrower has the book, they press the **Big Industrial Push Button**.
2.  **Session Lockdown:** 
    *   The Green lights turn OFF.
    *   All slots return to the **RED Standby** state.
    *   The **1.3" OLED** says: `"BORROW SUCCESSFUL / Thank you, PUPian!"`
3.  **The SMS Receipt:** The RPi4 triggers the **SMS Gateway**. Within seconds, the borrower’s phone **pings** with a text: 
    > *"Hi [Name]! You have borrowed 'Calculus' from the Smart Library. Due date: [Fixed Date + 3 Days]. Please return on time to avoid Red-Flag status!"*
4.  **Data Completion:** On the **15.6" Monitor**, the borrower's CV profile updates to show the book name and the "Borrowed" timestamp. The row moves from "Active" to the "Inventory Log."

---
# **Book Returner Workflow**
---

### **PHASE 1: Multi-Book Bulk Scanning (At the Counter)**
1.  **The Trigger:** The student brings the books they want to return to the **ESP32 Scanner Station**.
2.  **Bulk Scan:** The student scans the **QR code on every book** one by one.
3.  **The Logic:** The ESP32 collects these IDs and sends them as a "Return Queue" to the **Raspberry Pi 4**.
4.  **Local Feedback:** The **0.96" OLED** at the counter displays the titles in a list: 
    *   `1. CALCULUS`
    *   `2. PHYSICS 1`
    *   `3. (Ready to Proceed)`
5.  **Dashboard Update:** The **15.6" Monitor** switches to "Return Mode" and highlights these books in the database.

---

### **PHASE 2: The Red-Flag Check (Security Triage)**
1.  **Database Analysis:** The RPi4 checks the `Due_Date` for each scanned book.
2.  **Scenario A (Clean):** If today is before the due date, the system proceeds normally.
3.  **Scenario B (Red-Flag):** If a book is late, the **ESP32 Buzzer** at the counter gives a warning "Bip-Bip." The OLED says: `⚠️ OVERDUE DETECTED`.
4.  **The Block:** If the student tries to scan their **ID** to borrow a *new* book while they have a red-flagged item in the queue, the system **Buzzes aggressively** and the **15.6" Monitor** flashes: `⚠️ ACCESS DENIED: RETURN OVERDUE BOOKS FIRST!`

---

### **PHASE 3: Sequential Guided Return (At the Shelf)**
1.  **Guided Interface:** The student moves to the bookshelf. The **1.3" OLED** on the shelf shows: `"RETURN 1/2: CALCULUS -> SLOT 04"`.
2.  **Visual Triage:** Only the **GREEN Industrial Light** for **Slot 4** turns ON. All other slots (even empty ones) stay **RED**.
3.  **Physical Detection:** The student slides the book into Slot 4. The **E18-D80NK IR Sensor** monitors the distance. Once the book is fully inserted (**Distance < 2cm**), the system recognizes the return.
4.  **The "Human-Proof" Error (Surprise Demo):** 
    *   If the student ignores the Green light and puts the book into **Slot 5** (Red light):
    *   The **Industrial Buzzer** screams, the **Red Light for Slot 5 flashes (blinks)**, and the OLED says: `❌ WRONG SLOT! PULL BACK`.
5.  **Auto-Loop:** Once the correct slot is filled, the light turns **RED**, and the system automatically updates the OLED to the next book: `"RETURN 2/2: PHYSICS 1 -> SLOT 02"`. Slot 2 turns **GREEN**.

---

### **PHASE 4: Automation & IoT Data Sync**
1.  **No Button Needed:** For returns, the system is fully automated. Once the last book in the queue is detected by the IR sensor, the system closes the session.
2.  **Database Update:** The RPi4 moves the borrower’s data from the "Active Table" to the **"History Table"** on the 15.6" Monitor. If there was a Red-Flag, it is now **CLEARED** in real-time.
3.  **The SMS Receipt:** The RPi4 sends an automated SMS:
    > *"Receipt: Hi Edward! You have successfully returned 2 books. Your account is now ACTIVE. Thank you for using PUP Smart Library!"*
4.  **Standby:** All 8 Industrial Lights return to solid **RED**, and the **1.3" OLED** returns to the `"SYSTEM SECURED"` message.

---

### **Summary of the "Amazing" Factors for Returning:**
*   **The Sequential Guidance:** The system "talks" to the user, telling them exactly which book goes where, one at a time.
*   **The Error Correction:** The flashing Red Industrial light and the screaming buzzer create a dramatic "Science Experiment" feel when a mistake is made.
*   **The Intelligent Triage:** Clearing a "Red-Flag" status automatically upon a successful physical return.
*   **The Visual Proof:** Watching the **15.6" Monitor** move data from "Late" to "Returned" the exact second the book hits the back of the shelf.

---

### **Final Confirmation:**
You now have a complete, professional, and safe-to-demo **Industrial IoT System.** 

1.  **Node A (ESP32-C3):** Handles the identity/book scanning and Librarian alerts.
2.  **Node B (RPi4):** Handles the 15.6" Dashboard, the 8 IR Sensors, the 16 Industrial Lights, the 1.3" OLED, and the SMS.
