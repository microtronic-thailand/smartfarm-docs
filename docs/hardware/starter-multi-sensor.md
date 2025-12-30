# 📦 Grids Multi-Sensor Node (Mini Hub)

โมดูลระดับกลางที่รวมเซนเซอร์หลายชนิดไว้ในโหนดเดียว เพื่อเป็นตัวอย่างก่อนขยับไปใช้ Pro Master Hub

---

## 📦 รายการอุปกรณ์ (BOM)
1. ESP32 DevKit V1
2. DHT22 (Temp/Humid)
3. BH1750 (Light Intensity I2C)
4. SSD1306 OLED Display (I2C)

---

## 🔌 การต่อสาย (I2C Shared Bus)
- **VCC/GND** -> Shared
- **SDA** -> GPIO 21
- **SCL** -> GPIO 22
- **DHT22 Data** -> GPIO 4

---

## 💻 Integrated Code Preview
โหนดนี้จะแสดงค่าผ่านหน้าจอ OLED และส่งขึ้น Cloud พร้อมกัน

```cpp
#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <BH1750.h>
#include <Adafruit_SSD1306.h>

// Initialize all sensors...
// Loop to read all data and send a combined JSON payload
```
