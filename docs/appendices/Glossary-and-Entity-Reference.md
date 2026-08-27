# Appendix M — Glossary & Entity Reference

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** Quick Reference Guide  
> **Status:** Current system reference

## Overview

This appendix provides a quick-reference guide to the terminology, entity IDs, devices, automations and configuration components used by the Home Assistant Smart Sprinkler System.

It is intended to be used when:

- Reading or editing automations.
- Troubleshooting the system.
- Rebuilding Home Assistant or ESPHome.
- Checking entity IDs.
- Reviewing documentation.
- Making future improvements to the irrigation system.

The system can be viewed as four connected layers:

```text
Weather Data & Irrigation Model
              │
              ▼
      Home Assistant Logic
              │
              ▼
       ESPHome Controller
              │
              ▼
    Relays, Valves & Sprinklers
```

---

# 1. Quick System Reference

| Component | Current Reference |
|---|---|
| Home Automation Platform | Home Assistant |
| ESPHome Device Name | `esphome-web-fd4824` |
| ESPHome Friendly Name | `Sprinklers` |
| Controller | ESP32 |
| Framework | ESP-IDF |
| Active Garden Zones | 7 |
| Available ESPHome Relay Channels | 9 |
| Relay Type | Active-Low |
| Normal Watering Trigger | Sunrise + 30 minutes |
| Virtual Soil Moisture Update | 03:45 |
| Water Requirement Calculation | 04:00 |
| ESPHome Safety Timeout | 31 minutes per zone |

---

# 2. Core Automations

The sprinkler system currently uses three core Home Assistant automations.

| Automation Name | Purpose |
|---|---|
| `Sprinklers - Update Virtual Soil Moisture` | Updates the virtual soil moisture model based on temperature |
| `Sprinklers - Calculate Water Requirement` | Calculates irrigation requirements using moisture and rainfall information |
| `Sprinklers - Morning Cycle and Soak` | Controls the sequential watering cycle across the seven active zones |

## Automation Flow

```text
Sprinklers - Update Virtual Soil Moisture
                │
                ▼
Sprinklers - Calculate Water Requirement
                │
                ▼
Sprinklers - Morning Cycle and Soak
                │
                ▼
          ESP32 / ESPHome
                │
                ▼
        Sprinkler Zones 1–7
```

---

# 3. Input Boolean Entities

Input booleans are used as simple on/off controls.

## Sprinklers Enabled

**Entity ID:**

```text
input_boolean.sprinklers_enabled
```

**Purpose:**  
Master enable control for the sprinkler system.

When this helper is off, the calculation and watering automations are prevented from running under their normal conditions.

---

## Sprinkler Recovery Mode

**Entity ID:**

```text
input_boolean.sprinkler_recovery_mode
```

**Purpose:**  
Enables a more aggressive irrigation calculation when the lawn is recovering from dry or stressed conditions.

The water requirement calculation uses a different deficit multiplier depending on whether recovery mode is enabled.

---

# 4. Input Number Entities

Input numbers store configuration values and calculated values used by the irrigation system.

## Sprinkler Cycle Runtime

**Entity ID:**

```text
input_number.sprinkler_cycle_runtime
```

**Purpose:**  
Defines the maximum runtime for each watering pass before the system moves into a soak period.

This value is used by the cycle-and-soak automation.

---

## Sprinkler Moisture Target

**Entity ID:**

```text
input_number.sprinkler_moisture_target
```

**Purpose:**  
Represents the desired target value for the virtual soil moisture model.

The water requirement automation compares the current virtual soil moisture value against this target.

---

## Sprinkler Rain Skip Threshold

**Entity ID:**

```text
input_number.sprinkler_rain_skip_threshold
```

**Purpose:**  
Defines the forecast rainfall level at which automatic irrigation is skipped.

If the 24-hour rainfall forecast meets or exceeds this threshold, the water requirement and total runtime are set to zero.

---

## Sprinkler Runtime

**Entity ID:**

```text
input_number.sprinkler_runtime
```

**Purpose:**  
Stores a runtime value used as part of the sprinkler system configuration.

The current water requirement automation updates this helper using the configured cycle runtime, limited to a maximum of 30 minutes.

> **Note:** The main cycle-and-soak automation uses `input_number.sprinkler_total_runtime` and `input_number.sprinkler_cycle_runtime` to control watering.

---

## Sprinkler Soak Time

**Entity ID:**

