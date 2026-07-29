# Helper Entities

| | |
|---|---|
| **Project** | Home Assistant Smart Sprinklers |
| **Document** | 05 – Helper Entities |
| **Version** | 1.0 |
| **Platform** | Home Assistant |
| **Status** | Live System |
| **Last Updated** | July 2026 |

---

## Summary

Helper entities provide the user-configurable settings and working values used throughout the sprinkler system.

Unlike sensors, helpers either store user preferences or hold values calculated by the automation system.

Nearly every aspect of the irrigation controller can be adjusted by changing one or more helper values, eliminating the need to edit YAML during normal operation.

---

# Contents

- Helper Overview
- Boolean Helpers
- Numeric Helpers
- Template Sensors
- Helper Relationships
- Typical Configuration
- Key Takeaways

---

# Helper Overview

The project currently uses three categories of entities to store irrigation data.

| Type | Purpose |
|------|---------|
| Input Boolean | Enable or disable features |
| Input Number | Store user settings and calculated values |
| Template Sensor | Derived values calculated from weather data |

The helpers are used by the Home Assistant automations and are not accessed directly by the ESP32 firmware.

---

# Input Boolean Helpers

## Sprinklers Enabled

| Property | Value |
|----------|-------|
| Friendly Name | Sprinklers Enabled |
| Entity ID | `input_boolean.sprinklers_enabled` |
| Type | Input Boolean |
| Default | On |

### Purpose

Acts as the master enable switch for the entire irrigation system.

When turned off:

- No watering calculations are performed.
- No sprinkler zones are operated.
- Existing helper values are retained.

### Used By

- Sprinklers - Calculate Water Requirement
- Sprinklers - Morning Cycle and Soak

---

## Sprinkler Recovery Mode

| Property | Value |
|----------|-------|
| Friendly Name | Sprinkler Recovery Mode |
| Entity ID | `input_boolean.sprinkler_recovery_mode` |
| Type | Input Boolean |
| Default | Off |

### Purpose

Temporarily increases the amount of water applied when the lawn requires recovery from drought stress or prolonged hot weather.

When enabled, the irrigation calculation uses a larger refill factor than normal.

### Used By

- Sprinklers - Calculate Water Requirement

---

# Input Number Helpers

## Sprinkler Soil Moisture

| Property | Value |
|----------|-------|
| Friendly Name | Sprinkler Soil Moisture |
| Entity ID | `input_number.sprinkler_soil_moisture` |

### Purpose

Represents the current virtual soil moisture level.

This value decreases each morning according to estimated evaporation and increases after irrigation.

### Modified By

- Sprinklers - Update Virtual Soil Moisture

---

## Sprinkler Moisture Target

| Property | Value |
|----------|-------|
| Friendly Name | Sprinkler Moisture Target |
| Entity ID | `input_number.sprinkler_moisture_target` |

### Purpose

Defines the desired soil moisture level.

The irrigation system attempts to maintain this value over time.

### Used By

- Sprinklers - Calculate Water Requirement

---

## Sprinkler Rain Skip Threshold

| Property | Value |
|----------|-------|
| Friendly Name | Sprinkler Rain Skip Threshold |
| Entity ID | `input_number.sprinkler_rain_skip_threshold` |

### Purpose

Defines the forecast rainfall required before watering is skipped.

If the forecast rainfall equals or exceeds this value, the calculated watering runtime becomes zero.

### Used By

- Sprinklers - Calculate Water Requirement

---

## Sprinkler Water Target

| Property | Value |
|----------|-------|
| Friendly Name | Sprinkler Water Target |
| Entity ID | `input_number.sprinkler_water_target` |

### Purpose

Stores the calculated amount of water, in millimetres, that should be applied during the current irrigation cycle.

### Modified By

- Sprinklers - Calculate Water Requirement

---

## Sprinkler Total Runtime

