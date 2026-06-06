# Healthcare Monitoring System 🏥

An ESP32-based patient monitoring prototype built with **FreeRTOS** and **Adafruit IO (MQTT)**.  
Simulates or measures body temperature, heart rate, SpO₂, and motion, then displays readings on an OLED and pushes them to the cloud.

▶ **[Open in Wokwi Simulator](https://wokwi.com/projects/465912631455841281)**

---

## Features

| Feature | Details |
|---|---|
| **Temperature** | DS18B20 one-wire sensor (GPIO 14) |
| **Heart Rate** | Simulated random value (replace with MAX30102 or similar) |
| **SpO₂** | Simulated random value (replace with MAX30102 or similar) |
| **Motion** | PIR sensor (GPIO 12) |
| **Display** | SSD1306 OLED 128×64 via I²C |
| **Alerts** | 4 individual buzzers (GPIOs 4, 16, 17, 5) |
| **Cloud** | Adafruit IO MQTT feeds |
| **RTOS** | Two FreeRTOS tasks + two queues |

---

## Hardware / Wiring

| ESP32 Pin | Component | Signal |
|---|---|---|
| 3.3 V | All VCC rails | Power |
| GND | All GND rails | Ground |
| GPIO 14 | DS18B20 DQ | 1-Wire data |
| GPIO 21 | OLED SDA | I²C data |
| GPIO 22 | OLED SCL | I²C clock |
| GPIO 12 | PIR OUT | Motion signal |
| GPIO 4 | Buzzer 1 | Temp alert |
| GPIO 16 | Buzzer 2 | Heart-rate alert |
| GPIO 17 | Buzzer 3 | SpO₂ alert |
| GPIO 5 | Buzzer 4 | Motion alert |

---

## Alert Thresholds

| Vital | Condition | Buzzer |
|---|---|---|
| Body temperature | > 38 °C | Buzzer 1 |
| Heart rate | < 60 bpm **or** > 100 bpm | Buzzer 2 |
| SpO₂ | < 85 % | Buzzer 3 |
| Motion | PIR triggered | Buzzer 4 |

---

## Adafruit IO Feeds

| Feed key | Data |
|---|---|
| `body-temperature-degrees-c` | °C (float) |
| `heart-rate-monitor` | bpm (int) |
| `spo2-percent` | % (int) |
| `motion-detector` | 0 / 1 |
| `flag` | Status string |

---

## FreeRTOS Architecture

```
┌───────────────────┐          ┌────────────────────────────┐
│   sensorTask      │          │   processingTask           │
│   (Core 0, P=2)   │          │   (Core 1, P=1)            │
│                   │          │                            │
│  random HR/SpO2   │─hrQueue─▶│  Read HR, SpO2             │
│  every 1 s        │─spo2Q──▶│  Read DS18B20 temp         │
│                   │          │  Update OLED               │
│  (replace with    │          │  Trigger buzzers           │
│   real sensor     │          │  Publish to Adafruit IO    │
│   driver here)    │          │  every 1 s                 │
└───────────────────┘          └────────────────────────────┘
```

---

## Getting Started

### Wokwi (browser – no hardware needed)

1. Open the [Wokwi project link](https://wokwi.com/projects/465912631455841281).
2. Click **▶ Start Simulation**.

### Arduino IDE

1. Install the libraries listed in `libraries.txt` via the Library Manager.
2. Set board to **ESP32 Dev Module**.
3. Edit `sketch.ino` – replace `IO_USERNAME` and `IO_KEY` with your Adafruit IO credentials.
4. Upload and open Serial Monitor at **115200 baud**.

### PlatformIO (VS Code)

```bash
git clone https://github.com/<your-username>/healthcare-monitoring-esp32.git
cd healthcare-monitoring-esp32
pio run --target upload
```

A `platformio.ini` targeting `esp32dev` is recommended:

```ini
[env:esp32dev]
platform  = espressif32
board     = esp32dev
framework = arduino
lib_deps  =
    paulstoffregen/OneWire
    milesburton/DallasTemperature
    adafruit/Adafruit GFX Library
    adafruit/Adafruit SSD1306
    adafruit/Adafruit MQTT Library
```

---

## Configuration

Edit the top of `sketch.ino`:

```cpp
#define WIFI_SSID  "your-network"
#define WIFI_PASS  "your-password"
#define IO_USERNAME "your-adafruit-username"
#define IO_KEY      "your-adafruit-io-key"
```

---

## Bugs Fixed vs. Original Wokwi Sketch

| Location | Original | Fixed |
|---|---|---|
| `WifiClient` | Incorrect capitalisation | `WiFiClient` |
| `sop2` variable | Typo (`sop2`) | `spo2` |
| `xqueueOverwrite` | Wrong case | `xQueueOverwrite` |
| `xWueuePeek` | Typo | `xQueuePeek` |
| `display.setCurusor` | Typo | `display.setCursor` |
| `display.clearDisplay()` | Missing semicolon | Added `;` |
| Heart-rate upper alert | `heartRate > 300` | `heartRate > 100` (physiological limit) |
| `hrFeed` publish | Used before declaration | Declared properly |
| `MOTION_DETECTOR` macro | Redefined as both pin and feed string | Separated into `MOTION_PIN` and `MOTION_FEED` |

---

## License

MIT – see [LICENSE](LICENSE)
