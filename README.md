# ESP8266 Multi-Zone Temperature-Controlled Heating System

ESPHome configuration for controlling a **4-zone heating system** based on temperature readings from Shelly H&T Gen3 devices or Dallas sensors.

## Hardware

- **ESP8266 NodeMCU v3**
- **Shelly H&T Gen3 (S3SN-0U12A)** - WiFi temperature/humidity sensor (or multiple for each zone)
- **5x Dallas DS18B20** temperature sensors (already configured)
- **4x Heating control relays** connected to GPIO pins (one per zone)

## Features

### Multi-Zone Control
- **4 independent heating zones** with separate thermostats
- Each zone has its own GPIO output for relay control
- Individual temperature control per zone
- All zones can use the same temperature sensor or different sensors

### Temperature Control
- Reads temperature from Shelly H&T Gen3 via Home Assistant integration
- Can use Dallas DS18B20 sensors as temperature inputs
- Full climate control with presets (Home, Away, Sleep) per zone
- Independent operation - zones don't interfere with each other

### Additional Sensors
- 5 Dallas DS18B20 temperature sensors for local monitoring
- All sensors reported to Home Assistant
- Easy to assign different sensors to different zones

### Safety Features
- Minimum on/off times to prevent rapid cycling (2 minutes)
- Independent safety timers per zone
- Fallback WiFi hotspot if connection fails
- Status LED indicator
- Restore mode (heating off after reboot)

## Setup Instructions

### 1. Prepare Secrets File

Copy the template and add your credentials:

```bash
cp secrets.yaml.template secrets.yaml
```

Edit `secrets.yaml` with your actual values:
- WiFi SSID and password
- API encryption key
- OTA password
- Fallback hotspot password

### 2. Configure Shelly H&T in Home Assistant

Make sure your Shelly H&T Gen3 is integrated in Home Assistant:

1. Go to **Settings → Devices & Services**
2. Add Shelly integration if not already added
3. Find your Shelly H&T device
4. Note the entity ID for the temperature sensor (e.g., `sensor.shelly_handt_temperature`)

### 3. Update Configuration

Edit `kotlowniaesp.yaml` and change these values:

#### Required Changes:

1. **Shelly sensor entity ID** (line ~75):
   ```yaml
   entity_id: sensor.termometr_jarek_temperature  # Change to your actual entity ID
   ```

2. **GPIO pins are pre-configured for 4 zones**:
   - Zone 1: GPIO5 (D1) - line 87
   - Zone 2: GPIO14 (D5) - line 95
   - Zone 3: GPIO12 (D6) - line 103
   - Zone 4: GPIO13 (D7) - line 111

   Change these if you're using different pins.

#### GPIO Pin Reference for NodeMCU v3:
- D0 = GPIO16 (built-in LED / status LED)
- D1 = GPIO5 (Zone 1 heating output)
- D2 = GPIO4 (currently used for Dallas sensors)
- D3 = GPIO0
- D4 = GPIO2 (Dallas one-wire bus)
- D5 = GPIO14 (Zone 2 heating output)
- D6 = GPIO12 (Zone 3 heating output)
- D7 = GPIO13 (Zone 4 heating output)
- D8 = GPIO15

3. **Configure different temperature sensors per zone** (optional):

   By default, all 4 zones use the same Shelly sensor. To use different sensors:

   ```yaml
   # Zone 1 (line ~153)
   sensor: shelly_temperature

   # Zone 2 (line ~189)
   sensor: temp2  # Use Dallas sensor #2

   # Zone 3 (line ~225)
   sensor: temp3  # Use Dallas sensor #3

   # Zone 4 (line ~261)
   sensor: temp_cwu1_up  # Use Dallas sensor #1
   ```

### 4. Thermostat Control

The configuration uses **thermostat mode** for all 4 zones:
- Full climate control in Home Assistant
- Supports multiple presets (Home, Away, Sleep) per zone
- Better protection against rapid cycling
- Independent operation for each zone
- Adjustable deadband (hysteresis) to prevent frequent switching

### 5. Upload to ESP8266

Using ESPHome:

```bash
esphome run kotlowniaesp.yaml
```

Or in Home Assistant:
1. Go to **ESPHome Dashboard**
2. Upload the `kotlowniaesp.yaml` file or paste the configuration
3. Click **Install**
4. Choose installation method (OTA, USB, etc.)

### 6. Configure in Home Assistant

After successful upload:

1. The device should appear in Home Assistant automatically
2. Go to **Settings → Devices & Services → ESPHome**
3. Find "KotlowniaESP"
4. You'll see 4 independent thermostats - configure each:
   - Set target temperature per zone
   - Choose preset (Home/Away/Sleep) per zone
   - Monitor heating status for each zone
   - Manually override any zone using the switch entities

## How It Works

### 4-Zone Independent Control

Each of the 4 zones operates independently:

1. ESP8266 imports temperature from configured sensor (Shelly H&T or Dallas)
2. Each thermostat compares its sensor temperature with its target temperature
3. When temp < (target - 1°C), that zone's heating turns ON
4. When temp ≥ target, that zone's heating turns OFF
5. Minimum on/off times prevent rapid cycling (2 minutes each) per zone
6. Zones can be ON/OFF simultaneously without interfering with each other

