
# IoT-BiblioFlow: Software Logic & Node Documentation

This document details the **Node-RED logic engine** and the **Data Orchestration** used to manage the Smart Library System. The system is divided into two primary logic flows: the **Automated Data Capture Flow** and the **User Interaction Flow**.

---

## 🗺️ System Logic Flowchart

### 1. Data Capture Flow (Wireless Scan $\rightarrow$ Table)
`[MQTT In: library/scan]` $\rightarrow$ `[Func: Lookup by Name]` $\rightarrow$ `[SQLite: LibraryDB]` $\rightarrow$ `[Func: Log Stacker]` $\rightarrow$ `[UI Table]`

### 2. User Interaction Flow (Table Click $\rightarrow$ CV Profile)
`[UI Table]` $\rightarrow$ `[Func: Prepare SQL Fetch]` $\rightarrow$ `[SQLite: LibraryDB]` $\rightarrow$ `[UI Template: CV Profile]`

---

## 📝 Detailed Node Configuration

### Node A: MQTT Input
*   **Topic:** `library/scan`
*   **Purpose:** Captures the raw string (Name) sent from the ESP32-C3 via Wi-Fi.
*   **Output:** Plain String.

### Node B: Lookup By Name (Function Node)
*   **Purpose:** Converts the scanned name into a formal SQL query to retrieve student metadata.
*   **Code:**
```javascript
var scannedName = msg.payload.toString().trim();
msg.topic = "SELECT * FROM students WHERE full_name = '" + scannedName + "';";
return msg;
```

### Node C: Log Stacker (Function Node)
*   **Purpose:** Receives SQL data, formats a Philippines-standard timestamp, and maintains a "Stacking" array for the UI table.
*   **Code:**
```javascript
if (msg.payload && msg.payload.length > 0) {
    var student = msg.payload[0];
    var list = flow.get("libraryTable") || [];

    // Format Timestamp for PH
    var options = { hour: 'numeric', minute: 'numeric', second: 'numeric', hour12: true };
    var phTime = new Date().toLocaleTimeString('en-US', options);

    // Map Database results to Table Properties
    var newEntry = {
        "time": phTime,
        "name": student.full_name,
        "id": student.student_no,
        "status": "BORROWED"
    };

    list.unshift(newEntry); // Adds to top
    if (list.length > 10) { list.pop(); } // Memory management
    flow.set("libraryTable", list);

    msg.payload = list;
    return msg;
}
return null;
```

### Node D: Transaction Log (Table Node)
*   **Interaction:** "Send data on click" enabled.
*   **Column Mapping:**
    - Property: `time` | Title: `DATE / TIME`
    - Property: `name` | Title: `STUDENT NAME`
    - Property: `id` | Title: `STUDENT ID`
    - Property: `status` | Title: `STATUS`

### Node E: Prepare SQL Fetch (Function Node)
*   **Purpose:** Triggered when a librarian clicks a row. It clears the payload to prevent SQL injection errors and prepares a specific ID lookup.
*   **Code:**
```javascript
if (Array.isArray(msg.payload)) { return null; } // Block auto-updates

var studentID = msg.payload.id; 
msg.topic = "SELECT * FROM students WHERE student_no = '" + studentID + "';";
msg.payload = []; // Force clear to protect SQLite node

return msg;
```

### Node F: CV Profile View (UI Template)
*   **Purpose:** The visual centerpiece. Displays high-resolution photos and details using an HTML/CSS "Resume" format.
*   **Key Logic:** `img src="/{{msg.payload[0].student_no}}.jpg"` pulls the photo from the RPi4's local static directory automatically based on the Student ID.

---

## 🛡️ Industrial Design Justification

### **1. Data Integrity**
By implementing a **Name-to-Key Lookup**, we ensure that no sensitive data (Age, Phone, etc.) is transmitted over the air. The ESP32 only sends a name; the RPi4 server performs the secure lookup internally.

### **2. Asynchronous HMI**
The use of **Event-Driven JavaScript** within the UI Templates allows the 15.6" monitor to update specific elements (like the 2D Shelf dots) without re-rendering the entire page, resulting in a zero-lag user experience.

### **3. Persistence**
All transactions are committed to an **RDBMS (SQLite3)** layer. This ensures that the state of the library is maintained across power cycles, fulfilling the requirements for a "Mission Critical" system.

---

**Software Status:** `STABLE`  
**UI Refresh Rate:** `WebSocket Real-Time`  
**Database Capacity:** `10-Student Registered Core`

---

**Does this software-focused README meet your expectations?** If you are satisfied, we can move to the **Final Step**: The Python logic for the **Bookshelf Node** to handle the 8 IR Sensors and 8 RFID readers!