```text
input_number.sprinkler_soak_time
```

**Purpose:**  
Defines the delay between complete watering passes when additional runtime remains.

The soak period allows water to absorb into the soil before the next pass begins.

---

## Sprinkler Soil Moisture

**Entity ID:**

```text
input_number.sprinkler_soil_moisture
```

**Purpose:**  
Stores the current value of the virtual soil moisture model.

This value is reduced by the virtual soil moisture automation and is used by the water requirement calculation.

It is a calculated model value rather than a reading from a physical soil moisture sensor.

---

## Sprinkler Total Runtime

**Entity ID:**

```text
input_number.sprinkler_total_runtime
```

**Purpose:**  
Stores the calculated total amount of watering required for a zone.

The cycle-and-soak automation divides this total into individual watering passes using the cycle runtime.

---

## Sprinkler Water Target

**Entity ID:**

```text
input_number.sprinkler_water_target
```

**Purpose:**  
Stores the calculated amount of irrigation required by the water requirement automation.

This value represents the calculated water requirement before it is converted into the corresponding runtime.

---

# 5. Weather and Template Sensors

The irrigation model uses weather information to estimate moisture loss and forecast rainfall.

## Met Office Temperature

**Entity ID:**

```text
sensor.met_office_ashby_de_la_zouch_temperature
```

**Purpose:**  
Provides the temperature value used by the virtual soil moisture automation.

The temperature is used to select an estimated daily moisture loss value.

---

## Sprinkler Forecast Rain 24h

**Entity ID:**

```text
sensor.sprinkler_forecast_rain_24h
```

**Type:** Template Sensor

**Purpose:**  
Provides the forecast rainfall value used by the water requirement automation.

The forecast is compared against:

```text
input_number.sprinkler_rain_skip_threshold
```

to determine whether watering should be skipped.

---

# 6. ESPHome Sprinkler Switches

The ESP32 controller exposes individual Home Assistant switches for the sprinkler relay channels.

The current automatic irrigation system uses the first seven zones.

| Zone | Home Assistant Entity |
|---|---|
| Garden Sprinkler 1 | `switch.sprinklers_garden_sprinkler_1` |
| Garden Sprinkler 2 | `switch.sprinklers_garden_sprinkler_2` |
| Garden Sprinkler 3 | `switch.sprinklers_garden_sprinkler_3` |
| Garden Sprinkler 4 | `switch.sprinklers_garden_sprinkler_4` |
| Garden Sprinkler 5 | `switch.sprinklers_garden_sprinkler_5` |
| Garden Sprinkler 6 | `switch.sprinklers_garden_sprinkler_6` |
| Garden Sprinkler 7 | `switch.sprinklers_garden_sprinkler_7` |

The ESPHome firmware also defines Garden Sprinklers 8 and 9 for future or additional use.

---

# 7. ESPHome Internal Zone IDs

Within the ESPHome firmware, each relay channel has an internal ID.

| Sprinkler | ESPHome ID |
|---|---|
| Garden Sprinkler 1 | `zone_1` |
| Garden Sprinkler 2 | `zone_2` |
| Garden Sprinkler 3 | `zone_3` |
| Garden Sprinkler 4 | `zone_4` |
| Garden Sprinkler 5 | `zone_5` |
| Garden Sprinkler 6 | `zone_6` |
| Garden Sprinkler 7 | `zone_7` |
| Garden Sprinkler 8 | `zone_8` |
| Garden Sprinkler 9 | `zone_9` |

These IDs are used internally by the ESPHome firmware, including the independent safety shut-off actions.

---

# 8. ESP32 GPIO Reference

The current ESPHome firmware uses the following GPIO assignments.

| Sprinkler | ESP32 GPIO |
|---|---:|
| Garden Sprinkler 1 | GPIO13 |
| Garden Sprinkler 2 | GPIO14 |
| Garden Sprinkler 3 | GPIO16 |
| Garden Sprinkler 4 | GPIO17 |
| Garden Sprinkler 5 | GPIO18 |
| Garden Sprinkler 6 | GPIO19 |
| Garden Sprinkler 7 | GPIO21 |
| Garden Sprinkler 8 | GPIO22 |
| Garden Sprinkler 9 | GPIO23 |

The onboard ESP32 LED is configured on:

```text
GPIO2
```

The relay outputs are configured as:

```yaml
inverted: true
```

because the relay board is active-low.

---

# 9. Key Irrigation Terminology

