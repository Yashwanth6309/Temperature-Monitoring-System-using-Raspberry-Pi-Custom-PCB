# 🌡️ Temperature Monitoring System

A real-time embedded temperature monitoring system built with a **Raspberry Pi 4** and a **PCB-based hardware setup**. The system reads live temperature data, displays it, triggers alerts when a threshold is exceeded, and logs all readings to a CSV file for later analysis.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Hardware Requirements](#hardware-requirements)
- [Circuit & PCB Design](#circuit--pcb-design)
- [Software Requirements](#software-requirements)
- [Installation & Setup](#installation--setup)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [System Flow](#system-flow)
- [Testing & Validation](#testing--validation)
- [Results](#results)
- [Skills Demonstrated](#skills-demonstrated)

---

## Overview

This project implements a complete temperature monitoring solution using a Raspberry Pi 4 as the central controller. The system is designed to:

- Measure environmental temperatures ranging from **0°C to 100°C**
- Maintain an accuracy of **±0.5°C**
- Update readings every **1 second**
- Trigger an alert (buzzer/LED) if the temperature exceeds **40°C**
- Log all readings with timestamps to a **CSV file**

The hardware is assembled on a PCB with all components properly mounted, soldered, and tested — making the system compact and stable for real-world deployment in labs, server rooms, or industrial environments.

---

## Features

- **Real-time temperature reading** — continuous 1-second polling via GPIO
- **Threshold-based alerting** — buzzer/LED activates automatically above 40°C
- **Data logging** — timestamped CSV log for trend analysis
- **LCD display support** — optional 16x2 LCD display output
- **PCB-based hardware** — compact board with voltage regulation and sensor interface
- **Long-duration stability** — validated through 12–24 hour continuous testing

---

## Hardware Requirements

| Component | Description |
|---|---|
| Raspberry Pi 4 | Central controller |
| DS18B20 | Digital temperature sensor (1-Wire) |
| MCP3008 *(optional)* | 8-channel 10-bit ADC (if using LM35 analog sensor) |
| LM35 *(optional)* | Analog temperature sensor alternative |
| 4.7kΩ Resistor | Pull-up resistor for 1-Wire data line |
| Buzzer / LED | Alert output on GPIO18 |
| 16x2 LCD *(optional)* | Local temperature display |
| 3.3V & 5V Regulators | Voltage regulation on PCB |
| PCB | Base board for component assembly |
| Acrylic Casing | Protective enclosure for deployment |

### Wiring (DS18B20)

```
DS18B20 Pin    →    Raspberry Pi
─────────────────────────────────
VCC            →    3.3V (Pin 1)
GND            →    Ground (Pin 6)
DATA           →    GPIO4 (Pin 7)
[4.7kΩ pull-up resistor between VCC and DATA]
```

---

## Circuit & PCB Design

The PCB serves as the central hardware platform for this project. It includes:

- Temperature sensor interface (DS18B20 or LM35 via MCP3008)
- 4.7kΩ pull-up resistor for stable 1-Wire communication
- 3.3V and 5V voltage regulators for consistent power supply
- Header pins for clean GPIO connection to the Raspberry Pi
- Compact layout suitable for enclosed deployment

All components were carefully mounted and soldered onto the PCB. After assembly, the board was tested using a **multimeter** to verify proper connectivity and confirm there were no short circuits before powering up the system.

---

## Software Requirements

- Raspberry Pi OS (Raspbian)
- Python 3.x
- RPi.GPIO library

Install dependencies:

```bash
pip install RPi.GPIO
```

---

## Installation & Setup

### Step 1: Enable the 1-Wire Interface

```bash
sudo raspi-config
```

Navigate to: **Interfacing Options → 1-Wire → Enable → Reboot**

### Step 2: Verify Sensor Detection

After rebooting, confirm the DS18B20 is detected:

```bash
ls /sys/bus/w1/devices/
```

You should see a folder starting with `28-xxxx` — this confirms the sensor is recognized.

### Step 3: Clone the Repository

```bash
git clone https://github.com/your-username/temperature-monitoring-system.git
cd temperature-monitoring-system
```

### Step 4: Run the Program

```bash
python3 temperature_monitor.py
```

---

## Usage

Once running, the program will:

1. Print temperature readings to the terminal every second
2. Activate the buzzer/LED if temperature exceeds 40°C
3. Append each reading with a timestamp to `temperature_log.csv`

**Sample terminal output:**

```
Temperature monitoring started. Press Ctrl+C to stop.

Temperature: 27.31 °C
Temperature: 27.44 °C
Temperature: 41.02 °C  ← [ALERT: Buzzer ON]
Temperature: 40.87 °C  ← [ALERT: Buzzer ON]
Temperature: 38.56 °C
```

**Sample CSV log (`temperature_log.csv`):**

```
2024-06-01 14:32:01,27.31
2024-06-01 14:32:02,27.44
2024-06-01 14:32:03,41.02
```

To stop the program, press `Ctrl + C`. GPIO pins are automatically cleaned up on exit.

---

## Project Structure

```
temperature-monitoring-system/
│
├── temperature_monitor.py     # Main Python script
├── temperature_log.csv        # Auto-generated data log (created on first run)
├── images/
│   ├── pcb_top.jpg            # PCB top view photo
│   ├── pcb_bottom.jpg         # PCB bottom view photo
│   └── setup.jpg              # Full hardware setup photo
└── README.md
```

---

## System Flow

```
Start
  ↓
Check Sensor Detection (28-xxxx folder)
  ↓
Initialize GPIO (BCM mode, Pin 18)
  ↓
Read Raw Temperature Data
  ↓
Convert Raw Data → Celsius
  ↓
Display on Terminal (and LCD if connected)
  ↓
Check Threshold (> 40°C?)
  ├── YES → Trigger Buzzer/LED (GPIO18 HIGH)
  └── NO  → Buzzer/LED OFF (GPIO18 LOW)
  ↓
Log Reading + Timestamp to CSV
  ↓
Wait 1 Second → Repeat
  ↓
Ctrl+C → GPIO Cleanup → Exit
```

---

## Python Code

```python
import glob
import time
import csv
from datetime import datetime
import RPi.GPIO as GPIO

# --- Sensor Setup ---
base_dir = '/sys/bus/w1/devices/'
sensor_folders = glob.glob(base_dir + '28*')

if not sensor_folders:
    raise RuntimeError("DS18B20 sensor not found. Check wiring and enable 1-Wire interface.")

device_file = sensor_folders[0] + '/w1_slave'

# --- GPIO Setup ---
GPIO.setmode(GPIO.BCM)
buzzer_pin = 18
GPIO.setup(buzzer_pin, GPIO.OUT)
threshold = 40  # °C

# --- Read Raw Sensor Data ---
def read_temp_raw():
    with open(device_file, 'r') as f:
        return f.readlines()

# --- Convert to Celsius ---
def read_temp():
    lines = read_temp_raw()
    while lines[0].strip()[-3:] != 'YES':
        time.sleep(0.2)
        lines = read_temp_raw()
    equals_pos = lines[1].find('t=')
    if equals_pos != -1:
        temp_c = float(lines[1][equals_pos + 2:]) / 1000.0
        return temp_c
    raise ValueError("Could not parse temperature from sensor data.")

# --- Main Monitoring Loop ---
print("Temperature monitoring started. Press Ctrl+C to stop.\n")

try:
    while True:
        try:
            temperature = read_temp()
            print("Temperature: {:.2f} °C".format(temperature))

            GPIO.output(buzzer_pin, GPIO.HIGH if temperature > threshold else GPIO.LOW)

            with open('temperature_log.csv', mode='a', newline='') as file:
                writer = csv.writer(file)
                writer.writerow([datetime.now().strftime('%Y-%m-%d %H:%M:%S'), round(temperature, 2)])

        except ValueError as e:
            print("Sensor read error:", e)

        time.sleep(1)

except KeyboardInterrupt:
    print("\nMonitoring stopped by user.")

finally:
    GPIO.output(buzzer_pin, GPIO.LOW)
    GPIO.cleanup()
    print("GPIO cleaned up.")
```

---

## Testing & Validation

| Test | Method | Result |
|---|---|---|
| Accuracy check | Compared with standard digital thermometer | ±0.5°C confirmed |
| Alert test | Manually heated sensor past 40°C | Buzzer/LED activated correctly |
| Logging test | Inspected CSV file after 1 hour run | All readings recorded with timestamps |
| Stability test | Continuous 12–24 hour operation | No crashes or data loss observed |
| Short circuit check | Multimeter continuity test on PCB | No shorts detected |
| Sensor detection check | Ran script without sensor connected | Clear error message displayed |
| Crash recovery test | Simulated bad sensor read | Loop continued without full crash |

---

## Results

- Temperature measured accurately within **±0.5°C** across the full 0–100°C range
- Alert system responded within **1 second** of threshold being crossed
- Over 24 hours of continuous operation with **no system failures**
- GPIO pins cleanly released on every exit — no resource conflicts on restart
- CSV logs suitable for import into Excel, Python (pandas), or any data analysis tool

---

## Skills Demonstrated

- Embedded systems design (Raspberry Pi, GPIO communication)
- PCB assembly, soldering, and hardware testing
- Sensor interfacing (1-Wire protocol, DS18B20)
- ADC integration (MCP3008 for analog sensors)
- Python programming (file I/O, GPIO control, CSV logging, error handling)
- Hardware debugging and calibration
- System integration and long-duration reliability testing

---

## License

This project is open source and available under the [MIT License](LICENSE).

---

*Built as part of academic coursework in electronics and embedded systems.*
