# คู่มือการติดตั้ง Edge Hub (Local Gateway) 🏠

คู่มือนี้สำหรับผู้ใช้งานที่ต้องการเปลี่ยน Raspberry Pi หรือ PC ให้เป็น **Edge Gateway** เพื่อใช้ในโหมด **Hybrid (Offline)** ซึ่งจะช่วยให้ฟาร์มของคุณทำงานได้แม้ไม่มีอินเทอร์เน็ต

---

## 📋 ความต้องการของระบบ (System Requirements)

- **อุปกรณ์**: Raspberry Pi 4 (แรม 4GB ขึ้นไปแนะนำ) หรือ Mini PC
- **ระบบปฏิบัติการ**: 
  - Raspberry Pi OS (64-bit) 
  - Ubuntu 22.04+ (สำหรับ PC/Linux)
  - Windows 10/11 (สำหรับ PC ผ่าน Docker Desktop)
- **อินเทอร์เน็ต**: จำเป็นเฉพาะช่วงติดตั้งและซิงค์ข้อมูลไลเซนส์

---

## 🛠️ วิธีที่ 1: ติดตั้งผ่าน Docker (แนะนำ - ง่ายและเร็วที่สุด)

การใช้ Docker จะช่วยให้ระบบแยกส่วนจาก OS หลักและทำงานได้เสถียรที่สุด

### 1. ติดตั้ง Docker บน Linux / Raspberry Pi
เปิด Terminal แล้วรันคำสั่ง:
```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```
*จากนั้นทำการ Logout แล้ว Login ใหม่*

### 2. สร้างไฟล์ Docker Compose
สร้างโฟลเดอร์ชื่อ `smartfarm-edge` และสร้างไฟล์ `docker-compose.yml`:
```yaml
version: '3.8'
services:
  # MQTT Broker สำหรับรับส่งข้อมูลในฟาร์ม
  mqtt-broker:
    image: eclipse-mosquitto:latest
    container_name: sf-mqtt
    ports:
      - "1883:1883"
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data

  # Edge Hub Core (Logic Engine)
  edge-hub:
    image: gridsmicro/smartfarm-edge-hub:latest
    container_name: sf-edge-hub
    environment:
      - PLATFORM_TOKEN=YOUR_DEVICE_TOKEN
      - CLOUD_URL=https://api.smartfarm.com
    depends_on:
      - mqtt-broker
    restart: always
```

### 3. เริ่มทำงาน
```bash
docker-compose up -d
```

---

## 🛠️ วิธีที่ 2: ติดตั้งแบบ Manual (สำหรับนักพัฒนา)

### 1. ติดตั้ง MQTT Broker (Mosquitto)
```bash
sudo apt update
sudo apt install -y mosquitto mosquitto-clients
sudo systemctl enable mosquitto
```

### 2. ติดตั้ง Node.js (สำหรับรัน Edge Hub Service)
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

### 3. ดาวน์โหลดและรัน Edge Hub
```bash
git clone https://github.com/GridsMicro/smartfarm-edge-hub.git
cd smartfarm-edge-hub
npm install
npm start
```

---

## ⚙️ การตั้งค่าอุปกรณ์ (Device Configuration)

เมื่อ Edge Hub เริ่มทำงานแล้ว บอร์ด ESP32/Hardware ของคุณต้องถูกตั้งค่าให้ชี้มาที่ IP ของ Raspberry Pi:

1. แก้ไขโค้ดในบอร์ดตัวลูก:
```cpp
// ชี้มาที่ IP ของ Raspberry Pi ในวงแลนเดียวกัน
const char* local_mqtt_server = "192.168.1.100"; 
```
2. บอร์ดจะทำการส่งข้อมูลเข้าสู่ Edge Hub ก่อน
3. Edge Hub จะเก็บข้อมูลไว้ในเครื่อง (Local Cache) และซิงค์ขึ้น Cloud เมื่อมีเน็ต

---

## 🔍 การตรวจสอบสถานะ

- **ตรวจสอบการเชื่อมต่อ**: `mosquitto_sub -h localhost -t "farm/telemetry"`
- **ดู Log ของระบบ**: `docker logs sf-edge-hub -f`

---

> **หมายเหตุ**: ระบบ Edge Hub จะต้องมีการเชื่อมต่ออินเทอร์เน็ตอย่างน้อย **ทุกๆ 30 วัน** เพื่อทำการยืนยันสิทธิ์การใช้งาน (License Check) กับเซิร์ฟเวอร์หลัก
