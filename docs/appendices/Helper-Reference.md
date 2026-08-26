# Appendix C — Helper Reference

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** Home Assistant Helpers  
> **Status:** Current configuration reference

## Overview

The sprinkler system uses Home Assistant helpers to store settings, calculated values and the current state of the virtual irrigation model.

These helpers allow the sprinkler system to be adjusted from Home Assistant without editing the automation YAML. They also provide a simple way to inspect what the irrigation system is currently planning to do.

The helpers fall into two main categories:

- **Input Numbers** — used for values such as runtimes, targets, rainfall thresholds and virtual soil moisture.
- **Input Booleans** — used as on/off switches for enabling the system and activating recovery mode.

---

# Helper Summary

| Helper Name | Entity ID | Type | Primary Purpose |
|---|---|---|---|
| Sprinkler Cycle Runtime | `input_number.sprinkler_cycle_runtime` | Input Number | Length of each watering cycle |
| Sprinkler Moisture Target | `input_number.sprinkler_moisture_target` | Input Number | Target virtual soil moisture level |
| Sprinkler Rain Skip Threshold | `input_number.sprinkler_rain_skip_threshold` | Input Number | Rainfall amount that causes watering to be skipped |
| Sprinkler Recovery Mode | `input_boolean.sprinkler_recovery_mode` | Input Boolean | Enables increased watering while recovering stressed grass |
| Sprinkler Runtime | `input_number.sprinkler_runtime` | Input Number | Runtime value maintained by the calculation automation |
| Sprinkler Soak Time | `input_number.sprinkler_soak_time` | Input Number | Delay between watering passes |
| Sprinkler Soil Moisture | `input_number.sprinkler_soil_moisture` | Input Number | Virtual representation of soil moisture |
| Sprinkler Total Runtime | `input_number.sprinkler_total_runtime` | Input Number | Total watering runtime calculated for each zone |
| Sprinkler Water Target | `input_number.sprinkler_water_target` | Input Number | Calculated amount of water required |
| Sprinklers Enabled | `input_boolean.sprinklers_enabled` | Input Boolean | Master enable switch |

> **Note:** The sprinkler system also uses `sensor.sprinkler_forecast_rain_24h`. Although it appears alongside the helpers in Home Assistant, it is a template sensor rather than an input helper and is documented separately in **Appendix D — Template Sensor Reference**.

---

# 1. Sprinklers Enabled

**Entity ID:** `input_boolean.sprinklers_enabled`  
**Type:** Input Boolean

## Purpose

This is the master on/off switch for the sprinkler automation system.

When switched off, the main watering calculations and the morning watering automation will not proceed.

## Used By

- `Sprinklers - Calculate Water Requirement`
- `Sprinklers - Morning Cycle and Soak`

## Recommended Use

Leave this helper switched **on** during normal automatic operation.

Switch it **off** when:

- Working on the sprinkler system.
- Performing maintenance.
- Testing hardware manually.
- You want to temporarily prevent automatic watering.

> Turning this helper off provides a simple way to disable the automatic system without deleting or editing any automations.

---

# 2. Sprinkler Soil Moisture

**Entity ID:** `input_number.sprinkler_soil_moisture`  
**Type:** Input Number

## Purpose

This helper stores the system's **virtual soil moisture value**.

The system does not currently use a physical soil moisture sensor. Instead, it maintains an estimated value based on the irrigation model.

Each morning, the `Sprinklers - Update Virtual Soil Moisture` automation reduces this value according to the temperature-based daily loss calculation.

The value is then used by the water requirement calculation to determine how much irrigation may be required.

## Used By

- `Sprinklers - Update Virtual Soil Moisture`
- `Sprinklers - Calculate Water Requirement`

## How It Works

The system follows this basic cycle:

```text
Current Virtual Soil Moisture
          │
          ▼
Estimated Daily Moisture Loss
          │
          ▼
Reduced Virtual Moisture Value
          │
          ▼
Compare Against Moisture Target
          │
          ▼
Calculate Moisture Deficit
```

A higher value represents a higher estimated level of soil moisture within the model.

> **Important:** This is a virtual model rather than a direct measurement of physical soil moisture.

---

# 3. Sprinkler Moisture Target

**Entity ID:** `input_number.sprinkler_moisture_target`  
**Type:** Input Number

## Purpose

This helper defines the desired target for the virtual soil moisture model.

The water requirement automation compares the current `sprinkler_soil_moisture` value against this target.

If the current moisture value is already equal to or greater than the target, no irrigation requirement is calculated.

## Used By

- `Sprinklers - Calculate Water Requirement`

## Relationship to Soil Moisture

