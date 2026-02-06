# acurite2mqtt-test
Acurite to MQTT for Home Assistant with Device Discovery and sensors nested under main gateway

# Acurite to Home Assistant Add-on (TESTING)

This Home Assistant add-on receives 433MHz signals from Acurite sensors using an RTL-SDR dongle and publishes them to MQTT with automatic Home Assistant discovery.

## Features

- **Automatic MQTT Discovery**: If you have the MQTT integration configured in Home Assistant, this add-on will automatically discover and use your MQTT broker settings
- **RTL-SDR Support**: Uses RTL-SDR USB dongle to receive 433MHz signals
- **Home Assistant Auto-Discovery**: Sensors automatically appear in Home Assistant without manual configuration
- **Whitelist Support**: Filter specific sensors by ID
- **Customizable Protocols**: Configure which RTL-433 protocols to listen for

## Installation

1. Add this repository to your Home Assistant add-on store
2. Install the "Acurite to Home Assistant_Modified" add-on
3. Configure the add-on (see Configuration section)
4. Start the add-on

## Configuration

### Automatic MQTT Discovery (Recommended)

If you have MQTT integration already set up in Home Assistant:

1. Leave `mqtt_host`, `mqtt_user`, and `mqtt_password` **empty** in the add-on configuration
2. The add-on will automatically discover and use your MQTT broker

### Manual MQTT Configuration

If auto-discovery doesn't work or you want to use a different MQTT broker:

```yaml
mqtt_host: "192.168.1.100"
mqtt_port: 1883
mqtt_user: "your_mqtt_user"
mqtt_password: "your_mqtt_password"
```

### Other Options

- **mqtt_topic**: Base MQTT topic (default: `rtl_433`)
- **mqtt_retain**: Retain MQTT messages (default: `true`)
- **protocol**: RTL-433 protocol filters (default: `-R 11 -R 40 -R 41 -R 55 -R 74` for Acurite sensors)
  - `-R 11`: Acurite 609TXC
  - `-R 40`: Acurite 592TXR
  - `-R 41`: Acurite 986
  - `-R 55`: Acurite 606TX
  - `-R 74`: Acurite tower sensor
- **whitelist_enable**: Enable sensor ID filtering (default: `false`)
- **whitelist**: Comma-separated list of sensor IDs to allow
- **expire_after**: Seconds before sensor is marked unavailable (0 = disabled)
- **units**: Unit system (`si` or `customary`)
- **discovery_prefix**: Home Assistant discovery prefix (default: `homeassistant`)
- **discovery_interval**: Seconds between discovery announcements (default: `600`)
- **auto_discovery**: Enable Home Assistant auto-discovery (default: `true`)
- **debug**: Enable debug logging (default: `false`)

## Hardware Requirements

- RTL-SDR USB dongle (RTL2832U chipset)
- Acurite 433MHz sensors (temperature, humidity, etc.)

## How It Works

1. The RTL-SDR dongle receives 433MHz radio signals from Acurite sensors
2. RTL-433 software decodes the signals
3. The Python script publishes sensor data to MQTT
4. Home Assistant MQTT Discovery automatically creates sensor entities
5. Your sensors appear in Home Assistant!

## Troubleshooting

### MQTT Connection Failed

- Check that MQTT integration is configured in Home Assistant
- Or manually configure `mqtt_host`, `mqtt_user`, and `mqtt_password`
- Verify MQTT broker is running and accessible

### No Sensors Appearing

- Check add-on logs for received signals
- Verify RTL-SDR dongle is connected (check USB devices in System tab)
- Ensure `auto_discovery` is set to `true`
- Check that your sensors are within range and transmitting

### Wrong Units

- Change `units` to `si` (Celsius/metric) or `customary` (Fahrenheit/imperial)

## Support

Report issues at: https://github.com/rob7419/rob7419_acurite2mqtt_addons/issues
