# Appendix E — Hardware Specification

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** Hardware & Physical Installation  
> **Status:** Current configuration reference

## Overview

The Home Assistant Smart Sprinkler System uses an ESP32 running ESPHome to control a multi-channel relay board. The relay board operates the individual garden sprinkler zones, allowing Home Assistant to run each zone independently.

The irrigation system currently has **seven active sprinkler zones**, although the relay controller and ESPHome configuration provide **nine available channels**, allowing for future expansion.

The overall hardware system consists of:

- An **ESP32 development board** running ESPHome.
- A **UCtronics 9-channel active-low relay board**.
- Seven currently configured and active sprinkler zones.
- Individual irrigation valves connected to the relay outputs.
- Rain Bird 5004 sprinkler heads.
- A mains water supply feeding the garden irrigation manifold.

---

# 1. Hardware Architecture

The basic control path is:

```text
Home Assistant
      │
      │ Home Assistant API / Wi-Fi
      ▼
ESP32 running ESPHome
      │
      │ GPIO control signals
      ▼
9-Channel Relay Board
      │
      ├── Channel 1 ──► Sprinkler Zone 1
      ├── Channel 2 ──► Sprinkler Zone 2
      ├── Channel 3 ──► Sprinkler Zone 3
      ├── Channel 4 ──► Sprinkler Zone 4
      ├── Channel 5 ──► Sprinkler Zone 5
      ├── Channel 6 ──► Sprinkler Zone 6
      ├── Channel 7 ──► Sprinkler Zone 7
      ├── Channel 8 ──► Spare / Future Expansion
      └── Channel 9 ──► Spare / Future Expansion
```

Only one sprinkler zone is normally operated at a time by the Home Assistant watering automation.

---

# 2. ESP32 Controller

## Board

The sprinkler controller uses an ESP32 development board based on the **ESP32-WROOM-32** platform.

The board runs ESPHome and connects to Home Assistant over Wi-Fi.

## ESPHome Device Details

| Property | Value |
|---|---|
| ESPHome Device Name | `esphome-web-fd4824` |
| Friendly Name | Sprinklers |
| Platform | ESP32 |
| Framework | ESP-IDF |
| Minimum ESPHome Version | `2026.4.0` |
| Network Connection | Wi-Fi |
| Home Assistant Connection | ESPHome API |
| Firmware Updates | OTA enabled |

## Power Saving

The ESPHome configuration uses:

```text
power_save_mode: LIGHT
```

This provides reduced Wi-Fi power consumption while retaining network connectivity for the sprinkler controller.

---

# 3. Relay Controller

The ESP32 controls a **UCtronics 9-channel active-low relay board**.

An active-low relay board means that the ESP32 output logic is inverted so that the appropriate GPIO state activates the relay.

The ESPHome configuration handles this using:

```yaml
inverted: true
```

for each relay output.

## Available Relay Channels

| Relay Channel | ESPHome Zone | Current Use |
|---|---|---|
| 1 | Garden Sprinkler 1 | Active |
| 2 | Garden Sprinkler 2 | Active |
| 3 | Garden Sprinkler 3 | Active |
| 4 | Garden Sprinkler 4 | Active |
| 5 | Garden Sprinkler 5 | Active |
| 6 | Garden Sprinkler 6 | Active |
| 7 | Garden Sprinkler 7 | Active |
| 8 | Garden Sprinkler 8 | Spare / Available |
| 9 | Garden Sprinkler 9 | Spare / Available |

The final two channels are available for future sprinkler zones or other garden automation.

---

# 4. Active Sprinkler Zones

The Home Assistant irrigation automation currently operates seven zones.

