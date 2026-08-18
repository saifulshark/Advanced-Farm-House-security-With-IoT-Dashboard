<!-- # 🏡 Advanced Farm House Security with IoT Dashboard

An IoT-based smart farm house security system that combines **Face Recognition, IoT Sensors, two ESP32 boards, an Arduino Uno, a Servo Motor, and a Live Web Dashboard**.

The system recognizes authorized faces using a laptop camera, controls the gate automatically, monitors temperature, humidity, and gas levels, and sends sensor data to a web dashboard.

## Overview

The system uses a laptop webcam and a Python face-recognition application to identify authorized people.

When a recognized person is detected, the laptop sends an access command to **ESP32 #2 (Bridge)**. ESP32 #2 forwards the command over Wi-Fi to **ESP32 #1 (Main)**, which controls the gate Servo Motor.

At the same time, an **Arduino Uno** reads data from the DHT sensor and gas sensor and sends the readings to ESP32 #1. The Main ESP32 then sends the sensor data to the dashboard server through HTTP.

The overall system consists of:

- Laptop / PC with USB Webcam
- Python Face Recognition application
- ESP32 #1 — Main Controller
- ESP32 #2 — Communication Bridge
- Arduino Uno — Sensor Controller
- DHT11 / DHT22 — Temperature and Humidity Sensor
- MQ-2 / MQ-5 — Gas Sensor
- SG90 Servo Motor — Gate Controller
- Flask Dashboard Server
- Wi-Fi Router

## System Components

### Hardware

| Component | Quantity | Purpose |
|---|---:|---|
| Laptop / PC | 1 | Runs the Python face-recognition application |
| USB Webcam | 1 | Captures live video |
| ESP32 (Main) | 1 | Controls the gate, receives sensor data, and communicates with the dashboard |
| ESP32 (Bridge) | 1 | Transfers commands from the laptop to the Main ESP32 |
| Arduino Uno | 1 | Reads environmental sensors |
| DHT11 / DHT22 | 1 | Measures temperature and humidity |
| MQ-2 / MQ-5 Gas Sensor | 1 | Detects gas/smoke level |
| SG90 Servo Motor | 1 | Opens and closes the gate |
| Wi-Fi Router | 1 | Provides the main local network |
| Dashboard Server | 1 | Receives and displays sensor data |

### Software

| File | Language / Type | Purpose |
|---|---|---|
| `main.py` | Python | Main face-recognition application and command sender |
| `EncodeGenerator.py` | Python | Generates the face-encoding database |
| `EncodeFile.p` | Binary | Stores encoded faces |
| `Esp32WithArduinoAndDashboard.ino` | C++ | Main ESP32 firmware |
| `Esp32withLaptop.ino` | C++ | Bridge ESP32 firmware |

## How the System Works

### 1. Face Recognition

The laptop webcam continuously captures video.

The Python application:

1. Captures an image from the webcam.
2. Detects faces.
3. Compares detected faces with the stored face encodings.
4. Generates an access command.

Example commands:

```text
87,1
```

indicates an authorized face.

```text
87,0
```

indicates an unauthorized face.

### 2. ESP32 #2 — Bridge

ESP32 #2 receives the command from the laptop through USB Serial.

The Bridge ESP32 then forwards the command to ESP32 #1 through its own Wi-Fi Access Point.

```text
Laptop
   │
   │ USB Serial
   ▼
ESP32 #2
   │
   │ Wi-Fi
   ▼
ESP32 #1
```

The communication uses:

```text
SSID: No_Network
IP: 192.168.4.1
Port: 80
```

### 3. ESP32 #1 — Gate Control

ESP32 #1 receives and parses the command.

For:

```text
87,1
```

the Main ESP32 sets the Servo Motor to:

```text
90°
```

The gate opens and remains open for approximately five seconds.

After five seconds:

```text
Servo → 0°
```

and the gate closes.

For:

```text
87,0
```

the gate remains closed.

### 4. Arduino Uno — Sensor Monitoring

The Arduino Uno continuously reads the connected sensors.

Example sensor output:

```text
Humidity: 65, Temperature: 28, Gas Sensor: 150
```

The Arduino sends this data to ESP32 #1 using serial communication.

```text
Arduino Uno
    │
    │ UART
    ▼
ESP32 #1
```

### 5. Dashboard Communication

ESP32 #1 sends sensor data to the Flask dashboard using an HTTP POST request.

Endpoint:

```text
POST /sensor-data
```

Example request:

