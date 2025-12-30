# 🛡️ Smart Home: Security & Intrusion Detection

โครงการระบบความปลอดภัยเบื้องต้น แจ้งเตือนเมื่อมีการเคลื่อนไหวหรือการเปิดประตูในบ้าน

---

## 📦 รายการอุปกรณ์ (BOM)
1. ESP32 DevKit V1
2. HC-SR501 PIR Motion Sensor
3. Reed Switch (เซนเซอร์แม่เหล็กติดประตู)

---

## 🔌 การต่อสาย (Wiring)
- **PIR Data Out** -> **GPIO 27**
- **Reed Switch One End** -> **GPIO 26** (Pull-up)
- **Reed Switch Other End** -> GND

---

## 💻 Starter Code (Arduino IDE)

```cpp
#define PIR_PIN 27
#define DOOR_PIN 26

void setup() {
  pinMode(PIR_PIN, INPUT);
  pinMode(DOOR_PIN, INPUT_PULLUP);
  // ... (WiFi & MQTT Setup)
}

void loop() {
  bool motion = digitalRead(PIR_PIN);
  bool doorOpen = !digitalRead(DOOR_PIN);

  StaticJsonDocument<128> doc;
  doc["motion_detected"] = motion;
  doc["door_status"] = doorOpen ? "Open" : "Closed";

  char buffer[128];
  serializeJson(doc, buffer);
  client.publish("telemetry/home/security", buffer);
  
  if(motion || doorOpen) {
    // โค้ดแจ้งเตือน Line Notify หรือเปิดไฟ
  }
  
  delay(1000);
}
```
