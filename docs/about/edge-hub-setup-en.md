# Edge Hub Installation Guide (Local Gateway) 🏠

This guide is for users who want to turn a Raspberry Pi or a PC into an **Edge Gateway** to support **Hybrid (Offline) Mode**. This ensures your farm remains operational even without an internet connection.

---

## 📋 System Requirements

- **Hardware**: Raspberry Pi 4 (4GB RAM+ recommended) or Mini PC.
- **Operating System**: 
  - Raspberry Pi OS (64-bit) 
  - Ubuntu 22.04+ (For PC/Linux)
  - Windows 10/11 (For PC via Docker Desktop)
- **Internet**: Required only during installation and periodic license synchronization.

---

## 🛠️ Method 1: Installation via Docker (Recommended)

Using Docker is the easiest and most stable way to run the Edge Hub stack.

### 1. Install Docker on Linux / Raspberry Pi
Run the following commands in Terminal:
```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```
*Note: Logout and log back in for changes to take effect.*

### 2. Create Docker Compose File
Create a directory named `smartfarm-edge` and create a `docker-compose.yml` file:
```yaml
version: '3.8'
services:
  # MQTT Broker for local field communication
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

### 3. Start the Stack
```bash
docker-compose up -d
```

---

## 🛠️ Method 2: Manual Installation (For Developers)

### 1. Install MQTT Broker (Mosquitto)
```bash
sudo apt update
sudo apt install -y mosquitto mosquitto-clients
sudo systemctl enable mosquitto
```

### 2. Install Node.js
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

### 3. Clone and Run Edge Hub
```bash
git clone https://github.com/GridsMicro/smartfarm-edge-hub.git
cd smartfarm-edge-hub
npm install
npm start
```

---

## ⚙️ Device Configuration

Once the Edge Hub is running, your IoT devices (ESP32/Hardware) must be configured to point to the Raspberry Pi's IP address:

1. Update the code on your node:
```cpp
// Set this to your Raspberry Pi's local IP address
const char* local_mqtt_server = "192.168.1.100"; 
```
2. The board will now send data to the Local Edge Hub first.
3. Edge Hub stores data locally and syncs to the Cloud when internet is available.

---

## 🔍 Verification

- **Check Local MQTT**: `mosquitto_sub -h localhost -t "farm/telemetry"`
- **Monitor Logs**: `docker logs sf-edge-hub -f`

---

> **Note**: The Edge Hub must connect to the internet at least **once every 30 days** to perform a "Managed Heartbeat" license check with the main server.
