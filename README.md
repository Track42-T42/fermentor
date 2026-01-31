# Fermentation Chamber (ESPHome)

This repo contains the ESPHome configuration and documentation for a sophisticated fermentation chamber controller designed for various fermentation projects like beer, sourdough, koji, black garlic, and miso.

## What This System Does

This is an **ESPHome-based fermentation chamber controller** with precise temperature and humidity control, multiple safety features, and support for both normal and high-temperature fermentation processes.

### Hardware Components

The system controls 6 relays connected to:
- **Relay 1**: Interior light (auto-on when door opens)
- **Relay 2**: Heating mat (primary heat source)
- **Relay 3**: Ultrasonic mist maker (for humidity)
- **Relay 4**: Circulation fan
- **Relay 5**: Compressor/cooling system
- **Relay 6**: Turbo heater (for high-temperature applications)

**Sensors**:
- AHT20 temperature/humidity sensor (I2C)
- Two external DS18B20 temperature probes (for direct product monitoring)
- Magnetic door sensor

### Key Features

#### 1. Dual-Mode Operation
- **Normal Mode**: For typical fermentation (max 57°C soft limit)
- **High-Temp Mode**: For black garlic, miso, etc. (max 78°C soft limit)
- Hard safety limit of 85°C across all modes

#### 2. Climate Control
- PID-style thermostat with heating and cooling
- Four presets: Ferment (22°C), Sourdough (26°C), ColdCrash (4°C), BlackGarlic (70°C)
- Selectable temperature source:
  - AHT20 (chamber air temperature)
  - External 1 (direct product monitoring)
  - External 2 (direct product monitoring)
  - Average (Ext 1+2) - ideal for dual ferments

#### 3. Safety Interlocks
- Heating and cooling **never** run simultaneously
- Overtemperature protection shuts down all heaters
- Sensor failure detection cuts power to all climate devices
- Door-open safety: disables turbo heater in high-temp mode if door opens above 40°C
- Cooling lockout: 60-minute delay after exiting High-Temp mode
- Compressor won't run above 35°C or in High-Temp mode

#### 4. Smart Fan Control
- Auto mode runs fan whenever heating/cooling/misting is active
- Continues running for configurable post-run period after equipment stops (default 3 min)
- Extended 10-minute post-run after turbo heater use

