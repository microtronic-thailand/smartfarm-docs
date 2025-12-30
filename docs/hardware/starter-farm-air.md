# 💨 Smart Farm: Air Quality & Gas Monitor

โครงการสำหรับตรวจวัดคุณภาพอากาศภายในโรงเรือนหรือฟาร์มปิด เพื่อป้องกันก๊าซพิษที่อาจส่งผลต่อพืชและสัตว์

---

## 📦 รายการอุปกรณ์ (BOM)
1. ESP32 DevKit V1
2. MQ-135 Sensor (Air Quality/NH3)
3. MQ-2 Sensor (Smoke/LPG)
4. Active Buzzer (สำหรับแจ้งเตือน)

---

## 🔌 การต่อสาย (Wiring)
- **MQ-135 AO** -> **GPIO 32**
- **MQ-2 AO** -> **GPIO 33**
- **Buzzer (+)** -> **GPIO 25**

---

## 💻 Starter Code (Arduino IDE)

```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>

#define MQ135_PIN 32
#define MQ2_PIN 33
#define BUZZER_PIN 25

void setup() {
  Serial.begin(115200);
  pinMode(BUZZER_PIN, OUTPUT);
  // ... (WiFi & MQTT Setup)
}

void loop() {
  int gas1 = analogRead(MQ135_PIN);
  int gas2 = analogRead(MQ2_PIN);

  // แจ้งเตือนถ้าค่าก๊าซเกินกำหนด
  if (gas1 > 2000 || gas2 > 2000) {
    digitalWrite(BUZZER_PIN, HIGH);
  } else {
    digitalWrite(BUZZER_PIN, LOW);
  }

  StaticJsonDocument<128> doc;
  doc["air_quality"] = gas1;
  doc["smoke_level"] = gas2;
  doc["alert"] = (gas1 > 2000 || gas2 > 2000);

  char buffer[128];
  serializeJson(doc, buffer);
  client.publish("telemetry/farm/air", buffer);
  
  delay(5000);
}
```
