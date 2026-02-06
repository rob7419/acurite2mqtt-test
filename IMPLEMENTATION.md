# Implementation Guide: Adding MQTT Discovery to Acurite2MQTT Add-on

## Overview
This guide explains how to add automatic MQTT broker discovery to your Acurite2MQTT Home Assistant add-on.

## Changes Required

### 1. config.yaml Modifications

**Key additions:**
```json
"hassio_api": true,
"hassio_role": "default",
"auth_api": true,
"services": ["mqtt:want"],
"discovery": ["mqtt"],
```

**Schema changes:**
- Changed `mqtt_host`, `mqtt_user`, `mqtt_password` from required (`str`) to optional (`str?`, `password?`)
- This allows them to be empty when using auto-discovery
- Changed default values to empty strings

**What this does:**
- `services: ["mqtt:want"]` - Tells Home Assistant this add-on wants to use MQTT service
- `discovery: ["mqtt"]` - Enables MQTT service discovery
- `hassio_api: true` - Allows add-on to query Home Assistant services
- Optional fields - Users can leave MQTT settings blank for auto-discovery

### 2. entry.sh Modifications

**New discovery logic:**
```bash
# Check if MQTT discovery is available
if bashio::services.available "mqtt"; then
    MQTT_HOST=$(bashio::services "mqtt" "host")
    MQTT_PORT=$(bashio::services "mqtt" "port")
    MQTT_USERNAME=$(bashio::services "mqtt" "username")
    MQTT_PASSWORD=$(bashio::services "mqtt" "password")
else
    # Fallback to manual configuration
    MQTT_HOST="$(bashio::config 'mqtt_host')"
    # ... etc
fi
```

**What this does:**
- First checks if MQTT service is available via Home Assistant
- If found, automatically pulls MQTT connection details
- If not found, falls back to manual configuration
- Provides clear logging for debugging

### 3. How Users Will Use It

**With Auto-Discovery (Recommended):**
1. User has MQTT integration set up in Home Assistant
2. User installs your add-on
3. User leaves `mqtt_host`, `mqtt_user`, `mqtt_password` blank
4. Add-on automatically discovers and connects to MQTT
5. Sensors appear in Home Assistant automatically

**With Manual Configuration:**
1. User doesn't have MQTT integration (or wants different broker)
2. User manually fills in MQTT connection details
3. Add-on uses those settings instead

## Installation Steps

1. Replace your current `config.yaml` with the updated version
2. Replace your current `entry.sh` with the updated version
3. Increment version number (e.g., `0.3.27_rob7419`)
4. Rebuild and push your add-on
5. Update users via README about the new auto-discovery feature

## Testing

1. **Test Auto-Discovery:**
   - Configure MQTT integration in Home Assistant
   - Install add-on with blank MQTT settings
   - Verify it connects automatically
   - Check logs for "MQTT service discovered!" message

2. **Test Manual Configuration:**
   - Remove MQTT integration (or use different broker IP)
   - Configure MQTT settings manually in add-on
   - Verify it connects using manual settings
   - Check logs for "using manual configuration" message

3. **Test Fallback:**
   - Remove MQTT integration
   - Leave settings blank
   - Verify add-on shows error and doesn't start

## Benefits

- **Easier Setup**: Users don't need to duplicate MQTT configuration
- **Less Error-Prone**: No typos in hostnames/passwords
- **Better Integration**: Follows Home Assistant best practices
- **Backward Compatible**: Still works with manual configuration
- **User Friendly**: Automatically works when MQTT is already set up

## Troubleshooting

If discovery doesn't work:
1. Check that MQTT integration is installed and configured in Home Assistant
2. Check add-on logs for discovery messages
3. Verify `services: ["mqtt:want"]` is in config.yaml
4. Ensure Home Assistant Supervisor is up to date

## Next Steps

After implementing these changes, consider:
- Adding a configuration UI with discovery status
- Implementing USB device auto-detection for RTL-SDR
- Adding sensor entity customization options
- Creating automation templates for common use cases
