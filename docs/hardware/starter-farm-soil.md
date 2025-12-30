# 🌱 Smart Farm Starter: Soil Moisture Node

โครงการพื้นฐานสำหรับผู้เริ่มต้น เพื่อทดสอบการส่งค่าความชื้นในดินเข้าสู่แพลตฟอร์ม Grids

---

## 📦 รายการอุปกรณ์ (BOM)
1. ESP32 DevKit V1
2. Soil Moisture Sensor (Analog/Capacitive)
3. Jumper Wires

---

## 🔌 การต่อสาย (Wiring)
- **Soil Sensor VCC** -> 3.3V
- **Soil Sensor GND** -> GND
- **Soil Sensor AUOT/Signal** -> **GPIO 34** (Analog Input)

---

## 💻 Starter Code (Arduino IDE)
เน้นความง่ายเพื่อให้เห็นข้อมูลบน Dashboard ทันที

```cpp
#include <WiFi.h>
#include <PubSubClient.h> // ติดตั้งผ่าน Library Manager
#include <ArduinoJson.h>

const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";
const char* mqtt_server = "ga760710.ala.asia-southeast1.emqxsl.com"; // EMQX Cloud
const char* client_id = "FARM-STARTER-001";

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) { delay(500); }
  
  client.setServer(mqtt_server, 1883);
}

void loop() {
  if (!client.connected()) {
    if (client.connect(client_id, "user", "pass")) {
      Serial.println("Connected to Grids Cloud!");
    }
  }
  client.loop();

  static uint32_t lastMsg = 0;
  if (millis() - lastMsg > 10000) {
    lastMsg = millis();
    
    int rawValue = analogRead(34);
    int percent = map(rawValue, 4095, 1500, 0, 100); // ปรับจูนตามเซนเซอร์

    StaticJsonDocument<128> doc;
    doc["soil_moisture"] = percent;
    doc["status"] = (percent < 30) ? "Dry" : "Good";

    char buffer[128];
    serializeJson(doc, buffer);
    client.publish("telemetry/farm/soil", buffer);
    Serial.println("Sent telemetry!");
  }
}
```

---

## 🚀 การแสดงผล
ข้อมูลจะไปปรากฏที่หน้า **"มอนิเตอร์สด" (Live Monitor)** ในช่องข้อมูลการเกษตร
