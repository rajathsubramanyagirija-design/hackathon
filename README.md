# Satellite Fault Detection and Autonomous Power Supply

**Detect the cause of a power drop first, then keep the power on.**

An Arduino Mega based system that monitors a solar panel, cleans dust off it, tells low light apart from a real panel fault, and switches the load to a 4×AA battery backup only when it is truly needed. A live web dashboard shows everything in real time.

Built by **Team Trikal** (ID: AV26-20D89CF9A4) at Adichunchanagiri Institute of Technology, Chikkamagaluru, for **Avishkar 2K26 – Innovation Parampara**.

## The problem

Dust blocks sunlight from reaching solar cells, so output falls. But a power drop does not reveal its cause. It could be dust, low light, or a failed panel, and each needs a different response. Guess wrong and the system either wastes backup power or loses power completely.

## Our approach

Cause-aware switching: the INA219 power sensor and the LDR light sensor together separate dust, low light and real faults.

| Cause | How the system knows | Response |
|---|---|---|
| Low light | LDR reports low light | Stay on backup, retry later (not counted as a fault) |
| Dust | Current recovers after cleaning | Switch to solar |
| Panel fault | Light is normal but current stays low after cleaning | Stay on backup, count a failure |

Cleaning is mechanical, using a servo-driven carbon fiber brush with no electrostatic force.

## How it works

1. The system **starts on backup** (relay selects the battery).
2. Every **30 seconds** it runs a recovery check.
3. It **checks light** first. If light is low, it stays on backup and retries (not a fault).
4. If light is normal, the servo brush **cleans the panel** (two sweeps).
5. The relay switches to solar and, after a 2 s settle, the system **tests the solar current**.
6. If current is **at least 6 mA**, it runs on **solar**. Otherwise it returns to backup and counts one failed attempt.
7. After **3 failed attempts** it latches a **SOLAR FAULT** and stays on backup.
8. While on solar, **3 consecutive low-current readings** switch it back to backup.

## Hardware

| Part | Job |
|---|---|
| Arduino Mega | Runs the decision logic |
| Solar panel | Primary source, wired through the INA219 to the relay NC contact |
| INA219 | Measures voltage, current and power |
| Relay | Selects solar (NC) or backup (NO) |
| 4×AA battery pack | Backup source, wired to the relay NO contact |
| LDR module (digital out) | Detects light level |
| DHT11 | Temperature and humidity |
| Servo + carbon fiber brush | Cleans the panel (own 5 V adapter) |
| 16×2 LCD | Local status |
| Buzzer | Audible alerts |
| Red LED + 220 Ω resistor | Load indicator on relay COM |

### Pin map (Arduino Mega)

| Signal | Pin |
|---|---|
| Servo | 9 |
| LDR digital out | 2 |
| Relay | 30 (active LOW) |
| DHT11 | 4 |
| Buzzer | 8 |
| LCD RS, E, D4–D7 | 22, 23, 24, 25, 26, 27 |
| INA219 SDA / SCL | 20 / 21 |

## Software

### Arduino firmware

Libraries: `Wire`, `Adafruit_INA219`, `Servo`, `DHT`, `LiquidCrystal`.

```cpp
const float LOW_CURRENT_MA = 6.0;            // solar current threshold
const unsigned long RETRY_SOLAR_MS = 30000;  // recovery check interval
const int MAX_SOLAR_FAILURES = 3;            // failures before fault latch
const int MAX_LOW_READINGS = 3;              // low readings before leaving solar
```

Serial output at 9600 baud, once per second:

```
DATA,mode,status,busV,mA,mW,tempC,humidity,lightOK
```

### Live dashboard (Python)

A small local web server that reads the Arduino serial stream and serves a live dashboard showing power mode, system status, voltage, current, power, light condition, temperature and humidity.

```bash
pip install pyserial
python dashboard.py
```

1. Set `PORT` in the script to your Arduino's port (default `COM9`).
2. Run the script. Your browser opens at `http://127.0.0.1:8765`.
3. Keep the Python window open while using the dashboard.

## Getting started

1. Wire the hardware as above.
2. Open the sketch in the Arduino IDE, install the libraries, select **Arduino Mega**, and upload.
3. Close the Serial Monitor (it blocks the port).
4. Run the dashboard script.

## Limitations

This is a working proof of concept built from standard parts, not flight hardware. A space-ready version would need space-rated electronics and mechanisms tested for vacuum and temperature extremes. Switchover time and long-term brush wear have not been measured yet.

## Future scope

- Monitor each panel separately in a multi-panel array
- Send fault and cleaning status to a ground station
- Extend to other dust-prone systems such as solar farms

## Team Trikal

- Khushi Ravi Acharya
- Prajwal G V
- Rajath S K
- Ramesh K S
