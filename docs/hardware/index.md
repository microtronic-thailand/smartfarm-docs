# 🛠️ แนะนำฮาร์ดแวร์ (Hardware Guides)

โครงการ Smart Farm & Smart Home ของเราถูกออกแบบมาให้มอดูลาร์ (Modular) โดยเริ่มจากโปรเจกต์ง่ายๆ แยกตามการใช้งาน ไปจนถึงระดับมืออาชีพที่รวบรวมทุกฟังก์ชันไว้ด้วยกัน

---

## 🌱 โครงการเริ่มต้นการเกษตร (Smart Farm Starter)
เน้นการทดสอบเซนเซอร์ทีละจุด เพื่อเรียนรู้การส่งค่าและการทำงานพื้นฐาน:
- [Soil Moisture Node](starter-farm-soil.md) - วัดความชื้นในดิน (Basic)
- [Air Quality Monitor](starter-farm-air.md) - ตรวจวัดคุณภาพอากาศและก๊าซพิษ
- [Water Level Management](starter-farm-water.md) - ตรววัดระดับน้ำในถังและควบคุมปั๊ม

---

## 🏠 โครงการเริ่มต้นบ้านอัจฉริยะ (Smart Home Starter)
เปลี่ยนบ้านธรรมดาให้เป็นบ้านที่ควบคุมได้ผ่าน Cloud:
- [Climate Control](starter-home-climate.md) - มอนิเตอร์อุณหภูมิและความชื้น พร้อมสั่งเปิด/ปิดไฟ
- [Security & Intrusion](starter-home-security.md) - ระบบเตือนการเคลื่อนไหวและการเปิดประตู

---

## ⌚ สุขภาพอัจฉริยะ (Smart Health)
- [Grids Life-Node (Wearable)](life-node-v1.md) - อุปกรณ์สวมใส่เพื่อวิเคราะห์สุขภาพด้วย AI Doctor

---

## ⚡ สำหรับระดับมืออาชีพ (Advanced Modules)
การรวมหลายฟังก์ชันไว้ในฮาร์ดแวร์ชุดเดียวเพื่อการใช้งานจริง:
- [📦 Multi-Sensor Mini Hub](starter-multi-sensor.md) - โมดูลพื้นฐานที่รวมหลายเซนเซอร์และมีหน้าจอ OLED
- [👑 Grids Master Hub (Pro)](pro-master-hub.md) - ระบบควบคุมรวมศูนย์ประสิทธิภาพสูง (High Performance Edge Gateway)
- [📡 Edge Hub Setup](esp32-emqx-cloud.md) - การเชื่อมต่อและสถาปัตยกรรมข้อมูลขั้นสูง

---

## 🔌 คู่มือการตั้งค่า (Setup Guides)
- [ESP32 Configuration](esp32-setup.md)
- [STM32 Support](stm32-getting-started.md)
- [Power & Off-Grid Solutions](offgrid-power.md)
