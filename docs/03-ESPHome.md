# ESPHome Firmware

| | |
|---|---|
| **Project** | Home Assistant Smart Sprinklers |
| **Document** | 03 – ESPHome Firmware |
| **Version** | 1.0 |
| **Controller** | AZ-Delivery ESP32 DevKitC V4 |
| **Firmware** | ESPHome |
| **Status** | Live System |
| **Last Updated** | July 2026 |

---

## Summary

This document explains the ESPHome firmware that runs on the irrigation controller.

The ESP32 provides the physical interface between Home Assistant and the sprinkler relay board. While Home Assistant decides **when** watering should occur, ESPHome is responsible for safely switching the relay outputs, providing status indication and ensuring that no sprinkler valve can remain energised indefinitely.

The complete firmware source is stored in:

```
esphome/sprinklers.yaml
```

---

# Contents

- Overview
- Hardware Responsibilities
- ESPHome Configuration
- Network Configuration
- Status LED
- LED Flash Script
- Relay Outputs
- Safety Features
- GPIO Allocation
- Firmware Behaviour
- Updating the Firmware
- Troubleshooting
- Key Takeaways

---

# Overview

The ESP32 controller performs the following tasks:

- Connects to the local Wi-Fi network.
- Exposes all sprinkler zones to Home Assistant.
- Operates the relay board.
- Flashes the onboard LED whenever a relay changes state.
- Automatically turns off any relay after 31 minutes.
- Accepts firmware updates over the network.

Unlike Home Assistant, ESPHome continues to enforce hardware safety even if the automation platform becomes unavailable.

---

# Firmware Structure

The firmware is organised into the following sections:

```
esphome:
esp32:
logger:
api:
ota:
wifi:
output:
script:
switch:
```

Each section performs a specific role.

---

# ESPHome Configuration

```yaml
esphome:
  name: esphome-web-fd4824
  friendly_name: Sprinklers
```

## Purpose

Defines the identity of the controller.

### Name

```
esphome-web-fd4824
```

The internal device identifier used by ESPHome.

### Friendly Name

```
Sprinklers
```

The name displayed within Home Assistant.

---

# ESP32 Platform

```yaml
esp32:
  variant: esp32
  framework:
    type: esp-idf
```

The project uses the ESP-IDF framework rather than the Arduino framework.

### Why ESP-IDF?

Advantages include:

- Official Espressif framework.
- Excellent hardware support.
- Stable networking.
- Good long-term compatibility.
- Reliable GPIO behaviour.

---

# Logging

```yaml
logger:
```

Enables the serial logging system.

During development this allows firmware messages to be viewed directly from ESPHome.

Useful information includes:

- Boot messages
- Wi-Fi status
- Relay activity
- Error messages

---

# Home Assistant API

```yaml
api:
```

Enables native communication with Home Assistant.

This allows Home Assistant to expose every sprinkler zone as a switch entity without requiring MQTT.

---

# Over-The-Air Updates

```yaml
ota:
- platform: esphome
```

Firmware can be uploaded over the network after the initial USB installation.

Benefits include:

- No need to disconnect the controller.
- Faster firmware deployment.
- Safer maintenance.

---

# Wi-Fi Configuration

```yaml
wifi:
```

The Wi-Fi credentials are stored separately using Home Assistant secrets.

```yaml
ssid: !secret wifi_ssid
password: !secret wifi_password
```

This keeps passwords out of the firmware source code.

---

## Power Saving

```yaml
power_save_mode: LIGHT
```

Light power saving reduces energy consumption while maintaining reliable communication with Home Assistant.

---

# Status LED

The ESP32's onboard blue LED is configured as an output.

```yaml
output:
```

GPIO2 is used to provide simple visual feedback.

The LED flashes twice whenever any relay changes state.

This provides immediate confirmation that the controller has received and executed the requested command.

---

# LED Flash Script

Rather than duplicating LED code nine times, the firmware uses a reusable script.

```yaml
script:
```

Whenever a sprinkler turns on or off:

1. LED ON
2. Short delay
3. LED OFF
4. Short delay
5. LED ON
6. Short delay
7. LED OFF

Using a reusable script makes the configuration shorter and easier to maintain.

---

