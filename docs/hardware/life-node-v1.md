# 🏥 Grids Life-Node Hardware Guide (V1)

แผนภาพและตัวอย่างโค้ดสำหรับการสร้างอุปกรณ์สวมใส่ตรวจจับสุขภาพ

---

## 🔌 การต่อสาย (Wiring)
- **MAX30102 (I2C)**: 
  - SDA -> GPIO 21
  - SCL -> GPIO 22
  - VCC -> 3.3V
  - GND -> GND

---

## 🖥️ ESP32 Firmware Logic

```cpp
#include <Wire.h>
#include "MAX30102_lib.h" // ต้องติดตั้ง Library ก่อน
#include <SmartFarmIoT.h> // ใช้ Library ที่เราทำไว้ได้เลย!

SmartFarmIoT device("LIFE-NODE-001", "TOKEN_XYZ");
MAX30102 pulseSensor;

void setup() {
  Serial.begin(115200);
  
  // เริ่มต้นเซนเซอร์
  if (!pulseSensor.begin(Wire, I2C_SPEED_FAST)) {
    Serial.println("MAX30102 was not found. Please check wiring/power.");
    while (1);
  }

  // เริ่มต้นการเชื่อมต่อแพลตฟอร์ม
  device.begin("WIFI_SSID", "WIFI_PASS", "ga760710.ala.asia-southeast1.emqxsl.com");
}

void loop() {
  device.loop();

  static uint32_t lastReport = 0;
  if (millis() - lastReport > 5000) { // ส่งทุก 5 วินาที
    lastReport = millis();

    // จำลองการอ่านค่า (ในเครื่องจริงใช้ค่าจาก pulseSensor)
    float activeHR = pulseSensor.getHeartRate();
    float activeSPO2 = pulseSensor.getSpO2();

    // สร้างข้อมูลส่งเข้า Dashboard
    StaticJsonDocument<200> doc;
    doc["heartRate"] = activeHR;
    doc["spo2"] = activeSPO2;
    doc["temp"] = 36.5; // เพิ่มเซนเซอร์อุณหภูมิภายหลัง
    
    // ส่งเข้า Topic พิเศษสำหรับสุขภาพ
    device.sendTelemetry(doc.as<JsonObject>());
  }
}
```

---

## 📦 MQTT Payload Structure
เมื่ออุปกรณ์เชื่อมต่อสำเร็จ ให้ส่งข้อมูลมาที่ Topic: `telemetry/[device_id]/health` 
โดยใช้รูปแบบ JSON ดังนี้ เพื่อให้ระบบ AI และ Dashboard นำไปแสดงผลได้ทันที:

```json
{
  "heartRate": 75,
  "spo2": 98,
  "temp": 36.6,
  "movement": 120, 
  "isSitting": true,
  "timestamp": "2025-12-30T03:30:00Z"
}
```

---

## 🚑 หมายเหตุสำหรับการทดสอบ AI
ในรูปที่คุณส่งมาล่าสุด หมอตอบว่า **"ขออภัย หมอไม่สามารถวิเคราะห์ได้"** สาเหตุหลักคือ:
1.  **API Key**: คุณยังไม่ได้เปลี่ยน `GEMINI_API_KEY` ในไฟล์ `.env.local` ครับ (ต้องเอาจาก Google AI Studio มาใส่)
2.  **Credit**: ระบบตรวจสอบ Credit ของ User (ผมตั้ง Demo ไว้ที่ 10 ถ้าหมดจะใช้ไม่ได้)

**ถ้า Hardware เสร็จ ข้อมูลไหลเข้า Dashboard ปุ๊บ... หมอ AI จะเริ่มทักทายคุณทันทีครับ!**
