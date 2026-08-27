# Appendix F — GPIO & ESP32 Pin Reference

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** ESP32 GPIO & Relay Wiring Reference  
> **Status:** Current configuration reference

## Overview

This appendix documents the GPIO pins used by the ESP32 sprinkler controller and their corresponding functions within the ESPHome configuration.

The sprinkler controller uses an ESP32 development board running ESPHome and a **9-channel active-low UCtronics relay board**. Each relay channel is controlled by an individual ESP32 GPIO pin.

The current Home Assistant automation uses seven sprinkler zones, while the ESPHome configuration provides nine relay channels for future expansion.

> **Important:** This reference is based on the current ESPHome configuration. When rebuilding or modifying the controller, the ESPHome YAML should always be treated as the definitive software configuration.

---

# 1. Pin Allocation Summary

| ESP32 GPIO | Function | ESPHome ID | Relay / Zone |
|---|---|---|---|
| GPIO2 | Onboard blue status LED | `onboard_led` | Status indicator |
| GPIO13 | Relay output | `zone_1` | Garden Sprinkler 1 |
| GPIO14 | Relay output | `zone_2` | Garden Sprinkler 2 |
| GPIO16 | Relay output | `zone_3` | Garden Sprinkler 3 |
| GPIO17 | Relay output | `zone_4` | Garden Sprinkler 4 |
| GPIO18 | Relay output | `zone_5` | Garden Sprinkler 5 |
| GPIO19 | Relay output | `zone_6` | Garden Sprinkler 6 |
| GPIO21 | Relay output | `zone_7` | Garden Sprinkler 7 |
| GPIO22 | Relay output | `zone_8` | Garden Sprinkler 8 — Spare |
| GPIO23 | Relay output | `zone_9` | Garden Sprinkler 9 — Spare |

---

# 2. ESP32 to Relay Board Wiring

The relay board receives one control signal for each channel.

```text
ESP32 GPIO                    Relay Board
──────────                    ───────────

GPIO13  ────────────────────►  IN1
GPIO14  ────────────────────►  IN2
GPIO16  ────────────────────►  IN3
GPIO17  ────────────────────►  IN4
GPIO18  ────────────────────►  IN5
GPIO19  ────────────────────►  IN6
GPIO21  ────────────────────►  IN7
GPIO22  ────────────────────►  IN8
GPIO23  ────────────────────►  IN9
```

The relay board is configured as **active-low**, meaning the ESPHome configuration inverts the GPIO output logic for each relay.

The relevant ESPHome configuration pattern is:

```yaml
pin:
  number: GPIO13
  inverted: true
```

This pattern is repeated for each of the nine relay outputs.

---

# 3. Garden Sprinkler Zone Pin Reference

## Zone 1

| Property | Value |
|---|---|
| Name | Garden Sprinkler 1 |
| ESPHome ID | `zone_1` |
| ESP32 Pin | GPIO13 |
| Home Assistant Entity | `switch.sprinklers_garden_sprinkler_1` |
| Current Status | Active |

## Zone 2

| Property | Value |
|---|---|
| Name | Garden Sprinkler 2 |
| ESPHome ID | `zone_2` |
| ESP32 Pin | GPIO14 |
| Home Assistant Entity | `switch.sprinklers_garden_sprinkler_2` |
| Current Status | Active |

## Zone 3

| Property | Value |
|---|---|
| Name | Garden Sprinkler 3 |
| ESPHome ID | `zone_3` |
| ESP32 Pin | GPIO16 |
| Home Assistant Entity | `switch.sprinklers_garden_sprinkler_3` |
| Current Status | Active |

## Zone 4

| Property | Value |
|---|---|
| Name | Garden Sprinkler 4 |
| ESPHome ID | `zone_4` |
| ESP32 Pin | GPIO17 |
| Home Assistant Entity | `switch.sprinklers_garden_sprinkler_4` |
| Current Status | Active |

## Zone 5

| Property | Value |
|---|---|
| Name | Garden Sprinkler 5 |
| ESPHome ID | `zone_5` |
| ESP32 Pin | GPIO18 |
| Home Assistant Entity | `switch.sprinklers_garden_sprinkler_5` |
| Current Status | Active |

## Zone 6

| Property | Value |
|---|---|
| Name | Garden Sprinkler 6 |
| ESPHome ID | `zone_6` |
| ESP32 Pin | GPIO19 |
| Home Assistant Entity | `switch.sprinklers_garden_sprinkler_6` |
| Current Status | Active |

## Zone 7

| Property | Value |
|---|---|
| Name | Garden Sprinkler 7 |
| ESPHome ID | `zone_7` |
| ESP32 Pin | GPIO21 |
| Home Assistant Entity | `switch.sprinklers_garden_sprinkler_7` |
| Current Status | Active |

## Zone 8

| Property | Value |
|---|---|
| Name | Garden Sprinkler 8 |
| ESPHome ID | `zone_8` |
| ESP32 Pin | GPIO22 |
| Home Assistant Entity | `switch.sprinklers_garden_sprinkler_8` |
| Current Status | Spare / Available |

Zone 8 is configured in ESPHome but is not currently included in the automatic morning watering sequence.

## Zone 9

| Property | Value |
|---|---|
| Name | Garden Sprinkler 9 |
| ESPHome ID | `zone_9` |
| ESP32 Pin | GPIO23 |
| Home Assistant Entity | `switch.sprinklers_garden_sprinkler_9` |
| Current Status | Spare / Available |

Zone 9 is configured in ESPHome but is not currently included in the automatic morning watering sequence.

---

# 4. Onboard Status LED