| Zone | Home Assistant Entity |
|---|---|
| Zone 1 | `switch.sprinklers_garden_sprinkler_1` |
| Zone 2 | `switch.sprinklers_garden_sprinkler_2` |
| Zone 3 | `switch.sprinklers_garden_sprinkler_3` |
| Zone 4 | `switch.sprinklers_garden_sprinkler_4` |
| Zone 5 | `switch.sprinklers_garden_sprinkler_5` |
| Zone 6 | `switch.sprinklers_garden_sprinkler_6` |
| Zone 7 | `switch.sprinklers_garden_sprinkler_7` |

The `Sprinklers - Morning Cycle and Soak` automation operates these zones sequentially.

This means the system is designed so that the available water supply is directed to one zone at a time rather than attempting to operate all sprinkler heads simultaneously.

---

# 5. ESP32 GPIO Allocation

The following GPIO pins are allocated to the nine relay channels.

| Zone | ESPHome Switch ID | GPIO Pin |
|---|---|---:|
| Garden Sprinkler 1 | `zone_1` | GPIO13 |
| Garden Sprinkler 2 | `zone_2` | GPIO14 |
| Garden Sprinkler 3 | `zone_3` | GPIO16 |
| Garden Sprinkler 4 | `zone_4` | GPIO17 |
| Garden Sprinkler 5 | `zone_5` | GPIO18 |
| Garden Sprinkler 6 | `zone_6` | GPIO19 |
| Garden Sprinkler 7 | `zone_7` | GPIO21 |
| Garden Sprinkler 8 | `zone_8` | GPIO22 |
| Garden Sprinkler 9 | `zone_9` | GPIO23 |

## Onboard LED

The ESP32's onboard blue LED is controlled using:

```text
GPIO2
```

The LED is used as a visual activity indicator and flashes when a sprinkler relay is switched on or off.

---

# 6. Relay Safety Behaviour

Each relay switch in the ESPHome configuration uses:

```text
restore_mode: ALWAYS_OFF
```

This is an important safety feature.

If the ESP32 restarts or loses power, the sprinkler relay switches are configured to return to the **off** state rather than automatically restoring their previous state.

## Automatic Safety Timeout

Each sprinkler relay also contains an independent automatic shutdown timer.

When a relay is turned on, ESPHome waits for:

```text
31 minutes
```

and then automatically turns that relay off.

This acts as a safety backstop in case Home Assistant fails to send the expected switch-off command.

The normal Home Assistant irrigation cycles should remain below this limit.

```text
Home Assistant automation
        │
        ▼
Normal scheduled switch-off
        │
        │ If this fails...
        ▼
ESPHome 31-minute safety timeout
        │
        ▼
Relay automatically switched off
```

> The safety timeout is not intended to control normal watering duration. It provides an additional layer of protection against a relay remaining energised unexpectedly.

---

# 7. Visual Status Indicator

The ESPHome configuration includes a reusable script named:

```text
flash_led_twice
```

Whenever a sprinkler relay turns on or off, the onboard LED flashes twice.

This provides a simple visual indication that the ESP32 has received and processed a relay command.

The LED sequence is:

```text
ON  →  100 ms
OFF →  100 ms
ON  →  100 ms
OFF
```

---

# 8. Sprinkler Heads

The garden uses **Rain Bird 5004** sprinkler heads.

The installation includes a mixture of full-sweep and half-sweep coverage.

The full-sweep sprinklers use the largest nozzle configuration, while the sprinklers covering approximately half the area use a reduced-flow nozzle arrangement to help balance the water distribution.

The sprinkler heads are arranged along the garden to provide overlapping coverage.

## Coverage Strategy

The sprinklers are positioned along the length of the garden, alternating from side to side.

This arrangement is intended to provide relatively even coverage across the main lawn area.

Some dry areas have been observed around the outer edges of the garden, which can be useful information when refining the irrigation runtime or sprinkler positioning in the future.

---

# 9. Water Supply and Pipework

The irrigation system is supplied from the property's mains water supply.

The current physical arrangement is:

