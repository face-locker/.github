# 🔐 Face Locker - Smart Locker System with AI FaceID and Mobile App

> An intelligent centralized locker management system integrating AI-powered facial recognition and IoT mobile application for seamless, keyless storage solutions.

## 📋 Project Overview

**Face Locker** is a smart automated locker system designed using a centralized Kiosk model. It utilizes a single processor and camera to manage multiple locker compartments, optimizing costs and enhancing practicality for deployment in supermarkets, shopping centers, and similar venues.

## ✨ Key Features

- **Smart Storage**: Automatically finds and assigns available locker compartments - no physical keys required
- **Multi-Authentication Methods**:
  - 🤖 **AI FaceID**: Facial recognition for locker access
  - 📱 **IoT Mobile App**: QR code scanning via app for locker selection and remote unlocking
- **Real-time Monitoring**: Door sensors track open/close status with LED indicators showing occupancy
- **Admin Dashboard**: Usage statistics, compartment status tracking, and hardware fault/tampering alerts

## 🏗️ System Architecture

The system operates on a centralized control model (Kiosk):

- **Master Unit (Raspberry Pi 4)**: Acts as the "brain" - processes video stream from webcam for facial recognition (AI FaceID), runs web server for mobile app communication via Internet, and manages locker allocation logic
- **Slave Unit (ARM ESP32)**: Acts as the "executor" - receives commands from Raspberry Pi via UART protocol to control relays (unlock) and reads limit switch sensors (door monitoring), then sends feedback to Pi

## 🔄 Business Process Flow

### Scenario 1: Check-in (Deposit Items)

1. User presses "Deposit" button on touchscreen or app
2. System activates camera and performs AI FaceID to capture facial feature vector
3. Backend (on Pi) queries database to find an available locker compartment
4. If found, Pi sends UART packet (including locker ID + unlock command) to ESP32 kit
5. ESP32 receives command → Activates corresponding relay → Solenoid lock opens
6. User places items and closes door. Limit switch detects door closure → ESP32 sends signal to Pi → Pi updates locker status to "In Use" in database

### Scenario 2: Check-out (Retrieve Items)

1. User selects "Retrieve" and authenticates via FaceID (or QR scan on app)
2. System matches data. If matching with depositor's information:
3. Pi sends unlock command for that specific compartment to ESP32
4. Door opens, user retrieves items and closes door
5. System updates locker status back to "Available" and archives/clears the transaction session

## 🛠️ Technical Components

### Hardware & Circuits

- Locker frame assembly with 12V solenoid locks and limit switches
- Relay module circuit design for lock activation
- Power supply circuit (12V for locks, 5V/3.3V for microcontrollers)
- UART communication wiring between Raspberry Pi and ESP32

### Embedded Systems & Firmware

- ESP32 GPIO configuration for relay control and sensor reading
- UART protocol implementation (Frame transmission with Header, Command, Data, Checksum)
- Hardware error handling (e.g., lock jamming)

### AI & Backend

- Face data collection and model training/fine-tuning for Raspberry Pi deployment
- Inference time optimization on embedded devices
- Database management (locker list, open/close history)
- Smart locker allocation algorithm
- Mobile app API development

### Mobile Application

- Dynamic QR code generation for locker access
- Real-time locker status visualization
- Usage statistics and hardware fault alerts

## 👥 Team Members

| Name                 | Class   | Section |
| -------------------- | ------- | ------- |
| Dương Văn Chí Bảo    | 23T_DT4 | 23Nh15A |
| Võ Hoàng Thái Bảo    | 23T_DT4 | 23Nh15A |
| Huỳnh Ngọc Tiến Nhật | 23T_DT4 | 23Nh15A |
| Mai Thanh Tân        | 23T_DT4 | 23Nh15A |

## 📚 References

1. A. Bahga and V. Madisetti, _Internet of Things: A Hands-on Approach_. Vpt, 2014.
2. OASIS Standard, "MQTT Version 5.0," OASIS, Mar. 2019.
3. Raspberry Pi Foundation, "Raspberry Pi 4 Model B Datasheet," 2019.
4. Espressif Systems, "ESP32 Series Datasheet," ver. 3.7, 2023.

---

  <div align="center">

**PBL5 Project - Group 6**

_Da Nang University of Science and Technology_

  </div>
