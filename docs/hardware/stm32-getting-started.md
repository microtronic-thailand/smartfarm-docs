# STM32 Getting Started Guide for Smart Farm IoT Platform

## What is STM32?

**STM32** is a family of 32-bit microcontrollers from STMicroelectronics based on ARM Cortex-M cores. They are more powerful than ESP32/ESP8266 and widely used in industrial applications.

### Why STM32 for Smart Farm?

✅ **More Powerful**: Faster processing (up to 480 MHz)  
✅ **Industrial Grade**: Better reliability and temperature range  
✅ **Low Power**: Excellent for battery-powered sensors  
✅ **Rich Peripherals**: More ADC channels, timers, communication interfaces  
⚠️ **No Built-in WiFi**: Requires external WiFi module (ESP-01, ESP8266, or SIM800)

---

## Recommended STM32 Boards for Beginners

### 1. **STM32F103C8T6 "Blue Pill"** (Best for Beginners)
- **Price**: ~$2-3 USD
- **CPU**: 72 MHz ARM Cortex-M3
- **RAM**: 20 KB
- **Flash**: 64 KB (some have 128 KB)
- **GPIO**: 37 pins
- **ADC**: 10 channels (12-bit)
- **Where to Buy**: Shopee, Lazada, AliExpress

**Pros:**
- Very cheap
- Large community support
- Compatible with Arduino IDE
- Easy to find tutorials

**Cons:**
- Needs USB-to-Serial adapter for programming (ST-Link V2)
- No built-in WiFi

### 2. **STM32F401CCU6 "Black Pill"** (Recommended)
- **Price**: ~$4-5 USD
- **CPU**: 84 MHz ARM Cortex-M4
- **RAM**: 64 KB
- **Flash**: 256 KB
- **USB**: Built-in USB (no need for ST-Link after first flash)

**Pros:**
- More powerful than Blue Pill
- Built-in USB
- Better for complex projects

### 3. **STM32 Nucleo Boards** (Official, Best Quality)
- **Price**: ~$10-15 USD
- **Includes**: Built-in ST-Link programmer
- **Quality**: Official from STMicroelectronics

---

## Required Hardware

### Minimum Setup (Blue Pill):
1. **STM32F103C8T6 Blue Pill** (~$2)
2. **ST-Link V2 Programmer** (~$2)
3. **USB-to-Serial (FTDI/CH340)** for debugging (~$1)
4. **ESP-01 WiFi Module** (~$2) or **ESP8266 NodeMCU** (~$3)
5. **Breadboard & Jumper Wires** (~$3)

**Total**: ~$10-13 USD

### Sensors (Same as ESP32):
- DHT22 (Temperature/Humidity)
- Soil Moisture Sensor
- Relay Module

---

## Software Setup

### Step 1: Install STM32CubeIDE (Official, Recommended)

