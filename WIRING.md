# Wiring Diagram - 4-Zone System

## Overview

This document provides detailed wiring instructions for the ESP8266 **4-zone** temperature-controlled heating system.

## Components Required

1. ESP8266 NodeMCU v3
2. 5x Dallas DS18B20 temperature sensors
3. 1x 4.7kΩ resistor (pull-up for Dallas sensors)
4. **4x Relay modules** (5V or 3.3V compatible) - one per zone
5. Shelly H&T Gen3 (S3SN-0U12A) - WiFi connected, no physical wiring to ESP
6. Jumper wires
7. Breadboard (optional, for prototyping)
8. Power supply capable of powering ESP + 4 relays (~1-2A recommended)

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

### 2. Four Relay Modules for 4-Zone Heating Control

Each zone has its own relay connected to a different GPIO pin.

#### Option A: 5V Relay Modules (Recommended)

All 4 relays can share the same VCC and GND lines from NodeMCU:

```
Zone 1 Relay                 NodeMCU v3              Zone 2 Relay
┌──────────────┐            ┌──────────┐            ┌──────────────┐
│              │            │          │            │              │
│  VCC  ───────┼────────────┤ VIN (5V) ├────────────┼───── VCC     │
│              │            │          │            │              │
│  GND  ───────┼────────────┤ GND      ├────────────┼───── GND     │
│              │            │          │            │              │
│  IN (Signal) ├────────────┤ D1/GPIO5 │            │              │
│              │            │          │            │              │
└──────────────┘            │          │            │  IN (Signal) ├────┐
      ││                    │          │            │              │    │
      ││ Zone 1 Heating     │          │            └──────────────┘    │
      ▼▼                    │          │                  ││            │
                            │          │                  ││ Zone 2     │
Zone 3 Relay                │          │                  ▼▼            │
┌──────────────┐            │          │                                │
│              │            │          │            Zone 4 Relay        │
│  VCC  ───────┼────────────┤ (shared) │            ┌──────────────┐   │
│              │            │          │            │              │   │
│  GND  ───────┼────────────┤ (shared) │            │  VCC  (shared)   │
│              │            │          │            │              │   │
│  IN (Signal) ├────────────┤ D6/GPIO12│            │  GND  (shared)   │
│              │            │          │            │              │   │
└──────────────┘            │          │            │  IN (Signal) ├───┤
      ││                    │          │            │              │   │
      ││ Zone 3 Heating     │          │            └──────────────┘   │
      ▼▼                    └──────────┘                  ││            │
                                 │                        ││ Zone 4     │
                                 │                        ▼▼            │
                                 │                                      │
                            D5/GPIO14 ──────────────────────────────────┘
                            D7/GPIO13 ────────────────────────────────────┐
                                                                          │
                                                                          └─── Zone 4 IN
```

**Simplified Connection Table:**

| Zone | GPIO Pin | NodeMCU Pin | Relay Signal Pin |
|------|----------|-------------|------------------|
| 1    | GPIO5    | D1          | IN (Relay 1)     |
| 2    | GPIO14   | D5          | IN (Relay 2)     |
| 3    | GPIO12   | D6          | IN (Relay 3)     |
| 4    | GPIO13   | D7          | IN (Relay 4)     |

**All relays share:**
- VCC → NodeMCU VIN (5V)
- GND → NodeMCU GND

#### Option B: 3.3V Relay Modules

Same wiring as Option A, but connect all VCC pins to NodeMCU 3.3V instead of VIN:

```
All Relay VCC → NodeMCU 3.3V
All Relay GND → NodeMCU GND
Zone 1 IN → D1/GPIO5
Zone 2 IN → D5/GPIO14
Zone 3 IN → D6/GPIO12
Zone 4 IN → D7/GPIO13
```

**Relay Wiring to Heating Systems:**

Each relay controls its own heating zone independently. Wire each relay to its respective heating element:

```
Zone 1 Heating System:              Zone 2 Heating System:
Power (+) ────┐                     Power (+) ────┐
              │                                   │
        [Zone 1 Heater]                     [Zone 2 Heater]
              │                                   │
              └──► COM (Relay 1)                  └──► COM (Relay 2)
                   NO ──► Power (-)                    NO ──► Power (-)

Zone 3 Heating System:              Zone 4 Heating System:
Power (+) ────┐                     Power (+) ────┐
              │                                   │
        [Zone 3 Heater]                     [Zone 4 Heater]
              │                                   │
              └──► COM (Relay 3)                  └──► COM (Relay 4)
                   NO ──► Power (-)                    NO ──► Power (-)
```