```json
{
  "writeApiKey": "YOUR_API_KEY",
  "sensorName": "DHT",
  "value": 150
}
```

The dashboard server receives the data and updates the monitoring interface.

## System Architecture

```text
                    ┌─────────────────────┐
                    │      USB Webcam     │
                    └──────────┬──────────┘
                               │
                               │ USB
                               ▼
                    ┌─────────────────────┐
                    │       Laptop        │
                    │  Python Application  │
                    │   Face Recognition   │
                    └──────────┬──────────┘
                               │
                               │ USB Serial
                               ▼
                    ┌─────────────────────┐
                    │    ESP32 #2         │
                    │      Bridge         │
                    └──────────┬──────────┘
                               │
                               │ Wi-Fi
                               ▼
                    ┌─────────────────────┐
                    │    ESP32 #1         │
                    │       Main          │
                    └──────┬─────┬─────┬──┘
                           │     │     │
                     UART  │     │     │ Wi-Fi
                           │     │     │
                           ▼     ▼     ▼
                    ┌────────┐ ┌─────┐ ┌─────────────────┐
                    │Arduino │ │Servo│ │ Dashboard       │
                    │  Uno   │ │Motor│ │ Flask Server    │
                    └───┬────┘ └─────┘ └─────────────────┘
                        │
                  ┌─────┴─────┐
                  │            │
               DHT Sensor   Gas Sensor
```

## Communication Flow

```text
Laptop
  │
  │ USB Serial
  │ "87,1"
  ▼
ESP32 #2 — Bridge
  │
  │ Wi-Fi
  │ No_Network
  ▼
ESP32 #1 — Main
  │
  ├───────────────► Servo Motor
  │                  Gate Control
  │
  ├───────────────► Dashboard Server
  │                  HTTP POST
  │
  ▲
  │ UART
  │ Sensor Data
  │
Arduino Uno
  ▲
  │
DHT + Gas Sensors
```

The system uses two separate wireless connections:

- **ESP32 #2 → ESP32 #1:** ESP32 #1 provides the `No_Network` Access Point.
- **ESP32 #1 → Dashboard:** ESP32 #1 connects to the main Wi-Fi network, such as `Jalish`.

## Data Formats

### Laptop → ESP32 #2

```text
"value1,value2\n"
```

Example:

```text
87,1
```

### ESP32 #2 → ESP32 #1

```text
"value1,value2\n"
```

Example:

```text
87,1
```

### Arduino Uno → ESP32 #1

```text
"Humidity: X, Temperature: Y, Gas Sensor: Z\n"
```

Example:

```text
Humidity: 65, Temperature: 28, Gas Sensor: 150
```

### ESP32 #1 → Dashboard

Data is transmitted using HTTP POST with JSON.

```json
{
  "writeApiKey": "YOUR_API_KEY",
  "sensorName": "DHT",
  "value": 150
}
```

## Project Structure

```text
Advanced-Farm-House-security-With-IoT-Dashboard/
│
├── Esp32WithDashboardAndArduino/
│   └── Esp32WithArduinoAndDashboard.ino
│
├── Esp32withLaptop/
│   └── Esp32withLaptop.ino
│
├── SensorProject/
│   ├── main.py
│   ├── Duplicate.py
│   ├── EncodeGenerator.py
│   ├── video.py
│   ├── test.py
│   ├── EncodeFile.p
│   ├── Images/
│   ├── Files/
│   ├── Files.zip
│   └── Resources/
│       ├── background.png
│       └── Modes/
│
└── README.md
```

## Hardware Wiring

### Arduino Uno → ESP32 #1

```text
Arduino Uno          ESP32 #1
───────────          ────────
TX (Pin 1)   ──────► GPIO16 (RX2)
RX (Pin 0)   ◄────── GPIO17 (TX2)
GND          ──────► GND
```

### Servo Motor → ESP32 #1

```text
Servo Motor          ESP32 #1
───────────          ────────
Signal       ──────► GPIO5
Power        ──────► 5V / 3.3V
GND          ──────► GND
```

### ESP32 #2 → Laptop

```text
ESP32 #2 Bridge       Laptop
───────────────       ──────
USB Port       ─────► USB Port
```

### Main ESP32 Pin Map

| ESP32 Pin | Function | Connected To |
|---|---|---|
| GPIO16 | Serial2 RX | Arduino TX |
| GPIO17 | Serial2 TX | Arduino RX |
| GPIO5 | Servo PWM | SG90 Signal |
| GND | Ground | Arduino and Servo GND |
| 5V / 3.3V | Power | Servo Power |

