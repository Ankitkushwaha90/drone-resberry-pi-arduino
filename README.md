import React from 'react';

# 🚁 Wi-Fi Drone with Arduino + Raspberry Pi

A complete project setup for a real-time HD video streaming drone with full flight control using Arduino + Raspberry Pi. Add GPS, IMU, and telemetry with modular flexibility.

---

## ✅ What This Build Does

- 🧠 **Arduino**: Flight control using IMU and ESC PWM
- 🎥 **Raspberry Pi**: Handles video streaming + joystick/WebSocket control
- 💻 **Laptop**: Connects over Wi-Fi, receives stream, sends commands

---

## 📦 Components Required

### 🎯 Flight Control Side (Arduino-based)

| Component | Purpose |
|----------|---------|
| Arduino Mega / Nano | Handles IMU + PWM motor output |
| MPU6050 | IMU sensor (gyro + accel) |
| GPS Module (optional) | NEO-6M or BN-880 |
| 4x ESC (or 4-in-1) | BLHeli_32, controlled via PWM |
| Brushless Motors | 2207 2450KV (for 5” props) |
| 5" Frame + Props | Carbon fiber recommended |
| Battery | 4S 1500–2200mAh LiPo |
| UBEC (5V Regulator) | Powers Arduino + Pi safely |

---

### 🎥 Video + Wi-Fi Side (Raspberry Pi)

| Component | Purpose |
|----------|---------|
| Raspberry Pi 4 (2GB or 4GB) | Camera stream + Wi-Fi comm |
| Pi Camera V2 / USB Webcam | 720p–1080p video |
| Dual-band USB Wi-Fi Adapter | For long-range directional antenna |
| Battery Power (5V 3A) | Use step-down from LiPo |

---

## 🔗 Wiring Diagram Overview

```plaintext
[ Raspberry Pi ]
  └── USB Wi-Fi + Camera
  └── UART TX/RX to Arduino

[ Arduino Mega ]
  └── MPU6050 (I2C)
  └── ESCs via PWM (D3, D5, D6, D9)
  └── GPS (optional via Serial)
```

### 🖥️ Raspberry Pi Software Setup
- 1. Install MJPEG-Streamer for Live Camera
```bash
sudo apt update && sudo apt install cmake libjpeg-dev
git clone https://github.com/jacksonliam/mjpg-streamer.git
cd mjpg-streamer/mjpg-streamer-experimental
make
./mjpg_streamer -i "./input_uvc.so -y -n -f 20 -r 640x480" -o "./output_http.so -w ./www"
```
View video in browser at: http://<drone-ip>:8080

- 2. Setup WebSocket Server (Python)
```python
# ws_to_serial.py
import asyncio, websockets, serial

arduino = serial.Serial("/dev/ttyS0", 57600)

async def handler(websocket):
    async for message in websocket:
        print("Received:", message)
        arduino.write((message + "\n").encode())

start_server = websockets.serve(handler, "0.0.0.0", 8765)
asyncio.get_event_loop().run_until_complete(start_server)
asyncio.get_event_loop().run_forever()
```
- Run it with:

```bash
pip3 install pyserial websockets
python3 ws_to_serial.py
```
### 🧠 Arduino Code (Flight Controller)
```cpp
#include <Wire.h>
#include <MPU6050.h>
MPU6050 mpu;

const int motors[] = {3, 5, 6, 9};

void setup() {
  Serial.begin(57600);
  Wire.begin();
  mpu.initialize();
  for (int i = 0; i < 4; i++) pinMode(motors[i], OUTPUT);
}

void loop() {
  if (Serial.available()) {
    String input = Serial.readStringUntil('\n');  // "CTRL:0.2,0.1,0.6"
    if (input.startsWith("CTRL:")) {
      float pitch, roll, throttle;
      sscanf(input.c_str(), "CTRL:%f,%f,%f", &pitch, &roll, &throttle);
      int base = map(throttle * 100, 0, 100, 0, 255);

      // Basic throttle mix
      analogWrite(motors[0], base + roll * 50);   // M1
      analogWrite(motors[1], base - roll * 50);   // M2
      analogWrite(motors[2], base + pitch * 50);  // M3
      analogWrite(motors[3], base - pitch * 50);  // M4
    }
  }
}
```
### 🎮 Web Joystick Controller (HTML/JS)
```html
<script>
const ws = new WebSocket("ws://<drone-ip>:8765");
setInterval(() => {
  const gp = navigator.getGamepads()[0];
  if (gp) {
    let pitch = (-gp.axes[1]).toFixed(2);
    let roll = gp.axes[0].toFixed(2);
    let throttle = gp.buttons[7].value.toFixed(2);
    ws.send(`CTRL:${pitch},${roll},${throttle}`);
  }
}, 50);
</script>
```
### ✅ Optional: Add Dashboard
Show live MJPEG feed

Show GPS + battery (send JSON from Arduino)

Add map (e.g., Leaflet.js) for GPS location

### 📦 Project Summary

## Part	Use
Arduino Mega	Motor control, IMU, PWM
Raspberry Pi 4	Video + control relay
Wi-Fi + Antenna	Long-range link
Joystick / Browser UI	Pilot controls
Camera	Live stream (via MJPEG or RTSP)
## 🛠️ Would You Like:
✅ As a downloadable package?

✅ With a full HTML dashboard + telemetry page?

✅ With real-time motor feedback via Web UI?

Let me know, and I’ll package it up for you! 🚀
