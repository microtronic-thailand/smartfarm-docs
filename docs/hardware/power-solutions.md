# Off-Grid Power Solutions for Smart Farm IoT Devices

## Why Renewable Energy for Smart Farm?

เกษตรกรส่วนใหญ่มีพื้นที่ฟาร์มห่างจากแหล่งไฟฟ้า การใช้พลังงานทดแทนจึงเป็นทางเลือกที่ดีที่สุด:

✅ **ประหยัดค่าไฟ** - ไม่ต้องจ่ายค่าไฟรายเดือน  
✅ **ติดตั้งได้ทุกที่** - ไม่ต้องรอการไฟฟ้าเข้าพื้นที่  
✅ **เป็นมิตรกับสิ่งแวดล้อม** - ลดการปล่อย CO₂  
✅ **ทำงานได้ 24/7** - ใช้แบตเตอรี่สำรอง  

---

## Power Consumption of IoT Devices

### Typical Smart Farm Node:
- **ESP32**: 80-160 mA (Active), 5-10 mA (Deep Sleep)
- **STM32**: 30-50 mA (Active), 1-2 mA (Sleep)
- **DHT22 Sensor**: 1-1.5 mA
- **Relay Module**: 50-70 mA (when ON)
- **Total Average**: ~100-200 mA @ 5V = 0.5-1W

### Daily Energy Consumption:
```
Scenario 1: Always Active
100 mA × 24 hours = 2.4 Ah/day × 5V = 12 Wh/day

Scenario 2: Deep Sleep (Wake every 5 min)
Average 20 mA × 24 hours = 0.48 Ah/day × 5V = 2.4 Wh/day
```

**Recommendation**: Use Deep Sleep mode to save 80% energy!

---

## Solution 1: Solar Power System

### 🌞 Small Solar Setup (Single Node)

**Components:**
1. **Solar Panel**: 10W (6V) - ~฿200-300
2. **Charge Controller**: TP4056 or CN3791 - ~฿30-50
3. **Battery**: 18650 Li-ion 3.7V 3000mAh (2-3 cells) - ~฿100-150
4. **DC-DC Converter**: LM2596 (Step-down to 5V) - ~฿30
5. **Wires & Connectors** - ~฿50

**Total Cost**: ~฿400-600 (~$12-18 USD)

### Wiring Diagram:

```
Solar Panel (6V 10W)
    |
    v
[Charge Controller] ---> Battery (3.7V 6000mAh)
    |                        |
    +------------------------+
                |
                v
        [DC-DC Converter 5V]
                |
                v
            ESP32/STM32
```

---

## Solution 2: Wind Power System

### 💨 Small Wind Turbine Setup

**Components:**
1. **Mini Wind Turbine**: 100W 12V - ~฿1,500-2,500
2. **Charge Controller**: PWM 12V 10A - ~฿200-300
3. **Battery**: 12V 7Ah Lead-Acid or LiFePO4 - ~฿400-800
4. **DC-DC Converter**: Buck converter 12V to 5V - ~฿50

**Total Cost**: ~฿2,200-3,700 (~$65-110 USD)

### When to Use Wind Power?

✅ **Open areas** with consistent wind (>3 m/s average)  
✅ **Coastal regions** or hilltops  
✅ **Supplement to solar** during rainy season  
⚠️ **Not recommended** for enclosed farms or low-wind areas  

---

## Solution 3: Hybrid Solar + Wind

### 🌞💨 Best of Both Worlds

**Why Hybrid?**
- Solar works during day
- Wind works during night/rainy season
- Maximum uptime (99%+)

**Components:**
1. Solar Panel: 20W 12V - ~฿400
2. Wind Turbine: 100W 12V - ~฿2,000
3. Hybrid Charge Controller: 12V 20A - ~฿500
4. Battery: 12V 12Ah LiFePO4 - ~฿1,200
5. DC-DC Converter: 12V to 5V - ~฿50

**Total Cost**: ~฿4,200 (~$125 USD)

---

## Battery Selection Guide

### 1. **18650 Li-ion** (Best for small systems)
- **Voltage**: 3.7V
- **Capacity**: 2000-3500 mAh
- **Pros**: Cheap, high energy density
- **Cons**: Needs protection circuit

### 2. **Lead-Acid** (Budget option)
- **Voltage**: 12V
- **Capacity**: 7-20 Ah
- **Pros**: Very cheap, robust
- **Cons**: Heavy, lower lifespan

### 3. **LiFePO4** (Best for long-term)
- **Voltage**: 12.8V
- **Capacity**: 7-20 Ah
- **Pros**: Long life (2000+ cycles), safe
- **Cons**: More expensive

**Recommendation**: 
- **Small farm (1-2 nodes)**: 18650 Li-ion
- **Medium farm (3-5 nodes)**: 12V 12Ah LiFePO4
- **Large farm (10+ nodes)**: 12V 20Ah LiFePO4

---

## Power Management Tips

### 1. **Use Deep Sleep Mode**

**ESP32 Example:**
```cpp
#include <esp_sleep.h>

void setup() {
  // Read sensors
  // Send data via MQTT
  
  // Sleep for 5 minutes
  esp_sleep_enable_timer_wakeup(5 * 60 * 1000000); // microseconds
  esp_deep_sleep_start();
}
```

**Power Savings**: 80 mA → 5 mA = **94% reduction!**

---

## Monitoring Power System

### Add Voltage Monitoring to Your IoT Node

**Circuit:**
```
Battery (+) ---[10kΩ]---+---[10kΩ]--- GND
                        |
                     ADC Pin
```

**Code (ESP32):**
```cpp
float readBatteryVoltage() {
  int raw = analogRead(34); // ADC pin
  float voltage = (raw / 4095.0) * 3.3 * 2; // Voltage divider ratio
  return voltage;
}

void loop() {
  float batteryVoltage = readBatteryVoltage();
  
  if (batteryVoltage < 3.3) {
    Serial.println("WARNING: Low battery!");
    // Send alert via MQTT
  }
  
  // Send battery status to dashboard
  sendMQTT("battery_voltage", batteryVoltage);
}
```

---

## Installation Tips

### Solar Panel Placement:
1. **Face South** (in Northern Hemisphere) or **Face North** (in Southern Hemisphere)
2. **Tilt angle** = Your latitude (e.g., Bangkok ~13°)
3. **No shadows** from 10 AM - 2 PM
4. **Clean regularly** (dust reduces efficiency by 20-30%)

---

## Cost Comparison

| Solution | Initial Cost | Monthly Cost | Uptime | Best For |
|----------|-------------|--------------|--------|----------|
| **Grid Power** | ฿500-1,000 | ฿100-300 | 95% | Near power lines |
| **Solar Only** | ฿400-600 | ฿0 | 85-90% | Sunny areas |
| **Wind Only** | ฿2,200-3,700 | ฿0 | 70-80% | Windy areas |
| **Hybrid** | ฿4,200+ | ฿0 | 95-99% | Remote farms |

---

## Need Help?

If you have questions about power systems for your Smart Farm, contact us!

**LINE ID**: @smartfarm
