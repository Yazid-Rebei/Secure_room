# Secure_room

A Room That Enforces Its Own Privacy Policy — In Hardware

ESP32 Distributed RF Transparency System

📌 Overview

ConsentMesh is a distributed embedded system that detects active recording-capable devices inside a room using passive RF monitoring.

The system:

Passively listens to WiFi probe requests (802.11)

Scans Bluetooth Low Energy advertisements

Infers behavioral device states

Aggregates results across multiple ESP32 nodes

Displays real-time device activity on a public screen

No signals are blocked.
No personal data is stored.
The room simply makes active recording visible.

🎯 Objective

Create a passive transparency layer for confidential environments:

Detect devices entering a room

Infer screen activity

Detect high transmission bursts

Flag possible recording behavior

Display device states publicly in real time

🏗 System Architecture
1️⃣ Distributed Nodes (5x ESP32)

Each node:

Runs WiFi in promiscuous mode

Captures probe requests and data frames

Scans BLE advertisements

Performs local behavioral inference

Sends sightings to mesh network

2️⃣ Mesh Layer (ESP-NOW)

Nodes exchange device sightings

Multi-node confirmation reduces false positives

Simple consensus mechanism determines device state

3️⃣ Public Dashboard

Displays:

Number of devices detected

Device behavioral state

Screen activity indicators

Transmission bursts

Possible recording flags

No MAC addresses displayed.
Only anonymized device states.

🔍 Behavioral States

Each detected device transitions between:

PRESENT

SCREEN_ACTIVE

TRANSMITTING

POSSIBLE_RECORDING

State transitions are based on:

Probe request frequency

RSSI stability

Data burst rate

BLE advertisement interval

Multi-node confirmation

🧠 Technical Stack
Hardware

5x ESP32 DevKit V1

5x OLED SSD1306

1 Public Display (Laptop / HDMI screen)

USB power banks

Software

C / C++ (Arduino framework)

ESP32 WiFi promiscuous mode

BLE scanning

ESP-NOW mesh protocol

Lightweight state machine

Serial-to-dashboard communication

📊 Demo Scenario

Room empty → Dashboard shows 0 devices

Phone enters pocket → Device appears (PRESENT)

Phone unlocked → State updates (SCREEN_ACTIVE)

Recorder app opened → RF burst detected (POSSIBLE_RECORDING)

Phone locked → Flag clears

Room empty → List clears

⚖ Ethical Design Principles

No signal jamming

No device blocking

No packet injection

No personal identification

No persistent storage

Pure transparency

This is counter-surveillance, not surveillance.

🛠 Project Structure
ConsentMesh/
│
├── firmware/
│   ├── node/
│   │   ├── capture.cpp
│   │   ├── ble_scan.cpp
│   │   ├── state_machine.cpp
│   │   ├── mesh_comm.cpp
│   │   └── main.ino
│   │
│   └── dashboard/
│       ├── parser.py
│       └── display_ui.py
│
├── documentation/
│   ├── cahier_des_charges.pdf
│   ├── architecture.png
│   └── demo_script.md
│
└── README.md
🚀 What You Learn

802.11 frame structure

BLE advertisement parsing

ESP32 promiscuous mode

Behavioral inference modeling

Distributed consensus logic

Privacy-by-design system architecture