```text
Main Stopcock
      │
      ▼
Approximately 1 metre of 15 mm Copper Pipe
      │
      ▼
Approximately 10–15 metres of 32 mm MDPE
      │
      ▼
Irrigation Manifold
      │
      ▼
Individual 20 mm MDPE Runs
      │
      ▼
Sprinkler Heads
```

The individual runs from the manifold to the sprinkler heads vary in length.

The approximate shortest run is:

```text
7 metres
```

The approximate longest run is:

```text
35 metres
```

The differing pipe lengths and the mixture of sprinkler sweep patterns are factors to consider when refining the real-world watering performance.

---

# 10. Irrigation Layout

The garden is a long, relatively straight layout with sprinkler heads arranged along its length.

The general design principle is:

```text
Side A        Side B

Sprinkler 1
              Sprinkler 2
Sprinkler 3
              Sprinkler 4
Sprinkler 5
              Sprinkler 6
Sprinkler 7
```

The alternating arrangement is designed to distribute water across the width of the garden rather than concentrating the sprinkler heads along a single side.

> The exact physical layout, distances and coverage areas can be documented further alongside the final garden irrigation diagrams.

---

# 11. Estimated Precipitation Rate

The current Home Assistant irrigation calculations use an assumed precipitation rate of:

```text
12 mm per hour
```

This figure is used to convert the calculated water requirement into a sprinkler runtime.

The value is currently a starting estimate and may be refined in the future based on observed lawn condition and real-world watering performance.

The basic calculation used by the automation is:

```text
Runtime (minutes) = Water Required (mm) ÷ 12 × 60
```

Accurate precipitation measurements could allow this value to be improved over time.

---

# 12. Hardware Safety Features

The system includes several layers of protection against unintended watering.

| Safety Feature | Protection Provided |
|---|---|
| `restore_mode: ALWAYS_OFF` | Relays remain off following a restart |
| 31-minute ESPHome timeout | Prevents a relay remaining on indefinitely |
| Master Sprinklers Enabled helper | Allows automatic watering to be disabled |
| Sequential zone operation | Prevents all zones operating together |
| Cycle runtime control | Limits normal continuous watering periods |

These safeguards are intentionally spread across both Home Assistant and ESPHome, providing protection even if one part of the system does not behave as expected.

---

# 13. Future Hardware Expansion

The relay board and ESPHome configuration already provide two additional channels:

- Garden Sprinkler 8
- Garden Sprinkler 9

These could be used in the future for:

- Additional irrigation zones.
- A separate flower bed watering circuit.
- Greenhouse irrigation.
- Other garden water controls.

Any future expansion should also be added to the Home Assistant watering automation if the new zone is intended to participate in the automatic cycle-and-soak schedule.

---

# Hardware Rebuild Checklist

When rebuilding the physical controller:

- [ ] Install and power the ESP32 controller.
- [ ] Flash the ESPHome sprinkler configuration.
- [ ] Confirm Wi-Fi connectivity.
- [ ] Confirm the device appears in Home Assistant.
- [ ] Connect the relay board to the configured GPIO pins.
- [ ] Confirm the relay board is configured for active-low operation.
- [ ] Test each relay channel individually.
- [ ] Confirm all relays default to off after a restart.
- [ ] Confirm the 31-minute safety timeout operates correctly.
- [ ] Connect and test each sprinkler zone.
- [ ] Confirm only one zone operates at a time during automatic watering.
- [ ] Check sprinkler coverage and adjust the irrigation system if required.

---

## Related Documentation

- [ESPHome Firmware Reference](ESPHome-Firmware-Reference.md)
- [GPIO Reference](GPIO-Reference.md)
- [Automation Reference](Automation-Reference.md)
- [Helper Reference](Helper-Reference.md)
- [Irrigation Logic](../docs/07-Irrigation-Logic.md)
- [Installation & Setup](../docs/09-Installation-and-Setup.md)

---

*Home Assistant Smart Sprinkler System — Appendix E: Hardware Specification*