```text
Current Moisture ≥ Target
        │
        └──► No watering required

Current Moisture < Target
        │
        └──► Calculate moisture deficit
                 │
                 ▼
             Calculate irrigation
```

The difference between the current moisture value and this target is referred to as the **moisture deficit**.

---

# 4. Sprinkler Rain Skip Threshold

**Entity ID:** `input_number.sprinkler_rain_skip_threshold`  
**Type:** Input Number

## Purpose

This helper defines the amount of forecast rainfall that is considered sufficient to skip irrigation.

The `Sprinklers - Calculate Water Requirement` automation compares the 24-hour rainfall forecast against this value.

If the forecast rainfall is equal to or greater than the threshold, the system sets the watering requirement and total runtime to zero.

## Used By

- `Sprinklers - Calculate Water Requirement`

## Logic

```text
Forecast Rain ≥ Rain Skip Threshold
                 │
                 ▼
           Skip Watering
                 │
                 ▼
     Water Target = 0
     Total Runtime = 0
```

This prevents unnecessary irrigation when sufficient rainfall is expected.

---

# 5. Sprinkler Recovery Mode

**Entity ID:** `input_boolean.sprinkler_recovery_mode`  
**Type:** Input Boolean

## Purpose

Recovery Mode increases the proportion of the calculated moisture deficit that the system attempts to replace.

It is intended for periods when the lawn is under stress and requires additional watering.

## Used By

- `Sprinklers - Calculate Water Requirement`

## Watering Factors

| Mode | Deficit Replacement Factor |
|---|---:|
| Normal operation | 30% |
| Recovery Mode | 45% |

## Logic

```text
Recovery Mode OFF
        │
        ▼
Water Required = Deficit × 0.30

Recovery Mode ON
        │
        ▼
Water Required = Deficit × 0.45
```

> Recovery Mode does not directly turn the sprinklers on. It changes the amount of water calculated when the normal watering calculation runs.

---

# 6. Sprinkler Water Target

**Entity ID:** `input_number.sprinkler_water_target`  
**Type:** Input Number

## Purpose

This helper stores the amount of water calculated by the irrigation model.

When watering is required, the `Sprinklers - Calculate Water Requirement` automation calculates the water requirement from the virtual soil moisture deficit and stores the result here.

If watering is skipped because of sufficient forecast rainfall or because the moisture target has already been reached, this value is set to zero.

## Used By

- `Sprinklers - Calculate Water Requirement`

## Calculation

```text
Moisture Deficit
      │
      ▼
Multiply by Normal or Recovery Factor
      │
      ▼
Sprinkler Water Target
```

The current system uses this value as an intermediate result before converting the requirement into a total sprinkler runtime.

---

# 7. Sprinkler Total Runtime

**Entity ID:** `input_number.sprinkler_total_runtime`  
**Type:** Input Number

## Purpose

This helper stores the total amount of watering time calculated for **each sprinkler zone**.

It is the primary output of the water requirement calculation and is used by the morning cycle-and-soak automation.

## Used By

- `Sprinklers - Calculate Water Requirement`
- `Sprinklers - Morning Cycle and Soak`

## Calculation

The current irrigation model assumes a precipitation rate of approximately **12 mm per hour**.

The calculated water requirement is converted into minutes:

```text
Total Runtime = (Water Required ÷ 12) × 60
```

The automation caps the calculated value at **180 minutes**.

## Important

The total runtime is not necessarily one continuous watering session.

The `Sprinklers - Morning Cycle and Soak` automation divides this runtime into shorter cycles using `input_number.sprinkler_cycle_runtime`.

For example:

```text
Total Runtime: 20 minutes
Cycle Runtime: 5 minutes

Result:
4 watering passes of 5 minutes per zone
with soak periods between passes
```

---

# 8. Sprinkler Cycle Runtime

**Entity ID:** `input_number.sprinkler_cycle_runtime`  
**Type:** Input Number

## Purpose

This helper determines the maximum amount of time a sprinkler zone runs during a single watering pass.

It is the key setting controlling the **cycle** portion of the cycle-and-soak system.

## Used By

- `Sprinklers - Morning Cycle and Soak`
- `Sprinklers - Calculate Water Requirement`

## Example

With:

```text
Total Runtime: 15 minutes
Cycle Runtime: 5 minutes
```

Each zone receives three separate five-minute watering cycles rather than one continuous 15-minute watering session.

## Safety Consideration

The ESPHome firmware independently turns off each sprinkler relay after 31 minutes as a safety precaution.

The cycle runtime should therefore remain below that safety limit.

---

# 9. Sprinkler Soak Time

**Entity ID:** `input_number.sprinkler_soak_time`  
**Type:** Input Number

## Purpose

This helper defines the delay between complete watering passes.

After every active sprinkler zone has received one cycle of water, the automation waits for the configured soak time before beginning the next pass.

