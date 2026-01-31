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
- Selectable temperature source (AHT20 or either external probe)

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
- Disabled in High-Temp mode (ultrasonic humidifiers can't handle high temps)

#### 6. Door Light Automation
- Light turns on automatically when door opens
- Stays on for 10 minutes after door closes
- Smart logic preserves user's manual light state

#### 7. Turbo Heater Shadow Mode
- In High-Temp mode, turbo heater automatically follows the heating mat
- When main heater turns on, turbo activates for faster heating
- Extends fan post-run to dissipate residual heat safely

### Integration
- WiFi connectivity with fallback AP
- Home Assistant API integration (port 6053)
- Web server on port 80
- OTA updates supported

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
   - Connect to the web interface at `http://fermenter.local`
   - Or integrate with Home Assistant via the API

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

## Documentation

- [Wiring Guide](docs/wiring.md) - Complete wiring diagrams and pin assignments
- [Project Notes](docs/notes.md) - Development notes and future improvements

## License

This project is provided as-is for educational purposes. Use at your own risk.