## Active-Low Relay

A relay configuration where the output is activated when the control signal is driven low.

In the ESPHome configuration, this is handled using:

```yaml
inverted: true
```

---

## Cycle and Soak

A watering technique where the required irrigation is divided into shorter watering periods separated by soak periods.

The general sequence is:

```text
Water
  │
  ▼
Soak
  │
  ▼
Water
  │
  ▼
Soak
  │
  ▼
Repeat if Required
```

The purpose is to give water time to absorb into the soil rather than applying the entire amount continuously.

---

## Cycle Runtime

The maximum amount of time a zone runs during one watering pass.

The current value is stored in:

```text
input_number.sprinkler_cycle_runtime
```

---

## Soak Time

The delay between complete watering passes.

The current value is stored in:

```text
input_number.sprinkler_soak_time
```

---

## Total Runtime

The calculated total watering time required by the irrigation model.

The current value is stored in:

```text
input_number.sprinkler_total_runtime
```

This total may be divided into multiple shorter cycles.

---

## Virtual Soil Moisture

A calculated estimate of soil moisture maintained by Home Assistant.

The system does not currently depend on a physical soil moisture sensor.

The value is stored in:

```text
input_number.sprinkler_soil_moisture
```

---

## Moisture Target

The desired level used by the irrigation model when calculating a moisture deficit.

The current target is stored in:

```text
input_number.sprinkler_moisture_target
```

---

## Moisture Deficit

The difference between the configured moisture target and the current virtual soil moisture value.

Conceptually:

```text
Moisture Target
      -
Current Soil Moisture
      =
Moisture Deficit
```

The water requirement automation uses this deficit as part of its irrigation calculation.

---

## Rain Skip Threshold

The amount of forecast rainfall that causes the system to skip automatic irrigation.

The value is stored in:

```text
input_number.sprinkler_rain_skip_threshold
```

---

## Recovery Mode

A manual mode intended for periods when the lawn requires additional irrigation recovery.

When enabled, the water requirement calculation uses a larger proportion of the calculated moisture deficit.

---

# 10. Virtual Soil Moisture Loss Levels

The current virtual soil moisture automation estimates daily moisture loss using temperature ranges.

| Temperature | Estimated Daily Loss |
|---|---:|
| Below 10°C | 1 |
| 10°C to below 15°C | 3 |
| 15°C to below 20°C | 5 |
| 20°C to below 25°C | 8 |
| 25°C to below 30°C | 12 |
| 30°C and above | 15 |

These values are part of the current virtual soil moisture model and may be refined in the future as the system is calibrated against real garden conditions.

---

# 11. Water Requirement Calculation Terms

The calculation automation uses several values to determine whether irrigation is required.

```text
Current Virtual Soil Moisture
              │
              ▼
       Compare to Target
              │
              ▼
       Calculate Deficit
              │
              ▼
Check Forecast Rain & Recovery Mode
              │
              ▼
     Calculate Water Required
              │
              ▼
      Convert to Runtime
```

The current calculation uses different deficit multipliers:

| Mode | Current Multiplier |
|---|---:|
| Normal | 0.30 |
| Recovery Mode | 0.45 |

The calculated water requirement is then converted to runtime using the current assumed irrigation rate.

---

# 12. Irrigation Rate Reference

The current water requirement calculation uses an assumed precipitation rate of:

```text
12 mm per hour
```

This value is used to convert calculated water requirements into sprinkler runtime.

Conceptually:

```text
Water Required
      │
      ▼
Divide by 12 mm/hour
      │
      ▼
Convert Hours to Minutes
      │
      ▼
Total Runtime
```

The assumed precipitation rate can be refined through practical irrigation calibration.

---

# 13. Safety Terminology

## Restore Mode

The ESPHome sprinkler switches use:

```yaml
restore_mode: ALWAYS_OFF
```

This is intended to ensure that relay outputs start in the off state after the ESP32 restarts.

---

## Safety Timeout

Each sprinkler zone includes an independent automatic shut-off action.

The current firmware turns a zone off after:

```text
31 minutes
```

This provides a safeguard if a sprinkler switch is accidentally left on.

The normal Home Assistant watering logic should not rely on this timeout for routine operation.

---

# 14. Common Entity Naming Patterns

The sprinkler system uses consistent naming conventions.

## Automations

```text
Sprinklers - [Function Name]
```

Examples:

