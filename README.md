# ESP8266 Temperature-Controlled Heating System

ESPHome configuration for controlling a heating system based on temperature readings from a Shelly H&T Gen3 device.

## Hardware

- **ESP8266 NodeMCU v3**
- **Shelly H&T Gen3 (S3SN-0U12A)** - WiFi temperature/humidity sensor
- **5x Dallas DS18B20** temperature sensors (already configured)
- **Heating control relay** connected to GPIO pin

## Features

### Temperature Control
- Reads temperature from Shelly H&T Gen3 via Home Assistant integration
- Controls heating output based on temperature thresholds
- Two control methods available:
  1. **Thermostat mode** (recommended) - Full climate control with presets
  2. **Simple automation** - Basic on/off control with hysteresis

### Additional Sensors
- 5 Dallas DS18B20 temperature sensors for local monitoring
- All sensors reported to Home Assistant

### Safety Features
- Minimum on/off times to prevent rapid cycling
- Configurable temperature thresholds
- Fallback WiFi hotspot if connection fails
- Status LED indicator

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

1. **Shelly sensor entity ID** (line ~61):
   ```yaml
   entity_id: sensor.shelly_handt_temperature  # Change to your actual entity ID
   ```

2. **GPIO pin for heating control** (line ~93):
   ```yaml
   pin: GPIO5  # D1 on NodeMCU - change if using different pin
   ```

#### GPIO Pin Reference for NodeMCU v3:
- D0 = GPIO16 (built-in LED)
- D1 = GPIO5
- D2 = GPIO4
- D3 = GPIO0
- D4 = GPIO2 (currently used for Dallas sensors)
- D5 = GPIO14
- D6 = GPIO12
- D7 = GPIO13
- D8 = GPIO15

3. **Temperature thresholds** (optional, can be changed in Home Assistant):
   ```yaml
   initial_value: 18.0  # Lower threshold (heating turns ON)
   initial_value: 22.0  # Upper threshold (heating turns OFF)
   ```

### 4. Choose Control Method

The configuration includes two control methods:

#### Method 1: Thermostat (Default - Recommended)
- Provides full climate control in Home Assistant
- Supports multiple presets (Home, Away, Sleep)
- Better protection against rapid cycling
- **Currently active in the configuration**

#### Method 2: Simple Automation (Alternative)
- Basic on/off control with hysteresis
- More straightforward logic
- To use this method:
  1. Comment out the entire `climate:` section (lines ~109-150)
  2. Uncomment the `interval:` section (lines ~153-171)

### 5. Upload to ESP8266

Using ESPHome:

```bash
esphome run kotlowniaesp.yaml
```

Or in Home Assistant:
1. Go to **ESPHome Dashboard**
2. Upload the `kotlowniaesp.yaml` file
3. Click **Install**
4. Choose installation method (OTA, USB, etc.)

### 6. Configure in Home Assistant

After successful upload:

1. The device should appear in Home Assistant automatically
2. Go to **Settings → Devices & Services → ESPHome**
3. Find "KotlowniaESP"
4. Configure the thermostat:
   - Set target temperature
   - Choose preset (Home/Away/Sleep)
   - Monitor heating status

## How It Works

### Thermostat Mode

1. ESP8266 imports temperature from Shelly H&T via Home Assistant API
2. Built-in thermostat compares current temp with target temperature
3. When temp < (target - 1°C), heating turns ON
4. When temp ≥ target, heating turns OFF
5. Minimum on/off times prevent rapid cycling (2 minutes each)

### Temperature Thresholds

- **Lower Threshold (18°C default)**: Heating turns ON when temp drops below this
- **Upper Threshold (22°C default)**: Heating turns OFF when temp rises above this
- Both thresholds can be adjusted from Home Assistant UI

## Monitoring

The system exposes these entities to Home Assistant:

### Sensors
- `sensor.cwu1_up_temperature` - Dallas sensor 1
- `sensor.temperature_2` through `sensor.temperature_5` - Dallas sensors 2-5
- `sensor.shelly_temperature` - Temperature from Shelly H&T

### Controls
- `climate.room_thermostat` - Main thermostat control
- `number.heating_lower_threshold` - Lower temperature setpoint
- `number.heating_upper_threshold` - Upper temperature setpoint

### Status
- `switch.heating_output` - Manual override for heating
- `binary_sensor.heating_active` - Shows if heating is currently ON

## Wiring

### Dallas Temperature Sensors
- VCC → 3.3V
- GND → GND
- Data → GPIO2 (D4) with 4.7kΩ pull-up resistor to 3.3V

### Heating Control Relay
- Control pin → GPIO5 (D1) or your chosen pin
- VCC → 5V (if relay requires 5V)
- GND → GND
- **Important**: Use appropriate relay module for your heating system voltage

## Troubleshooting

### Shelly temperature not updating
- Check that Shelly is online in Home Assistant
- Verify entity_id matches your Shelly sensor
- Check ESP8266 logs: `esphome logs kotlowniaesp.yaml`

### Heating not turning on/off
- Verify GPIO pin number matches your wiring
- Check relay module is powered correctly
- Test manual control via `switch.heating_output` in HA
- Review logs for temperature threshold messages

### WiFi connection issues
- Ensure WiFi credentials in secrets.yaml are correct
- Check if fallback hotspot appears (SSID: "Kotlowniaesp Fallback Hotspot")
- Verify ESP8266 is within WiFi range

### Rapid heating cycling
- Increase `heat_deadband` in climate section (currently 1.0°C)
- Increase `min_heating_off_time` and `min_heating_run_time`
- Check if temperature sensor is placed correctly (not near heating element)

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
