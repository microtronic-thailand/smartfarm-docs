# คู่มือการติดตั้ง Edge Hub (Local Gateway) 🏠

คู่มือนี้สำหรับผู้ใช้งานที่ต้องการเปลี่ยน Raspberry Pi หรือ PC ให้เป็น **Edge Gateway** เพื่อใช้ในโหมด **Hybrid (Offline)** ซึ่งจะช่วยให้ฟาร์มของคุณทำงานได้แม้ไม่มีอินเทอร์เน็ต และรองรับทั้งการทดสอบแบบ Local และการเชื่อมต่อ Online

---

## 📋 ความต้องการของระบบ (System Requirements)

- **อุปกรณ์**: Raspberry Pi 4 (แรม 4GB ขึ้นไปแนะนำ) หรือ Mini PC
- **ระบบปฏิบัติการ**: 
  - Raspberry Pi OS (64-bit) 
  - Ubuntu 22.04+ (แนะนำสำหรับ Server)
  - Windows 10/11 (ผ่าน **WSL2** และ **Docker Desktop**)
- **อินเทอร์เน็ต**: จำเป็นเฉพาะช่วงติดตั้งและซิงค์ข้อมูลไลเซนส์

---

## 🛠️ วิธีที่ 1: ติดตั้งผ่าน Docker (แนะนำ - ง่ายและเร็วที่สุด)

การใช้ Docker จะช่วยให้ระบบแยกส่วนจาก OS หลักและทำงานได้เสถียรที่สุด โดยเราแนะนำ **EMQX** สำหรับระบบที่ต้องการความแม่นยำสูง

### 1. ติดตั้ง Docker
**สำหรับ Linux / Ubuntu:**
```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```
*Logout แล้ว Login ใหม่เพื่อให้คำสั่งทำงาน*

**สำหรับ Windows:**
1. ติดตั้ง [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)
2. ตรวจสอบว่าเปิดใช้งาน **WSL2 Backend** ใน Settings แล้ว
3. ติดตั้ง Ubuntu จาก Microsoft Store (เพื่อใช้รันคำสั่ง Linux)

### 2. สร้างไฟล์ Docker Compose
สร้างโฟลเดอร์ชื่อ `smartfarm-iot` และสร้างไฟล์ `docker-compose.yml`:
```yaml
version: '3.8'
services:
  # MQTT Broker (EMQX - แนะนำสำหรับ Production/Enterprise)
  emqx:
    image: emqx:5.3.0
    container_name: sf-emqx
    ports:
      - "18083:18083" # Dashboard (admin/public)
      - "1883:1883"   # MQTT Port
      - "8083:8083"   # WebSocket
    restart: always

  # Edge Hub Core (Logic Engine)
  edge-hub:
    image: gridsmicro/smartfarm-edge-hub:latest
    container_name: sf-edge-hub
    environment:
      - PLATFORM_TOKEN=YOUR_DEVICE_TOKEN
      - MQTT_BROKER_URL=mqtt://emqx:1883
    depends_on:
      - emqx
    restart: always
```

### 3. เริ่มทำงาน
```bash
docker-compose up -d
```

---

## 🌐 ทางเลือก: การใช้ Online MQTT Broker

หากคุณไม่ต้องการติดตั้งเครื่อง Server เอง สามารถเลือกใช้บริการ Online Broker เพื่อทดสอบระบบได้ทันที:

1. **EMQX Cloud** (แนะนำ): มี Dashboard สวยงามและตั้งค่าง่าย
2. **HiveMQ Cloud**: ฟรีสำหรับอุปกรณ์จำนวนน้อย
3. **Mosquitto Test**: `test.mosquitto.org` (ไม่แนะนำสำหรับข้อมูลสำคัญ)

**การตั้งค่าใน .env.local:**
```env
MQTT_BROKER_URL=mqtt://your-online-broker-url:1883
```

---

## 🖥️ คู่มือสำหรับผู้ใช้งาน Windows (WSL2)

สำหรับผู้ที่ใช้ Windows และต้องการจำลองสภาพแวดล้อมเหมือน Ubuntu (Local Testing):

1. **ติดตั้ง WSL2**: เปิด PowerShell (Admin) แล้วพิมพ์ `wsl --install -d Ubuntu`
2. **Setup Docker**: ใน Docker Desktop Settings -> Resources -> WSL Integration ให้ติ๊กถูกที่ `Ubuntu`
3. **การเข้าถึง Dashboard**: 
   - EMQX Dashboard จะเข้าได้ทาง `http://localhost:18083`
   - IP ของเครื่องที่บอร์ด ESP32 จะต่อเข้ามาคือ IP ของเครื่อง Windows (รัน `ipconfig` เพื่อดู IPv4)
4. **ความปลอดภัย**: อย่าลืมเปิด Firewall พอร์ต 1883 ใน Windows เพื่อให้บอร์ดส่งข้อมูลเข้ามาได้

---

## ⚙️ การตั้งค่าอุปกรณ์ (Device Configuration)

เมื่อ Edge Hub เริ่มทำงานแล้ว บอร์ด ESP32 ของคุณต้องถูกตั้งค่าให้ชี้มาที่ IP ของ Server:

1. **Local**: ใส่ IP ของเครื่อง Ubuntu หรือ Windows ในเครือข่ายเดียวกัน
2. **Online**: ใส่ URL ของ Online Broker ที่คุณสมัครไว้

```cpp
// ตัวอย่างการตั้งค่าใน Library
SmartFarmIoT farm;
farm.begin("Device_ID", "mqtt://192.168.1.100"); 
```

---

## 🔍 การตรวจสอบสถานะ

- **EMQX Dashboard**: เข้าผ่าน Browser `http://<IP-ADDRESS>:18083` (User: `admin`, Pass: `public`)
- **ดู Log ของระบบ**: `docker logs -f sf-edge-hub`
- **ทดสอบส่งข้อมูล**: `mosquitto_pub -h localhost -t "farm/test" -m "hello"`

---

> **คำแนะนำ**: ในการทดสอบช่วงแรก แนะนำให้ใช้ **Local EMQX via Docker** (วิธีที่ 1) เพราะจะช่วยให้คุณเห็น Flow ข้อมูลได้ชัดเจนผ่าน Dashboard โดยไม่ต้องกังวลเรื่องอินเทอร์เน็ตครับ
