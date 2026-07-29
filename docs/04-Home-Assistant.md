# Home Assistant

| | |
|---|---|
| **Project** | Home Assistant Smart Sprinklers |
| **Document** | 04 – Home Assistant |
| **Version** | 1.0 |
| **Platform** | Home Assistant |
| **Status** | Live System |
| **Last Updated** | July 2026 |

---

## Summary

This document describes the Home Assistant configuration that controls the irrigation system.

Home Assistant acts as the "brain" of the project. It gathers weather information, maintains the virtual soil moisture model, calculates irrigation requirements, and schedules watering. The ESP32 running ESPHome is responsible only for operating the relay hardware safely.

Separating the decision-making from the hardware control keeps the system modular, easier to maintain and more resilient.

---

# Contents

- System Architecture
- Integrations
- Entity Overview
- Helper Entities
- Template Sensors
- Automations
- Daily Operating Sequence
- Data Flow
- Failure Handling
- Related Files
- Key Takeaways

---

# System Architecture

The irrigation system is split into two distinct layers.

## Decision Layer

Implemented entirely within Home Assistant.

Responsibilities include:

- Weather monitoring
- Virtual soil moisture modelling
- Water requirement calculations
- Irrigation scheduling
- User configuration
- Runtime calculations

---

## Hardware Layer

Implemented within ESPHome.

Responsibilities include:

- Relay control
- Status indication
- Safety timeout
- Wi-Fi communication

---

> [!TIP]
> **Design Note**
>
> Home Assistant determines **when** watering should occur.
>
> ESPHome determines **how** the relay hardware behaves.
>
> Keeping these responsibilities separate allows each platform to focus on what it does best.

---

# Integrations

The following integrations are used by the project.

| Integration | Purpose |
|-------------|---------|
| ESPHome | Communicates with the ESP32 controller |
| Met Office | Supplies weather data |
| Helpers | Stores user-configurable values |
| Automations | Performs calculations and schedules watering |
| Template Sensors | Generates calculated values |

---

## ESPHome Integration

Provides communication with the sprinkler controller.

Home Assistant exposes every relay as an individual switch entity.

Example:

```
switch.sprinklers_garden_sprinkler_1
```

These switches are operated by the watering automation.

---

## Met Office Integration

The irrigation calculations rely on weather data supplied by the Met Office integration.

Current weather information used by the project includes:

- Ambient temperature
- Forecast rainfall

This information is updated automatically and forms the basis of the virtual soil moisture calculations.

---

# Entity Overview

The project uses four categories of entities.

| Type | Purpose |
|------|---------|
| Switches | Operate sprinkler zones |
| Helpers | Store user settings and calculated values |
| Sensors | Supply weather data |
| Automations | Perform calculations and watering |

---

# Sprinkler Zones

The current installation consists of seven active zones.

| Zone | Entity |
|------|--------|
| 1 | `switch.sprinklers_garden_sprinkler_1` |
| 2 | `switch.sprinklers_garden_sprinkler_2` |
| 3 | `switch.sprinklers_garden_sprinkler_3` |
| 4 | `switch.sprinklers_garden_sprinkler_4` |
| 5 | `switch.sprinklers_garden_sprinkler_5` |
| 6 | `switch.sprinklers_garden_sprinkler_6` |
| 7 | `switch.sprinklers_garden_sprinkler_7` |

Each switch represents one relay output on the ESP32.

---

# Helper Entities

Helper entities allow almost every aspect of the irrigation system to be configured without modifying YAML.

Current helper categories include:

## Control

- Sprinklers Enabled
- Recovery Mode

---

## Runtime

- Cycle Runtime
- Total Runtime
- Runtime

---

## Moisture

- Soil Moisture
- Moisture Target
- Water Target

---

## Weather

- Rain Skip Threshold

---

## Timing

- Soak Time

---

> [!NOTE]
> A complete reference for every helper is provided in **05 – Helpers.md**.

---

# Template Sensors

Template sensors are used to provide calculated information that is not supplied directly by Home Assistant integrations.

Current template sensors include:

| Sensor | Purpose |
|--------|---------|
| Forecast Rain (24h) | Total rainfall expected over the next 24 hours |

This value is used by the irrigation calculations to determine whether watering should be skipped.

---

# Automations

The project currently consists of three primary automations.

---

## 1. Update Virtual Soil Moisture

Runs every morning.

Purpose:

- Estimate daily evaporation.
- Reduce the virtual soil moisture level.

---

## 2. Calculate Water Requirement

Runs every two days.

Purpose:

- Calculate soil moisture deficit.
- Check rainfall forecast.
- Determine irrigation requirement.
- Calculate total runtime.

---

## 3. Morning Cycle & Soak

Runs at Sunrise +30 minutes.

Purpose:

- Operate each sprinkler zone.
- Divide watering into multiple passes.
- Allow soak periods.
- Complete watering safely.

---

> [!IMPORTANT]
> These automations work together as a pipeline. Each automation prepares data for the next stage of the process.

---

# Daily Operating Sequence

The irrigation controller follows the same sequence each day.

```
03:45

↓

Update Virtual Soil Moisture

↓

04:00

↓

Calculate Water Requirement

↓

Forecast Rain?

↓

Enough Rain
      │
      ├── Yes → Runtime = 0
      │
      └── No

↓

Sunrise +30 Minutes

↓

Morning Cycle & Soak

↓

Complete
```

---

# Data Flow

The following diagram illustrates how information moves through the system.

```
Temperature
        │
        ▼

Virtual Soil Moisture

        │

Rain Forecast

        │

Water Requirement

        │

Total Runtime

        │

Morning Watering

        │

ESPHome

        │

Relay Outputs

        │

Sprinklers
```

---

# Failure Handling

The project has been designed so that failures fail safely.

## Home Assistant Restart

If Home Assistant restarts:

- Calculations resume automatically.
- ESPHome remains connected.
- Relays remain OFF until instructed otherwise.

---

## ESP32 Restart

If the ESP32 restarts:

- Every relay defaults to OFF.
- Home Assistant reconnects automatically.
- Safety timeout remains active.

---

## Weather Unavailable

If weather information is unavailable:

The irrigation calculations may produce reduced or zero runtime depending on the availability of required sensor values.

Checking automation traces is recommended when diagnosing weather-related issues.

---

# Related Files

| File | Description |
|------|-------------|
| `homeassistant/automations.yaml` | Home Assistant automations |
| `docs/05-Helpers.md` | Helper reference |
| `docs/06-Automations.md` | Detailed automation documentation |
| `docs/03-ESPHome.md` | ESPHome firmware |

---

# Design Philosophy

The irrigation controller has been designed to minimise manual intervention.

Rather than asking the user to decide how long the sprinklers should run each day, Home Assistant determines the appropriate runtime using environmental data and configurable targets.

This approach keeps the day-to-day operation simple while allowing advanced behaviour to remain hidden behind a small number of helper entities.

---

# Key Takeaways

- Home Assistant is responsible for all irrigation decisions.
- ESPHome controls the relay hardware.
- Weather data influences irrigation requirements.
- Helper entities allow the system to be adjusted without editing YAML.
- Three automations work together to calculate and perform watering.
- The design separates calculation from hardware control for improved reliability and maintainability.

---

## Navigation

⬅️ Previous: [03 – ESPHome Firmware](03-ESPHome.md)

➡️ Next: [05 – Helpers](05-Helpers.md)
