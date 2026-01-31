# Project Notes - Fermentation Chamber Controller

## Operational Behavior

### Humidity Control and High-Temp Mode

**Important:** Automatic humidity control is **disabled in High-Temp mode** by design.

- **Reason**: Ultrasonic mist makers cannot handle high temperatures and will be damaged above ~40-50°C
- **Affected modes**: When the Mode selector is set to "High-Temp"
- **Behavior**: The humidity control loop exits early when High-Temp mode is active ([fermentor.yaml:648-651](../esphome/fermentor.yaml#L648-L651))
- **Manual control**: The mist maker relay can still be manually controlled, but automatic humidity regulation is disabled
- **Recommendation**: For high-temperature fermentation (black garlic, miso, etc.), rely on natural evaporation or use alternative humidity control methods

### Humidity Auto Switch

The humidity automation must be **manually enabled** after boot:

- **Default state**: OFF (for safety)
- **Location**: Web interface or Home Assistant - switch named "Fermenter Humidity Auto"
- **Why**: Prevents unexpected misting behavior during initial setup or testing
- **Tip**: Enable this switch after confirming:
  - Mist maker is connected and working
  - You're in Normal mode (not High-Temp)
  - Target humidity is set appropriately

### Mode Switching Behavior

When switching between Normal and High-Temp modes:

**Normal → High-Temp:**
- Humidity control automatically disables
- Compressor protection: 60-minute cooling lockout begins
- Turbo heater becomes available (shadows main heater)

**High-Temp → Normal:**
- 60-minute cooling lockout prevents compressor damage
- Compressor won't run until lockout expires
- Humidity control becomes available again (if Auto switch is ON)

### Fan Automation

The Fan Auto switch enables intelligent fan control:

- **When ON**: Fan runs whenever heating, cooling, misting, or turbo heater is active
- **Post-run**: Continues for 3 minutes after equipment stops (configurable)
- **Turbo post-run**: Extended to 10 minutes minimum after turbo heater use
- **When OFF**: Fan must be controlled manually

### Temperature Source Selection

The controller supports four temperature sources for climate control:

#### 1. AHT20 (Chamber Air Temperature)
- **Use case**: General fermentation when chamber air temp is adequate
- **Pros**: Fast response to temperature changes, measures overall chamber climate
- **Cons**: Doesn't directly measure product temperature
- **Best for**: Beer fermentation, general purpose

#### 2. External 1 (Direct Product Monitoring)
- **Use case**: Single ferment where precise product temperature is critical
- **Pros**: Measures actual fermentation temperature, not affected by air stratification
- **Cons**: Slower response time, only monitors one location
- **Best for**: Sourdough, koji, single batch ferments

#### 3. External 2 (Direct Product Monitoring)
- **Use case**: Alternative single ferment monitoring or backup sensor
- **Pros**: Same as External 1
- **Cons**: Same as External 1
- **Best for**: Second fermentation vessel or redundancy

#### 4. Average (Ext 1+2) - **NEW**
- **Use case**: **Multiple simultaneous ferments** requiring balanced temperature control
- **Pros**: Ensures both ferments stay within acceptable range, prevents one from overheating while the other is too cold
- **Cons**: Averaging may allow individual temperatures to drift if products have different thermal masses
- **Best for**: Dual batch beer fermentation, multiple koji trays, parallel sourdough batches
- **How it works**:
  - Calculates average of External 1 and External 2 temperatures
  - If one sensor fails, automatically falls back to the working sensor
  - If both fail, falls back to AHT20

#### Choosing the Right Source

**Single Ferment:**
- Use External 1 or 2 inserted directly into the fermenting product
- Most accurate control for that specific product

**Dual Ferments:**
- Use "Average (Ext 1+2)" with one probe in each ferment
- Ensures both products ferment at acceptable temperatures
- Controller maintains average, keeping both within range

**No External Probes:**
- Use AHT20 for chamber air control
- Adequate for most general fermentation needs

**Example Scenario - Dual Beer Fermentation:**
```
Batch A (External 1): 21.5°C
Batch B (External 2): 22.5°C
Control Temp (Average): 22.0°C
Setpoint: 22°C

Result: Controller maintains average at 22°C, both batches stay
within ±0.5°C of target despite thermal mass differences.
```

## Safety Features

### Temperature Safety Limits

The system has three tiers of temperature protection:

1. **Soft limit (Normal mode)**: 57°C - Triggers overtemp alarm, stops heating
2. **Soft limit (High-Temp mode)**: 78°C - Triggers overtemp alarm, stops heating
3. **Hard limit (All modes)**: 85°C - Absolute emergency cutoff

### Heating/Cooling Interlocks

**Critical safety feature**: Heating and cooling outputs are **mutually exclusive**

- When heating turns ON → cooling forced OFF
- When cooling turns ON → heating forced OFF
- When turbo heater turns ON → cooling forced OFF
- Prevents simultaneous operation that could damage equipment or waste energy

### Sensor Failure Protection

If the control temperature sensor fails (NaN/invalid reading):

- All heating outputs forced OFF (mat + turbo)
- Compressor forced OFF
- System enters safe state
- User must resolve sensor issue before normal operation resumes

### Door Safety in High-Temp Mode

Special safety feature for high-temperature operation:

- **Trigger**: Door opens when temperature > 40°C in High-Temp mode
- **Action**: Turbo heater immediately disabled
- **Reason**: Prevents user burns and rapid heat loss
- **Behavior**: Turbo remains off until conditions are safe

## Troubleshooting

### Humidity Not Controlling

Check in this order:

1. **Humidity Auto switch** - Must be ON
2. **Mode setting** - Must be "Normal" (not High-Temp)
3. **AHT20 sensor** - Verify it's reading humidity correctly
4. **Mist lockout** - Wait 60 seconds if mister recently hit max runtime
5. **Target/hysteresis** - Verify settings make sense (default: 70% ± 2%)
6. **Wiring** - Confirm mist maker is on Relay 3 (GPIO27)

### Compressor Won't Start

Possible causes:

1. **Temperature too high** - Compressor disabled above 35°C
2. **High-Temp mode active** - Compressor disabled in this mode
3. **Cooling lockout active** - Wait up to 60 minutes after mode switch
4. **Min off time** - Compressor requires 5 minutes between cycles (300s default)
5. **Thermostat deadband** - Temperature may not be high enough yet

### Turbo Heater Not Working

Expected behavior varies by mode:

- **Normal mode**: Turbo heater is OFF (only heating mat used)
- **High-Temp mode**: Turbo automatically follows heating mat
  - Turbo ON when heater ON
  - Turbo OFF when heater OFF or door opens above 40°C

### Fan Runs Continuously

If Fan Auto is ON, check:

1. **Post-run timer** - Fan continues 3-10 minutes after heating/cooling/misting
2. **Active equipment** - Something is still running (check heater/cooler/mister status)
3. **Disable Fan Auto** - Turn off the switch if manual control is preferred

## Configuration Tips

### Optimal Humidity Settings

- **Beer fermentation**: 60-70% RH
- **Sourdough proofing**: 75-85% RH
- **Cheese aging**: 80-95% RH
- **Dry aging**: 70-80% RH
- **Koji cultivation**: 70-75% RH

Set hysteresis appropriately:
- Lower hysteresis (1-2%) = tighter control, more misting cycles
- Higher hysteresis (3-5%) = looser control, fewer cycles, longer mister life

### Temperature Presets

The default presets can be customized in the YAML:

- **Ferment (22°C)**: Ale fermentation, general purpose
- **Sourdough (26°C)**: Optimal sourdough starter activity
- **ColdCrash (4°C)**: Beer clarification, cold storage
- **BlackGarlic (70°C)**: Black garlic, high-temp ferments

### Safety Threshold Tuning

Conservative defaults are set for safety. Adjust if needed:

- **Normal mode max (57°C)**: Safe for most ferments, prevents mist maker damage
- **High-Temp max (78°C)**: Suitable for black garlic (60-70°C range)
- **Hard limit (85°C)**: Emergency cutoff, should rarely be changed

### Temperature Source Best Practices

**Probe Placement:**
- Insert external probes into the center of fermenting products
- Avoid touching container walls (reads container temp, not product)
- For liquids: suspend probe in middle of liquid
- For solids (koji, sourdough): insert into thickest part

**Using Average Mode:**
- Ideal when running two ferments with similar temperature requirements
- Place one probe in each fermentation vessel
- Monitor individual temps via web interface to verify both are in range
- If one ferment finishes early, switch to single External source for remaining batch

**Sensor Failures:**
- System automatically falls back to AHT20 if selected source fails
- In Average mode, falls back to working external sensor if one fails
- Monitor logs for sensor failure warnings
- DS18B20 reading 85°C usually indicates sensor disconnection or failure

## Known Limitations

### Ultrasonic Humidifiers

- Cannot operate above ~40-50°C (component limitation)
- Require distilled water for best performance
- May need cleaning every 1-2 weeks with heavy use
- Dry detection alerts when water runs out

### Compressor Protection

- Minimum 5-minute off time between cycles (protects compressor)
- 60-minute lockout after exiting High-Temp mode (prevents thermal shock)
- Cannot run in High-Temp mode or above 35°C

### Temperature Control

- Not a true PID controller (uses ESPHome thermostat component)
- Hysteresis-based control may overshoot by 0.5-1°C
- For tighter control, reduce deadband values in YAML
- External temperature probes recommended for direct product monitoring

## Future Improvements

Ideas for future development:

- [ ] Add PID controller option for tighter temperature control
- [ ] Implement dew point calculation to prevent condensation
- [ ] Add data logging to SD card for offline monitoring
- [ ] Create scheduling system for automated fermentation profiles
- [ ] Add support for multiple chambers with one controller
- [ ] Implement push notifications for alerts (via Home Assistant)
- [ ] Add chamber light dimming control
- [ ] Support for steam-based humidification in High-Temp mode
- [ ] Advanced dry detection with automatic water level sensor

## Change Log

### 2026-01-31 (v1.1)
- **Added**: Average (Ext 1+2) temperature source option
  - Enables balanced temperature control for dual ferments
  - Automatically averages External 1 and External 2 sensor readings
  - Falls back to single sensor if one fails
- **Updated**: Comprehensive documentation for temperature source selection
- **Updated**: Best practices for probe placement and sensor usage

### 2026-01-31 (v1.0)
- Initial release
- Dual-mode operation (Normal/High-Temp)
- 6-relay control system
- Humidity auto-control with dry detection
- Multiple safety interlocks
- Home Assistant integration

## Contributing

If you make improvements to this configuration:

1. Test thoroughly in a safe environment
2. Document any changes in this file
3. Update wiring.md if pin assignments change
4. Consider submitting a pull request to share improvements

## Support

For issues or questions:
- Check troubleshooting section above
- Review ESPHome logs via web interface
- Post issues on the GitHub repository

---

**Remember**: This system controls mains voltage and temperature-sensitive processes. Always prioritize safety over convenience.