### Default Configuration

By default, all 4 zones use the same Shelly temperature sensor:
- Useful for testing all zones work correctly
- Can control heating in 4 different rooms based on one sensor
- Easy to change to different sensors later per zone

**Note:** If all zones use the same sensor and have different setpoints, they may compete. It's recommended to assign different temperature sensors to each zone for best results.

## Monitoring

The system exposes these entities to Home Assistant:

### Temperature Sensors
- `sensor.cwu1_up_temperature` - Dallas sensor 1
- `sensor.temperature_2` through `sensor.temperature_5` - Dallas sensors 2-5
- `sensor.shelly_temperature` - Temperature from Shelly H&T

### Climate Controls (Thermostats)
- `climate.zone_1_thermostat` - Zone 1 thermostat
- `climate.zone_2_thermostat` - Zone 2 thermostat
- `climate.zone_3_thermostat` - Zone 3 thermostat
- `climate.zone_4_thermostat` - Zone 4 thermostat

Each thermostat has:
- Target temperature control
- Preset modes (Home/Away/Sleep)
- Current temperature display
- Heating/Idle status

### Manual Control Switches
- `switch.zone_1_heating_output` - Manual override Zone 1
- `switch.zone_2_heating_output` - Manual override Zone 2
- `switch.zone_3_heating_output` - Manual override Zone 3
- `switch.zone_4_heating_output` - Manual override Zone 4

### Status Indicators
- `binary_sensor.zone_1_heating_active` - Zone 1 heating status
- `binary_sensor.zone_2_heating_active` - Zone 2 heating status
- `binary_sensor.zone_3_heating_active` - Zone 3 heating status
- `binary_sensor.zone_4_heating_active` - Zone 4 heating status

## Wiring

### Dallas Temperature Sensors (One-Wire Bus)
- VCC → 3.3V
- GND → GND
- Data → GPIO2 (D4) with 4.7kΩ pull-up resistor to 3.3V
- All 5 sensors share the same three wires

### 4-Zone Heating Control Relays

Each zone needs a relay module connected to its GPIO pin:

**Zone 1 Relay:**
- Control pin → GPIO5 (D1)
- VCC → 5V or 3.3V (depending on relay module)
- GND → GND

**Zone 2 Relay:**
- Control pin → GPIO14 (D5)
- VCC → 5V or 3.3V
- GND → GND

**Zone 3 Relay:**
- Control pin → GPIO12 (D6)
- VCC → 5V or 3.3V
- GND → GND

**Zone 4 Relay:**
- Control pin → GPIO13 (D7)
- VCC → 5V or 3.3V
- GND → GND

**Important**:
- Use appropriate relay modules rated for your heating system voltage and current
- All relays can share the same power (VCC/GND) lines
- Ensure proper electrical isolation between low voltage (ESP) and high voltage (heating)

See [WIRING.md](WIRING.md) for detailed wiring diagrams.

## Troubleshooting

### Shelly temperature not updating
- Check that Shelly is online in Home Assistant
- Verify entity_id matches your Shelly sensor (line ~75)
- Check ESP8266 logs: `esphome logs kotlowniaesp.yaml`
- Look for "Temperature received" debug messages

### Specific zone not turning on/off
- Verify GPIO pin number matches your wiring for that zone
- Check relay module is powered correctly
- Test manual control via `switch.zone_X_heating_output` in HA
- Check the zone's thermostat is not in "off" mode
- Review logs for that zone's heating action

### All zones not working
- Check ESP8266 is powered and connected to Home Assistant
- Verify WiFi connection is stable
- Check ESPHome logs for errors
- Verify relays are powered (VCC/GND connections)

### WiFi connection issues
- Ensure WiFi credentials in secrets.yaml are correct
- Check if fallback hotspot appears (SSID: "Kotlowniaesp Fallback Hotspot")
- Verify ESP8266 is within WiFi range
- Check router allows ESP8266 connection

### Rapid heating cycling in a zone
- Increase `heat_deadband` for that zone (currently 1.0°C)
- Increase `min_heating_off_time` and `min_heating_run_time` (currently 120s)
- Check if temperature sensor is placed correctly (not near heating element)
- Verify temperature readings are stable, not fluctuating

### Multiple zones conflict
- If all zones use the same sensor with different setpoints, they may compete
- Assign different temperature sensors to each zone
- Or set all zones to similar temperatures if using one sensor

## Safety Warnings

⚠️ **Important Safety Information:**

- Ensure proper electrical isolation between low voltage (ESP8266) and high voltage (heating system)
- Use appropriate relay rated for your heating system voltage and current
- Consider adding fuses and thermal protection
- Test thoroughly before deploying in production
- Never leave heating systems unattended during initial testing
- Comply with local electrical codes and regulations

## License

This configuration is provided as-is for educational purposes.

## Support

For ESPHome documentation: https://esphome.io/
For Home Assistant: https://www.home-assistant.io/
