# API Reference

The Smart Farm Platform provide a RESTful API and MQTT interface for interacting with your farm data.

## MQTT Interface (Primary)

Hardware devices should use the MQTT interface for real-time telemetry and commands.

### Telemetry Topic
`smartfarm/telemetry/{device_token}`

**Payload format:**
```json
{
  "device_id": "YOUR_DEVICE_ID",
  "data": {
    "temperature": 25.5,
    "humidity": 60,
    "soil_moisture": 45
  },
  "timestamp": "2025-12-29T10:00:00Z"
}
```

### Command Topic
`smartfarm/commands/{device_token}`

**Platform will send commands in this format:**
```json
{
  "type": "control",
  "action": "relay_on",
  "pin": 4
}
```

## REST API (External Integration)

You can use our REST API to fetch data or trigger actions from external applications.

### 1. Get Latest Telemetry
`GET /api/v1/telemetry/{device_id}`

### 2. Get Historical Data
`GET /api/v1/telemetry/{device_id}/history?start=...&end=...`

### 3. Send Manual Command
`POST /api/v1/commands/{device_id}`

---

**Note**: All API requests require an `X-API-KEY` header for authentication.
