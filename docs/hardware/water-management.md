# Smart Water Management System

## Why Water Management Matters

น้ำคือทรัพยากรที่มีค่าที่สุดในการเกษตร การใช้น้ำอย่างชาญฉลาดไม่เพียงช่วยประหยัดต้นทุน แต่ยังช่วยรักษาระบบนิเวศ:

✅ **ประหยัดน้ำ 30-50%** - ลดค่าน้ำและพลังงานสูบน้ำ  
✅ **เพิ่มผลผลิต** - พืชได้รับน้ำพอดี ไม่มากไม่น้อย  
✅ **ป้องกันโรคพืช** - ลดความชื้นส่วนเกิน  
✅ **รักษาระบบนิเวศ** - ไม่สูบน้ำบาดาลเกินไป  

---

## Water Monitoring Sensors

### 1. **Soil Moisture Sensor** (เซนเซอร์ความชื้นดิน)

**Types:**

#### A. Capacitive Soil Moisture Sensor (แนะนำ)
- **Price**: ~฿50-80
- **Pros**: ไม่เป็นสนิม, ทนทาน, แม่นยำ
- **Cons**: แพงกว่าแบบ Resistive นิดหน่อย
- **Lifespan**: 2-3 ปี

**Wiring:**
```
Sensor → ESP32/STM32
VCC    → 3.3V
GND    → GND
AOUT   → GPIO34 (Analog)
```

**Code Example (ESP32):**
```cpp
#define SOIL_PIN 34

void setup() {
  Serial.begin(115200);
}

void loop() {
  int soilValue = analogRead(SOIL_PIN);
  
  // Convert to percentage (calibrate for your soil)
  int soilMoisture = map(soilValue, 4095, 1500, 0, 100);
  soilMoisture = constrain(soilMoisture, 0, 100);
  
  Serial.print("Soil Moisture: ");
  Serial.print(soilMoisture);
  Serial.println("%");
  
  // Auto-irrigation logic
  if (soilMoisture < 30) {
    Serial.println("🚨 Soil too dry! Turning ON pump...");
    digitalWrite(RELAY_PIN, HIGH);
  } else if (soilMoisture > 70) {
    Serial.println("✅ Soil wet enough. Turning OFF pump.");
    digitalWrite(RELAY_PIN, LOW);
  }
  
  delay(5000);
}
```

### 2. **Water Level Sensor** (เซนเซอร์ระดับน้ำ)

#### A. Ultrasonic Sensor (HC-SR04)
- **Price**: ~฿30-50
- **Range**: 2-400 cm
- **Use**: วัดระดับน้ำในถัง/บ่อ

**Code Example:**
```cpp
#define TRIG_PIN 5
#define ECHO_PIN 18

float getWaterLevel() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  
  long duration = pulseIn(ECHO_PIN, HIGH);
  float distance = duration * 0.034 / 2; // cm
  
  // Tank height = 100cm, sensor at top
  float waterLevel = 100 - distance;
  return waterLevel;
}
```

### 3. **Water Flow Sensor** (เซนเซอร์การไหลของน้ำ)

**Model**: YF-S201 (1/2")
- **Price**: ~฿80-120
- **Use**: วัดปริมาณน้ำที่ใช้ (ลิตร/นาที)

---

## Smart Irrigation System

### Basic Drip Irrigation Setup

**Components:**
1. **Water Pump**: 12V DC (~฿200-400)
2. **Relay Module**: 5V (~฿30)
3. **Drip Irrigation Kit**: (~฿300-500)
4. **Soil Moisture Sensor**: (~฿50)
5. **Water Tank**: 100-200L (~฿500-1,000)

**Total**: ~฿1,100-2,000

---

## Water Quality Monitoring

### pH Sensor (วัดความเป็นกรด-ด่าง)

**Model**: Analog pH Sensor
- **Price**: ~฿400-600
- **Range**: pH 0-14

### TDS Sensor (วัดแร่ธาตุในน้ำ)

**Model**: TDS Meter
- **Price**: ~฿200-400
- **Use**: วัดความเข้มข้นของปุ๋ย

---

## Ecosystem-Friendly Practices

### 1. **Biofilter for Wastewater** (กรองน้ำเสียด้วยพืช)

**Plants for biofilter:**
- ผักตบชวา (Water Hyacinth) - ดูดซับโลหะหนัก
- ต้นกก (Cattail) - กรองตะกอน
- ผักบุ้งน้ำ - ดูดซับไนโตรเจน

### 2. **Aquaponics Integration** (ปลูกพืชเลี้ยงปลา)

**Concept:**
```
ปลา → ของเสีย (แอมโมเนีย) → แบคทีเรีย → ไนเตรต → พืช → กรองน้ำ → กลับไปหาปลา
```

---

## Cost-Benefit Analysis

| Solution | Initial Cost | Monthly Cost | ROI |
|----------|--------------|--------------|-----|
| **Traditional** | ฿0 | ฿100+ | N/A |
| **Smart** | ฿1,500 | ฿50 | ~2.7 years |

---

## Need Help?

Questions about water management for your farm?

**LINE ID**: @smartfarm
