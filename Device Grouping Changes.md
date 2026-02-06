# Device Grouping Changes - RTL-433 Gateway

## What Changed

The Python script has been modified to create a **hierarchical device structure** in Home Assistant:

### Before (Flat Structure):
```
Entities:
- Acurite Tower 12345 Temperature
- Acurite Tower 12345 Humidity
- Acurite Tower 12345 Battery
- Acurite 592TXR 67890 Temperature
- Acurite 592TXR 67890 Battery
```

### After (Nested Structure):
```
📱 RTL-433 Gateway (Main Device)
  └─ Gateway Status (sensor)

📱 Acurite Tower 12345 (Sensor Device)
  ├─ Temperature
  ├─ Humidity
  └─ Battery

📱 Acurite 592TXR 67890 (Sensor Device)
  ├─ Temperature
  └─ Battery
```

## Key Changes in rtl_433_mqtt_hass.py

### 1. New Gateway Device Function
```python
def publish_gateway_device(mqttc):
    """Publish the main RTL-433 Gateway device that will group all sensors."""
```

This creates a single "RTL-433 Gateway" device that represents your RTL-SDR receiver/add-on.

**Gateway Device Details:**
- **Name**: RTL-433 Gateway
- **Model**: Acurite2MQTT
- **Manufacturer**: RTL-SDR
- **Identifier**: `rtl433_gateway`
- **Status Sensor**: Shows online/offline status

### 2. Modified Sensor Device Linking
```python
sensor_device = {
    "identifiers": [instance],
    "name": f"{model.replace('-', ' ')} {instance}",
    "model": model_name,
    "manufacturer": manufacturer,
    "via_device": "rtl433_gateway"  # ← Links sensor to gateway
}
```

**The magic happens here:** `"via_device": "rtl433_gateway"`

This tells Home Assistant that each sensor device (e.g., "Acurite Tower 12345") is connected **through** the RTL-433 Gateway.

### 3. Automatic Gateway Creation
```python
if (auto_discovery == True):
    # Publish gateway device first (only happens once)
    publish_gateway_device(mqttc)
```

The gateway device is created automatically when the first sensor is discovered.

## Benefits

### 1. **Better Organization**
- All your 433MHz sensors are grouped under one gateway
- Easy to see which sensors are connected to which receiver
- Cleaner device list in Home Assistant

### 2. **Visual Hierarchy**
In Home Assistant's Devices page, you'll see:
- RTL-433 Gateway (the hub)
  - Connected sensors listed underneath

### 3. **Gateway Status Monitoring**
- The gateway has its own status sensor
- Shows "online" when add-on is running
- Shows "offline" when add-on stops

### 4. **Easier Troubleshooting**
- Click on the gateway device to see all connected sensors at once
- Quickly identify if sensors are missing or offline
- See signal strength and diagnostics for each sensor

## How It Appears in Home Assistant

### Devices Page
```
┌─────────────────────────────────────┐
│ RTL-433 Gateway                     │
│ RTL-SDR • Acurite2MQTT             │
│ 1 entity • 3 devices via this      │
└─────────────────────────────────────┘
  ├─ Acurite Tower 12345
  │  └─ 4 entities
  ├─ Acurite 592TXR 67890
  │  └─ 3 entities
  └─ Acurite 606TX 11111
     └─ 2 entities
```

### Individual Sensor Device
```
┌─────────────────────────────────────┐
│ Acurite Tower 12345                 │
│ Acurite • Tower                     │
│ Via: RTL-433 Gateway               │
│                                     │
│ Entities:                           │
│ • Temperature      72.5°F          │
│ • Humidity         65%             │
│ • Battery          100%            │
│ • Last Seen        2 min ago       │
└─────────────────────────────────────┘
```

## Implementation Details

### Gateway Device Properties
- **Identifier**: `rtl433_gateway` (unique ID)
- **Name**: RTL-433 Gateway
- **Model**: Acurite2MQTT
- **Manufacturer**: RTL-SDR
- **Software Version**: 0.3.27
- **Status Topic**: `rtl_433/status`

### Sensor Device Properties
Each discovered sensor creates its own device:
- **Identifier**: Sensor ID (e.g., `12345`)
- **Name**: Model + ID (e.g., "Acurite Tower 12345")
- **Model**: From sensor data
- **Manufacturer**: Extracted from model name
- **Via Device**: `rtl433_gateway` ← Links to gateway

### Entity Organization
Each sensor's entities (temperature, humidity, battery, etc.) are automatically grouped under their parent sensor device.

## Backwards Compatibility

**Good news:** This change is fully backwards compatible!

- Existing sensors will be re-organized under the gateway
- Entity IDs remain the same
- Automations and dashboards continue to work
- No manual reconfiguration needed

## Testing the Changes

1. **Stop the add-on**
2. **Update the `rtl_433_mqtt_hass.py` file** with the new version
3. **Restart the add-on**
4. **Go to Settings → Devices & Services → MQTT**
5. **Look for "RTL-433 Gateway" device**
6. **Click on it to see all connected sensors**

## Customization Options

### Change Gateway Name
Edit line in `publish_gateway_device()`:
```python
"name": "RTL-433 Gateway",  # Change to your preferred name
```

### Change Gateway Model
```python
"model": "Acurite2MQTT",  # Change to describe your setup
```

### Add Gateway Icon
The gateway status sensor has icon:
```python
"icon": "mdi:radio-tower"  # Can change to mdi:antenna, mdi:broadcast, etc.
```

## Troubleshooting

### Gateway doesn't appear
- Check that `auto_discovery` is set to `true`
- Verify at least one sensor has been received
- Check add-on logs for "Published RTL-433 Gateway device" message

### Sensors not linked to gateway
- Ensure you're using the updated Python script
- Check that `via_device` is in the config
- Restart Home Assistant if devices seem cached

### Duplicate devices
- Delete old devices from MQTT integration
- Restart add-on to republish with new structure

## Advanced: Multiple Gateways

If you have multiple RTL-SDR receivers, you can run multiple instances with different gateway identifiers:

```python
# In publish_gateway_device(), change:
"identifiers": ["rtl433_gateway_1"],  # For receiver #1
"identifiers": ["rtl433_gateway_2"],  # For receiver #2
```

And update the via_device accordingly in each instance.
