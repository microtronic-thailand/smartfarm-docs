# Smart Irrigation System Guide

คู่มือการสร้างระบบรดน้ำอัจฉริยะที่มีระบบตัดสินใจในตัว (Local Logic) และรองรับการสั่งงานทางไกล

## Overview
ระบบนี้จะรดน้ำอัตโนมัติเมื่อความชื้นในดินต่ำกว่าเกณฑ์ที่กำหนด แม้ในช่วงที่อินเทอร์เน็ตล่ม (Offline) ระบบก็ยังทำงานได้ปกติ

## Required Components
- ESP32 DevKit V1
- Capacitive Soil Moisture Sensor
- 1-Channel Relay Module
- Water Pump (DC 3-6V or AC with proper circuit)
- External Power Supply (แนะนำ 5V หรือ 12V ตามประเภทปั๊ม)

## Wiring Diagram

| Component Pin | ESP32 Pin |
|---|---|
| Soil Sensor (Analog) | GPIO 34 |
| Relay Signal | GPIO 5 |

## Source Code

```cpp
#include <SmartFarmIoT.h>
#include <SmartFarmSensors.h>

const char* device_id = "IRR-002";
const char* token     = "your-secret-token";
SmartFarmIoT sf(device_id, token);

SoilMoistureSensor soil(34); // Analog Pin 34
const int RELAY_PIN = 5;

// คำสั่งจาก Cloud / Mobile App
void handleCommand(String command, JsonObject params) {
  if (command == "SET_PUMP") {
    bool state = params["state"];
    digitalWrite(RELAY_PIN, state ? LOW : HIGH); // รีเลย์ส่วนใหญ่เป็น Active Low
    sf.sendCommandResponse("req-id", true, "Pump toggled");
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, HIGH); // เริ่มต้นที่ ปิด
  
  sf.begin("Farm-WiFi", "password", "mqtt.smartfarm.com");
  sf.allowHybrid("10.0.0.5"); // ต่อไปยัง Edge Gateway (ถ้ามี)
  
  sf.onCommand(handleCommand);
}

void loop() {
  sf.loop();

  static unsigned long lastSend = 0;
  if (millis() - lastSend > 10000) {
    lastSend = millis();
    
    int moisture = soil.readMoisture();
    
    // 🧠 Local Logic: ทำงานได้แม้ออฟไลน์
    if (moisture < 30) {
       digitalWrite(RELAY_PIN, LOW); // รดน้ำทันที
    }

    StaticJsonDocument<200> data;
    data["moisture"] = moisture;
    data["pump"]     = digitalRead(RELAY_PIN) == LOW ? "ON" : "OFF";
    
    sf.sendTelemetry(data.as<JsonObject>());
  }
}
```

## Key Features
- **Local Autonomy**: ปั๊มรดน้ำได้เองแม้ไม่มีสัญญาณอินเทอร์เน็ต
- **Manual Override**: คุณสามารถสั่งเปิด/ปิดปั๊มน้ำด้วยตนเองผ่านมือถือได้ทุกเมื่อ
- **Status Reporting**: รายงานสถานะปั๊มน้ำกลับไปยังฐานข้อมูลแบบ Real-time