## Installation and Setup

### 1. Install Python Dependencies

```bash
pip install opencv-python face_recognition numpy pyserial requests cmake dlib
```

### 2. Generate the Face Database

Place images of authorized people inside:

```text
SensorProject/Images/
```

Use the person's name as the image filename.

Example:

```text
rifat.jpg
```

Then run:

```bash
cd SensorProject
python EncodeGenerator.py
```

This generates:

```text
EncodeFile.p
```

which is used by `main.py`.

### 3. Configure Arduino Uno

Install the following libraries through the Arduino IDE:

- DHT sensor library by Adafruit
- Adafruit Unified Sensor

Upload the sensor sketch to the Arduino Uno.

The expected serial output is similar to:

```text
Humidity: 65, Temperature: 28, Gas Sensor: 150
```

### 4. Configure Main ESP32

Open:

```text
Esp32WithDashboardAndArduino/Esp32WithArduinoAndDashboard.ino
```

Update the Wi-Fi and dashboard configuration before uploading:

```cpp
const char* ssid        = "YOUR_WIFI_NAME";
const char* password    = "YOUR_WIFI_PASSWORD";
const char* ap_ssid     = "No_Network";
const char* ap_password = "YOUR_AP_PASSWORD";
const char* serverName  = "http://YOUR_DASHBOARD_IP:5000/sensor-data";
const char* writeApiKey = "YOUR_API_KEY";
```

Select:

```text
Board: ESP32 Dev Module
```

and upload the firmware.

### 5. Configure Bridge ESP32

Open:

```text
Esp32withLaptop/Esp32withLaptop.ino
```

Configure the connection to the Main ESP32:

```cpp
const char* ssid       = "No_Network";
const char* password   = "YOUR_AP_PASSWORD";
const char* serverIP   = "192.168.4.1";
const int serverPort   = 80;
```

Select:

```text
Board: ESP32 Dev Module
```

and upload the firmware.

### 6. Connect the Hardware

```text
Arduino TX        → ESP32 #1 GPIO16
Arduino RX        → ESP32 #1 GPIO17
Arduino GND       → ESP32 #1 GND
ESP32 #1 GPIO5    → Servo Signal
ESP32 #2 USB      → Laptop USB
```

### 7. Run the Python Application

```bash
cd SensorProject
python main.py
```

The system is now ready to operate.

## Configuration

### Python Serial Port

For Windows:

```python
ser = serial.Serial('COM3', 115200)
```

For Linux/macOS:

```python
ser = serial.Serial('/dev/ttyUSB0', 115200)
```

### Main ESP32

```cpp
// Main Wi-Fi
const char* ssid     = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";

// ESP32 #1 Access Point
const char* ap_ssid     = "No_Network";
const char* ap_password = "YOUR_AP_PASSWORD";

// Serial2
#define RX2 16
#define TX2 17

// Servo
myServo.attach(5);

// Dashboard
const char* serverName = "http://YOUR_DASHBOARD_IP:5000/sensor-data";
const char* writeApiKey = "YOUR_API_KEY";
```

### Bridge ESP32

```cpp
const char* ssid     = "No_Network";
const char* password = "YOUR_AP_PASSWORD";
const char* serverIP = "192.168.4.1";
const int serverPort = 80;
```

## Code Flow

### Main ESP32

The Main ESP32 performs four primary tasks:

1. Reads sensor data from the Arduino Uno.
2. Receives access commands from ESP32 #2.
3. Controls the Servo Motor based on the access command.
4. Sends sensor data to the dashboard using HTTP POST.

Conceptually:

```text
Arduino Sensor Data
        │
        ▼
   Parse Sensor Data
        │
        ▼
Update Sensor Values
        │
        ├──────────────► Dashboard
        │
        ▼
Receive Gate Command
        │
        ▼
   Parse Command
        │
        ▼
  Authorized?
    /       \
  Yes        No
   │          │
   ▼          ▼
Servo 90°   Gate Closed
   │
   ▼
Wait 5 seconds
   │
   ▼
Servo 0°
```

### Bridge ESP32

The Bridge ESP32 continuously checks for commands from the laptop.

```text
Laptop Serial Input
        │
        ▼
Read Command
        │
        ▼
Parse Command
        │
        ▼
Connect to Main ESP32
        │
        ▼
Forward Command
```

## API Reference

### `POST /sensor-data`

Receives sensor data from ESP32 #1.

