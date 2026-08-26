# Appendix D — Template Sensor Reference

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** Home Assistant Template Sensor  
> **Status:** Current configuration reference

## Overview

The sprinkler system uses a template sensor to provide the estimated amount of rainfall forecast over the next 24 hours.

This value is used by the irrigation calculation to decide whether automatic watering should be skipped because sufficient rainfall is expected.

The sensor is a Home Assistant **Template** entity with the following details:

| Property | Value |
|---|---|
| Name | Sprinkler Forecast Rain 24h |
| Entity ID | `sensor.sprinkler_forecast_rain_24h` |
| Platform | Template |
| Unit of Measurement | `mm` |
| Icon | `mdi:weather-rainy` |

---

# Sprinkler Forecast Rain 24h

## Entity ID

```text
sensor.sprinkler_forecast_rain_24h
```

## Purpose

This sensor represents the amount of rainfall forecast for the next 24 hours, expressed in millimetres.

It provides the rainfall information required by the sprinkler system without requiring the main irrigation automation to perform the rainfall calculation itself.

The sensor acts as an input to the watering decision:

```text
24-Hour Rainfall Forecast
          │
          ▼
Sprinkler Forecast Rain 24h
          │
          ▼
Compare with Rain Skip Threshold
          │
          ├── Forecast is sufficient ──► Skip watering
          │
          └── Forecast is insufficient ─► Calculate watering requirement
```

---

# Role in the Irrigation System

The sensor is used by:

**Sprinklers - Calculate Water Requirement**

The automation reads the sensor value using:

```jinja
{{ states('sensor.sprinkler_forecast_rain_24h') | float }}
```

The resulting value is stored temporarily as `forecast_rain` during the automation.

It is then compared with:

```text
input_number.sprinkler_rain_skip_threshold
```

---

# Rain Skip Logic

The sprinkler system checks whether the forecast rainfall is greater than or equal to the configured rain skip threshold.

The logic is:

```text
Forecast Rain ≥ Rain Skip Threshold
```

If this condition is true, the system assumes that sufficient rainfall is expected and cancels the calculated watering requirement.

The following values are set to zero:

```text
input_number.sprinkler_water_target
input_number.sprinkler_total_runtime
```

This prevents the morning watering automation from running.

## Decision Flow

```text
          Forecast Rain
                │
                ▼
     Is rain ≥ skip threshold?
             │       │
            Yes      No
             │       │
             ▼       ▼
        Skip watering  Continue
             │       │
             ▼       ▼
       Runtime = 0   Check soil
                    moisture level
```

---

# Relationship with the Rain Skip Threshold

The amount of rainfall required to skip watering is controlled separately by the following helper:

```text
input_number.sprinkler_rain_skip_threshold
```

This separation is useful because the rainfall sensor can continue to calculate and report the forecast independently, while the required amount of rain can be adjusted from the Home Assistant interface.

For example:

```text
Forecast Rain:       6 mm
Rain Skip Threshold: 5 mm

Result: Watering is skipped
```

If the forecast is below the configured threshold, the system continues with its normal moisture deficit calculation.

---

# Sensor State

The sensor reports its value in:

```text
mm
```

A value of:

```text
0
```

means that the sensor is currently reporting no forecast rainfall within the value it is calculating.

The value should be interpreted as an irrigation input rather than a direct measurement from a physical rain gauge.

---

# Current Configuration Information

The current Home Assistant entity configuration identifies this sensor as a template entity named **Sprinkler Forecast Rain 24h**, using millimetres as its unit of measurement and the `mdi:weather-rainy` icon.

The exact template responsible for calculating the 24-hour rainfall total is not included in the sprinkler automation YAML and should be documented separately once the source template configuration is available.

> **Important:** The calculation logic has deliberately not been reconstructed or guessed in this document. When rebuilding the system, the actual template YAML should be backed up alongside this appendix.

---

# Template YAML Backup

## Current Calculation Template

The exact YAML used to calculate `sensor.sprinkler_forecast_rain_24h` should be inserted below when available.

```yaml
# Template YAML for sensor.sprinkler_forecast_rain_24h
#
# Add the current Home Assistant template configuration here.
#
# The sensor should retain:
#   - Entity: sensor.sprinkler_forecast_rain_24h
#   - Name: Sprinkler Forecast Rain 24h
#   - Unit: mm
#   - Icon: mdi:weather-rainy
```

---

# Dependencies

The rainfall forecast sensor is part of the wider irrigation decision process.

```text
Weather Forecast Source
          │
          ▼
Sprinkler Forecast Rain 24h
          │
          ▼
Sprinklers - Calculate Water Requirement
          │
          ├── Rain ≥ Threshold ──► No watering
          │
          └── Rain < Threshold ──► Calculate moisture deficit
                                      │
                                      ▼
                              Calculate total runtime
                                      │
                                      ▼
                          Morning Cycle and Soak
```

The sensor therefore does not directly control any sprinkler relay. Instead, it provides information used by the water requirement calculation.

---

# Rebuild Checklist

When rebuilding the template sensor:

- [ ] Create a Home Assistant Template sensor.
- [ ] Use the entity name `Sprinkler Forecast Rain 24h`.
- [ ] Confirm the resulting entity ID is `sensor.sprinkler_forecast_rain_24h`.
- [ ] Configure the unit of measurement as `mm`.
- [ ] Use the `mdi:weather-rainy` icon if desired.
- [ ] Restore the original rainfall calculation template.
- [ ] Confirm the sensor returns a numeric value.
- [ ] Test the rain skip logic using a value above and below the configured threshold.
- [ ] Confirm the `Sprinklers - Calculate Water Requirement` automation can read the sensor successfully.

---

# Related Documentation

- [Automation Reference](Automation-Reference.md)
- [Helper Reference](Helper-Reference.md)
- [Weather Integration](../docs/08-Weather.md)
- [Installation & Setup](../docs/09-Installation-and-Setup.md)

---

*Home Assistant Smart Sprinkler System — Appendix D: Template Sensor Reference*