The ESP32's onboard blue LED is controlled using:

```text
GPIO2
```

## ESPHome Output ID

```text
onboard_led
```

The LED is used by the reusable ESPHome script:

```text
flash_led_twice
```

This script flashes the LED twice whenever a sprinkler relay changes state.

## Status Sequence

```text
Relay switched ON or OFF
            │
            ▼
      flash_led_twice
            │
            ▼
LED ON  ─── 100 ms
LED OFF ─── 100 ms
LED ON  ─── 100 ms
LED OFF
```

The LED is purely a visual status indicator and does not control the sprinkler operation.

---

# 5. Active-Low Relay Operation

The UCtronics relay board used by the sprinkler system is an **active-low** design.

This means that the logic signal used to activate the relay is inverted compared with a conventional active-high output.

ESPHome handles this through:

```yaml
inverted: true
```

Without this setting, the relay behaviour may be reversed.

> **Important:** The active-low behaviour is handled in software. The physical GPIO wiring should not be changed simply because the relay board is active-low.

---

# 6. GPIO Connection Checklist

When connecting or rebuilding the controller, confirm each signal wire against the following list:

```text
Relay IN1  → GPIO13
Relay IN2  → GPIO14
Relay IN3  → GPIO16
Relay IN4  → GPIO17
Relay IN5  → GPIO18
Relay IN6  → GPIO19
Relay IN7  → GPIO21
Relay IN8  → GPIO22
Relay IN9  → GPIO23
```

Before connecting the sprinkler valves, it is recommended to test each relay individually from Home Assistant.

A successful test should:

1. Toggle the correct relay channel.
2. Cause the ESP32 status LED to flash.
3. Show the correct switch state in Home Assistant.
4. Automatically turn the relay off after 31 minutes if it has not already been switched off.

---

# 7. Pin Assignment Diagram

The following is a simplified logical representation of the ESP32 connections.

```text
                 ┌───────────────────────────┐
                 │          ESP32            │
                 │                           │
                 │  GPIO2  ───► Status LED   │
                 │                           │
                 │  GPIO13 ───► Relay IN1    │
                 │  GPIO14 ───► Relay IN2    │
                 │  GPIO16 ───► Relay IN3    │
                 │  GPIO17 ───► Relay IN4    │
                 │  GPIO18 ───► Relay IN5    │
                 │  GPIO19 ───► Relay IN6    │
                 │  GPIO21 ───► Relay IN7    │
                 │  GPIO22 ───► Relay IN8    │
                 │  GPIO23 ───► Relay IN9    │
                 │                           │
                 └───────────────────────────┘
```

This is a **logical connection diagram** rather than a representation of the physical pin positions on the ESP32 development board.

A physical pin-location diagram should be used alongside this appendix when working directly on the controller.

---

# 8. Important Safety Notes

## Do Not Assume Pin Locations

ESP32 development boards can have different physical layouts even when they use the same ESP32 chip.

Always identify the GPIO labels printed on the actual board rather than relying solely on the physical position of a pin.

For example, `GPIO13` is the important identifier used by ESPHome, regardless of where that pin appears on a particular ESP32 board layout.

## Restart Behaviour

All sprinkler relay switches use:

```yaml
restore_mode: ALWAYS_OFF
```

This ensures the relays should remain off after an ESP32 restart.

## Independent Safety Timeout

Each relay includes a 31-minute automatic shut-off within ESPHome.

This provides an independent safety layer in addition to the Home Assistant watering automation.

---

# 9. Troubleshooting Pin Connections

If a sprinkler zone does not operate correctly:

### Check the ESPHome GPIO Assignment

Confirm the GPIO number in the ESPHome configuration matches the physical wire connected to the relay board.

### Check the Relay Channel

Confirm that the expected ESP32 GPIO is connected to the correct relay input.

For example:

```text
Garden Sprinkler 1
       │
       ▼
     GPIO13
       │
       ▼
    Relay IN1
```

### Check Active-Low Configuration

Confirm that:

```yaml
inverted: true
```

is present for the relay output.

### Test the Channel Independently

Use the corresponding Home Assistant switch to test the relay without running the complete watering automation.

### Confirm the Relay Board Power Supply

Confirm that the relay board has an appropriate and stable power supply independent of the ESP32's GPIO signalling requirements.

> Take appropriate care when working with relay wiring and any mains-powered equipment. This GPIO reference documents the control side of the system and does not replace the manufacturer's electrical safety guidance.

---

# 10. Quick Reference Card

For convenience, the complete current GPIO allocation is:

| GPIO | Connection |
|---:|---|
| GPIO2 | Onboard blue status LED |
| GPIO13 | Relay IN1 — Sprinkler 1 |
| GPIO14 | Relay IN2 — Sprinkler 2 |
| GPIO16 | Relay IN3 — Sprinkler 3 |
| GPIO17 | Relay IN4 — Sprinkler 4 |
| GPIO18 | Relay IN5 — Sprinkler 5 |
| GPIO19 | Relay IN6 — Sprinkler 6 |
| GPIO21 | Relay IN7 — Sprinkler 7 |
| GPIO22 | Relay IN8 — Sprinkler 8 |
| GPIO23 | Relay IN9 — Sprinkler 9 |

---

## Related Documentation

- [Hardware Specification](Hardware-Specification.md)
- [ESPHome Firmware Reference](ESPHome-Firmware-Reference.md)
- [Automation Reference](Automation-Reference.md)
- [Installation & Setup](../docs/09-Installation-and-Setup.md)

---

*Home Assistant Smart Sprinkler System — Appendix F: GPIO & ESP32 Pin Reference*