**Connection Notes:**
- **NO** = Normally Open (heating OFF when relay is OFF) - **Use this for safety**
- **NC** = Normally Closed (heating ON when relay is OFF) - **Not recommended**
- **COM** = Common (connects to one side of heating element)
- Each zone can have different voltage/current ratings
- Each relay must be rated for its specific heating system's voltage and current
- All 4 zones operate independently

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

**Option A: USB Power (For light loads)**
```
5V USB Power Supply ──► USB Port (NodeMCU)
```

**Option B: External 5V Supply (Recommended for 4 relays)**
```
5V Power Supply (+) ──► VIN (NodeMCU)
5V Power Supply (-) ──► GND (NodeMCU)
```

**Power Requirements:**
- NodeMCU: ~80-170mA (WiFi active)
- Dallas sensors: ~1.5mA each (7.5mA total for 5 sensors)
- **Relay modules: ~15-70mA each × 4 = 60-280mA**
- **Total: ~150-460mA** → **1-2A power supply recommended**

**Note:** If using high-current relays or all 4 zones simultaneously, use a 2A power supply for stability.

## Complete Wiring Diagram - 4-Zone System

```
                              NodeMCU v3
                        ┌─────────────────────┐
                        │                     │
5V Power Supply ────────┤ VIN (5V)            ├───┐ VCC (shared by 4 relays)
    (1-2A)              │                     │   │
         │              │                     │   │
         └──────────────┤ GND                 ├───┼─┐ GND (shared by 4 relays)
                        │                     │   │ │
                   ┌────┤ 3.3V                │   │ │
                   │    │                     │   │ │
                   │    │ D4/GPIO2 ├──────────┼───┼─┼──► Dallas Sensors DATA (x5)
                   │    │          │          │   │ │            + 4.7kΩ pull-up
                   │    │ D1/GPIO5 ├──────────┼───┼─┼──► Zone 1 Relay IN
                   │    │          │          │   │ │
                   │    │ D5/GPIO14├──────────┼───┼─┼──► Zone 2 Relay IN
                   │    │          │          │   │ │
                   │    │ D6/GPIO12├──────────┼───┼─┼──► Zone 3 Relay IN
                   │    │          │          │   │ │
                   │    │ D7/GPIO13├──────────┼───┼─┼──► Zone 4 Relay IN
                   │    │          │          │   │ │
                   │    │ D0/GPIO16│ (Status LED)  │ │
                   │    │          │          │   │ │
                   │    └──────────┴──────────┘   │ │
                   │                               │ │
                   └──► VCC (All Dallas sensors)   │ │
                                                   │ │
    Relay 1 (Zone 1)        Relay 2 (Zone 2)      │ │
    ┌──────────────┐        ┌──────────────┐      │ │
    │ VCC ◄────────┼────────┼──────────────┼──────┘ │
    │ GND ◄────────┼────────┼──────────────┼────────┘
    │ IN  ◄── D1   │        │ IN  ◄── D5   │
    │ COM/NO/NC    │        │ COM/NO/NC    │
    └──────┬───────┘        └──────┬───────┘
           │ Zone 1                │ Zone 2
           │ Heating               │ Heating
           ▼                       ▼

    Relay 3 (Zone 3)        Relay 4 (Zone 4)
    ┌──────────────┐        ┌──────────────┐
    │ VCC ◄────────┼────────┼──────────────┼──── (shared VCC)
    │ GND ◄────────┼────────┼──────────────┼──── (shared GND)
    │ IN  ◄── D6   │        │ IN  ◄── D7   │
    │ COM/NO/NC    │        │ COM/NO/NC    │
    └──────┬───────┘        └──────┬───────┘
           │ Zone 3                │ Zone 4
           │ Heating               │ Heating
           ▼                       ▼
```

**Connection Summary:**
- **Power:** 5V supply → VIN and GND
- **Dallas Sensors:** All 5 share GPIO2 (D4) with 4.7kΩ pull-up to 3.3V
- **4 Relays:** Share VCC/GND, each has individual signal pin
  - Zone 1: GPIO5 (D1)
  - Zone 2: GPIO14 (D5)
  - Zone 3: GPIO12 (D6)
  - Zone 4: GPIO13 (D7)
