# 📷 Object Detection Using ESP32-CAM & CircuitDigest Cloud

[![Arduino](https://img.shields.io/badge/Arduino-IDE-00979D?style=for-the-badge&logo=arduino&logoColor=white)](https://www.arduino.cc/)
[![ESP32](https://img.shields.io/badge/ESP32-CAM-E7352C?style=for-the-badge&logo=espressif&logoColor=white)](https://www.espressif.com/)
[![Cloud API](https://img.shields.io/badge/CircuitDigest-Cloud%20API-6C63FF?style=for-the-badge)](https://circuitdigest.cloud/)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

> **Real-time object detection using a compact ESP32-CAM and the CircuitDigest Cloud API — no model training, no dataset creation, just plug and detect!**

---

## 📖 Table of Contents

- [About the Project](#-about-the-project)
- [How It Works](#-how-it-works)
- [Components Required](#-components-required)
- [Circuit Diagram](#-circuit-diagram)
- [Step-by-Step Setup](#-step-by-step-setup)
- [Code Overview](#-code-overview)
- [Output](#-output)
- [Troubleshooting](#-troubleshooting)
- [Advantages & Limitations](#-advantages--limitations)
- [FAQ](#-faq)
- [Relevant Links](#-relevant-links)

---

## 🧠 About the Project

Object detection is a computer vision technique that identifies real-world objects — such as cell phones, remotes, laptops, cars, cups, books, and more — by drawing **bounding boxes** around them along with a **confidence score** indicating detection accuracy.

This project implements a **low-cost, cloud-powered object detection system** using an ESP32-CAM module. Unlike traditional systems that require high-end hardware and complex ML model training, this project leverages the **CircuitDigest Cloud API** to handle all the heavy processing in the cloud.

### ✨ Key Highlights

- ✅ No dataset creation, labelling, or model training required
- ✅ Supports **75+ object classes** (people, cars, animals, electronics, etc.)
- ✅ Compact and portable — just an ESP32-CAM, a push button, and a laptop
- ✅ Results displayed on the **Serial Monitor** in real time
- ✅ Adjustable **confidence threshold** and **object class selection**

---

## ⚙️ How It Works

```
[ Push Button Pressed ]
        ↓
[ ESP32-CAM Captures Image ]
        ↓
[ Image Sent to CircuitDigest Cloud via HTTPS API ]
        ↓
[ Cloud Processes & Runs Object Detection ]
        ↓
[ JSON Response: Object Names + Count + Confidence ]
        ↓
[ Results Printed on Serial Monitor ]
```

1. The user presses the **push button** connected to GPIO13.
2. The **ESP32-CAM** captures a JPEG image.
3. The image is sent to the **CircuitDigest Cloud** via an HTTPS POST request.
4. The cloud processes the image and returns detected object names, counts, and confidence values.
5. Results are printed on the **Arduino Serial Monitor**.

> 💡 **Tip:** Ensure adequate lighting when capturing images — better lighting = higher detection accuracy.

---

## 🛒 Components Required

| S.No | Component | Purpose |
|------|-----------|---------|
| 1 | ESP32-CAM (AI Thinker) | Microcontroller + Camera Module |
| 2 | Push Button | Triggers image capture |
| 3 | Breadboard | Simplifies circuit connections |
| 4 | USB-to-Serial (FTDI) Adapter *(if no onboard USB)* | Programming the ESP32-CAM |
| 5 | USB Cable | Powers the system via laptop |

> **⚠️ Important:** If you are using the standard ESP32-CAM (without an onboard USB port), you will need a **USB-to-Serial (FTDI) adapter** for programming.  
> Connect: `FTDI TX → ESP32-CAM RX (U0R)`, `RX → TX (U0T)`, `GND → GND`  
> Hold **GPIO0 LOW** during upload to enter flash mode.

---

## 🔌 Circuit Diagram

The push button is connected to **GPIO13** of the ESP32-CAM to trigger photo capture.

```
ESP32-CAM
  GPIO13 ──── Push Button ──── GND
  5V/3.3V ─── VCC
  GND ──────── GND
```

> Refer to the circuit diagram image in the `/assets` folder for a visual reference.

---

## 🚀 Step-by-Step Setup

### Step 1 — Create a CircuitDigest Cloud Account
- Go to [CircuitDigest Cloud](https://circuitdigest.cloud/)
- Sign up or log in to your account
- Scroll down and click on the **Object Detection** feature

### Step 2 — Configure the API Settings
- In the **Try API** section, set the desired **confidence level** (minimum probability threshold)
- A higher confidence value = more accurate but may miss some objects
- A lower value = detects more objects but with less certainty

### Step 3 — Select Object Classes
- Choose from **75+ available classes** (e.g., person, car, dog, laptop)
- You can select **all classes** or only **specific ones**

### Step 4 — Test Without Hardware (Optional)
- Upload any test image containing objects from your selected classes
- Click **Run Test** to see detection results instantly

### Step 5 — Get the ESP32-CAM Code
- Scroll to the **microcontroller selection** section
- Select **ESP32-CAM**
- Copy the generated code

### Step 6 — Flash the Code
- Open **Arduino IDE**
- Paste the copied code
- Update the following values in the code:
  ```cpp
  const char* ssid     = "YourWiFiSSID";
  const char* password = "YourWiFiPassword";
  const char* apiKey   = "YourAPIKey";
  const char* classes  = "[]";        // Leave empty for all classes
  const char* confidence = "0.2";     // Adjust as needed
  ```
- Select **AI Thinker ESP32-CAM** as the board in Arduino IDE
- Upload the sketch
- Connect the push button and open **Serial Monitor** at `115200` baud

### Step 7 — Detect Objects!
- Point the camera at objects
- Press the **push button**
- View results in the Serial Monitor 🎉

---

## 💻 Code Overview

### 1. Library Imports & Configuration

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include "esp_camera.h"

const char* ssid       = "YourSSID";
const char* password   = "YourPassword";

const char* serverName = "www.circuitdigest.cloud";
const char* serverPath = "/api/v1/object-detection/detect";
const int   serverPort = 443;
const char* apiKey     = "YourAPIKey";
const char* classes    = "[]";   // Empty = detect all classes
const char* confidence = "0.2";

#define TRIGGER_BTN 13
```

### 2. Camera Initialization

```cpp
void initCamera() {
  camera_config_t config;
  config.pixel_format = PIXFORMAT_JPEG;
  config.frame_size   = FRAMESIZE_VGA;
  esp_camera_init(&config);
}
```
> Adjust `frame_size` and quality settings based on your memory and accuracy needs.

### 3. Send Image to API

```cpp
String sendImageToAPI(camera_fb_t* fb) {
  client.connect(serverName, serverPort);
  client.println("POST /api HTTP/1.1");
  client.write(fb->buf, fb->len);
}
```
> This function performs an HTTPS POST request, uploading the JPEG frame buffer to the CircuitDigest Cloud.

### 4. Setup & Loop

```cpp
void setup() {
  Serial.begin(115200);
  pinMode(TRIGGER_BTN, INPUT_PULLUP);
  initCamera();
  WiFi.begin(ssid, password);
}

void loop() {
  if (digitalRead(TRIGGER_BTN) == LOW) {
    camera_fb_t* fb = esp_camera_fb_get();
    String result = sendImageToAPI(fb);
    Serial.println(result);
  }
}
```

---

## 📊 Output

When the push button is pressed, the Serial Monitor displays results like:

```
Detected Objects: 3
1. laptop     - Confidence: 91%
2. cell phone - Confidence: 87%
3. mouse      - Confidence: 78%
```

You can also view **detection logs** and **API usage** (daily/monthly) directly on the CircuitDigest Cloud dashboard.

---

## 🛠️ Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| **Camera capture failed** | Insufficient PSRAM / memory | Reduce frame size, lower JPEG quality, enable PSRAM in board settings |
| **ESP32-CAM keeps restarting** | Insufficient USB power | Use an external 5V power supply; check connections |
| **Blurry or unclear images** | Focus or lighting issues | Manually adjust the lens; improve lighting; tune brightness/contrast |
| **Camera initialization failed** | Wrong pin config or board selection | Select **AI Thinker ESP32-CAM** in Arduino IDE; verify GPIO pin mapping |
| **No detection / wrong results** | Poor image quality or wrong confidence level | Improve lighting, adjust confidence threshold, reposition the camera |

---

## ⚖️ Advantages & Limitations

### ✅ Advantages

| # | Advantage |
|---|-----------|
| 1 | Real-time object detection within a few seconds |
| 2 | Low-cost system using ESP32-CAM with built-in camera and Wi-Fi |
| 3 | Supports detection of multiple object types (people, cars, animals, etc.) |
| 4 | Easy to change object classes and confidence settings |
| 5 | Small size and portable design |

### ⚠️ Limitations

| # | Limitation |
|---|------------|
| 1 | Cannot work without cloud API access |
| 2 | Requires an active internet connection |
| 3 | Blurry or low-quality images may produce incorrect results |
| 4 | Subject to daily/monthly API usage limits |
| 5 | Captures single images — not continuous live video detection |

---

## ❓ FAQ

**Q1. What happens if the API limit is exceeded?**  
The server returns a "limit exceeded" response, and no detections will occur until the limit resets or a new API key is used.

**Q2. Can we detect specific objects only?**  
Yes! Modify the `classes` parameter in the API request to target only selected object types.

**Q3. How can detection accuracy be improved?**  
- Ensure good lighting conditions  
- Adjust camera brightness and contrast settings  
- Use an appropriate confidence threshold  
- Position the camera properly and close to the subject  

**Q4. What is the role of the confidence parameter?**  
It sets the minimum probability threshold. Higher values = more precise but may miss objects. Lower values = detects more but with less certainty.

**Q5. Why use cloud processing instead of on-device?**  
The ESP32-CAM has limited processing power. Complex object detection algorithms require significant computation, which is efficiently handled by cloud APIs.

**Q6. Can the system work offline?**  
No. The system relies entirely on cloud-based processing. Internet connectivity is required.

---

## 🔗 Relevant Links

- 🔗 [License Plate Recognition Using ESP32-CAM](https://circuitdigest.com/projects/license-plate-recognition-using-esp32-cam)
- 🔗 [DIY Smart WiFi Video Doorbell Using ESP32](https://circuitdigest.com/microcontroller-projects/diy-smart-wifi-video-doorbell-using-esp32-and-camera)
- 🔗 [ESP32-CAM Face Mask Detection](https://circuitdigest.com/microcontroller-projects/esp32-cam-based-face-mask-detection)
- 🔗 [GitHub Repository](https://github.com/Circuit-Digest/Object-detection-Using-ESP32-Cam)
- 🔗 [CircuitDigest Cloud](https://circuitdigest.cloud/)

---

## 📜 License

This project is licensed under the **MIT License** — feel free to use, modify, and distribute with attribution.

---

<p align="center">
  Made with ❤️ using ESP32-CAM & CircuitDigest Cloud
</p>
