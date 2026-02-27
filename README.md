
# SecureRoom

### A Room That Enforces Its Own Privacy Policy — In Hardware

Distributed Passive RF Transparency System using ESP32

---

## 1. Project Overview

ConsentMesh is a distributed embedded system that passively monitors radio-frequency activity inside a room and makes the presence of recording-capable devices visible in real time.

The system does not block, jam, or interfere with any signals.
It only observes publicly emitted RF behavior and displays anonymized device states on a shared screen.

The objective is to create transparency in confidential environments such as:

* Boardrooms
* Therapy offices
* Legal consultation rooms
* Research labs
* Government spaces

The room becomes a neutral enforcer of its own privacy policy.

---

## 2. Core Principles

* Passive RF monitoring only
* No signal jamming
* No packet injection
* No personal identification
* No persistent personal data storage
* Behavioral detection instead of identity tracking

This is counter-surveillance through transparency.

---

## 3. System Architecture

ConsentMesh consists of three layers:

### 3.1 Detection Nodes (ESP32 Mesh)

Each node:

* Runs WiFi in promiscuous mode
* Captures 802.11 probe requests and data frames
* Scans BLE advertisements
* Extracts RSSI and frame metadata
* Performs local behavioral inference
* Shares sightings with other nodes

Recommended deployment: 4–5 nodes placed in room corners.

---

### 3.2 Mesh Aggregation Layer

Communication protocol: ESP-NOW

Responsibilities:

* Exchange device sightings
* Confirm presence using multi-node detection
* Apply consensus logic
* Reduce false positives
* Synchronize device states

---

### 3.3 Public Dashboard

Displays in real time:

* Number of detected devices
* Behavioral state per device
* Transmission intensity indicators
* Possible recording flags

No MAC addresses are shown.
Only anonymized device states are displayed.

---

## 4. Behavioral State Model

Each detected device transitions between states based on RF activity patterns.

### States

* PRESENT
* SCREEN_ACTIVE
* TRANSMITTING
* POSSIBLE_RECORDING

### Inference Based On

* Probe request frequency
* Data burst rate
* RSSI persistence
* BLE advertisement interval
* Multi-node confirmation

The system performs behavioral inference — not identification.

---

## 5. Hardware Requirements

| Component                              | Quantity |
| -------------------------------------- | -------- |
| ESP32 DevKit V1                        | 5        |
| OLED SSD1306 (0.96")                   | 5        |
| Public Display (Laptop / HDMI Monitor) | 1        |
| USB Power Banks                        | 5        |
| Micro USB cables                       | 5        |

Estimated total cost: ~100 euros

---

## 6. Software Stack

### Firmware (ESP32)

* Arduino Framework
* WiFi Promiscuous Mode API
* BLE scanning API
* ESP-NOW mesh communication
* Finite State Machine logic

### Dashboard

* Python (serial parser)
* Lightweight UI (Tkinter / Web UI optional)

---

## 7. Repository Structure

```
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
│       ├── display.py
│       └── requirements.txt
│
├── documentation/
│   ├── architecture_diagram.png
│   ├── state_machine.png
│   └── cahier_des_charges.pdf
│
└── README.md
```

---

## 8. Installation & Setup

### 8.1 Flashing ESP32 Nodes

1. Install Arduino IDE
2. Add ESP32 board package
3. Upload firmware from `firmware/node/`
4. Configure unique node IDs
5. Power using USB power banks

---

### 8.2 Running Dashboard

1. Connect main node to laptop via USB
2. Install Python dependencies
3. Run:

```
python display.py
```

The dashboard will display live device activity.

---

## 9. Development Phases

### Phase 1 – RF Capture Validation

* Enable promiscuous mode
* Count probe requests
* Display device presence locally

### Phase 2 – Behavioral Detection

* Measure probe frequency
* Detect screen-on patterns
* Define transmission burst thresholds

### Phase 3 – BLE Integration

* Scan BLE advertisements
* Merge WiFi and BLE sightings

### Phase 4 – Mesh Communication

* Implement ESP-NOW
* Share device sightings
* Add multi-node confirmation

### Phase 5 – Dashboard Integration

* Serialize device state
* Build live public UI

### Phase 6 – Calibration & Testing

* Phone in pocket
* Phone unlocked
* Voice recorder app
* Video recording app
* Multiple devices simultaneously

---

## 10. Technical Challenges

* MAC address randomization
* RF noise interference
* False positive reduction
* RSSI fluctuation filtering
* Behavioral threshold tuning

Mitigation strategy:
Use state persistence and multi-node consensus instead of single-event detection.

---

## 11. Limitations

* Cannot detect offline recording devices with no RF emission
* Cannot identify device owner
* Detection is probabilistic, not absolute
* Modern MAC randomization reduces tracking reliability

The product goal is transparency, not forensic precision.

---

## 12. Ethical and Legal Considerations

ConsentMesh:

* Does not interfere with communication
* Does not decrypt traffic
* Does not log personal data
* Does not identify individuals

It only observes publicly emitted RF metadata.

Deployment should comply with local RF monitoring regulations.

---

## 13. Educational Value

This project provides hands-on experience in:

* 802.11 frame analysis
* BLE advertisement parsing
* Embedded systems design
* Distributed consensus logic
* Privacy-by-design architecture
* RF signal behavior analysis

---

## 14. Future Improvements

* Advanced statistical inference
* Machine learning behavioral classification
* Web-based dashboard
* Audit log generation
* Encrypted event storage
* Consent terminal mode at room entrance

---

## 15. License

This project is developed for academic purposes.
Further commercialization requires regulatory analysis.

---