#### Headers

```http
Content-Type: application/json
```

#### Request Body

```json
{
  "writeApiKey": "YOUR_API_KEY",
  "sensorName": "DHT",
  "value": 150
}
```

#### Success Response

```json
{
  "status": "success"
}
```

## Troubleshooting

### `EncodeFile.p not found`

The face database has not been generated.

Run:

```bash
python EncodeGenerator.py
```

### Serial Port Error

Check the correct COM port in Windows Device Manager and update the port in `main.py`.

### ESP32 #1 Cannot Connect to Wi-Fi

Check the configured Wi-Fi SSID and password.

### ESP32 #2 Cannot Connect

Make sure ESP32 #1 is powered on and its `No_Network` Access Point is running.

### Servo Does Not Move

Check that the received command is being parsed correctly and verify the Servo signal connection to GPIO5.

### Dashboard Does Not Receive Data

Check the dashboard server IP and make sure the endpoint is reachable:

```text
http://YOUR_DASHBOARD_IP:5000/sensor-data
```

### Arduino Data Parsing Fails

Verify that the Arduino sends data using the expected format:

```text
Humidity: 65, Temperature: 28, Gas Sensor: 150
```

### Face Recognition Fails

Update the images in:

```text
SensorProject/Images/
```

and regenerate:

```text
EncodeFile.p
```

## Dependencies

### Python

```bash
pip install opencv-python face_recognition numpy pyserial requests cmake dlib
```

Main libraries:

- `opencv-python` — webcam and image processing
- `face_recognition` — face detection and encoding
- `numpy` — numerical processing
- `pyserial` — serial communication
- `requests` — HTTP communication
- `dlib` — backend used by `face_recognition`
- `cmake` — required for building certain dependencies

### Arduino / ESP32

- `WiFi.h` — Wi-Fi Station and Access Point functionality
- `HTTPClient.h` — HTTP communication with the dashboard
- `ESP32Servo.h` — Servo Motor control
- `HardwareSerial` — UART communication with Arduino Uno
- `DHT.h` — DHT11/DHT22 sensor readings

## Quick Start

```bash
# 1. Generate the face database
cd SensorProject
python EncodeGenerator.py

# 2. Upload the sensor firmware to Arduino Uno

# 3. Configure and upload the Main ESP32 firmware
#    Esp32WithArduinoAndDashboard.ino

# 4. Configure and upload the Bridge ESP32 firmware
#    Esp32withLaptop.ino

# 5. Connect the hardware
#    Arduino TX     → ESP32 #1 GPIO16
#    Arduino RX     → ESP32 #1 GPIO17
#    Arduino GND    → ESP32 #1 GND
#    ESP32 #1 GPIO5 → Servo Signal
#    ESP32 #2 USB   → Laptop

# 6. Start the Python application
python main.py
```

## Future Enhancements

- [ ] Replace `delay(5000)` with a non-blocking `millis()`-based gate timer
- [ ] Send temperature and humidity as separate dashboard values
- [ ] Add SMS or email alerts for unauthorized access
- [ ] Integrate a cloud dashboard such as ThingSpeak, Firebase, or MQTT
- [ ] Add remote gate control through a mobile application
- [ ] Add an OLED display for local ESP32 status
- [ ] Add multi-camera support

## Author

