# Appendix G — ESPHome Firmware Reference

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** ESP32 / ESPHome Controller Firmware  
> **Status:** Current configuration reference

## Overview

This appendix documents the ESPHome firmware running on the ESP32 controller that operates the garden sprinkler system.

The ESP32 provides the physical connection between Home Assistant and the 9-channel relay board. Home Assistant determines when and how long the sprinklers should operate, while ESPHome receives those commands and controls the individual relay outputs.

The firmware also provides several important safety and status features:

- Individual control of nine relay channels.
- Support for the seven currently active sprinkler zones.
- Active-low relay operation.
- Automatic relay shut-off after 31 minutes.
- Relays defaulting to off following a restart.
- A flashing onboard LED to indicate relay activity.
- Wi-Fi connectivity and the Home Assistant ESPHome API.
- Over-the-air firmware updates.

The configuration below should be treated as the primary firmware reference for rebuilding the sprinkler controller.

---

# 1. Device Configuration

The ESPHome device is configured with the following identity:

| Property | Value |
|---|---|
| ESPHome Device Name | `esphome-web-fd4824` |
| Friendly Name | `Sprinklers` |
| Name MAC Suffix | Disabled |
| Minimum ESPHome Version | `2026.4.0` |
| Platform | ESP32 |
| Variant | `esp32` |
| Framework | ESP-IDF |

The device name is intentionally configured without a MAC-address suffix:

```yaml
name_add_mac_suffix: false
```

This keeps the device name consistent rather than automatically adding characters derived from the ESP32's MAC address.

---

# 2. ESP32 Platform

The sprinkler controller is configured as an ESP32 using the ESP-IDF framework.

```yaml
esp32:
  variant: esp32
  framework:
    type: esp-idf
```

The ESP-IDF framework is the official development framework used by Espressif for ESP32 devices.

---

# 3. Home Assistant Integration

The ESP32 communicates directly with Home Assistant using the ESPHome API.

```yaml
api:
```

This allows Home Assistant to discover and control the switches exposed by the sprinkler controller.

The individual sprinkler zones appear in Home Assistant as switch entities, allowing them to be controlled manually or by automations.

---

# 4. Wi-Fi Connectivity

The controller connects to the local Wi-Fi network using credentials stored in the ESPHome secrets file.

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

The Wi-Fi credentials are deliberately not stored directly in the firmware configuration.

Instead, they are referenced using ESPHome secrets:

```text
!secret wifi_ssid
!secret wifi_password
```

This helps prevent network credentials from being accidentally included in a public repository.

## Wi-Fi Power Saving

The current configuration uses light Wi-Fi power saving:

```yaml
power_save_mode: LIGHT
```

The sprinkler controller is mains-powered and needs to remain reliably available to Home Assistant, so network reliability should always take priority over unnecessary power savings.

---

# 5. Logging

ESPHome logging is enabled:

```yaml
logger:
```

This allows diagnostic information to be viewed when troubleshooting the ESP32.

Logs can be useful for checking:

- ESP32 startup.
- Wi-Fi connectivity.
- Home Assistant API connectivity.
- Relay state changes.
- Firmware errors.
- Unexpected restarts.

---

# 6. Over-the-Air Firmware Updates

Over-the-air updates are enabled:

```yaml
ota:
  - platform: esphome
```

This allows the ESPHome firmware to be updated over the local network without physically reconnecting the ESP32 to a computer.

> When making changes to the sprinkler firmware, it is good practice to retain a backup of the working YAML configuration before uploading experimental changes.

---

# 7. Onboard Status LED

The ESP32's onboard blue LED is configured as a GPIO output.

```yaml
output:
  - platform: gpio
    pin: GPIO2
    id: onboard_led
```

The LED is used as a simple visual indicator of relay activity.

## LED Activity Script

The firmware defines a reusable script named:

```text
flash_led_twice
```

The script flashes the onboard LED twice.

```yaml
script:
  - id: flash_led_twice
    mode: restart
    then:
      - output.turn_on: onboard_led
      - delay: 100ms
      - output.turn_off: onboard_led
      - delay: 100ms
      - output.turn_on: onboard_led
      - delay: 100ms
      - output.turn_off: onboard_led
```

The script is configured with:

```yaml
mode: restart
```

This means a new request to run the script restarts the flashing sequence rather than allowing multiple flashing sequences to build up.

The LED flashes whenever a sprinkler relay is turned on or off.

---

