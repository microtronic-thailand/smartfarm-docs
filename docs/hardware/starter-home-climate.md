# 🏠 Smart Home Starter: Climate & Light Control

โครงการพื้นฐานสำหรับผู้เริ่มต้น เพื่อควบคุมไฟและตรวจวัดอุณหภูมิภายในบ้าน

---

## 📦 รายการอุปกรณ์ (BOM)
1. ESP32 DevKit V1
2. DHT11 หรือ DHT22 Sensor
3. 1-Channel Relay Module (5V/3.3V)

---

## 🔌 การต่อสาย (Wiring)
- **DHT11 Data** -> **GPIO 4**
- **Relay Input** -> **GPIO 5** (ควบคุมหลอดไฟ)

---

## 💻 Starter Code (Arduino IDE)

```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>
#include <ArduinoJson.h>

#define DHTPIN 4
#define DHTTYPE DHT11
#define RELAY_PIN 5

DHT dht(DHTPIN, DHTTYPE);
WiFiClient espClient;
PubSubClient client(espClient);

void callback(char* topic, byte* payload, unsigned int length) {
  // รับคำสั่งเปิด/ปิดไฟจาก Dashboard
  if (payload[0] == '1') digitalWrite(RELAY_PIN, HIGH);
  else digitalWrite(RELAY_PIN, LOW);
}

void setup() {
  Serial.begin(115200);
  pinMode(RELAY_PIN, OUTPUT);
  dht.begin();
  // ... (WiFi & MQTT Setup เหมือนเดิม)
  client.setCallback(callback);
}

void loop() {
  // ... (MQTT Reconnect)
  client.loop();

  static uint32_t lastMsg = 0;
  if (millis() - lastMsg > 15000) {
    lastMsg = millis();
    
    float h = dht.readHumidity();
    float t = dht.readTemperature();

    StaticJsonDocument<128> doc;
    doc["humidity"] = h;
    doc["temperature"] = t;
    doc["light_status"] = digitalRead(RELAY_PIN);

    char buffer[128];
    serializeJson(doc, buffer);
    client.publish("telemetry/home/climate", buffer);
  }
}
```

---

## 🚀 การจัดการ
คุณสามารถกดปุ่มสวิตช์หน้า **"บ้านอัจฉริยะ"** เพื่อสั่งงาน Relay ตัวนี้ได้ทันที
