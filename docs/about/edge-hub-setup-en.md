# Edge Hub Installation Guide (Local Gateway) 🏠

This guide is for users who want to turn a Raspberry Pi or a PC into an **Edge Gateway** to support **Hybrid (Offline) Mode**. This ensures your farm remains operational even without an internet connection and supports both local testing and online cloud connectivity.

---

## 📋 System Requirements

- **Hardware**: Raspberry Pi 4 (4GB RAM+ recommended) or Mini PC.
- **Operating System**: 
  - Raspberry Pi OS (64-bit) 
  - Ubuntu 22.04+ (Recommended for Server usage)
  - Windows 10/11 (Via **WSL2** and **Docker Desktop**)
- **Internet**: Required only during installation and periodic license synchronization.

---

## 🛠️ Method 1: Installation via Docker (Recommended)

Using Docker is the easiest and most stable way to run the Edge Hub stack. We recommend **EMQX** as the high-performance MQTT broker for this platform.

### 1. Install Docker
**For Linux / Ubuntu:**
```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```
*Note: Logout and log back in for changes to take effect.*

**For Windows:**
1. Install [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)
2. Ensure **WSL2 Backend** is enabled in Settings.
3. Install Ubuntu from the Microsoft Store (to run Linux commands).

### 2. Create Docker Compose File
Create a directory named `smartfarm-iot` and create a `docker-compose.yml` file:
```yaml
version: '3.8'
services:
  # MQTT Broker (EMQX - Recommended for Production/Enterprise)
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

### 3. Start the Stack
```bash
docker-compose up -d
```

---

## 🌐 Alternative: Using Online MQTT Brokers

If you don't want to host your own server, you can use an Online Broker to test the system immediately:

1. **EMQX Cloud** (Recommended): Features a beautiful dashboard and easy setup.
2. **HiveMQ Cloud**: Free tier available for a small number of devices.
3. **Mosquitto Test**: `test.mosquitto.org` (Not recommended for sensitive data).

**Configuration in .env.local:**
```env
MQTT_BROKER_URL=mqtt://your-online-broker-url:1883
```

---

## 🖥️ Windows User Guide (WSL2)

For Windows users who want to simulate a Linux environment (Local Testing):

1. **Install WSL2**: Open PowerShell (Admin) and run `wsl --install -d Ubuntu`.
2. **Setup Docker**: In Docker Desktop Settings -> Resources -> WSL Integration, enable it for `Ubuntu`.
3. **Accessing Dashboard**: 
   - The EMQX Dashboard will be accessible at `http://localhost:18083`.
   - Your local IP for IoT devices (ESP32) is your Windows IP (run `ipconfig` in CMD to find IPv4).
4. **Firewall**: Ensure port 1883 is open in your Windows Firewall to allow incoming data from your hardware.

---

## ⚙️ Device Configuration

Once the Edge Hub is running, your IoT devices (ESP32/Hardware) must be configured to point to your server's IP:

1. **Local**: Use the IP of your Ubuntu/Windows machine on the same network.
2. **Online**: Use the URL of the Online Broker provider.

```cpp
// Library Configuration Example
SmartFarmIoT farm;
farm.begin("Device_ID", "mqtt://192.168.1.100"); 
```

---

## 🔍 Verification

- **EMQX Dashboard**: Access via browser at `http://<IP-ADDRESS>:18083` (User: `admin`, Pass: `public`).
- **Monitor Logs**: `docker logs -f sf-edge-hub`.
- **Test Message**: `mosquitto_pub -h localhost -t "farm/test" -m "hello"`.

---

> **Tip**: For initial development, we highly recommend using **Local EMQX via Docker** (Method 1). It provides excellent visibility into data flows through its dashboard without relying on internet stability.