# 8. Relay Control

The sprinkler controller operates a 9-channel active-low relay board.

Each relay is represented in ESPHome as a GPIO switch.

The relay channels are configured using the following GPIO pins:

| Relay Channel | GPIO Pin | ESPHome ID | Name |
|---|---:|---|---|
| 1 | GPIO13 | `zone_1` | Garden Sprinkler 1 |
| 2 | GPIO14 | `zone_2` | Garden Sprinkler 2 |
| 3 | GPIO16 | `zone_3` | Garden Sprinkler 3 |
| 4 | GPIO17 | `zone_4` | Garden Sprinkler 4 |
| 5 | GPIO18 | `zone_5` | Garden Sprinkler 5 |
| 6 | GPIO19 | `zone_6` | Garden Sprinkler 6 |
| 7 | GPIO21 | `zone_7` | Garden Sprinkler 7 |
| 8 | GPIO22 | `zone_8` | Garden Sprinkler 8 |
| 9 | GPIO23 | `zone_9` | Garden Sprinkler 9 |

Zones 1–7 are currently used by the automatic irrigation system.

Zones 8 and 9 are configured and available for future expansion.

---

# 9. Active-Low Relay Configuration

The relay board is an active-low design.

Each GPIO switch therefore uses:

```yaml
inverted: true
```

This setting reverses the output logic so that ESPHome operates the relay correctly.

The current configuration pattern is:

```yaml
pin:
  number: GPIO13
  inverted: true
```

The GPIO number changes for each relay channel, but the inverted configuration is retained across all nine channels.

> If the relay board is replaced in the future, the active-low configuration should be checked before assuming that `inverted: true` remains appropriate.

---

# 10. Restart Safety

Every sprinkler relay uses:

```yaml
restore_mode: ALWAYS_OFF
```

This is an important safety feature.

Following an ESP32 restart or reboot, the sprinkler outputs are configured to initialise in the off state rather than automatically restoring a previously active watering zone.

The expected behaviour is:

```text
ESP32 Restart
      │
      ▼
ESPHome Starts
      │
      ▼
Sprinkler Relay Outputs Initialise
      │
      ▼
All Zones OFF
```

This provides protection against a sprinkler continuing to run simply because the ESP32 restarted during a watering cycle. ESPHome documents `ALWAYS_OFF` as a switch restore mode that initialises the switch in the off state on boot. :contentReference[oaicite:0]{index=0}

---

# 11. Independent Relay Safety Timeout

Each sprinkler zone includes an independent automatic shut-off timer.

When a relay is turned on, the firmware performs the following sequence:

```text
Relay Turned ON
      │
      ▼
Flash Status LED
      │
      ▼
Wait up to 31 minutes
      │
      ▼
Automatically Turn Relay OFF
```

The relevant ESPHome pattern is:

```yaml
on_turn_on:
  - script.execute: flash_led_twice
  - delay: 31min
  - switch.turn_off: zone_1
```

The same safety behaviour is configured for each of the nine relay channels.

The Home Assistant automation normally turns zones off much sooner than this limit.

The 31-minute timeout acts as an independent failsafe in case:

- A Home Assistant automation fails.
- A communication problem prevents the normal switch-off command.
- A watering cycle is interrupted unexpectedly.

> The safety timeout is intentionally separate from the normal irrigation schedule. Normal watering duration is controlled by Home Assistant.

---

# 12. Relay State Change Behaviour

Each relay performs actions when switched on and off.

## When a Relay Turns On

The ESP32:

1. Flashes the onboard LED twice.
2. Starts the 31-minute safety timer.
3. Automatically turns the relay off when the safety timeout expires.

## When a Relay Turns Off

The ESP32:

1. Flashes the onboard LED twice.

The general configuration pattern is:

```yaml
on_turn_on:
  - script.execute: flash_led_twice
  - delay: 31min
  - switch.turn_off: zone_x

on_turn_off:
  - script.execute: flash_led_twice
```

Where `zone_x` is replaced by the appropriate zone ID.

---

# 13. Current Complete ESPHome Configuration

The following is the current sprinkler controller firmware configuration.

