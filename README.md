# 🌡️ Temperature Monitoring System using Raspberry Pi & Custom PCB

## 📌 Overview
This project is a real-time Temperature Monitoring System developed using Raspberry Pi 4 and a custom-designed PCB. It continuously measures environmental temperature, processes data in real time, triggers alerts when thresholds are exceeded, and logs data for analysis. The system demonstrates complete embedded systems design including hardware, software, and PCB integration.

---

## 🚀 Features
- Real-time temperature monitoring (1-second interval)
- Temperature range: 0°C to 100°C
- Accuracy: ±0.5°C
- DS18B20 (digital) and LM35 (analog via MCP3008 ADC) support
- Threshold-based alert system (buzzer/LED)
- CSV data logging with timestamps
- Custom PCB for stable hardware integration

---

## 🛠️ Hardware Components
- Raspberry Pi 4
- DS18B20 Temperature Sensor / LM35 Sensor
- MCP3008 ADC (for analog conversion)
- 4.7kΩ Pull-up resistor
- Buzzer / LED
- Custom PCB (KiCad design)

---

## ⚙️ Software Requirements
- Python 3
- RPi.GPIO library
- Built-in libraries: os, glob, time, csv, datetime

---

## 🔌 Hardware Setup

### Sensor Connections (DS18B20)
- VCC → 3.3V  
- GND → Ground  
- DATA → GPIO4  
- Pull-up resistor (4.7kΩ) between VCC and DATA  

---

### Verify Sensor Detection
Run the following command:
```bash
ls /sys/bus/w1/devices/
