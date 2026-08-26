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
