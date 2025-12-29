# SmartFarmIoT Arduino Library

To connect your hardware to the Smart Farm Platform, you **must** use the official `SmartFarmIoT` library. This library provides a standardized way to handle telemetry, commands, and security.

## Installation

1.  Download the library as a `.zip` file from our [GitHub repository](https://github.com/gridsmicro/smartfarm-iot-library).
2.  In Arduino IDE, go to `Sketch` -> `Include Library` -> `Add .ZIP Library...`.
3.  Select the downloaded file.

## Dependencies

The following libraries will be installed automatically or are required:

- `PubSubClient` (for MQTT)
- `ArduinoJson` (v6 or higher)
- `DHT sensor library`
- `SmartFarmSensors` (included in the bundle)

## Simple Example

```cpp
#include <SmartFarmIoT.h>

const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";
const char* deviceToken = "YOUR_DEVICE_TOKEN";

SmartFarmNode node(deviceToken);

void setup() {
  Serial.begin(115200);
  node.begin(ssid, password);
}

void loop() {
  node.loop();
  
  // Send telemetry every 10 seconds
  static unsigned long lastTime = 0;
  if (millis() - lastTime > 10000) {
    lastTime = millis();
    
    float temp = 25.5; // Replace with real sensor data
    float hum = 60.0;
    
    node.sendTelemetry("temperature", temp);
    node.sendTelemetry("humidity", hum);
  }
}
```

## Features

- **Automatic Reconnection**: Automatically handles WiFi and MQTT drops.
- **Secure Communication**: Uses unique device tokens for authentication.
- **Standardized Payloads**: Formats data into the platform's required JSON structure.
- **Over-the-Air (OTA)**: Built-in support for remote firmware updates (ESP32/ESP8266).
