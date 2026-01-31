# Home Assistant Integration Troubleshooting

## Entity Not Appearing in Home Assistant

If entities (especially binary sensors) aren't showing up in Home Assistant after setting up the ESPHome integration, follow these steps:

### 1. Verify ESPHome Device is Connected

**Check in Home Assistant:**
- Go to Settings → Devices & Services → ESPHome
- Verify the "Fermenter" device shows as "Connected"
- If offline, check WiFi connectivity and ESP32 logs

### 2. Check ESPHome Logs

**Via Web Interface:**
- Open `http://fermenter.local` (or IP: 10.42.102.38)
- Click "Logs"
- Look for lines showing entities being exposed to API
- Should see: `[api:102] Exposing binary sensor 'Fermenter Overtemp Trip'`

**Via Home Assistant:**
- Settings → Devices & Services → ESPHome
- Click on "Fermenter" device
- Click "Logs" button
- Check for API connection errors

### 3. Reload ESPHome Integration

**After making changes to dashboard or YAML:**
- Settings → Devices & Services → ESPHome
- Click the three dots (⋮) on the ESPHome card
- Click "Reload"
- Wait 10-15 seconds for device to reconnect

### 4. Check Entity Registry

**Find all entities (including disabled ones):**
- Developer Tools → States tab
- Filter by "fermenter" in the entity field
- Scroll through all entities to see if binary sensors are listed

**Or check entity registry:**
- Settings → Devices & Services → ESPHome
- Click on "Fermenter" device
- Scroll down to see all entities
- Check if any entities show as "Disabled"
- Click on disabled entities to enable them

### 5. Verify API Encryption Key

**Important:** The API encryption key in the YAML must match what Home Assistant has stored.

**Check the key in ESPHome YAML:**
```yaml
api:
  encryption:
    key: "CXcjj0aUsTEuJP/ny2nrsnuNVOfvukaE0b4ArdgeBlg="
```

**If key mismatch occurs:**
- Delete the ESPHome integration in Home Assistant
- Go to Settings → Devices & Services → ESPHome
- Click three dots (⋮) → Delete
- Restart Home Assistant
- The device should be auto-discovered again
- Click "Configure" and enter the encryption key from the YAML

### 6. Force Entity Discovery

**Restart the ESP32 device:**
- Via web interface: Navigate to `http://fermenter.local`
- Click "Restart"

**Or via Home Assistant:**
- Settings → Devices & Services → ESPHome → Fermenter device
- Click on `button.fermenter_fermenter_restart`
- Press the button

**After restart:**
- Wait 30-60 seconds
- All entities should re-register with Home Assistant

### 7. Check Entity Naming

**Correct Entity IDs:**
The ESPHome device name is "fermenter" and entity names include "Fermenter" prefix.
This creates a double prefix pattern:

**Climate:**
- `climate.fermenter_fermenter_temperature`

**Sensors:**
- `sensor.fermenter_fermenter_control_temp`
- `sensor.fermenter_fermenter_air_temp`
- `sensor.fermenter_fermenter_humidity_aht20`
- `sensor.fermenter_fermenter_external_1`
- `sensor.fermenter_fermenter_external_2`
- `sensor.fermenter_fermenter_wifi_rssi`
- `sensor.fermenter_fermenter_uptime`

**Binary Sensors:**
- `binary_sensor.fermenter_fermenter_overtemp_trip`
- `binary_sensor.fermenter_fermenter_door`
- `binary_sensor.fermenter_fermenter_humidifier_dry`

**Switches:**
- `switch.fermenter_fermenter_light`
- `switch.fermenter_fermenter_heating_mat`
- `switch.fermenter_fermenter_mist_maker`
- `switch.fermenter_fermenter_fan`
- `switch.fermenter_fermenter_compressor_cool`
- `switch.fermenter_fermenter_turbo_heater`
- `switch.fermenter_fermenter_humidity_auto`
- `switch.fermenter_fermenter_fan_auto`
- `switch.fermenter_fermenter_humidifier_dry_alerts`

**Selects:**
- `select.fermenter_fermenter_temperature_source`
- `select.fermenter_fermenter_mode`

**Numbers:**
- `number.fermenter_fermenter_target_humidity`
- `number.fermenter_fermenter_rh_hysteresis`
- `number.fermenter_fermenter_mist_max_on_min`
- `number.fermenter_fermenter_fan_post_run_min`
- And all other configurable parameters...

### 8. Common Issues

**Issue: Binary sensors show as unavailable**
- **Cause:** ESPHome device not connected to Home Assistant
- **Fix:** Check WiFi connection, verify API key, restart device

**Issue: Entities show with wrong names**
- **Cause:** Dashboard using incorrect entity IDs
- **Fix:** Update dashboard YAML with correct double-prefix pattern

**Issue: Some entities missing**
- **Cause:** Entities might be disabled by default
- **Fix:** Go to device page, scroll to entity list, enable disabled entities

**Issue: "Failed to connect" in ESPHome**
- **Cause:** API encryption key mismatch or network issue
- **Fix:** Delete integration and re-add, or check firewall settings (port 6053)

### 9. Network Troubleshooting

**Check ESPHome API Port:**
- ESPHome uses port **6053** for native API
- Ensure this port is not blocked by firewall
- Test connectivity: `telnet 10.42.102.38 6053`

**mDNS Issues:**
- If `fermenter.local` doesn't resolve, use IP address directly: `10.42.102.38`
- Some routers/networks don't support mDNS properly
- Add static DHCP lease for the ESP32 in your router

### 10. Complete Reset (Last Resort)

If nothing else works:

1. **Remove from Home Assistant:**
   - Settings → Devices & Services → ESPHome
   - Delete "Fermenter" device
   - Restart Home Assistant

2. **Flash ESP32 again:**
   ```bash
   cd ~/Documents/fermenter
   esphome run esphome/fermentor.yaml
   ```

3. **Wait for auto-discovery:**
   - Device should appear in Settings → Devices & Services
   - Click "Configure" and add

4. **Import dashboard:**
   - Use the corrected `homeassistant_dashboard.yaml`
   - All entities should now be visible

## Verification Checklist

- [ ] ESPHome device shows "Connected" in Home Assistant
- [ ] All expected entities appear in device page
- [ ] Binary sensors are not disabled
- [ ] Entity IDs use double prefix (fermenter_fermenter_*)
- [ ] Dashboard loads without entity errors
- [ ] Can control switches from dashboard
- [ ] Sensor values update in real-time
- [ ] Climate control responds to temperature changes

---

**Last Updated:** 2026-01-31
