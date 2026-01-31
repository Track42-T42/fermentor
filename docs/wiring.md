# Wiring Guide - Fermentation Chamber Controller

This document details the complete wiring for the ESP32-based fermentation chamber controller.

## ⚠️ SAFETY WARNING

**This project involves mains voltage (120V/240V AC) which can be LETHAL.**

- Only proceed if you are qualified to work with mains voltage
- Use proper enclosures rated for mains voltage
- Install appropriate fuses and circuit breakers
- Ensure all equipment is properly grounded/earthed
- Use appliance-rated wire for all mains connections
- Have your work inspected by a qualified electrician
- Test all safety interlocks before operation

**Do not attempt this project if you are not experienced with electrical wiring.**

---

## ESP32 GPIO Pin Assignments

### Sensors

| GPIO | Function | Device | Notes |
|------|----------|--------|-------|
| GPIO21 | I2C SDA | AHT20 Temperature/Humidity | Pull-up resistor recommended (4.7kΩ) |
| GPIO22 | I2C SCL | AHT20 Temperature/Humidity | Pull-up resistor recommended (4.7kΩ) |
| GPIO4 | One-Wire | DS18B20 External Sensor 1 | 4.7kΩ pull-up to 3.3V required |
| GPIO33 | One-Wire | DS18B20 External Sensor 2 | 4.7kΩ pull-up to 3.3V required |
| GPIO32 | Digital Input | Magnetic Door Sensor | Internal pull-up enabled in software |

### Relay Outputs

| GPIO | Relay # | Controls | Load Type | Notes |
|------|---------|----------|-----------|-------|
| GPIO25 | Relay 1 | Interior Light | Incandescent/LED | Low current |
| GPIO26 | Relay 2 | Heating Mat | Resistive heater | Medium current |
| GPIO27 | Relay 3 | Mist Maker | Ultrasonic humidifier | Low current, 12V/24V DC |
| GPIO14 | Relay 4 | Circulation Fan | AC/DC fan | Medium current |
| GPIO23 | Relay 5 | Compressor (Cooling) | Refrigerator compressor | High current, inductive load |
| GPIO5 | Relay 6 | Turbo Heater | Resistive heater | ⚠️ **Strapping pin - see notes** |

### Important GPIO Notes

**GPIO5 (Turbo Heater):**
- This is a strapping pin on ESP32 and may cause boot issues
- The YAML includes a warning to move this to GPIO 16, 17, 18, 19, or 13 if possible
- If you experience boot problems, rewire to a non-strapping pin

**Safe GPIO pins for outputs:** 12, 13, 14, 16, 17, 18, 19, 23, 25, 26, 27, 32, 33

---

## Component Wiring Details

### 1. AHT20 Temperature/Humidity Sensor (I2C)

```
AHT20 Module        ESP32
─────────────────────────────
VCC (3.3V)    →    3.3V
GND           →    GND
SDA           →    GPIO21 (with 4.7kΩ pull-up to 3.3V)
SCL           →    GPIO22 (with 4.7kΩ pull-up to 3.3V)
```

**Notes:**
- I2C address: 0x38 (default for AHT20)
- Use short wires (< 30cm) or shielded cable for longer runs
- Some AHT20 modules have built-in pull-ups; external pull-ups may not be needed

### 2. DS18B20 Temperature Probes (One-Wire)

**External Sensor 1:**
```
DS18B20 #1          ESP32
─────────────────────────────
VCC (Red)     →    3.3V or 5V
GND (Black)   →    GND
Data (Yellow) →    GPIO4 + 4.7kΩ resistor to VCC
```

**External Sensor 2:**
```
DS18B20 #2          ESP32
─────────────────────────────
VCC (Red)     →    3.3V or 5V
GND (Black)   →    GND
Data (Yellow) →    GPIO33 + 4.7kΩ resistor to VCC
```

**Notes:**
- Each One-Wire bus needs its own 4.7kΩ pull-up resistor
- DS18B20 can be powered by 3.3V or 5V (5V recommended for long cables)
- Use waterproof probes for direct food contact
- Cable lengths up to 10m are supported with proper pull-ups