## Used By

- `Sprinklers - Morning Cycle and Soak`

## Example

With:

```text
Cycle Runtime: 5 minutes
Soak Time: 45 minutes
Total Runtime: 10 minutes
```

The sequence is:

```text
Pass 1
├── Zone 1: 5 minutes
├── Zone 2: 5 minutes
├── Zone 3: 5 minutes
├── Zone 4: 5 minutes
├── Zone 5: 5 minutes
├── Zone 6: 5 minutes
└── Zone 7: 5 minutes

45-minute soak period

Pass 2
├── Zone 1: 5 minutes
├── Zone 2: 5 minutes
├── Zone 3: 5 minutes
├── Zone 4: 5 minutes
├── Zone 5: 5 minutes
├── Zone 6: 5 minutes
└── Zone 7: 5 minutes
```

Because the zones operate sequentially, each zone also receives additional natural soaking time while the other zones are being watered.

---

# 10. Sprinkler Runtime

**Entity ID:** `input_number.sprinkler_runtime`  
**Type:** Input Number

## Purpose

This helper is maintained by the `Sprinklers - Calculate Water Requirement` automation.

The current automation sets its value to the lower of:

- The configured `Sprinkler Cycle Runtime`.
- 30 minutes.

The relevant calculation is:

```text
minimum(Sprinkler Cycle Runtime, 30 minutes)
```

## Used By

- `Sprinklers - Calculate Water Requirement`

## Current Role

The current `Sprinklers - Morning Cycle and Soak` automation uses `input_number.sprinkler_cycle_runtime` directly when determining how long each zone should run.

As a result, `input_number.sprinkler_runtime` is currently maintained as a separate reference value and is not directly used by the cycle-and-soak execution automation.

This helper has been retained as part of the current system configuration and may be useful for future automation development or dashboard displays.

---

# How the Helpers Work Together

The helpers form the adjustable settings and stored state of the sprinkler system.

```text
                 USER SETTINGS
        ┌─────────────────────────────┐
        │ Moisture Target             │
        │ Rain Skip Threshold         │
        │ Recovery Mode               │
        │ Cycle Runtime               │
        │ Soak Time                   │
        └──────────────┬──────────────┘
                       │
                       ▼
              IRRIGATION CALCULATION
                       │
                       ▼
        ┌─────────────────────────────┐
        │ Water Target                │
        │ Total Runtime               │
        │ Sprinkler Runtime           │
        └──────────────┬──────────────┘
                       │
                       ▼
               MORNING WATERING
                       │
                       ▼
              SEVEN SPRINKLER ZONES
```

The `Sprinkler Soil Moisture` helper sits at the centre of the virtual irrigation model and is updated daily before the water requirement is calculated.

---

# Recommended Normal Operation

During normal use, the helpers can generally be left alone while the automation system manages the calculated values.

The settings most likely to be adjusted manually are:

| Setting | When You Might Adjust It |
|---|---|
| Sprinklers Enabled | Temporarily disable automatic watering |
| Sprinkler Moisture Target | Adjust the desired level in the virtual moisture model |
| Sprinkler Rain Skip Threshold | Change how much forecast rain is required to skip watering |
| Sprinkler Recovery Mode | Temporarily increase watering for stressed grass |
| Sprinkler Cycle Runtime | Adjust watering cycle length |
| Sprinkler Soak Time | Adjust the time allowed for water to soak into the soil |

The calculated helpers should normally be managed automatically by the system:

- `Sprinkler Water Target`
- `Sprinkler Total Runtime`
- `Sprinkler Runtime`

---

# Rebuild Checklist

When rebuilding the sprinkler system in Home Assistant:

- [ ] Recreate all input number helpers.
- [ ] Recreate both input boolean helpers.
- [ ] Ensure the entity IDs match those used by the automations.
- [ ] Configure sensible starting values for the irrigation model.
- [ ] Confirm the master `Sprinklers Enabled` switch is in the desired state.
- [ ] Test that each helper can be adjusted from Home Assistant.
- [ ] Test the automations after recreating the helpers.

> **Configuration Note:** This appendix documents the helper names, entity IDs and roles currently used by the sprinkler automations. The exact minimum, maximum, step and default settings should be preserved from the Home Assistant helper configuration when rebuilding, as those values are not contained within the automation YAML itself.

---

## Related Documentation

- [Automation Reference](Automation-Reference.md)
- [Template Sensor Reference](Template-Sensor-Reference.md)
- [ESPHome Firmware Reference](ESPHome-Firmware-Reference.md)
- [Installation & Setup](../docs/09-Installation-and-Setup.md)

---

*Home Assistant Smart Sprinkler System — Appendix C: Helper Reference*