```text
Sprinklers - Update Virtual Soil Moisture
Sprinklers - Calculate Water Requirement
Sprinklers - Morning Cycle and Soak
```

## Helpers

```text
input_number.sprinkler_[function]
input_boolean.sprinkler_[function]
```

Examples:

```text
input_number.sprinkler_soil_moisture
input_number.sprinkler_total_runtime
input_boolean.sprinkler_recovery_mode
```

## Sprinkler Zones

```text
switch.sprinklers_garden_sprinkler_[number]
```

Examples:

```text
switch.sprinklers_garden_sprinkler_1
switch.sprinklers_garden_sprinkler_7
```

Maintaining these naming patterns will help keep future additions understandable and consistent.

---

# 15. Quick Entity Lookup Table

| Description | Entity ID |
|---|---|
| Master sprinkler enable | `input_boolean.sprinklers_enabled` |
| Recovery mode | `input_boolean.sprinkler_recovery_mode` |
| Cycle runtime | `input_number.sprinkler_cycle_runtime` |
| Moisture target | `input_number.sprinkler_moisture_target` |
| Rain skip threshold | `input_number.sprinkler_rain_skip_threshold` |
| Sprinkler runtime | `input_number.sprinkler_runtime` |
| Soak time | `input_number.sprinkler_soak_time` |
| Virtual soil moisture | `input_number.sprinkler_soil_moisture` |
| Total runtime | `input_number.sprinkler_total_runtime` |
| Water target | `input_number.sprinkler_water_target` |
| Temperature | `sensor.met_office_ashby_de_la_zouch_temperature` |
| 24-hour rainfall forecast | `sensor.sprinkler_forecast_rain_24h` |

---

# 16. Quick Automation Lookup

| Automation | Trigger | Primary Function |
|---|---|---|
| `Sprinklers - Update Virtual Soil Moisture` | 03:45 | Estimates daily moisture loss |
| `Sprinklers - Calculate Water Requirement` | 04:00 | Calculates irrigation requirements |
| `Sprinklers - Morning Cycle and Soak` | Sunrise + 30 minutes | Waters zones using cycle and soak |

---

# 17. Quick Troubleshooting Reference

| Symptom | First Area to Check |
|---|---|
| No watering occurs | `input_boolean.sprinklers_enabled` |
| ESP32 switches unavailable | ESPHome device and Wi-Fi connection |
| One zone does not operate | Relay channel, wiring or valve |
| All zones fail | ESP32, relay power or water supply |
| Runtime is zero | Water requirement calculation and rain skip logic |
| Watering is skipped unexpectedly | Forecast rain sensor and threshold |
| Soil moisture does not change | Virtual soil moisture automation |
| Incorrect watering sequence | Morning Cycle and Soak automation |
| Sprinkler remains on too long | ESPHome safety timeout and automation |

---

# 18. Related Documentation

For more detailed information, refer to:

- [Automation Reference](Automation-Reference.md)
- [Helper Reference](Helper-Reference.md)
- [Template Sensor Reference](Template-Sensor-Reference.md)
- [Hardware Specification](Hardware-Specification.md)
- [GPIO & ESP32 Pin Reference](GPIO-ESP32-Pin-Reference.md)
- [ESPHome Firmware Reference](ESPHome-Firmware-Reference.md)
- [Irrigation Calibration](Irrigation-Calibration.md)
- [Installation Troubleshooting](Installation-Troubleshooting.md)
- [System Maintenance](System-Maintenance.md)
- [Quick Recovery & Rebuild Guide](Quick-Recovery-and-Rebuild-Guide.md)
- [Testing & Commissioning Checklist](Testing-and-Commissioning-Checklist.md)

---

## Glossary Summary

```text
Virtual Soil Moisture
    A calculated estimate of soil moisture.

Moisture Deficit
    The difference between the target and current moisture level.

Cycle and Soak
    Watering in shorter passes with absorption time between them.

Total Runtime
    The total calculated watering time required.

Cycle Runtime
    The maximum duration of one watering pass.

Soak Time
    The delay between watering passes.

Rain Skip Threshold
    The forecast rainfall amount that prevents automatic watering.

Recovery Mode
    A mode that increases the calculated irrigation response.

Active-Low Relay
    A relay activated by a low control signal.

Safety Timeout
    The ESPHome automatic shut-off after 31 minutes.
```

---

*Home Assistant Smart Sprinkler System — Appendix M: Glossary & Entity Reference*