- **Status LED:** Built-in on GPIO16 (D0)

## Testing Procedure - 4-Zone System

### 1. Initial Power-Up Test
1. Connect only ESP8266 to power supply (no relays connected yet)
2. Verify status LED blinks (WiFi connecting)
3. Check ESPHome logs for WiFi connection
4. Verify device appears in Home Assistant

### 2. Dallas Sensor Test
1. Connect Dallas sensors one at a time
2. Verify each sensor appears in Home Assistant
3. Check temperature readings are reasonable
4. All 5 sensors should show up with correct names

### 3. Relay Test (No Load) - Test Each Zone
1. Connect all 4 relay modules (no heating systems yet)
2. Test each zone individually:
   - Zone 1: Toggle `switch.zone_1_heating_output` in Home Assistant
   - Zone 2: Toggle `switch.zone_2_heating_output` in Home Assistant
   - Zone 3: Toggle `switch.zone_3_heating_output` in Home Assistant
   - Zone 4: Toggle `switch.zone_4_heating_output` in Home Assistant
3. Listen for relay click and observe LED indicator for each
4. Verify zones operate independently

### 4. Shelly Integration Test
1. Verify Shelly temperature appears in ESP logs
2. Test each thermostat by setting different target temperatures:
   - `climate.zone_1_thermostat`
   - `climate.zone_2_thermostat`
   - `climate.zone_3_thermostat`
   - `climate.zone_4_thermostat`
3. Observe correct relays toggle based on temperatures

### 5. Full System Test - One Zone at a Time
1. Connect heating system to Zone 1 relay first (appropriate voltage/current)
2. Set Zone 1 thermostat to test temperature
3. Monitor for at least one heating cycle (on → off)
4. Verify minimum on/off times are respected (2 minutes)
5. Repeat for remaining zones one at a time
6. Finally test all 4 zones simultaneously if needed

## Safety Checklist - 4-Zone System

- [ ] All connections are secure and properly insulated
- [ ] **Each relay is rated for its specific heating system voltage and current**
- [ ] No exposed high-voltage connections on any zone
- [ ] Power supply provides adequate current (1-2A for ESP + 4 relays)
- [ ] Proper electrical isolation between low voltage (ESP) and high voltage (heating) for all zones
- [ ] Fuses/circuit breakers in place for each heating zone
- [ ] Temperature sensors not placed near direct heat sources
- [ ] Emergency shutoff accessible for all zones
- [ ] Each zone tested without load before connecting heating element
- [ ] All 4 zones can operate simultaneously without overloading power supply
- [ ] Complies with local electrical codes and regulations

## Troubleshooting - 4-Zone System

### Dallas Sensors Not Detected
- Check 4.7kΩ pull-up resistor is connected
- Verify wiring: VCC to 3.3V, GND to GND, DATA to D4
- Test with single sensor first
- Check for loose connections
- Ensure one-wire bus isn't too long (max 10-20m)

### Specific Zone Relay Not Switching
- Verify correct GPIO connection for that zone:
  - Zone 1: GPIO5 (D1)
  - Zone 2: GPIO14 (D5)
  - Zone 3: GPIO12 (D6)
  - Zone 4: GPIO13 (D7)
- Check that specific relay's power supply (VCC and GND)
- Test with multimeter: GPIO pin should output 3.3V when ON
- Test manual switch control in Home Assistant
- Swap relay module to verify if relay is faulty

### All Relays Not Working
- Check shared VCC/GND connections from NodeMCU
- Verify power supply has enough current (2A recommended)
- Check if ESP8266 is responding in Home Assistant
- Review ESPHome logs for errors

### Only Some Zones Working
- Check individual signal wire connections (D1, D5, D6, D7)
- Verify each relay has power (VCC/GND may be loose)
- Test non-working zones with manual switch control
- Check for GPIO pin conflicts in configuration

### Intermittent WiFi Connection
- Check power supply quality and current capacity (should be stable 1-2A)
- Reduce distance to WiFi router
- Avoid placing ESP near sources of interference
- Consider external antenna for NodeMCU
- Verify relays switching doesn't cause voltage drops

### Power Supply Issues
- If ESP resets when multiple relays activate: increase power supply amperage
- If status LED dims during relay activation: inadequate power supply
- Use dedicated 2A power supply for stable operation with 4 relays

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
