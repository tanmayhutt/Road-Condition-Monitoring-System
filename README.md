# Road Surface Monitoring Device

## Overview

Road Surface Monitoring Device is a vehicle-mounted embedded sensing system designed to detect road anomalies and transmit their locations to a backend server.

The system combines:

* STM32F446RE for sensor acquisition and processing
* LSM6DS3 IMU for shock detection
* HC-SR04 ultrasonic sensor for surface height measurement
* NEO-6M GPS module for location tracking
* ESP32-S3 for wireless telemetry transmission

Road events are detected locally on the STM32 and transmitted to an ESP32 gateway, which forwards the data to a backend server over Wi-Fi.

---

# Team

* **Sumed Sreedhar** — System Architecture, Firmware Integration, Sensor Processing
* **Adarsh Audupa** — Embedded Driver Development, STM32 Peripheral Support
* **Tanmay Hutt** — Backend Infrastructure, Telemetry Pipeline, IoT Integration
* **Pratham** — Data Logging, Dataset Support, GPS Data Validation

---

# System Architecture

```text
LSM6DS3 IMU
      │
      ▼
Vertical Shock Detection
      │
      ▼
Road Event Classification
      │
      ├────► Rough Road
      │
      └────► Pothole
                    │
                    ▼
             GPS Location
                    │
                    ▼
            STM32 Telemetry
                    │
                    ▼
                 ESP32
                    │
                    ▼
              HTTP Upload
                    │
                    ▼
             Backend Server
```

---

# Hardware Platform

## Processing Nodes

* STM32F446RE
* ESP32-S3

## Sensors

* LSM6DS3 6-Axis IMU
* HC-SR04 Ultrasonic Sensor
* NEO-6M GPS Receiver

## Communication Interfaces

* UART (STM32 ↔ ESP32)
* UART (STM32 ↔ NEO-6M)
* I2C (STM32 ↔ LSM6DS3)
* Wi-Fi (ESP32 ↔ Backend)

---

# Sensor Subsystems

## LSM6DS3 Shock Detection

The LSM6DS3 accelerometer continuously measures vehicle motion.

A low-pass gravity estimation algorithm separates:

```text
Measured Acceleration
        │
        ▼
Gravity Estimate
        │
        ▼
Dynamic Acceleration
        │
        ▼
Vertical Shock Value
```

The firmware projects dynamic acceleration onto the gravity vector to calculate vertical shock magnitude.

### Implemented Features

* I2C communication
* Accelerometer configuration
* WHO_AM_I validation
* Gravity estimation
* Low-pass filtering
* Vertical shock extraction
* Real-time processing

---

## HC-SR04 Surface Measurement

The HC-SR04 is used to measure distance between the sensor and the road surface.

The driver uses:

* Timer input capture
* Rising edge detection
* Falling edge detection
* Pulse width calculation

Distance is calculated directly from echo pulse duration.

### Implemented Features

* TIM5 input capture
* Hardware timing measurement
* Non-blocking acquisition
* Interrupt-driven distance calculation

---

## NEO-6M GPS Telemetry

The NEO-6M continuously streams NMEA messages.

The firmware implements:

* UART interrupt reception
* Ring buffer architecture
* NMEA sentence reconstruction
* GPGGA parsing

Extracted data includes:

* UTC Time
* Latitude
* Longitude
* Altitude
* Satellite Count
* GPS Fix Quality

### Implemented Features

* Interrupt-driven UART
* Circular buffering
* Producer-consumer processing
* GPS data extraction

---

# Road Event Classification

The STM32 classifies road events using sensor measurements.

## Rough Road Event

Generated when:

* Significant vertical shock is detected

Packet format:

```text
R,shock,latitude,longitude,timestamp
```

---

## Pothole Event

Generated when:

* Significant vertical shock is detected
* Ultrasonic measurement indicates a surface deviation

Packet format:

```text
P,shock,distance,latitude,longitude,timestamp
```

---

# Telemetry Pipeline

The STM32 transmits structured packets over UART to the ESP32.

```text
STM32
   │
   ▼
UART Packet
   │
   ▼
ESP32
   │
   ▼
HTTP POST
   │
   ▼
Backend API
```

The ESP32 parses incoming packets and uploads event data to the backend server.

---

# Firmware Architecture

## STM32 Firmware

```text
main.c
│
├── LSM6DS3.c
├── HC_SR04.c
├── NEO_6M.c
└── ESP32.c
```

### Responsibilities

* Sensor acquisition
* Shock detection
* GPS processing
* Distance measurement
* Event classification
* UART telemetry generation

---

## ESP32 Firmware

```text
road_surface_monitor.ino
```

### Responsibilities

* UART packet reception
* Wi-Fi connectivity
* HTTP communication
* Backend data upload
* Vehicle control support

---

# Concepts Demonstrated

* Embedded systems design
* Multi-sensor integration
* Sensor fusion fundamentals
* UART communication
* Interrupt-driven firmware
* Ring buffer architectures
* GPS telemetry systems
* I2C peripheral integration
* Timer input capture
* Real-time event detection
* Embedded-to-cloud telemetry
* Distributed processing architecture

---

# Repository Structure

```text
Road_Surface_Monitoring_Device/
│
├── Core/
│   ├── Src/
│   │   ├── main.c
│   │   ├── LSM6DS3.c
│   │   ├── HC_SR04.c
│   │   ├── NEO_6M.c
│   │   └── ESP32.c
│   │
│   └── Inc/
│
├── ESP32_Firmware/
│   └── road_surface_monitor.ino
│
├── Drivers/
│
└── README.md
```

---

# Future Improvements

* Decimal degree GPS conversion
* SD card logging
* Adaptive classification thresholds
* Kalman filtering
* Mobile dashboard
* Cloud analytics
* Machine-learning-based classification
* V2X integration

---

# Current Status

Functional prototype demonstrating:

* IMU-based shock detection
* Ultrasonic distance measurement
* GPS location acquisition
* UART telemetry transfer
* Wi-Fi cloud communication
* Backend event ingestion

The system is capable of detecting road anomalies, associating them with GPS coordinates, and transmitting event data to a backend server for further analysis.
