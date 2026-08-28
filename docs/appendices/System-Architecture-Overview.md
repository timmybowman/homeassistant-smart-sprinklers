# Appendix O — System Architecture Overview

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** Overall System Architecture  
> **Status:** Current system reference

## Overview

The Home Assistant Smart Sprinkler System is a weather-aware, automated garden irrigation system controlled by Home Assistant and an ESP32 running ESPHome.

Rather than operating as a simple fixed timer, the system uses a virtual soil moisture model, weather information and forecast rainfall to estimate whether irrigation is required. When watering is needed, the system calculates a runtime and applies it across the garden using a cycle-and-soak watering strategy.

The system is made up of four main layers:

```text
┌──────────────────────────────────────────────────────┐
│                   WEATHER & DATA                     │
│                                                      │
│  Temperature ────┐                                   │
│                  ├──► Virtual Soil Moisture Model    │
│  Rain Forecast ──┘                                   │
└──────────────────────────┬───────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────┐
│                  HOME ASSISTANT                      │
│                                                      │
│  Helpers ──► Calculations ──► Automations            │
│                                                      │
└──────────────────────────┬───────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────┐
│                    ESPHOME / ESP32                   │
│                                                      │
│       Home Assistant API ──► GPIO Outputs            │
│                                                      │
└──────────────────────────┬───────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────┐
│                  IRRIGATION HARDWARE                  │
│                                                      │
│        Relay Board ──► Valves ──► Sprinklers         │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

# 1. System Responsibilities

Each layer of the system has a specific responsibility.

| Layer | Primary Responsibility |
|---|---|
| Weather & Data | Provides information used by the irrigation model |
| Home Assistant | Makes irrigation decisions and controls the watering schedule |
| ESPHome | Provides reliable communication between Home Assistant and the controller |
| ESP32 | Controls the physical relay outputs |
| Relay Board | Switches the irrigation zones |
| Irrigation Hardware | Delivers water to the garden |

Separating these responsibilities makes the system easier to understand, troubleshoot and rebuild.

---

# 2. Weather and Environmental Data

The irrigation model uses weather information rather than relying entirely on a fixed watering schedule.

The primary weather entities currently referenced by the system are:

```text
sensor.met_office_ashby_de_la_zouch_temperature

sensor.sprinkler_forecast_rain_24h
```

The temperature sensor is used to estimate daily moisture loss, while the rainfall forecast is used to determine whether expected rain should reduce or eliminate the need for irrigation.

The general information flow is:

```text
Temperature
     │
     ▼
Estimated Daily Moisture Loss
     │
     ▼
Virtual Soil Moisture


Forecast Rain
     │
     ▼
Rain Skip Decision
     │
     ▼
Watering Required?
```

Weather information is therefore an input to the irrigation model rather than a direct command to operate the sprinklers.

---

# 3. Virtual Soil Moisture Model

The system maintains a virtual representation of soil moisture using:

```text
input_number.sprinkler_soil_moisture
```

This is not a reading from a physical soil sensor. Instead, it is a value maintained by Home Assistant and adjusted according to the irrigation model.

The automation responsible for updating this value is:

```text
Sprinklers - Update Virtual Soil Moisture
```

The current model estimates daily moisture loss based on temperature ranges.

```text
Temperature
     │
     ▼
Estimated Daily Loss
     │
     ▼
Reduce Virtual Soil Moisture
```

The resulting value is then used by the irrigation calculation.

---

# 4. Irrigation Decision Layer

The central irrigation decision is made by:

```text
Sprinklers - Calculate Water Requirement
```

This automation considers several inputs:

```text
Current Soil Moisture
        │
        ├──► Moisture Target
        │
        ├──► Forecast Rain
        │
        ├──► Rain Skip Threshold
        │
        └──► Recovery Mode
                  │
                  ▼
           Water Requirement
                  │
                  ▼
             Total Runtime
```

The automation stores its calculated results in helper entities including:

```text
input_number.sprinkler_water_target

input_number.sprinkler_total_runtime
```

If the rainfall forecast meets the configured rain skip threshold, or if the virtual soil moisture is already at the configured target, the required runtime is set to zero.

---

# 5. Watering Execution Layer

The physical watering sequence is controlled by:

```text
Sprinklers - Morning Cycle and Soak
```

Under normal operation, the automation triggers at:

```text
Sunrise + 30 minutes
```

Before watering begins, the automation checks:

```text
Are sprinklers enabled?
        │
        ▼
Is total runtime greater than zero?
        │
        ▼
      Watering begins