```yaml
esphome:
  name: esphome-web-fd4824
  friendly_name: Sprinklers
  min_version: 2026.4.0
  name_add_mac_suffix: false

esp32:
  variant: esp32
  framework:
    type: esp-idf

# Enable logging
logger:

# Enable Home Assistant API
api:

# Allow Over-The-Air updates
ota:
  - platform: esphome

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Power saving settings
  power_save_mode: LIGHT

# Define the onboard blue LED
output:
  - platform: gpio
    pin: GPIO2
    id: onboard_led

# Reusable flash effect template
script:
  - id: flash_led_twice
    mode: restart
    then:
      - output.turn_on: onboard_led
      - delay: 100ms
      - output.turn_off: onboard_led
      - delay: 100ms
      - output.turn_on: onboard_led
      - delay: 100ms
      - output.turn_off: onboard_led

# Control for 9 channels on the UCtronics Active-Low relay board
switch:
  - platform: gpio
    name: "Garden Sprinkler 1"
    id: zone_1
    pin:
      number: GPIO13
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_1
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 2"
    id: zone_2
    pin:
      number: GPIO14
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_2
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 3"
    id: zone_3
    pin:
      number: GPIO16
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_3
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 4"
    id: zone_4
    pin:
      number: GPIO17
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_4
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 5"
    id: zone_5
    pin:
      number: GPIO18
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_5
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 6"
    id: zone_6
    pin:
      number: GPIO19
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_6
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 7"
    id: zone_7
    pin:
      number: GPIO21
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_7
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 8"
    id: zone_8
    pin:
      number: GPIO22
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_8
    on_turn_off:
      - script.execute: flash_led_twice

  - platform: gpio
    name: "Garden Sprinkler 9"
    id: zone_9
    pin:
      number: GPIO23
      inverted: true
    restore_mode: ALWAYS_OFF
    on_turn_on:
      - script.execute: flash_led_twice
      - delay: 31min
      - switch.turn_off: zone_9
    on_turn_off:
      - script.execute: flash_led_twice
```

---

# 14. Firmware Rebuild Checklist

If the ESP32 controller needs to be rebuilt or replaced:

- [ ] Obtain a compatible ESP32 development board.
- [ ] Install ESPHome.
- [ ] Create the sprinkler device configuration.
- [ ] Configure Wi-Fi credentials using the ESPHome secrets file.
- [ ] Copy the ESPHome configuration from this appendix.
- [ ] Connect the relay inputs to the correct GPIO pins.
- [ ] Confirm the relay board operates correctly as an active-low device.
- [ ] Upload the firmware to the ESP32.
- [ ] Confirm the device connects to Wi-Fi.
- [ ] Confirm the device appears in Home Assistant.
- [ ] Test each relay individually.
- [ ] Confirm the onboard LED flashes during relay state changes.
- [ ] Confirm all relays initialise in the off state following a restart.
- [ ] Confirm the 31-minute safety timeout remains active.

---

# 15. Configuration Dependencies

The ESPHome configuration depends on the following external items:

| Dependency | Purpose |
|---|---|
| `wifi_ssid` secret | Wi-Fi network name |
| `wifi_password` secret | Wi-Fi network password |
| Home Assistant ESPHome integration | Controller communication |
| UCtronics active-low relay board | Physical switching of sprinkler circuits |

The Wi-Fi secrets should not be committed to a public GitHub repository.

If the repository is ever made public, ensure that the actual secrets file remains excluded from version control.

---

# 16. Relationship to Home Assistant

The division of responsibility between Home Assistant and ESPHome is:

```text
HOME ASSISTANT
────────────────────────────────────
• Calculates watering requirements
• Monitors virtual soil moisture
• Checks rainfall forecasts
• Determines watering runtime
• Controls cycle and soak behaviour
• Selects when zones should operate

                │
                │ ESPHome API
                ▼

ESP32 / ESPHOME
────────────────────────────────────
• Receives switch commands
• Controls physical GPIO outputs
• Operates relay channels
• Provides relay safety timeout
• Defaults relays to OFF on restart
• Provides visual LED feedback
```

This separation provides an additional layer of safety.

Home Assistant controls the irrigation logic, while the ESP32 retains responsibility for directly controlling the physical relay outputs and enforcing the independent safety timeout.

---

# 17. Related Documentation

- [Hardware Specification](Hardware-Specification.md)
- [GPIO & ESP32 Pin Reference](GPIO-ESP32-Pin-Reference.md)
- [Automation Reference](Automation-Reference.md)
- [Helper Reference](Helper-Reference.md)
- [Installation & Setup](../docs/09-Installation-and-Setup.md)

---

*Home Assistant Smart Sprinkler System — Appendix G: ESPHome Firmware Reference*