**Download**: [https://www.st.com/en/development-tools/stm32cubeide.html](https://www.st.com/en/development-tools/stm32cubeide.html)

1. Create a free ST account
2. Download STM32CubeIDE for Windows
3. Install (takes ~10 minutes)

### Step 2: Install ST-Link Drivers

**Download**: [https://www.st.com/en/development-tools/stsw-link009.html](https://www.st.com/en/development-tools/stsw-link009.html)

1. Download and install
2. Connect ST-Link V2 to your PC
3. Verify in Device Manager (should show "STMicroelectronics STLink dongle")

### Alternative: Arduino IDE (Easier for Beginners)

1. **Install Arduino IDE** (if not installed)
2. **Add STM32 Board Support:**
   - Open Arduino IDE
   - Go to: `File` → `Preferences`
   - Add to "Additional Boards Manager URLs":
     ```
     https://github.com/stm32duino/BoardManagerFiles/raw/main/package_stmicroelectronics_index.json
     ```
   - Go to: `Tools` → `Board` → `Boards Manager`
   - Search "STM32" and install "STM32 MCU based boards"

---

## Hardware Connections

### Connecting ST-Link V2 to Blue Pill

| ST-Link V2 | Blue Pill |
|------------|-----------|
| 3.3V       | 3.3V      |
| GND        | GND       |
| SWDIO      | DIO       |
| SWCLK      | DCLK      |

### Connecting ESP-01 (WiFi) to Blue Pill

| ESP-01 | Blue Pill |
|--------|-----------|
| VCC    | 3.3V      |
| GND    | GND       |
| TX     | A10 (RX)  |
| RX     | A9 (TX)   |
| CH_PD  | 3.3V      |

⚠️ **Important**: ESP-01 uses 3.3V, NOT 5V!

---

## Your First STM32 Program: Blink LED

### Using Arduino IDE:

```cpp
// Blink LED on PC13 (Blue Pill built-in LED)

void setup() {
  pinMode(PC13, OUTPUT);
}

void loop() {
  digitalWrite(PC13, HIGH);  // LED OFF (inverted on Blue Pill)
  delay(1000);
  digitalWrite(PC13, LOW);   // LED ON
  delay(1000);
}
```

**Upload Steps:**
1. Connect ST-Link V2 to Blue Pill
2. Select: `Tools` → `Board` → `STM32 boards groups` → `Generic STM32F1 series`
3. Select: `Tools` → `Board part number` → `BluePill F103C8`
4. Select: `Tools` → `Upload method` → `STLink`
5. Click **Upload** ⬆️

---

## Smart Farm Example: Read DHT22 Sensor

### Install Library:
- Arduino IDE → `Sketch` → `Include Library` → `Manage Libraries`
- Search "DHT sensor library" by Adafruit
- Install

### Code:

```cpp
#include <DHT.h>

#define DHTPIN PA0     // DHT22 connected to PA0
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  dht.begin();
  Serial.println("STM32 DHT22 Test");
}

void loop() {
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();

  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.print(" %\t");
  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println(" °C");

  delay(2000);
}
```

---

## Connecting to Smart Farm Platform (MQTT)

### Using ESP-01 as WiFi Module:

```cpp
#include <SoftwareSerial.h>

// ESP-01 connected to PA9 (TX) and PA10 (RX)
SoftwareSerial esp8266(PA10, PA9); // RX, TX

void setup() {
  Serial.begin(115200);
  esp8266.begin(115200);
  
  // Connect to WiFi
  sendCommand("AT+CWMODE=1", 1000);
  sendCommand("AT+CWJAP=\"YourWiFi\",\"YourPassword\"", 5000);
  
  // Connect to MQTT Broker
  // (Use AT commands or ESP8266 firmware like ESP-Link)
}

void sendCommand(String cmd, int timeout) {
  esp8266.println(cmd);
  long int time = millis();
  while((time + timeout) > millis()) {
    while(esp8266.available()) {
      char c = esp8266.read();
      Serial.print(c);
    }
  }
}
```

### Better Option: Use ESP8266 NodeMCU as WiFi Bridge

Instead of ESP-01, use a full ESP8266 NodeMCU:
- STM32 reads sensors
- Sends data to ESP8266 via UART
- ESP8266 handles WiFi/MQTT

**Benefits:**
- Easier to program
- More stable
- More GPIO for sensors

---

## Troubleshooting

### Problem: Can't upload code
**Solution:**
1. Check ST-Link connections (SWDIO, SWCLK, GND, 3.3V)
2. Try pressing RESET button on Blue Pill
3. In Arduino IDE, try: `Tools` → `Upload method` → `Serial` (if using USB-to-Serial)

### Problem: ESP-01 not responding
**Solution:**
1. Check voltage (must be 3.3V, NOT 5V!)
2. Connect CH_PD to 3.3V
3. Use external 3.3V power supply (Blue Pill might not provide enough current)

### Problem: DHT22 returns NaN
**Solution:**
1. Check wiring (VCC, GND, Data)
2. Add 10kΩ pull-up resistor between Data and VCC
3. Wait 2 seconds after power-on before reading

---

## STM32 vs ESP32 Comparison

| Feature | STM32F103 (Blue Pill) | ESP32 |
|---------|----------------------|-------|
| **Price** | $2-3 | $4-6 |
| **CPU Speed** | 72 MHz | 240 MHz |
| **WiFi** | ❌ (needs module) | ✅ Built-in |
| **Bluetooth** | ❌ | ✅ Built-in |
| **Power Consumption** | Lower | Higher |
| **Programming** | Harder (for beginners) | Easier |
| **Industrial Use** | ✅ Common | ⚠️ Less common |
| **Best For** | Battery sensors, industrial | WiFi projects, prototyping |

---

## Recommended Learning Path

### Week 1: Basics
1. ✅ Blink LED
2. ✅ Read button input
3. ✅ Serial communication

### Week 2: Sensors
1. ✅ Read DHT22 (Temperature/Humidity)
2. ✅ Read analog sensor (Soil moisture)
3. ✅ Control relay

### Week 3: Communication
1. ✅ UART communication with ESP8266
2. ✅ Send data to MQTT broker
3. ✅ Receive commands from platform

### Week 4: Integration
1. ✅ Complete Smart Farm node
2. ✅ Test with real sensors
3. ✅ Deploy in the field

---

## Additional Resources

### Official Documentation:
- [STM32 Official Site](https://www.st.com/en/microcontrollers-microprocessors/stm32-32-bit-arm-cortex-mcus.html)
- [STM32duino Wiki](https://github.com/stm32duino/wiki/wiki)

### Tutorials (Thai):
- [Arduino.in.th - STM32](https://www.arduino.in.th/tag/stm32/)
- [Cyberbrain - STM32 Tutorial](https://www.cyberbrain.co.th/stm32-tutorial/)

---

## Next Steps

1. ✅ Buy STM32 Blue Pill + ST-Link V2
2. ✅ Install STM32CubeIDE or Arduino IDE
3. ✅ Test Blink LED
4. ✅ Connect DHT22 sensor
5. ✅ Add ESP-01 or ESP8266 for WiFi
6. 🚀 Connect to Smart Farm Platform!