```

The master enable entity is:

```text
input_boolean.sprinklers_enabled
```

This provides a simple way to prevent normal automatic watering when the system is being maintained, tested or disabled.

---

# 6. Cycle and Soak Architecture

The system does not necessarily apply the entire calculated runtime continuously.

Instead, it divides watering into shorter passes using:

```text
input_number.sprinkler_cycle_runtime
```

and separates passes using:

```text
input_number.sprinkler_soak_time
```

The overall pattern is:

```text
PASS 1
Zone 1 ─► Zone 2 ─► Zone 3 ─► ... ─► Zone 7
                    │
                    ▼
                 SOAK TIME
                    │
                    ▼
PASS 2
Zone 1 ─► Zone 2 ─► Zone 3 ─► ... ─► Zone 7
                    │
                    ▼
            Repeat if Required
```

This approach gives water an opportunity to absorb into the soil between watering passes.

The cycle-and-soak automation continues until the calculated total runtime has been applied.

---

# 7. Home Assistant to ESPHome Communication

Home Assistant does not directly operate the ESP32 GPIO pins.

Instead, the system communicates through the ESPHome integration.

The architecture is:

```text
Home Assistant Automation
          │
          ▼
Home Assistant Switch Entity
          │
          ▼
       ESPHome API
          │
          ▼
         ESP32
          │
          ▼
       GPIO Output
```

For example, the first active zone is controlled using:

```text
switch.sprinklers_garden_sprinkler_1
```

This entity corresponds to the ESPHome switch configured for the first sprinkler relay channel.

---

# 8. ESP32 Controller

The physical controller is an ESP32 running ESPHome.

The ESPHome device configuration includes:

```text
Device Name:     esphome-web-fd4824
Friendly Name:   Sprinklers
Framework:       ESP-IDF
```

The ESP32 receives commands through the ESPHome API and controls the relay board using GPIO outputs.

The current system defines nine relay channels, allowing additional irrigation zones to be added in the future.

Seven channels are currently used by the automatic irrigation system.

---

# 9. Relay Control Architecture

The relay board is configured as active-low.

This means that the ESP32 configuration uses:

```yaml
inverted: true
```

for each relay output.

The relationship between the controller and the irrigation system is:

```text
ESP32 GPIO Output
        │
        ▼
 Active-Low Relay Input
        │
        ▼
      Relay Operates
        │
        ▼
 Irrigation Zone Activated
```

The current active zone GPIO assignments are:

| Zone | ESP32 GPIO |
|---|---:|
| Garden Sprinkler 1 | GPIO13 |
| Garden Sprinkler 2 | GPIO14 |
| Garden Sprinkler 3 | GPIO16 |
| Garden Sprinkler 4 | GPIO17 |
| Garden Sprinkler 5 | GPIO18 |
| Garden Sprinkler 6 | GPIO19 |
| Garden Sprinkler 7 | GPIO21 |

The ESPHome firmware also includes GPIO assignments for zones 8 and 9.

---

# 10. Irrigation Zones

The current automatic watering sequence uses seven active zones.

```text
┌─────────────┐
│   Zone 1    │
└──────┬──────┘
       ▼
┌─────────────┐
│   Zone 2    │
└──────┬──────┘
       ▼
┌─────────────┐
│   Zone 3    │
└──────┬──────┘
       ▼
┌─────────────┐
│   Zone 4    │
└──────┬──────┘
       ▼
┌─────────────┐
│   Zone 5    │
└──────┬──────┘
       ▼
┌─────────────┐
│   Zone 6    │
└──────┬──────┘
       ▼
┌─────────────┐
│   Zone 7    │
└─────────────┘
```

Only one zone is operated at a time by the current cycle-and-soak automation.

The Home Assistant entities used are:

```text
switch.sprinklers_garden_sprinkler_1
switch.sprinklers_garden_sprinkler_2
switch.sprinklers_garden_sprinkler_3
switch.sprinklers_garden_sprinkler_4
switch.sprinklers_garden_sprinkler_5
switch.sprinklers_garden_sprinkler_6
switch.sprinklers_garden_sprinkler_7
```

---

# 11. Safety Architecture

The system includes safety measures at more than one level.

## Home Assistant Level

The master enable control can prevent normal automatic watering:

```text
input_boolean.sprinklers_enabled
```

The watering automation also checks that a positive runtime has been calculated before starting.

## ESPHome Level

Each sprinkler switch is configured with:

```yaml
restore_mode: ALWAYS_OFF
```

This is intended to ensure that sprinkler outputs return to an off state following a restart.

Each zone also includes an independent automatic shut-off action after:

```text
31 minutes
```

The overall safety approach can be represented as:

```text
Home Assistant Logic
       │
       ├──► Master Enable Control
       │
       ├──► Runtime Checks
       │
       ▼
ESPHome Controller
       │
       ├──► Restore Outputs Off
       │
       └──► Independent Safety Timeout
