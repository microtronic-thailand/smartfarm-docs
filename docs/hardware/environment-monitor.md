# Environment Monitoring Project

เรียนรู้วิธีการสร้างระบบตรวจสอบสภาพอากาศพื้นฐาน (Temperature & Humidity) โดยใช้ ESP32 และเซนเซอร์ตระกูล DHT

## Overview
โปรเจกต์นี้เป็นการเริ่มต้นที่ง่ายที่สุดสำหรับผู้เริ่มต้น ช่วยให้คุณสามารถติดตามค่าความร้อนและความชื้นในโรงเรือนได้แบบ Real-time ผ่านทาง Dashboard

## Required Components
- ESP32 DevKit V1
- DHT11 หรือ DHT22 Sensor
- Jumper Wires
- Breadboard (optional)

## Wiring Diagram

| Sensor Pin | ESP32 Pin |
|---|---|
| VCC | 3.3V |
| GND | GND |
| DATA | GPIO 4 |

## Source Code

ก๊อปปี้โค้ดด้านล่างนี้ไปวางใน Arduino IDE:

```cpp
#include <SmartFarmIoT.h>
#include <SmartFarmSensors.h>

// 1. Initialize System
const char* device_id = "ENV-001";
const char* token     = "your-secret-token";
SmartFarmIoT sf(device_id, token);

// 2. Initialize Sensors
TemperatureHumiditySensor dht(4, DHT22); // Connected to Pin 4

void setup() {
  Serial.begin(115200);
  
  // 3. Setup Connection
  sf.begin("Home-WiFi", "password", "mqtt.smartfarm.com");
  
  // 4. Enable Hybrid failover (Recommended)
  sf.allowHybrid("192.168.1.50");
  
  dht.begin();
}

void loop() {
  sf.loop();

  static unsigned long lastSend = 0;
  if (millis() - lastSend > 5000) {
    lastSend = millis();
    
    StaticJsonDocument<200> sensors;
    sensors["temp"] = dht.readTemperature();
    sensors["hum"]  = dht.readHumidity();
    
    sf.sendTelemetry(sensors.as<JsonObject>());
  }
}
```

## How to Test
1. อัปโหลดโค้ดลงบอร์ด ESP32
2. เปิด Serial Monitor (Baud rate 115200) เพื่อดูสถานะการเชื่อมต่อ
3. เข้าสู่ระบบ Smart Farm Dashboard เพื่อดูข้อมูลในรูปแบบกราฟ
