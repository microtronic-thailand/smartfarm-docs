# คู่มือผู้ดูแลระบบ (Admin Guide) 🛠️

คู่มือนี้สำหรับวิศวกรและผู้ดูแลระบบ (System Administrator) ของ Grids Micro เพื่อใช้ในการบริหารจัดการโครงสร้างพื้นฐานของ Smart Farm IoT Platform

---

## 🏗️ โครงสร้างระบบ (System Overview)

แพลตฟอร์มของเราประกอบด้วย 3 เลเยอร์หลัก:
1.  **Cloud Layer (Next.js + Supabase)**: จัดการข้อมูลรวมศูนย์, การยืนยันตัวตน, และการแสดงผลผ่านเว็บ
2.  **Edge Layer (Raspberry Pi/PC)**: ทำหน้าที่เป็น Local Gateway (MQTT Broker + Logic Engine)
3.  **Device Layer (ESP32/Hardware)**: เซนเซอร์และอุปกรณ์ควบคุมหน้างาน

---

## 🔑 หน้าที่ของผู้ดูแลระบบ

### 1. การจัดการอุปกรณ์ (Device Provisioning)
- การออกรหัส **Device Token** สำหรับบอร์ดใหม่
- การแมป ID ของอุปกรณ์เข้ากับบัญชีผู้ใช้
- การตรวจสอบสถานะ Heartbeat ของอุปกรณ์ในระบบ

### 2. การดูแล Edge Hub
- บำรุงรักษาความปลอดภัยของระบบปฏิบัติการ (OS Patching)
- ตรวจสอบปริมาณการใช้ทรัพยากร (CPU/RAM/Disk) ของ Raspberry Pi
- การอัปเดตซอฟต์แวร์ Edge Hub ผ่าน Docker หรือระบบ OTA (ในอนาคต)

---

## 📚 ข้อมูลอ้างอิงทางเทคนิค (Technical Reference)

### พอร์ตที่สำคัญ (Important Ports)
| พอร์ต | โปรโตคอล | คำอธิบาย |
| :--- | :--- | :--- |
| 1883 | MQTT | สำหรับการรับส่งข้อมูลภายใน (Local Network) |
| 8883 | MQTTS | สำหรับการซิงค์ข้อมูลขึ้น Cloud (Secure) |
| 5432 | Postgres | การเชื่อมต่อฐานข้อมูล (Supabase) |
| 80/443 | HTTP/S | เว็บ Dashboard และ API |

### หัวข้อ MQTT (Topic Structure)
- `farm/telemetry/[device_id]`: ข้อมูลเซนเซอร์ (Publish)
- `farm/status/[device_id]`: สถานะเชื่อมต่อ (LWT)
- `farm/command/[device_id]`: คำสั่งจากระบบ (Subscribe)

---

## 🚑 การวินิจฉัยปัญหาขั้นสูง (Advanced Diagnostics)

### การตรวจสอบสถานะ Docker
```bash
docker ps                # ดู Container ที่กำลังรัน
docker stats             # ดูการใช้ทรัพยากร
docker-compose logs -f   # ดู Log แบบ Real-time
```

### การทดสอบ MQTT ด้วย Command Line
```bash
# ทดสอบฟังข้อมูลเซนเซอร์
mosquitto_sub -h localhost -t "farm/telemetry/#" -v
```

---

## 💾 แผนการสำรองและกู้คืนระบบ (Backup & Recovery Plan)

เนื่องจาก Edge Hub ทำงานบน SD Card (สำหรับ Raspberry Pi) ซึ่งมีโอกาสเสียได้ง่าย ผู้ดูแลระบบต้องปฏิบัติตามแผนดังนี้:

### 1. สิ่งที่ต้องสำรองข้อมูล (What to Backup)
- **ไฟล์คอมฟิก**: โฟลเดอร์ `sf-edge/mosquitto/config` และไฟล์ `docker-compose.yml`
- **ฐานข้อมูลท้องถิ่น**: โฟลเดอร์ `sf-edge/mosquitto/data` (หากมีการเก็บข้อความค้างไว้)
- **ค่าตัวแปรสภาพแวดล้อม**: ไฟล์ `.env` (ถ้ามี) ที่เก็บรหัสผ่านและ Token

### 2. กลยุทธ์การสำรองข้อมูล (Backup Strategy)
- **Daily Snapshot**: ใช้สคริปต์ตั้งเวลา (Cron Job) เพื่อบีบอัดโฟลเดอร์ `sf-edge` ทั้งหมดเป็นไฟล์ `.tar.gz`
- **External Storage**: ควรบันทึกไฟล์สำรองไว้ใน USB Drive แยกต่างหาก หรืออัปโหลดขึ้น Cloud Storage (เช่น Google Drive/S3) โดยอัตโนมัติ

### 3. ขั้นตอนการกู้คืนระบบ (Recovery Steps)
หากเครื่อง Edge Hub เดิมเสียหายจนใช้งานไม่ได้:
1.  เตรียมเครื่องใหม่ที่มี OS และ Docker พร้อมใช้งาน
2.  ก๊อปปี้โฟลเดอร์ `sf-edge` จากไฟล์สำรองลงที่เครื่องใหม่
3.  ตรวจสอบสิทธิ์ของโฟลเดอร์ (Ownership)
4.  รันคำสั่ง `docker-compose up -d`
5.  ระบบจะกลับมาทำงานด้วยค่าเดิมทันทีโดยไม่ต้องตั้งค่าอุปกรณ์ตัวลูกใหม่

---

## 🛡️ การป้องกันความเสียหาย (Prevention)

### 1. การยืดอายุ SD Card
- **Industrial Grade SD Card**: แนะนำให้ลูกค้าใช้การ์ดเกรดอุตสาหกรรม (Endurance)
- **Minimize Logging**: ลดการเขียน Log ลง Disk หากไม่จำเป็นเพื่อให้เขียนข้อมูลน้อยลง

### 2. ระบบไฟสำรอง (UPS)
- Raspberry Pi มีความอ่อนไหวสูงต่อไฟตก/ไฟกระชาก แนะนำให้ติดตั้ง Power Shield หรือ UPS ขนาดเล็กเสมอ

---

> **ข้อควรระวัง**: ห้ามเปิดเผยไฟล์ `.env` หรือ `Service Role Key` ของ Supabase ให้กับบุคคลภายนอกเด็ดขาด เพราะจะเป็นการเปิดการเข้าถึงฐานข้อมูลทั้งหมด
