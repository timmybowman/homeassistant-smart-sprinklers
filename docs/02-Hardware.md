# Hardware

| | |
|---|---|
| **Project** | Home Assistant Smart Sprinklers |
| **Document** | 02 – Hardware |
| **Version** | 1.0 |
| **Controller** | AZ-Delivery ESP32 DevKitC V4 |
| **Status** | Live System |
| **Last Updated** | July 2026 |

---

## Summary

This document describes the hardware used by the Home Assistant Smart Sprinklers project, explains why each component was selected, and documents how the various parts of the system are connected together.

The system has been designed using readily available components that are inexpensive, reliable and easy to replace.

---

# Contents

- Hardware Overview
- ESP32 Controller
- Relay Board
- Sprinkler Valves
- Sprinkler Heads
- Pipework
- Weather Integration
- Electrical Layout
- GPIO Allocation
- Expansion Capability
- Hardware Maintenance

---

# Hardware Overview

The irrigation controller consists of four primary systems:

1. Home Assistant
2. ESP32 Controller
3. Relay Interface
4. Irrigation Hardware

Together these provide a complete irrigation system capable of calculating watering requirements and operating seven independent sprinkler zones.

---

# System Overview

```text
                Home Assistant

                       │
                       │ Wi-Fi
                       │
                       ▼

             ESP32 DevKitC V4

                       │

                GPIO Outputs

                       │

           UCTronics Relay Board

                       │

      Solenoid Valves / Irrigation Zones

                       │

              Rain Bird 5004 Rotors
```

*A detailed SVG version of this diagram is provided in the `images` folder.*

---

# ESP32 Controller

## Controller

| Property | Value |
|-----------|-------|
| Model | AZ-Delivery ESP32 DevKitC V4 |
| MCU | ESP32-WROOM-32 |
| Firmware | ESPHome |
| Communication | Wi-Fi |
| Programming | USB / OTA |
| Power | USB 5V |

The ESP32 acts as the hardware interface between Home Assistant and the relay board.

Home Assistant determines **when** irrigation should occur.

ESPHome determines **how** the relays behave.

This separation keeps hardware safety independent of the automation logic.

---

## Firmware Features

The ESPHome firmware provides:

- Native Home Assistant API
- Over-The-Air firmware updates
- Active-low relay control
- Hardware safety timeout
- Boot-safe outputs
- Status LED indication
- Wi-Fi connectivity

---

# Relay Board

## Relay Module

| Property | Value |
|-----------|-------|
| Manufacturer | UCTronics |
| Channels | 9 |
| Relay Type | Active-Low |
| Used Outputs | 7 |
| Spare Outputs | 2 |

The relay board provides electrical isolation between the ESP32 and the sprinkler valve circuits.

Each relay is controlled from a dedicated GPIO output.

---

## Why Active-Low?

The relay module activates when the GPIO output is pulled LOW.

ESPHome therefore configures every output using:

```yaml
inverted: true
```

This simplifies operation while matching the relay hardware.

---

# GPIO Allocation

| GPIO | Function | Status |
|------|----------|--------|
| GPIO13 | Garden Sprinkler 1 | Used |
| GPIO14 | Garden Sprinkler 2 | Used |
| GPIO16 | Garden Sprinkler 3 | Used |
| GPIO17 | Garden Sprinkler 4 | Used |
| GPIO18 | Garden Sprinkler 5 | Used |
| GPIO19 | Garden Sprinkler 6 | Used |
| GPIO21 | Garden Sprinkler 7 | Used |
| GPIO22 | Garden Sprinkler 8 | Spare |
| GPIO23 | Garden Sprinkler 9 | Spare |
| GPIO2 | On-board Status LED | Used |

The complete ESP32 pinout is documented in `docs/03-ESPHome.md` and illustrated in `images/esp32_pinout.svg`.

---

# Safety Features

Every sprinkler output includes several safety mechanisms.

## Restore Mode

```yaml
restore_mode: ALWAYS_OFF
```

If the ESP32 loses power or restarts unexpectedly, all relay outputs remain OFF until instructed otherwise by Home Assistant.

---

## Automatic Timeout

Whenever a relay is switched on, ESPHome starts a 31-minute timer.

If Home Assistant fails to switch the relay off, ESPHome automatically deactivates the output.

This prevents valves remaining energised indefinitely.

---

## Status LED

The on-board LED flashes twice:

- when a relay turns on
- when a relay turns off

This provides a simple visual indication that commands are being received and executed.

---

# Sprinkler Zones

The current installation uses seven irrigation zones.

| Zone | ESPHome Name | GPIO |
|------|--------------|------|
| 1 | Garden Sprinkler 1 | GPIO13 |
| 2 | Garden Sprinkler 2 | GPIO14 |
| 3 | Garden Sprinkler 3 | GPIO16 |
| 4 | Garden Sprinkler 4 | GPIO17 |
| 5 | Garden Sprinkler 5 | GPIO18 |
| 6 | Garden Sprinkler 6 | GPIO19 |
| 7 | Garden Sprinkler 7 | GPIO21 |

Outputs 8 and 9 are currently reserved for future expansion.

---

# Sprinkler Heads

The irrigation system uses **Rain Bird 5004** rotor sprinklers.

The heads are arranged to provide overlapping coverage across the lawn, improving watering uniformity.

During commissioning, nozzle sizes were selected to balance water distribution while accounting for areas requiring half-circle coverage.

The design aims to maximise even water application across the lawn while minimising dry patches around the edges.

---

# Pipework

The irrigation supply is arranged as follows:

```text
Incoming Water Main
        │
        ▼
15 mm Copper Supply
        │
        ▼
32 mm MDPE Main Feed
        │
        ▼
Valve Manifold
        │
        ▼
20 mm MDPE Laterals
        │
        ▼
Rain Bird 5004 Sprinklers
```

This arrangement provides a low-resistance supply to the valve manifold before distributing water to the individual irrigation zones.

---

# Weather Integration

Weather data is supplied by the Met Office integration within Home Assistant.

The system currently uses:

- Ambient temperature
- Forecast rainfall

These values influence the irrigation calculations but are not processed directly by the ESP32.

---

# Expansion Capability

The current hardware allows several future enhancements.

Potential additions include:

- Flow meter
- Rain sensor
- Pressure sensor
- Additional relay outputs
- Solenoid valve monitoring
- Water usage logging

The two unused relay outputs (GPIO22 and GPIO23) are reserved for future expansion.

---

# Hardware Maintenance

The following maintenance tasks are recommended:

| Task | Frequency |
|------|-----------|
| Inspect sprinkler heads | Monthly |
| Check for blocked nozzles | Monthly |
| Verify relay operation | Every 6 months |
| Test ESPHome OTA updates | Every 6 months |
| Inspect pipework for leaks | Seasonally |
| Winterise irrigation system | Annually |
| Check electrical connections | Annually |

---

# Key Takeaways

- Home Assistant performs all irrigation calculations.
- ESPHome provides reliable hardware control.
- Every relay includes a firmware safety timeout.
- Active-low relay outputs are used throughout.
- Two relay outputs remain available for future expansion.
- The hardware has been designed to be reliable, maintainable and easily expandable.

---

## Navigation

⬅️ Previous: [01 – Introduction](01-Introduction.md)

➡️ Next: [03 – ESPHome](03-ESPHome.md)
