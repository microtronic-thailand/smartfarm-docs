# ⚡ Grids Master Hub (Integrated Module)

สำหรับผู้พัฒนาขั้นสูง: นี่คือโมดูลรวมศูนย์ที่รวบรวมเซนเซอร์หลายชนิดไว้ในกล่องเดียว (Super Box) เพื่อการจัดการระบบ Smart Ecosystem แบบเบ็ดเสร็จ

---

## 🏗️ Hardware Architecture (Pro Version)
โมดูลนี้ถูกออกแบบให้เป็น **Edge Gateway** ขนาดเล็กที่ทำหน้าที่เก็บข้อมูลจากทุุกมิติ:

- **Environment**: BME280 (Temp, Humid, Pressure)
- **Air Quality**: CCS811 (VOCs, CO2)
- **Power**: INA219 (Voltage/Current Monitor)
- **Human**: HC-SR501 (PIR Motion)
- **Storage**: MicroSD Card Slot (Offline Logging)

---

## 🛠️ Advanced Connectivity (MQTT Protocol)
Master Hub ใช้โครงสร้าง Topic แบบลำดับชั้นเพื่อให้รองรับการขยายตัว:
- `grids/master/[id]/env` -> ข้อมูลสภาพแวดล้อม
- `grids/master/[id]/power` -> ข้อมูลพลังงาน
- `grids/master/[id]/security` -> ข้อมูลความปลอดภัย

---

## 💻 Pro Firmware Logic (Multi-Tasking)
ใช้ระบบ **FreeRTOS** เพื่อให้การอ่านเซนเซอร์แต่ละประเภทไม่ขัดจังหวะกัน

```cpp
// ตัวอย่างโครงสร้างโค้ดแบบ Task-based
void TaskEnvironment(void *pvParameters) {
  for(;;) {
    // อ่าน BME280 และส่ง MQTT
    vTaskDelay(5000 / portTICK_PERIOD_MS);
  }
}

void TaskPower(void *pvParameters) {
  for(;;) {
    // อ่าน INA219 ตรวจสอบแบตเตอรี่
    vTaskDelay(1000 / portTICK_PERIOD_MS);
  }
}

void setup() {
  xTaskCreate(TaskEnvironment, "Env", 4096, NULL, 1, NULL);
  xTaskCreate(TaskPower, "Power", 2048, NULL, 1, NULL);
}
```

---

## 👑 ความแตกต่างจากรุ่น Starter
1. **Reliability**: รองรับระบบล่ม (Watchdog Timer) และการบันทึกข้อมูลแบบ Offline
2. **Speed**: ใช้ FreeRTOS ในการประมวลผลแทนการวน Loop ปกติ
3. **Data Security**: รองรับการเข้ารหัส TLS สำหรับการเชื่อมต่อ Cloud
4. **Expandability**: มีพอร์ต I2C/RS485 สำหรับต่อเซนเซอร์ภายนอกเพิ่มเติม