#### 5. Humidity Management
- Automatic misting to maintain target humidity (30-95%)
- Configurable hysteresis to prevent rapid cycling
- Safety timeout (max 45 minutes continuous misting)
- **Dry detection**: Monitors if humidity rises as expected when misting; alerts if humidifier runs dry
- Automatically disabled when temperature reaches 45°C or higher (ultrasonic humidifiers can't handle high temps)

#### 6. Door Light Automation
- Light turns on automatically when door opens
- Stays on for 10 minutes after door closes
- Smart logic preserves user's manual light state

#### 7. Turbo Heater Shadow Mode
- In High-Temp mode, turbo heater automatically follows the heating mat
- When main heater turns on, turbo activates for faster heating
- Extends fan post-run to dissipate residual heat safely

### Integration

**Standalone Operation:**
- Built-in web server on port 80 (`http://fermenter.local`)
- Control all relays, view sensors, adjust settings via web interface
- No additional software required - works independently

**Home Assistant Integration (Optional):**
- All sensors, switches, and controls automatically discovered
- Native API integration on port 6053
- Real-time updates and historical data logging
- Automation and notifications support
- Dashboard creation with fermentation metrics

**Other Features:**
- WiFi connectivity with fallback AP mode
- OTA (Over-The-Air) updates - no USB required after initial flash
- ESPHome native API for custom integrations

## Hardware Requirements

- ESP32 development board
- 6-channel relay module (5V, optoisolated)
- AHT20 temperature/humidity sensor (I2C)
- 2x DS18B20 waterproof temperature probes
- Magnetic door sensor (reed switch)
- 5V power supply (3A recommended)
- Heating mat or heating element
- Refrigeration unit with compressor
- Additional heater for high-temp mode (optional)
- Ultrasonic mist maker (12V/24V)
- Circulation fan
- Interior light

See [docs/wiring.md](docs/wiring.md) for complete wiring diagrams and pin assignments.

## ESPHome Setup

1. **Install ESPHome:**
   ```bash
   pip install esphome
   ```

2. **Clone this repository:**
   ```bash
   git clone <your-repo-url>
   cd fermenter
   ```

3. **Create secrets file:**
   Create `esphome/secrets.yaml`:
   ```yaml
   wifi_ssid: "YourWiFiSSID"
   wifi_password: "YourWiFiPassword"
   ```

4. **Review and customize configuration:**
   - Edit `esphome/fermentor.yaml` if needed
   - Adjust safety thresholds for your use case
   - Verify GPIO pin assignments match your wiring

5. **Build and flash:**
   ```bash
   esphome run esphome/fermentor.yaml
   ```

6. **Initial setup:**

   **Option A: Standalone Mode (Web Interface)**
   - Open a web browser and navigate to `http://fermenter.local` (or use the IP address)
   - Access all controls, sensors, and settings directly
   - No additional software needed - fully functional out of the box

   **Option B: Home Assistant Integration**
   - ESPHome devices are automatically discovered in Home Assistant
   - Go to Settings → Devices & Services → ESPHome
   - Click "Configure" on the discovered "Fermenter" device
   - All entities (sensors, switches, controls) will be available immediately
   - Create dashboards, automations, and notifications as needed

## Configuration

Key parameters you may want to adjust in the YAML:

- **Safety Thresholds:**
  - `safety_max_c_normal`: 57°C (soft limit for Normal mode)
  - `safety_max_c_high`: 78°C (soft limit for High-Temp mode)
  - `safety_hard_c`: 85°C (absolute hard limit)

- **Compressor Timing:**
  - `min_cooling_run_time`: 120s
  - `min_cooling_off_time`: 300s (protects compressor)

- **Temperature Presets:**
  - Ferment: 22°C
  - Sourdough: 26°C
  - ColdCrash: 4°C
  - BlackGarlic: 70°C

## Usage Modes

### Standalone Web Interface

The controller includes a built-in web server that works independently without any additional software:

**Access:** `http://fermenter.local` (or use IP address if mDNS doesn't work)

**Available Controls:**
- All relay switches (Light, Heater, Mist Maker, Fan, Compressor, Turbo)
- Temperature source selector
- Mode selector (Normal/High-Temp)
- Humidity settings (target, hysteresis, max on time)
- All automation toggles (Humidity Auto, Fan Auto, Dry Alerts)
- Real-time sensor readings (temperature, humidity, door status)
- Device information and logs

**Best for:**
- Simple setups without home automation
- Quick access during fermentation checks
- Troubleshooting and configuration
- Direct control without additional infrastructure

### Home Assistant Integration

When integrated with Home Assistant, the controller becomes part of your smart home ecosystem:

**Auto-Discovery:**
- ESPHome integration automatically discovers the device
- All sensors and controls appear as native entities
- No manual configuration required

**Available in Home Assistant:**
- **Sensors**: All temperature readings, humidity, WiFi signal, uptime
- **Switches**: All relays and automation toggles
- **Climate**: Thermostat control with presets
- **Selectors**: Temperature source, mode selection
- **Binary Sensors**: Door sensor, overtemperature alarm, humidifier dry status
- **Numbers**: All configurable parameters (humidity target, timings, etc.)

**Additional Capabilities:**
- **Historical Data**: Track temperature/humidity trends over weeks/months
- **Automations**: Trigger actions based on fermentation stages
  - Example: Notification when cold crash temperature reached
  - Example: Auto-enable cooling when fermentation complete
- **Dashboards**: Create custom fermentation monitoring displays
- **Notifications**: Push alerts for overtemperature, humidifier dry, etc.
- **Integrations**: Combine with other sensors (e.g., tilt hydrometers for beer)
- **Voice Control**: "Alexa, what's the fermentation temperature?"

**Example Home Assistant Automations:**
```yaml
# Alert when fermentation temperature drops (cold crash complete)
automation:
  - alias: "Cold Crash Complete"
    trigger:
      - platform: numeric_state
        entity_id: sensor.fermenter_control_temp
        below: 5
        for: "01:00:00"
    action:
      - service: notify.mobile_app
        data:
          message: "Cold crash complete - beer ready for kegging!"

# Enable humidifier when starting sourdough preset
automation:
  - alias: "Sourdough Mode Humidity"
    trigger:
      - platform: state
        entity_id: climate.fermenter_temperature
        attribute: preset_mode
        to: "Sourdough"
    action:
      - service: switch.turn_on
        entity_id: switch.fermenter_humidity_auto
```

**Best for:**
- Advanced home automation users
- Long-term data logging and analysis
- Complex automation workflows
- Remote monitoring away from home
- Multi-device smart home setups

### Which Mode Should You Use?

**Use Standalone Mode if:**
- You want simple, direct control
- You don't have Home Assistant
- You prefer accessing via web browser
- You only need basic monitoring

**Use Home Assistant if:**
- You already have a Home Assistant installation
- You want historical data and trends
- You need notifications and automations
- You want to integrate with other smart devices
- You want remote access (via Home Assistant Cloud or VPN)

**Use Both!**
- Both modes work simultaneously
- Web interface always available for quick checks
- Home Assistant provides advanced features
- No configuration needed - works out of the box

## Safety

⚠️ **CRITICAL SAFETY INFORMATION** ⚠️

**This project involves mains voltage (120V/240V AC) which can be LETHAL.**

- Only proceed if you are qualified to work with mains voltage
- Use proper enclosures rated for mains voltage
- Install appropriate fuses and circuit breakers
- Ensure all equipment is properly grounded/earthed
- Never operate with exposed mains connections
- Test all safety interlocks before first use
- Have your work inspected by a qualified electrician

**This project is NOT suitable for someone who does not understand electrical wiring.**

### Built-in Safety Features

- Heating and cooling interlocks (prevents simultaneous operation)
- Multi-tier overtemperature protection
- Sensor failure detection and automatic shutdown
- Door safety in high-temperature mode
- Compressor protection timing
- Humidifier dry detection
- Configurable safety limits per mode

## Home Assistant Dashboard

A pre-built, beautiful dashboard configuration is included for Home Assistant users:

**File:** [homeassistant_dashboard.yaml](homeassistant_dashboard.yaml)

### Dashboard Features

**4 Comprehensive Views:**

1. **Overview** - Main control panel
   - Real-time temperature gauges
   - Climate control thermostat
   - All equipment switches
   - Safety status indicators
   - Mode and automation controls

2. **Graphs** - Historical data visualization
   - 24-hour temperature history
   - 7-day temperature statistics
   - Humidity trends
   - Equipment activity timeline

3. **Advanced** - Detailed configuration
   - All tunable parameters
   - Fan and humidity settings
   - System information
   - Quick restart button

4. **Presets** - One-tap fermentation profiles
   - Large preset buttons (Ferment, Sourdough, ColdCrash, BlackGarlic)
   - Preset recommendations
   - Temperature source selector
   - Mode switching guide

### Installation

1. Open Home Assistant
2. Go to Settings → Dashboards
3. Click "+ Add Dashboard"
4. Choose "New dashboard from scratch"
5. Click "⋮" (three dots) → "Edit Dashboard"
6. Click "⋮" again → "Raw configuration editor"
7. Copy contents of `homeassistant_dashboard.yaml`
8. Paste and save

### Screenshots

The dashboard includes:
- 🌡️ Real-time temperature gauges with color-coded zones
- 📊 Historical graphs for trend analysis
- 🎯 One-tap preset activation buttons
- 🔌 Individual equipment controls
- 🤖 Automation toggle switches
- ⚠️ Safety status indicators
- 💧 Humidity management controls

## Documentation

- [Wiring Guide](docs/wiring.md) - Complete wiring diagrams and pin assignments
- [Project Notes](docs/notes.md) - Development notes and future improvements
- [Home Assistant Dashboard](homeassistant_dashboard.yaml) - Pre-built dashboard configuration
- [Home Assistant Troubleshooting](docs/homeassistant_troubleshooting.md) - Entity visibility and integration issues

## License

This project is provided as-is for educational purposes. Use at your own risk.