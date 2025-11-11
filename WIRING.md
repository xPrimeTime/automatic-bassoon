# Wiring Diagram

## Overview

This document provides detailed wiring instructions for the ESP8266 temperature-controlled heating system.

## Components Required

1. ESP8266 NodeMCU v3
2. 5x Dallas DS18B20 temperature sensors
3. 1x 4.7kΩ resistor (pull-up for Dallas sensors)
4. Relay module (5V or 3.3V compatible)
5. Shelly H&T Gen3 (S3SN-0U12A) - WiFi connected, no physical wiring to ESP
6. Jumper wires
7. Breadboard (optional, for prototyping)

## NodeMCU v3 Pinout Reference

```
                   +-----+
       3.3V        |     |        VIN (5V)
       GND         |     |        GND
       D0/GPIO16   |     |        D8/GPIO15
       D1/GPIO5    | USB |        D7/GPIO13
       D2/GPIO4    |     |        D6/GPIO12
       D3/GPIO0    +-----+        D5/GPIO14
       D4/GPIO2                   GND
       3.3V                       3.3V
```

## Connection Details

### 1. Dallas DS18B20 Temperature Sensors (One-Wire Bus)

All 5 Dallas sensors connect to the same one-wire bus:

```
DS18B20 Sensor (x5)           NodeMCU v3
┌─────────────┐               ┌──────────┐
│             │               │          │
│  VCC (Red)  ├───────────────┤ 3.3V     │
│             │               │          │
│  DATA (Yel) ├───────────────┤ D4/GPIO2 │
│             │      │        │          │
│  GND (Blk)  ├──────┼────────┤ GND      │
│             │      │        │          │
└─────────────┘      │        └──────────┘
                     │
                  ┌──┴──┐
                  │4.7kΩ│ Pull-up resistor
                  └──┬──┘
                     │
                  to 3.3V
```

**Connection Steps:**
1. Connect all VCC pins (red wires) together → NodeMCU 3.3V
2. Connect all DATA pins (yellow wires) together → NodeMCU D4 (GPIO2)
3. Connect all GND pins (black wires) together → NodeMCU GND
4. Add 4.7kΩ resistor between DATA line and 3.3V

**Notes:**
- The 4.7kΩ pull-up resistor is **required** for reliable one-wire communication
- All sensors share the same three wires (VCC, DATA, GND)
- Use good quality wires; long cable runs may require twisted pair or shielded cable
- Maximum recommended cable length: 10-20m per sensor

### 2. Relay Module for Heating Control

#### Option A: 5V Relay Module (Recommended)

```
Relay Module                  NodeMCU v3
┌────────────────┐           ┌──────────┐
│                │           │          │
│  VCC           ├───────────┤ VIN (5V) │
│                │           │          │
│  GND           ├───────────┤ GND      │
│                │           │          │
│  IN (Signal)   ├───────────┤ D1/GPIO5 │
│                │           │          │
└────────────────┘           └──────────┘
       ││
       ││  Relay contacts (NO/NC/COM)
       ││
       ▼▼
   To Heating System
```

#### Option B: 3.3V Relay Module

```
Relay Module                  NodeMCU v3
┌────────────────┐           ┌──────────┐
│                │           │          │
│  VCC           ├───────────┤ 3.3V     │
│                │           │          │
│  GND           ├───────────┤ GND      │
│                │           │          │
│  IN (Signal)   ├───────────┤ D1/GPIO5 │
│                │           │          │
└────────────────┘           └──────────┘
       ││
       ││  Relay contacts (NO/NC/COM)
       ││
       ▼▼
   To Heating System
```

**Relay Wiring to Heating System:**

```
Heating System Wiring:

Power Source (+) ────┐
                     │
                   [Heating Element]
                     │
                     └────► COM (Relay)

                            NO (Relay) ───► Return to Power (-)

                            NC (Relay) = Not connected
```

**Connection Notes:**
- **NO** = Normally Open (heating OFF when relay is OFF)
- **NC** = Normally Close (heating ON when relay is OFF) - **Not recommended**
- **COM** = Common (connects to one side of heating element)
- Use NO configuration for safety (heating off by default)
- Ensure relay is rated for your heating system's voltage and current

### 3. Shelly H&T Gen3 Setup

The Shelly H&T is a wireless device and requires no physical connection to the ESP8266.

**Setup Steps:**
1. Power on Shelly H&T (battery operated)
2. Connect to WiFi (same network as ESP8266)
3. Integrate with Home Assistant:
   - Settings → Devices & Services → Add Integration
   - Search for "Shelly"
   - Follow setup instructions
4. Note the temperature sensor entity ID (e.g., `sensor.shelly_handt_temperature`)
5. Update entity_id in `kotlowniaesp.yaml`

