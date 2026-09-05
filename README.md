---
for code visit the index.html.html file it will also give you other information
---
# 🌌 Stardance Hack Club: ESP32-CAM 3D Printer Live Stream

A low-cost, open-source solution to stream live video from your 3D printer using an ESP32-CAM board. Built using the **Arduino IDE**, this project lets you monitor prints remotely via a web browser or integrate the stream directly into **OctoPrint** and **Klipper (Mainsail/Fluidd)**.

---

## 🛠️ Hardware Requirements

* **ESP32-CAM Module** (AI-Thinker model recommended)
* **FTDI USB-to-TTL Serial Adapter** (for flashing the code)
* **Mini USB / USB-C Cable** (compatible with your FTDI adapter)
* **Jumper Wires** (Female-to-Female)
* **5V Power Source** (via the printer's mainboard or an external buck converter)

---

## 🔌 Pin Connections

### 1. Flashing Connections (To Upload Code)
To program the ESP32-CAM, connect the FTDI adapter pins to the board pins as follows. **Crucial:** You must tie GPIO 0 to GND to put the board into flashing mode.

* **FTDI VCC** ➡️ **ESP32-CAM 5V** (or 3.3V)
* **FTDI GND** ➡️ **ESP32-CAM GND**
* **FTDI RX** ➡️ **ESP32-CAM U0T**
* **FTDI TX** ➡️ **ESP32-CAM U0R**
* **ESP32-CAM GPIO 0** ➡️ **ESP32-CAM GND** (Bridge these two pins with a jumper wire)

### 2. Normal Operation Connections (To Stream on Printer)
Disconnect the FTDI adapter entirely. Power the board using your printer's power supply or a dedicated 5V source, and **remove the wire on GPIO 0**.

* **Power Supply 5V+** ➡️ **ESP32-CAM 5V**
* **Power Supply GND** ➡️ **ESP32-CAM GND**
* **ESP32-CAM GPIO 0** ➡️ *Leave disconnected*

---

## 💻 Arduino IDE Setup Steps

Follow these steps to prepare your environment and flash the board:

### Step 1: Install ESP32 Board Definitions
1. Open **Arduino IDE**.
2. Navigate to **File > Preferences**.
3. Locate the **Additional Boards Manager URLs** field and paste the following URL:
   ```text
   https://githubusercontent.com
   ```
4. Click **OK**.
5. Go to **Tools > Board > Boards Manager...**
6. Search for `esp32` and click **Install** on the package by **Espressif Systems**.

### Step 2: Configure the Project Code
1.open the code in index.html.html and paste it in the arduino ide.
3. Scroll down slightly to find the Wi-Fi credentials variables and fill in your network information:
   ```cpp
   const char* ssid = "YOUR_WIFI_NAME";
   const char* password = "YOUR_WIFI_PASSWORD";
   ```

### Step 3: Flash the Board
1. Wire the ESP32-CAM to your FTDI adapter according to the **Flashing Connections** listed above (ensure GPIO 0 is tied to GND).
2. Plug the FTDI adapter into your computer's USB port.
3. In Arduino IDE, go to **Tools > Board > ESP32 Arduino** and select **AI Thinker ESP32-CAM**.
4. Go to **Tools > Port** and select the active COM port associated with your FTDI adapter.
5. Click the **Upload** button. Wait for the terminal to display `100%` or `Hash of data verified...`.

---

## 🚀 Usage & Testing

1. **Unplug the jumper wire** connecting GPIO 0 to GND.
2. Open the **Serial Monitor** in Arduino IDE (**Tools > Serial Monitor**).
3. Set the baud rate in the bottom right corner of the Serial Monitor to **115200**.
4. Press the physical **RST (Reset)** button on the back of the ESP32-CAM board.
5. The Serial Monitor will print a local network URL once connected, for example:
   ```text
   Camera Ready! Use 'http://192.168.1.150' to connect
   ```
6. Open your web browser and navigate to that IP address to view the streaming dashboard.

---
for code visit the index.html.html file it will also give you other information
---

✨ *Built for the Stardance Hack Club community.*