| Property | Value |
|----------|-------|
| Friendly Name | Sprinkler Total Runtime |
| Entity ID | `input_number.sprinkler_total_runtime` |

### Purpose

Stores the total runtime, in minutes, required for each irrigation zone.

This value is calculated automatically from the estimated water requirement.

### Used By

- Sprinklers - Morning Cycle and Soak

---

## Sprinkler Cycle Runtime

| Property | Value |
|----------|-------|
| Friendly Name | Sprinkler Cycle Runtime |
| Entity ID | `input_number.sprinkler_cycle_runtime` |

### Purpose

Defines the maximum length of a single watering pass.

If the total runtime exceeds this value, the watering automation performs multiple passes separated by soak periods.

### Used By

- Sprinklers - Calculate Water Requirement
- Sprinklers - Morning Cycle and Soak

---

## Sprinkler Runtime

| Property | Value |
|----------|-------|
| Friendly Name | Sprinkler Runtime |
| Entity ID | `input_number.sprinkler_runtime` |

### Purpose

Stores the maximum runtime allowed for a single watering cycle.

The calculation automation updates this helper using the configured cycle runtime, capped at 30 minutes to align with the ESPHome safety timeout.

### Used By

- Sprinklers - Calculate Water Requirement

> [!NOTE]
> Although the ESPHome firmware allows a maximum runtime of 31 minutes before automatically switching a relay off, this helper deliberately limits individual watering cycles to **30 minutes**, providing an additional one-minute safety margin.

---

## Sprinkler Soak Time

| Property | Value |
|----------|-------|
| Friendly Name | Sprinkler Soak Time |
| Entity ID | `input_number.sprinkler_soak_time` |

### Purpose

Defines the pause between complete watering passes.

The soak period allows water to infiltrate the soil before the next watering cycle begins.

### Used By

- Sprinklers - Morning Cycle and Soak

---

# Template Sensors

## Forecast Rain (24 Hours)

| Property | Value |
|----------|-------|
| Friendly Name | Sprinkler Forecast Rain 24h |
| Entity ID | `sensor.sprinkler_forecast_rain_24h` |
| Type | Template Sensor |

### Purpose

Calculates the total rainfall expected during the next 24 hours using weather forecast data.

This value is compared with the configured rain skip threshold before irrigation is scheduled.

### Used By

- Sprinklers - Calculate Water Requirement

---

# Helper Relationships

The following diagram shows how helper values interact.

```text
Temperature
      │
      ▼

Virtual Soil Moisture

      │

Moisture Target

      │

Rain Skip Threshold

      │

Water Target

      │

Total Runtime

      │

Cycle Runtime

      │

Soak Time

      │

Morning Cycle & Soak
```

---

# Typical Configuration

The default configuration is designed to provide conservative irrigation suitable for a typical domestic lawn.

Most users will only adjust:

- Moisture Target
- Rain Skip Threshold
- Cycle Runtime
- Soak Time

The remaining helpers are generally updated automatically by the irrigation automations.

---

> [!TIP]
> **Design Note**
>
> By exposing the key irrigation parameters as helper entities, the system can be tuned directly from the Home Assistant user interface without modifying automation code or ESPHome firmware.

---

# Key Takeaways

- Helper entities provide the configurable behaviour of the irrigation system.
- Input Booleans enable or disable major features.
- Input Numbers store calculated values and user preferences.
- Template Sensors provide derived weather information.
- Most day-to-day adjustments can be made by changing helper values rather than editing YAML.
- The helpers form the link between weather data, irrigation calculations and sprinkler operation.

---

## Related Files

| File | Description |
|------|-------------|
| `homeassistant/automations.yaml` | Irrigation automations |
| `docs/04-Home-Assistant.md` | Home Assistant architecture |
| `docs/06-Automations.md` | Detailed automation reference |

---

## Navigation

⬅️ Previous: [04 – Home Assistant](04-Home-Assistant.md)

➡️ Next: [06 – Automations](06-Automations.md)
