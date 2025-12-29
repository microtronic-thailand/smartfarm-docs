# Off-Grid Power Monitoring

คู่มือการติดตั้งระบบตรวจสอบสุขภาพแบตเตอรี่และพลังงานสำหรับอุปกรณ์ที่ใช้โซลาร์เซลล์

## Overview
โปรเจกต์นี้ช่วยให้คุณติดตามแรงดันแบตเตอรี่ (Voltage) และสถานะการชาร์จ เพื่อป้องกันแบตเตอรี่เสื่อมสภาพจากการใช้งานจนเกลี้ยง (Deep Discharge)

## Required Components
- ESP32 หรือ STM32
- Voltage Divider Module (หรือใช้ตัวต้านทาน 2 ตัว R1=30k, R2=7.5k)
- แบตเตอรี่ Li-ion 18650 หรือ แบตเตอรี่ตะกั่วกรด 12V
- Solar Controller (กรณีใช้โซลาร์เซลล์)

## Wiring Diagram

| Component Pin | ESP32 Pin |
|---|---|
| Voltage Sensor Output | GPIO 32 |

## Source Code

```cpp
#include <SmartFarmIoT.h>
#include <SmartFarmSensors.h>

const char* device_id = "PWR-003";
const char* token     = "your-secret-token";
SmartFarmIoT sf(device_id, token);

// กำหนดขา Pin และอัตราทดแรงดัน (Voltage Divider Ratio)
VoltageSensor battery(32, 0.5); 

void setup() {
  Serial.begin(115200);
  
  sf.begin("Field-WiFi", "password", "mqtt.smartfarm.com");
  sf.allowHybrid("192.168.1.100");
}

void loop() {
  sf.loop();

  static unsigned long lastSend = 0;
  if (millis() - lastSend > 30000) { 
    lastSend = millis();
    
    float volt = battery.readVoltage();
    int percent = battery.readPercent(3.2, 4.2); // คำนวณเป็น % ตามช่วงแรงดัน
    
    StaticJsonDocument<200> data;
    data["voltage"] = volt;
    data["battery_pct"] = percent;
    
    // ส่งข้อมูลพร้อมระบุค่าแบตเตอรี่ลงใน System Field เพื่อการแจ้งเตือน
    sf.sendTelemetry(data.as<JsonObject>(), volt, sf.getRSSI());
  }
}
```

## Maintenance Tips
- **Deep Sleep**: สำหรับอุปกรณ์โซลาร์ แนะนำให้ใช้คำสั่ง `ESP.deepSleep()` เพื่อประหยัดพลังงาน
- **Calibration**: ควรใช้มัลติมิเตอร์วัดค่าจริงและปรับค่าในโค้ดให้ตรงกันเพื่อความแม่นยำ