# Relay Outputs

The controller currently exposes nine relay outputs.

Seven are actively used.

Two remain available for future expansion.

Each output follows the same design.

Example:

```yaml
switch:
  - platform: gpio
```

Every switch contains:

- GPIO assignment
- Active-low configuration
- Restore mode
- Turn-on actions
- Turn-off actions

---

# Active-Low Relays

The UCTronics relay board activates when its input is pulled LOW.

ESPHome therefore uses:

```yaml
inverted: true
```

This allows Home Assistant to continue presenting a normal ON/OFF interface while the electrical logic is handled transparently.

---

# Restore Mode

Every output includes:

```yaml
restore_mode: ALWAYS_OFF
```

This ensures that following a reboot or power failure all sprinkler zones remain OFF.

The controller never attempts to restore a previous watering state.

This behaviour prevents accidental irrigation after unexpected restarts.

---

# Automatic Safety Timeout

One of the most important firmware features is the automatic timeout.

Whenever a sprinkler zone is turned on:

```
Relay ON

↓

Start 31 minute timer

↓

Timer expires

↓

Relay OFF
```

This protection exists entirely within ESPHome.

Even if Home Assistant becomes unavailable, the relay will automatically switch off after 31 minutes.

---

# On / Off Actions

Every relay performs the same sequence.

## Turn On

- Flash onboard LED.
- Activate relay.
- Start timeout timer.

## Turn Off

- Deactivate relay.
- Flash onboard LED.

This provides consistent behaviour across every sprinkler zone.

---

# GPIO Allocation

| GPIO | Function |
|------|----------|
| GPIO13 | Garden Sprinkler 1 |
| GPIO14 | Garden Sprinkler 2 |
| GPIO16 | Garden Sprinkler 3 |
| GPIO17 | Garden Sprinkler 4 |
| GPIO18 | Garden Sprinkler 5 |
| GPIO19 | Garden Sprinkler 6 |
| GPIO21 | Garden Sprinkler 7 |
| GPIO22 | Garden Sprinkler 8 (Unused) |
| GPIO23 | Garden Sprinkler 9 (Unused) |
| GPIO2 | Status LED |

---

# Firmware Behaviour

A typical watering sequence is shown below.

```
Home Assistant

↓

Turn Zone ON

↓

ESPHome receives command

↓

LED flashes twice

↓

Relay energises

↓

31 minute timer starts

↓

Home Assistant turns zone OFF

↓

LED flashes twice

↓

Ready for next command
```

If Home Assistant does not issue an OFF command before the timer expires:

```
31 minute timer

↓

Relay OFF

↓

LED flashes twice

↓

Safe state restored
```

---

# Updating the Firmware

Routine firmware updates can normally be performed over Wi-Fi.

Recommended procedure:

1. Open ESPHome.
2. Compile the firmware.
3. Upload wirelessly.
4. Confirm successful reboot.
5. Test one sprinkler zone.

No physical access to the controller is normally required.

---

# Troubleshooting

## Relay Does Not Operate

Check:

- ESP32 online
- Wi-Fi connection
- ESPHome logs
- Relay wiring
- GPIO assignment

---

## Relay Turns Off Unexpectedly

This is usually expected behaviour.

The firmware intentionally switches every relay off after 31 minutes as a hardware safety feature.

---

## Device Offline

Check:

- Power supply
- Wi-Fi credentials
- Home Assistant API connectivity
- ESPHome logs

---

# Key Takeaways

- ESPHome provides the hardware interface for the irrigation system.
- Home Assistant decides when watering occurs.
- ESPHome decides how the relay hardware behaves.
- Every output is active-low.
- Every relay automatically turns off after 31 minutes.
- OTA firmware updates simplify maintenance.
- Hardware safety remains active independently of Home Assistant.

---

## Related Files

| File | Description |
|------|-------------|
| `esphome/sprinklers.yaml` | Complete ESPHome firmware |
| `docs/02-Hardware.md` | Hardware overview |
| `docs/04-Home-Assistant.md` | Home Assistant architecture |

---

## Navigation

⬅️ Previous: [02 – Hardware](02-Hardware.md)

➡️ Next: [04 – Home Assistant](04-Home-Assistant.md)
