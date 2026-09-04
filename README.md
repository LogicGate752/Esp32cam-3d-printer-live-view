# 🚀 My 3D Printer Camera Monitor Node 🚀

Welcome to my project repo! I built this cool local hardware camera server because I wanted to check on my 3D printer directly from my phone or bed. This prevents long prints from failing into plastic "spaghetti code" or trapping heat in the printer enclosure!

This project was built for **Hack Club Stardance**. It runs an embedded C++ server directly on the micro silicon chip without relying on any external cloud platforms or subscription fees.

---

## 🛠️ The Hardware System Map
*   **The Brain:** ESP32-S Dev Module (with 4MB of dual-channel external PSRAM to process video frames).
*   **The Lens:** OV2640 2-Megapixel camera module attached via a golden flat ribbon connector.
*   **The Base Shield:** ESP32-CAM-MB USB download daughterboard (handles auto-flashing and feeds stable 5V current so the chip doesn't brown out).
*   **The Shell:** A custom physical protective case enclosure that I custom-assembled today to protect the components from short-circuiting on the printer frame!

---

## 💻 Software & Logic Design (Arduino IDE)
I wrote the firmware in embedded C++ inside the Arduino IDE. The code manages dual memory buffers to execute multiple actions over my local Wi-Fi router simultaneously:
1.  **Live Optical Downlink (`/stream`):** Pushes continuous VGA video frames across the local network using a raw multipart boundary pipeline so it renders inside a standard web layout.
2.  **Flash Toggle Utility (`/toggle-light`):** Commands physical GPIO pin 4 to trigger the bright white onboard flash LED to illuminate dark printing beds at night.
3.  **Local Snapshot Matrix (`/capture` & `/get-latest`):** Freeze-frames images directly onto the internal `SPIFFS` flash file partition for instant gallery viewing.

---

## 🎨 The Web Control Panel Layout
The frontend interface is baked directly inside the C++ string memory. I designed it to be bright, retro, and easy to use. It features:
*   A classic, bright yellow high-visibility layout using clean comic fonts.
*   Chunky action buttons that change styles when clicked.
*   An inline asynchronous JavaScript fetch script that pauses data pipelines to trigger snapshot memory updates without breaking network connectivity.

---

## 🏆 Learning Milestones I Mastered
*   **Hardware Assembly:** Learning how to align and reverse thin golden ribbon connector contact rows so pins touch the board's metal teeth properly.
*   **Memory Management:** Juggling high-speed video processing arrays inside small 4MB storage bounds without overloading the microchip core.
*   **Full-Stack Programming:** Bridging low-level hardware registers (C++) with standard front-end design protocols (HTML/CSS/JS).

***
*Built for Hack Club Stardance
