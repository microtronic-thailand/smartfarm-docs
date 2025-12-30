# 💧 Smart Farm: Water Level & Tank Management

โครงการจัดการระบบน้ำและวัดระดับน้ำในถังพัก สำหรับฟาร์มที่ต้องการระบบแจ้งเตือนน้ำหมด

---

## 📦 รายการอุปกรณ์ (BOM)
1. ESP32 DevKit V1
2. HC-SR04 Ultrasonic Sensor
3. Submersible Water Pump (5V-12V) + Relay Module

---

## 🔌 การต่อสาย (Wiring)
- **HC-SR04 Trigger** -> **GPIO 5**
- **HC-SR04 Echo** -> **GPIO 18**
- **Relay Pin** -> **GPIO 19**

---

## 💻 Starter Code (Arduino IDE)

```cpp
#define TRIG_PIN 5
#define ECHO_PIN 18
#define RELAY_PIN 19

float getDistance() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  long duration = pulseIn(ECHO_PIN, HIGH);
  return duration * 0.034 / 2;
}

void loop() {
  float distance = getDistance();
  int tankHeight = 100; // สมมติถังสูง 100 cm
  int waterLevel = tankHeight - (int)distance;
  int percent = constrain(map(waterLevel, 0, tankHeight, 0, 100), 0, 100);

  StaticJsonDocument<128> doc;
  doc["water_level"] = percent;
  doc["pump_status"] = digitalRead(RELAY_PIN);

  char buffer[128];
  serializeJson(doc, buffer);
  client.publish("telemetry/farm/water", buffer);
  
  delay(2000);
}
```
