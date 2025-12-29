# ESP32 Connection to EMQX Cloud 🛰️

คู่มือการตั้งค่าบอร์ด ESP32 เพื่อเชื่อมต่อกับระบบ Smart Farm ผ่าน EMQX Cloud (MQTT)

---

## 🔧 อุปกรณ์ที่ต้องเตรียม
1.  บอร์ด ESP32 (รุ่นใดก็ได้)
2.  โปรแกรม Arduino IDE
3.  Library: `PubSubClient` โดย Nick O'Leary

---

## 📝 ตัวอย่าง Code (Arduino / C++)

ก๊อปปี้โค้ดด้านล่างนี้ไปใส่ใน Arduino IDE และแก้ไขค่าในส่วน `CONFIG` ให้เป็นของคุณ

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

// --- [CONFIG] ส่วนที่ต้องแก้ไข ---
const char* ssid = "ชื่อ_WiFi_ของคุณ";
const char* password = "รหัส_WiFi_ของคุณ";

// ข้อมูลจาก EMQX Cloud Overview
const char* mqtt_broker = "ga760710.ala.asia-southeast1.emqxsl.com"; 
const int mqtt_port = 1883;

// ข้อมูลจาก EMQX Cloud -> Authentication
const char* mqtt_username = "iot-platfrom";
const char* mqtt_password = "รหัสผ่านที่ตั้งไว้";

// ไอดีของอุปกรณ์ (ตั้งให้ไม่ซ้ำกัน)
const char* device_id = "ESP32_FARM_001";
// -----------------------------

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_broker, mqtt_port);
}

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi connected");
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    if (client.connect(device_id, mqtt_username, mqtt_password)) {
      Serial.println("connected");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // ส่งข้อมูลจำลองทุกๆ 10 วินาที
  static uint32_t last_send = 0;
  if (millis() - last_send > 10000) {
    last_send = millis();
    
    float temp = 25.0 + random(0, 100) / 10.0;
    
    // สร้าง JSON Payload
    String payload = "{\"value\": " + String(temp) + "}";
    
    // Topic: telemetry/[device_id]/temperature
    String topic = "telemetry/" + String(device_id) + "/temperature";
    
    Serial.print("Publishing to: ");
    Serial.println(topic);
    client.publish(topic.c_str(), payload.c_str());
  }
}
```

---

## 📈 วิธีตรวจสอบการเชื่อมต่อ
1.  เบิร์นโค้ดลงบอร์ด ESP32
2.  เปิด **Serial Monitor** ใน Arduino IDE เพื่อดูสถานะการเชื่อมต่อ
3.  เปิดหน้า **Dashboard** ของคุณ (localhost:3000/dashboard)
4.  เลือก ID อุปกรณ์ให้ตรงกับ `device_id` ในโค้ด
5.  คุณจะเห็นค่ากราฟขยับแบบ **Real-time** ทันทีที่มีข้อมูลเข้ามา!

---

> **แอดมินโน้ต**: หากบอร์ดเชื่อมต่อไม่ได้ ให้ตรวจสอบว่าได้เพิ่ม User ในเมนู **Authentication** ของหน้า EMQX Cloud หรือยัง