**Rifat** — [@rifat87](https://github.com/rifat87)

## License

This project is open source and available under the MIT License.

## Network Requirements

The **Laptop, ESP32 #1, and Dashboard Server** should be connected to the same main Wi-Fi network.

ESP32 #2 does not connect directly to this main network. Instead, it connects to the dedicated Access Point created by ESP32 #1:

```text
Main Wi-Fi Network
├── Laptop
├── ESP32 #1
└── Dashboard Server

ESP32 #1 Access Point
└── ESP32 #2
```

This separation allows ESP32 #2 to receive commands from the laptop through USB Serial and forward them to ESP32 #1 over its dedicated wireless connection.

# Path for python  code:
![image](https://github.com/user-attachments/assets/da16279a-1c78-43c6-84a0-a94ffefbaa2d)
-->

# 🏡 Advanced Farm House Security with IoT Dashboard

An IoT-based farm house security system combining **Face Recognition, ESP32, Arduino Uno, environmental sensors, automatic gate control, and a web dashboard**.

The system recognizes authorized users through a laptop webcam, controls the gate using a Servo Motor, monitors temperature, humidity, and gas levels, and sends sensor data to a local Flask dashboard.

## Features

- Face recognition-based gate access
- Automatic gate control using Servo Motor
- Temperature and humidity monitoring
- Gas/smoke level monitoring
- Two-ESP32 communication architecture
- Arduino Uno-based sensor reading
- Local Flask web dashboard
- Optional ThingSpeak cloud monitoring

## System Components

### Hardware

- Laptop / PC with USB Webcam
- 2 × ESP32
  - ESP32 #1 — Main Controller
  - ESP32 #2 — Bridge
- Arduino Uno
- DHT11 / DHT22
- MQ-2 / MQ-5 Gas Sensor
- SG90 Servo Motor
- Wi-Fi Router
- PIR(Intrusion Detection)
- Laser(security laser break)

### Software

- Python
- OpenCV
- `face_recognition`
- Arduino IDE
- Flask Dashboard
- ESP32 firmware

## How It Works and System Diagram or Architecture view

<img width="1536" height="959" alt="farmhouse" src="https://github.com/user-attachments/assets/8ffb7ed9-ed62-41cd-a732-6bf41db2fcf2" />



```text
                     USB Webcam
                          │
                          ▼
                  ┌───────────────┐
                  │    Laptop     │
                  │ Face          │
                  │ Recognition   │
                  └───────┬───────┘
                          │ USB Serial
                          ▼
                  ┌───────────────┐
                  │   ESP32 #2    │
                  │    Bridge     │
                  └───────┬───────┘
                          │ Wi-Fi
                          ▼
                  ┌───────────────┐
                  │   ESP32 #1    │
                  │     Main      │
                  └───┬────┬───┬──┘
                      │    │   │
                    UART  │   │ Wi-Fi
                      │   │   │
                      ▼   ▼   ▼
                  Arduino Servo Flask
                    Uno    │   Dashboard
                     │     │
                  Sensors  Gate
```

### Access Control Flow

1. The webcam captures the user's face.
2. Python compares it with the stored face database.
3. The laptop sends an access command to ESP32 #2.
4. ESP32 #2 forwards the command to ESP32 #1.
5. ESP32 #1 controls the Servo Motor.
6. Authorized access opens the gate to `90°`.
7. After approximately 5 seconds, the Servo returns to `0°`.

### Sensor Data Flow

1. Arduino Uno reads the DHT and gas sensors.
2. Sensor data is sent to ESP32 #1 through UART.
3. ESP32 #1 parses the readings.
4. ESP32 #1 sends the data to the Flask dashboard using HTTP POST.
5. The dashboard displays the received sensor data.

## Communication

| Connection | Method |
|---|---|
| Webcam → Laptop | USB |
| Laptop → ESP32 #2 | USB Serial |
| ESP32 #2 → ESP32 #1 | Wi-Fi |
| Arduino → ESP32 #1 | UART |
| ESP32 #1 → Dashboard | HTTP POST |

Example access command:

```text
87,1
```

Example sensor data:

```text
Humidity: 65, Temperature: 28, Gas Sensor: 150
```

## Project Structure

```text
Advanced-Farm-House-security-With-IoT-Dashboard/
│
├── Esp32WithDashboardAndArduino/
│   └── Esp32WithArduinoAndDashboard.ino
│
├── Esp32withLaptop/
│   └── Esp32withLaptop.ino
│
├── SensorProject/
│   ├── main.py
│   ├── Duplicate.py
│   ├── EncodeGenerator.py
│   ├── video.py
│   ├── test.py
│   ├── EncodeFile.p
│   ├── Images/
│   ├── Files/
│   └── Resources/
│
└── README.md
```

## Hardware Connections

### Arduino Uno → ESP32 #1

```text
Arduino TX  → ESP32 GPIO16
Arduino RX  → ESP32 GPIO17
Arduino GND → ESP32 GND
```

### Servo → ESP32 #1

```text
Servo Signal → GPIO5
Servo Power  → 5V / suitable external supply
Servo GND    → GND
```

### ESP32 #2 → Laptop

```text
ESP32 #2 USB → Laptop USB
```

## Installation

### 1. Install Python Dependencies

```bash
pip install opencv-python face_recognition numpy pyserial requests cmake dlib
```

### 2. Generate Face Encodings

Place authorized-user images in:

```text
SensorProject/Images/
```

Then run:

```bash
cd SensorProject
python EncodeGenerator.py
```

This creates `EncodeFile.p`.

### 3. Configure Main ESP32

Update the Wi-Fi and dashboard settings:

```cpp
const char* ssid        = "YOUR_WIFI_NAME";
const char* password    = "YOUR_WIFI_PASSWORD";
const char* ap_ssid     = "No_Network";
const char* ap_password = "YOUR_AP_PASSWORD";
const char* serverName  = "http://YOUR_DASHBOARD_IP:5000/sensor-data";
const char* writeApiKey = "YOUR_API_KEY";
```

Upload:

```text
Esp32WithArduinoAndDashboard.ino
```

using **ESP32 Dev Module**.

### 4. Configure Bridge ESP32

```cpp
const char* ssid       = "No_Network";
const char* password   = "YOUR_AP_PASSWORD";
const char* serverIP   = "192.168.4.1";
const int serverPort   = 80;
```

Upload:

```text
Esp32withLaptop.ino
```

### 5. Start the Python Application

```bash
cd SensorProject
python main.py
```

## Dashboard API

ESP32 #1 sends sensor data to:

```text
POST /sensor-data
```

Example:

```json
{
  "writeApiKey": "YOUR_API_KEY",
  "sensorName": "DHT",
  "value": 150
}
```

## Dependencies

### Python

- `opencv-python`
- `face_recognition`
- `numpy`
- `pyserial`
- `requests`
- `dlib`
- `cmake`

### Arduino / ESP32

- `WiFi.h`
- `HTTPClient.h`
- `ESP32Servo.h`
- `HardwareSerial`
- `DHT.h`

## Troubleshooting

**Face database not found**

```bash
python EncodeGenerator.py
```

**Serial error:** Check the correct COM port in `main.py`.

**ESP32 Wi-Fi error:** Verify SSID and password.

**ESP32 #2 connection error:** Make sure ESP32 #1 is running its `No_Network` Access Point.

**Servo not moving:** Check GPIO5, power, ground, and the received command.

**Dashboard not receiving data:** Verify the dashboard IP, port `5000`, endpoint, and API key.

## Project View Images(shots)
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/9475bbdb-2d16-44e7-b3a0-922bc8fb768b" />
<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/0a3fa093-467b-4b83-bb1e-fe3c8a4479b9" />
<img width="417" height="408" alt="image" src="https://github.com/user-attachments/assets/bbe471d1-f7c1-48b4-a399-cf175abff1f8" />
<img width="719" height="435" alt="image" src="https://github.com/user-attachments/assets/1214be7f-480a-48fb-a1b1-f484dc013657" />


## Optional: ThingSpeak Integration

ThingSpeak can be used as an optional cloud dashboard for storing and graphing sensor data.

1. Create a ThingSpeak channel and add:
   - Field 1 — Temperature
   - Field 2 — Humidity
   - Field 3 — Gas Level
2. Copy the channel's **Write API Key**.
3. Add the following to the Main ESP32 code:

```cpp
const char* thingSpeakServer = "api.thingspeak.com";
const char* thingSpeakApiKey = "YOUR_WRITE_API_KEY";
```

Add a function to send the three sensor values:

```cpp
void sendToThingSpeak(float temp, float hum, int gas) {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;

    String url = "http://" + String(thingSpeakServer) +
                 "/update?api_key=" + String(thingSpeakApiKey) +
                 "&field1=" + String(temp) +
                 "&field2=" + String(hum) +
                 "&field3=" + String(gas);

    http.begin(url);
    http.GET();
    http.end();
  }
}
```

Call it every **15 seconds** from `loop()`:

```cpp
static unsigned long lastThingSpeakUpdate = 0;

if (millis() - lastThingSpeakUpdate >= 15000) {
  sendToThingSpeak(temperature, humidity, gasSensorValue);
  lastThingSpeakUpdate = millis();
}
```

The ThingSpeak channel can then be used to view the sensor graphs remotely. The local Flask dashboard can continue running simultaneously.

## Future Improvements

- Non-blocking gate control using `millis()`
- Separate temperature, humidity, and gas dashboard fields
- SMS/email alerts for unauthorized access
- Cloud-based monitoring
- Mobile application for remote gate control
- OLED status display
- Multi-camera support


## License

This project is open source and available under the MIT License.

> **Network Note:** The Laptop, ESP32 #1, and Dashboard Server should be on the same main Wi-Fi network. ESP32 #2 connects to the `No_Network` Access Point created by ESP32 #1.

## Video Clips!
[Video Tutorial](https://youtu.be/m4eX0TFn8zo)
[Video Tutorial](https://youtu.be/XwVTBG5RGu8)