### 4. Power Supply

**Option A: USB Power**
```
5V USB Power Supply ──► USB Port (NodeMCU)
```

**Option B: External 5V Supply**
```
5V Power Supply (+) ──► VIN (NodeMCU)
5V Power Supply (-) ──► GND (NodeMCU)
```

**Power Requirements:**
- NodeMCU: ~80-170mA (WiFi active)
- Dallas sensors: ~1.5mA each (7.5mA total for 5 sensors)
- Relay module: ~15-70mA
- **Total: ~200-300mA** → 1A power supply recommended

## Complete Wiring Diagram

```
                              NodeMCU v3
                        ┌─────────────────────┐
                        │                     │
5V USB ─────────────────┤ USB/VIN             │
                        │                     │
                   ┌────┤ 3.3V                │
                   │    │                     │
                   │ ┌──┤ GND                 │
                   │ │  │                     │
                   │ │  │ D4/GPIO2 ├──────────┼──► Dallas Sensors DATA (x5)
                   │ │  │          │          │
                   │ │  │ D1/GPIO5 ├──────────┼──► Relay IN (Signal)
                   │ │  │          │          │
                   │ │  │ D0/GPIO16├──┐       │    Status LED (built-in)
                   │ │  │          │  │       │
                   │ │  └──────────┴──┼───────┘
                   │ │                │
                   │ │    4.7kΩ       │
                   │ └────[Pull-up]───┘
                   │
                   └──► VCC (All Dallas sensors)


                         Relay Module
                   ┌─────────────────────┐
    VIN (5V) ──────┤ VCC                 │
                   │                     │
    GND ───────────┤ GND                 │
                   │                     │
    D1/GPIO5 ──────┤ IN (Signal)         │
                   │                     │
                   │  COM ───┐           │
                   │         │ Contacts  │
                   │  NO ────┤           │
                   │         │           │
                   │  NC     │ (unused)  │
                   └─────────┼───────────┘
                             │
                        To Heating
                          System
```

## Testing Procedure

### 1. Initial Power-Up Test
1. Connect only ESP8266 to USB power (no relay connected yet)
2. Verify status LED blinks (WiFi connecting)
3. Check ESPHome logs for WiFi connection

### 2. Dallas Sensor Test
1. Connect Dallas sensors one at a time
2. Verify each sensor appears in Home Assistant
3. Check temperature readings are reasonable

### 3. Relay Test (No Load)
1. Connect relay module (no heating system yet)
2. Manually toggle `switch.heating_output` in Home Assistant
3. Listen for relay click and observe LED indicator

### 4. Shelly Integration Test
1. Verify Shelly temperature appears in ESP logs
2. Test thermostat by setting target temperature
3. Observe relay toggles based on temperature

### 5. Full System Test
1. Connect heating system to relay (appropriate voltage/current)
2. Set thermostat to test temperature
3. Monitor for at least one heating cycle (on → off)
4. Verify minimum on/off times are respected

## Safety Checklist

- [ ] All connections are secure and properly insulated
- [ ] Relay is rated for heating system voltage and current
- [ ] No exposed high-voltage connections
- [ ] Power supply provides adequate current
- [ ] Proper electrical isolation between low voltage (ESP) and high voltage (heating)
- [ ] Fuses/circuit breakers in place for heating system
- [ ] Temperature sensors not placed near direct heat source
- [ ] Emergency shutoff accessible
- [ ] System tested without load before connecting heating element
- [ ] Complies with local electrical codes

## Troubleshooting

### Dallas Sensors Not Detected
- Check 4.7kΩ pull-up resistor is connected
- Verify wiring: VCC to 3.3V, GND to GND, DATA to D4
- Test with single sensor first
- Check for loose connections

### Relay Not Switching
- Verify GPIO5 connection to relay IN pin
- Check relay power supply (VCC and GND)
- Test with multimeter: GPIO5 should output 3.3V when ON
- Try different GPIO pin if available

### Intermittent WiFi Connection
- Check power supply quality (should be stable)
- Reduce distance to WiFi router
- Avoid placing ESP near sources of interference
- Consider external antenna for NodeMCU

### Heating Cycles Too Rapidly
- Increase `heat_deadband` in configuration
- Increase minimum on/off times
- Verify temperature sensor placement (not too close to heating element)
- Check if temperature readings are stable

## Additional Resources

- ESPHome One-Wire: https://esphome.io/components/one_wire.html
- ESPHome Climate: https://esphome.io/components/climate/thermostat.html
- Dallas DS18B20 Datasheet: https://www.analog.com/media/en/technical-documentation/data-sheets/DS18B20.pdf
- NodeMCU Documentation: https://nodemcu.readthedocs.io/