### 3. Magnetic Door Sensor

```
Door Sensor         ESP32
─────────────────────────────
Terminal 1    →    GPIO32
Terminal 2    →    GND
```

**Notes:**
- Internal pull-up resistor is enabled in software
- Sensor closes circuit when door is closed, opens when door opens
- Use normally-closed (NC) type for fail-safe operation
- Can use any standard magnetic reed switch

### 4. 6-Channel Relay Module

Most relay modules have optoisolated inputs and can be controlled directly from ESP32 GPIO pins.

**Typical Relay Module Wiring:**

```
Relay Module        ESP32
─────────────────────────────
VCC           →    5V (from external supply, NOT ESP32)
GND           →    GND (common ground)
IN1           →    GPIO25 (Relay 1 - Light)
IN2           →    GPIO26 (Relay 2 - Heating Mat)
IN3           →    GPIO27 (Relay 3 - Mist Maker)
IN4           →    GPIO14 (Relay 4 - Fan)
IN5           →    GPIO23 (Relay 5 - Compressor)
IN6           →    GPIO5  (Relay 6 - Turbo Heater)
```

**Power Supply for Relay Module:**
- Use a separate 5V power supply (NOT from ESP32's 5V pin)
- ESP32 can only supply ~500mA; relay module may draw more
- Share common ground between ESP32 and relay power supply

**Relay Configuration:**
- Check if your module is active-high or active-low
- The YAML config uses `inverted: false` (active-high)
- If your relays are active-low, change to `inverted: true` in the YAML

---

## Load Wiring (AC Mains Side)

### General Relay Contact Wiring

Each relay has three terminals:
- **COM** (Common)
- **NO** (Normally Open) - Recommended for safety
- **NC** (Normally Closed)

For safety, use **NO** contacts so loads are OFF when controller loses power.

### Relay Load Connections

**Relay 1 - Interior Light:**
```
Mains Hot → Relay COM
Relay NO → Light → Mains Neutral
```

**Relay 2 - Heating Mat:**
```
Mains Hot → Relay COM
Relay NO → Heating Mat → Mains Neutral
```

**Relay 3 - Mist Maker:**
```
DC Power + → Relay COM
Relay NO → Mist Maker + terminal
Mist Maker - terminal → DC Power -
```
*Note: Most ultrasonic mist makers run on 12V or 24V DC, not mains voltage*

**Relay 4 - Circulation Fan:**
```
Mains Hot → Relay COM (or DC+ for DC fans)
Relay NO → Fan → Mains Neutral (or DC-)
```

**Relay 5 - Compressor (Cooling):**
```
Mains Hot → Relay COM
Relay NO → Compressor Live terminal
Compressor Neutral → Mains Neutral
Compressor Ground → Mains Ground
```
**⚠️ Critical:** Compressors are inductive loads and should use relays rated for motor starting current

**Relay 6 - Turbo Heater:**
```
Mains Hot → Relay COM
Relay NO → Heater → Mains Neutral
```

### Recommended Relay Ratings

| Load | Minimum Relay Rating | Recommended |
|------|---------------------|-------------|
| Light | 5A @ 250V AC | 10A @ 250V AC |
| Heating Mat | 10A @ 250V AC | 15A @ 250V AC |
| Mist Maker | 2A @ 24V DC | 5A @ 24V DC |
| Fan | 5A @ 250V AC | 10A @ 250V AC |
| Compressor | 15A @ 250V AC | 20A @ 250V AC (motor-rated) |
| Turbo Heater | 10A @ 250V AC | 15A @ 250V AC |

---

## Power Supply

### ESP32 Power

**Option 1: USB Power (Development/Testing)**
- 5V @ 1A USB power adapter
- Simple and safe for development
- Relay module needs separate 5V supply

**Option 2: Dedicated 5V Supply (Production)**
- 5V @ 3A power supply (e.g., Mean Well HDR-15-5)
- Powers both ESP32 and relay module
- More reliable than USB for 24/7 operation

### Complete Power Distribution

```
5V Power Supply
├── ESP32 5V pin (or USB)
└── Relay Module VCC

Mains Supply (with breaker/fuse)
├── Relay 1 → Light
├── Relay 2 → Heating Mat
├── Relay 4 → Fan (if AC)
├── Relay 5 → Compressor
└── Relay 6 → Turbo Heater

12V/24V DC Supply (for mist maker)
└── Relay 3 → Mist Maker
```

---

## Enclosure Recommendations

1. **Controller Box:**
   - IP65 rated enclosure for ESP32 and relay module
   - Ventilation holes or small fan for heat dissipation
   - DIN rail mounting for professional installation
   - Separate compartments for low-voltage (ESP32) and high-voltage (relays)

2. **Wire Entry:**
   - Cable glands for sensor wires
   - Strain relief for all cables
   - Separate entry for mains voltage

3. **Mounting:**
   - Mount outside fermentation chamber (relays generate heat)
   - Keep sensor cables away from mains wiring
   - Ensure good WiFi signal reception

---

## Wiring Checklist

- [ ] All sensor pull-up resistors installed
- [ ] Common ground between ESP32 and relay module
- [ ] Relay module powered from separate 5V supply
- [ ] All mains connections use appropriately rated wire
- [ ] All mains connections properly terminated and insulated
- [ ] Ground/earth connections on all metal appliances
- [ ] Relay contacts rated for loads
- [ ] Fuses/circuit breakers installed
- [ ] No exposed mains voltage conductors
- [ ] All wiring secured with strain relief
- [ ] Enclosure properly sealed and labeled
- [ ] WiFi credentials configured in secrets file
- [ ] GPIO5 moved to non-strapping pin if boot issues occur

---

## Testing Procedure

1. **Before Applying Power:**
   - Verify all connections with multimeter
   - Check for shorts between power rails
   - Confirm relay wiring polarity

2. **Initial Power-Up (No Loads):**
   - Power ESP32 only
   - Check serial output for boot messages
   - Verify WiFi connection
   - Test each relay activation via web interface
   - Confirm relays click but don't connect loads yet

3. **Sensor Testing:**
   - Verify AHT20 temperature/humidity readings
   - Check DS18B20 readings (should show room temp)
   - Test door sensor (open/close detection)

4. **Load Testing:**
   - Connect one load at a time
   - Test with low-power devices first
   - Verify proper switching
   - Check for overheating

5. **Safety Testing:**
   - Verify heating/cooling interlock (cannot run together)
   - Test overtemperature cutoff
   - Confirm sensor failure safety shutdown
   - Test door safety in high-temp mode

---

## Troubleshooting

| Problem | Possible Cause | Solution |
|---------|---------------|----------|
| ESP32 won't boot | GPIO5 strapping pin issue | Move Relay 6 to GPIO 16/17/18/19 |
| No I2C sensor detected | Missing pull-ups or wrong address | Add 4.7kΩ resistors, check wiring |
| DS18B20 reads 85°C | Missing pull-up resistor | Add 4.7kΩ pull-up to data line |
| Relay doesn't click | Wrong active high/low | Change `inverted` setting in YAML |
| Relay clicks but load doesn't turn on | Wiring to NC instead of NO | Check relay contact connections |
| Random WiFi disconnects | Power supply issues | Use dedicated 5V supply, check voltage |
| Compressor short-cycles | Insufficient min off time | Adjust `min_cooling_off_time` in YAML |

---

## Schematic Diagram

For a visual reference, see the full schematic in [schematic.pdf](schematic.pdf) (if available).

## Additional Resources

- [ESPHome Documentation](https://esphome.io/)
- [ESP32 Pinout Reference](https://randomnerdtutorials.com/esp32-pinout-reference-gpios/)
- [AHT20 Datasheet](https://server4.eca.ir/eshop/AHT20/Aosong_AHT20_en_draft_0c.pdf)
- [DS18B20 Guide](https://randomnerdtutorials.com/esp32-ds18b20-temperature-arduino-ide/)

---

**Last Updated:** 2026-01-31