```

These safeguards provide additional protection, but they do not replace sensible testing and supervision when making configuration changes.

---

# 12. Complete Daily System Flow

Under normal operation, the system follows this sequence:

```text
03:45
Update Virtual Soil Moisture
        │
        ▼
Estimate Daily Moisture Loss
        │
        ▼
04:00
Calculate Water Requirement
        │
        ├── Check Soil Moisture
        ├── Check Moisture Target
        ├── Check Forecast Rain
        └── Check Recovery Mode
                │
                ▼
         Calculate Runtime
                │
                ▼
        Store Runtime Result
                │
                ▼
Sunrise + 30 Minutes
        │
        ▼
Morning Cycle & Soak
        │
        ▼
Zone 1 → Zone 2 → Zone 3 → Zone 4
        │
        ▼
Zone 5 → Zone 6 → Zone 7
        │
        ▼
Soak and Repeat if Required
        │
        ▼
      Watering Complete
```

This sequence separates the decision-making stage from the physical execution stage.

---

# 13. System Dependencies

The sprinkler system depends on several components being available.

| Component | Required For |
|---|---|
| Home Assistant | Automation logic and scheduling |
| ESPHome Integration | Communication with the ESP32 |
| ESP32 | Physical relay control |
| Relay Board | Switching irrigation zones |
| Weather Temperature Sensor | Virtual soil moisture calculation |
| Rain Forecast Sensor | Rain skip decision |
| Helper Entities | System configuration and calculated values |
| Water Supply | Physical irrigation |

A failure at one level may affect the operation of the levels above or below it.

For example:

```text
ESP32 Offline
     │
     ▼
Home Assistant Cannot Operate Zone Switches
     │
     ▼
Automations May Run
     │
     ▼
Physical Watering Does Not Occur
```

Understanding these dependencies helps make troubleshooting more systematic.

---

# 14. Architecture Design Principles

The current system follows several useful design principles.

## Separation of Responsibilities

Home Assistant manages the irrigation logic, while ESPHome manages physical output control.

## Independent Safety Layer

The ESP32 includes safety behaviour that does not depend entirely on Home Assistant remaining available.

## Configurable Behaviour

Key values are stored in helpers rather than being permanently embedded in every automation.

This makes it easier to adjust:

- Cycle duration.
- Soak duration.
- Moisture target.
- Rain skip threshold.
- Recovery behaviour.

## Expandability

The ESPHome configuration currently defines nine relay channels while seven are used by the automatic watering sequence.

This leaves scope for future expansion.

---

# 15. Future Architecture Considerations

The architecture is designed to allow gradual development without requiring a complete redesign.

Potential future additions could include:

```text
Additional Zones
      │
      ├── Additional Home Assistant Switches
      ├── Updated Automation Zone List
      └── Updated Documentation

Improved Weather Model
      │
      ├── Additional Weather Inputs
      └── Refined Irrigation Calculations

Monitoring
      │
      ├── Watering History
      ├── Status Dashboard
      └── Notifications
```

These are potential development directions rather than features currently implemented by the documented system.

---

# 16. Architecture Summary

The system can be summarised in one flow:

```text
WEATHER
   │
   ▼
VIRTUAL SOIL MOISTURE MODEL
   │
   ▼
IRRIGATION REQUIREMENT CALCULATION
   │
   ▼
TOTAL WATERING RUNTIME
   │
   ▼
CYCLE & SOAK AUTOMATION
   │
   ▼
HOME ASSISTANT SWITCHES
   │
   ▼
ESPHOME API
   │
   ▼
ESP32 GPIO OUTPUTS
   │
   ▼
ACTIVE-LOW RELAY BOARD
   │
   ▼
SPRINKLER ZONES
```

This layered approach allows the irrigation system to make weather-aware watering decisions while keeping the physical controller and relay safety mechanisms independent from the higher-level scheduling logic.

---

## Related Documentation

- [Hardware Specification](Hardware-Specification.md)
- [GPIO & ESP32 Pin Reference](GPIO-ESP32-Pin-Reference.md)
- [ESPHome Firmware Reference](ESPHome-Firmware-Reference.md)
- [Automation Reference](Automation-Reference.md)
- [Helper Reference](Helper-Reference.md)
- [Template Sensor Reference](Template-Sensor-Reference.md)
- [Irrigation Calibration](Irrigation-Calibration.md)
- [Testing & Commissioning Checklist](Testing-and-Commissioning-Checklist.md)
- [Glossary & Entity Reference](Glossary-and-Entity-Reference.md)
- [Change Log & Upgrade Notes](Change-Log-and-Upgrade-Notes.md)

---

*Home Assistant Smart Sprinkler System — Appendix O: System Architecture Overview*
