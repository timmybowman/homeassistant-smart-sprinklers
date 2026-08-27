# Appendix L — Testing & Commissioning Checklist

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** System Testing & Commissioning  
> **Status:** Current system reference

## Overview

This checklist provides a structured procedure for testing the Home Assistant Smart Sprinkler System following:

- Initial installation.
- A complete system rebuild.
- ESP32 replacement or firmware changes.
- Home Assistant configuration changes.
- Significant automation changes.
- Changes to irrigation hardware.

The purpose of commissioning is to confirm that each layer of the system operates correctly before unattended automatic watering is enabled.

```text
Home Assistant Configuration
          │
          ▼
Weather & Helper Entities
          │
          ▼
Automations
          │
          ▼
ESPHome API
          │
          ▼
ESP32 Controller
          │
          ▼
Relay Board
          │
          ▼
Sprinkler Valves & Heads
```

> **Important:** Test the system in stages. Do not enable unattended automatic watering until the relevant checks have been completed.

---

# 1. Pre-Commissioning Safety Check

Before beginning testing:

- [ ] Confirm that all sprinkler switches are currently off.
- [ ] Check that no sprinkler zone is unintentionally watering.
- [ ] Inspect accessible wiring for obvious damage.
- [ ] Check the ESP32 and relay board enclosure for moisture or damage.
- [ ] Confirm that the irrigation water supply is in the expected state.
- [ ] Ensure that manual testing can be stopped quickly if required.

If electrical wiring needs to be changed, isolate the relevant power supply before working on the equipment.

---

# 2. Home Assistant Core Check

Confirm that Home Assistant is operating normally.

- [ ] Home Assistant is online and accessible.
- [ ] The ESPHome integration is available.
- [ ] The Sprinklers ESPHome device is visible.
- [ ] No critical errors relating to the sprinkler system are present.
- [ ] Required automations are visible.
- [ ] Required helpers are visible.

The core sprinkler automations are:

```text
Sprinklers - Update Virtual Soil Moisture
Sprinklers - Calculate Water Requirement
Sprinklers - Morning Cycle and Soak
```

---

# 3. ESP32 and ESPHome Commissioning

Confirm that the physical controller is operating correctly before testing the irrigation logic.

## Device Status

- [ ] The ESP32 powers on correctly.
- [ ] The ESPHome device is online.
- [ ] Wi-Fi connectivity is stable.
- [ ] The Home Assistant API connection is working.
- [ ] ESPHome logs do not show repeated unexpected restarts.

## Output Safety

- [ ] All sprinkler outputs initialise in the off state.
- [ ] No relay is unintentionally activated after a restart.
- [ ] The independent safety timeout remains present in the ESPHome configuration.

The current firmware uses `restore_mode: ALWAYS_OFF` for the sprinkler outputs and includes an automatic 31-minute timeout for each zone.

---

# 4. Individual Zone Testing

Test each active irrigation zone individually from Home Assistant.

> Run these tests for only as long as necessary to identify the correct zone and confirm operation.

## Zone 1

- [ ] Turn on `switch.sprinklers_garden_sprinkler_1`.
- [ ] Confirm the expected relay operates.
- [ ] Confirm the correct sprinkler zone operates.
- [ ] Confirm water reaches the expected area.
- [ ] Turn the zone off.
- [ ] Confirm the relay and sprinkler valve turn off.

## Zone 2

- [ ] Turn on `switch.sprinklers_garden_sprinkler_2`.
- [ ] Confirm the expected relay operates.
- [ ] Confirm the correct sprinkler zone operates.
- [ ] Turn the zone off.

## Zone 3

- [ ] Turn on `switch.sprinklers_garden_sprinkler_3`.
- [ ] Confirm the expected relay operates.
- [ ] Confirm the correct sprinkler zone operates.
- [ ] Turn the zone off.

## Zone 4

- [ ] Turn on `switch.sprinklers_garden_sprinkler_4`.
- [ ] Confirm the expected relay operates.
- [ ] Confirm the correct sprinkler zone operates.
- [ ] Turn the zone off.

## Zone 5

- [ ] Turn on `switch.sprinklers_garden_sprinkler_5`.
- [ ] Confirm the expected relay operates.
- [ ] Confirm the correct sprinkler zone operates.
- [ ] Turn the zone off.

## Zone 6

- [ ] Turn on `switch.sprinklers_garden_sprinkler_6`.
- [ ] Confirm the expected relay operates.
- [ ] Confirm the correct sprinkler zone operates.
- [ ] Turn the zone off.

## Zone 7

- [ ] Turn on `switch.sprinklers_garden_sprinkler_7`.
- [ ] Confirm the expected relay operates.
- [ ] Confirm the correct sprinkler zone operates.
- [ ] Turn the zone off.

---

# 5. Helper Entity Check

Confirm that the sprinkler helpers are available and contain sensible values.

## Input Booleans

- [ ] `input_boolean.sprinklers_enabled` is available.
- [ ] `input_boolean.sprinkler_recovery_mode` is available.

## Runtime and Cycle Helpers

- [ ] `input_number.sprinkler_cycle_runtime` is available.
- [ ] `input_number.sprinkler_runtime` is available.
- [ ] `input_number.sprinkler_soak_time` is available.
- [ ] `input_number.sprinkler_total_runtime` is available.

## Irrigation Model Helpers

- [ ] `input_number.sprinkler_soil_moisture` is available.
- [ ] `input_number.sprinkler_moisture_target` is available.
- [ ] `input_number.sprinkler_rain_skip_threshold` is available.
- [ ] `input_number.sprinkler_water_target` is available.

Review the values before testing.

Values should be sensible and appropriate for the current season and testing conditions.

---

# 6. Weather and Forecast Sensor Check

The irrigation logic depends on weather information.

Confirm that the following sensors are available and returning sensible values:

- [ ] `sensor.met_office_ashby_de_la_zouch_temperature`
- [ ] `sensor.sprinkler_forecast_rain_24h`

Check that:

- [ ] The temperature sensor is not unavailable or unknown.
- [ ] The rainfall forecast sensor is not unavailable or unknown.
- [ ] Values appear reasonable for current conditions.

If weather data is unavailable, investigate this before relying on the automatic irrigation calculations.

---

# 7. Virtual Soil Moisture Automation Test

The first core automation is:

```text
Sprinklers - Update Virtual Soil Moisture
```

Its purpose is to estimate daily moisture loss and update:

```text
input_number.sprinkler_soil_moisture
```

## Commissioning Check

- [ ] Confirm the automation is enabled.
- [ ] Confirm the temperature sensor is available.
- [ ] Record the current soil moisture value.
- [ ] Run or test the automation in a controlled manner.
- [ ] Confirm the automation completes successfully.
- [ ] Confirm the soil moisture value changes as expected.
- [ ] Review the automation trace for unexpected errors.

The daily loss model currently uses temperature ranges to estimate moisture depletion.

---

# 8. Water Requirement Calculation Test

The second core automation is:

```text
Sprinklers - Calculate Water Requirement
```

This automation evaluates:

```text
Virtual Soil Moisture
        │
        ├── Moisture Target
        │
        ├── Forecast Rain
        │
        ├── Rain Skip Threshold
        │
        └── Recovery Mode
                │
                ▼
        Water Requirement
                │
                ▼
         Total Runtime
```

## Commissioning Check

- [ ] Confirm the automation is enabled.
- [ ] Confirm `input_boolean.sprinklers_enabled` is set appropriately for testing.
- [ ] Confirm the weather and forecast sensors are available.
- [ ] Confirm the moisture target is sensible.
- [ ] Confirm the rain skip threshold is sensible.
- [ ] Run or trigger the automation in a controlled manner.
- [ ] Confirm the automation completes successfully.
- [ ] Check `input_number.sprinkler_water_target`.
- [ ] Check `input_number.sprinkler_total_runtime`.
- [ ] Review the automation trace.

The calculated result should make sense in relation to the current moisture level and forecast conditions.

---

# 9. Rain Skip Logic Test

The system should avoid unnecessary watering when forecast rainfall meets or exceeds the configured skip threshold.

Confirm that the logic behaves correctly when:

- [ ] Forecast rainfall is below the skip threshold.
- [ ] Forecast rainfall meets or exceeds the skip threshold.
- [ ] Virtual soil moisture is already at or above the moisture target.

When watering is skipped, the expected result is:

```text
Sprinkler Water Target = 0
Sprinkler Total Runtime = 0
```

---

# 10. Recovery Mode Test

Recovery mode allows the irrigation calculation to use a larger proportion of the calculated moisture deficit.

Test carefully by comparing the calculation with recovery mode:

```text
OFF
```

and:

```text
ON
```

Checklist:

- [ ] Confirm `input_boolean.sprinkler_recovery_mode` can be changed.
- [ ] Run the calculation using controlled test conditions.
- [ ] Confirm the calculated result changes as expected.
- [ ] Return recovery mode to the desired operational state after testing.

---

# 11. Cycle and Soak Automation Test

The main watering automation is:

```text
Sprinklers - Morning Cycle and Soak
```

It normally runs at:

```text
Sunrise + 30 minutes
```

For commissioning, it is preferable to use a short controlled runtime rather than waiting for a full unattended watering cycle.

## Basic Sequence Test

Confirm that the system operates in the expected order:

```text
Zone 1
   │
   ▼
Zone 2
   │
   ▼
Zone 3
   │
   ▼
Zone 4
   │
   ▼
Zone 5
   │
   ▼
Zone 6
   │
   ▼
Zone 7
```

Checklist:

- [ ] Confirm the automation is enabled.
- [ ] Confirm sprinklers are enabled.
- [ ] Set a suitable short test runtime.
- [ ] Confirm only one zone operates at a time.
- [ ] Confirm zones operate sequentially.
- [ ] Confirm each zone turns off before the next begins.
- [ ] Confirm the automation completes without errors.

---

# 12. Cycle and Soak Multi-Pass Test

Where the total runtime exceeds the configured cycle runtime, the system should perform multiple passes.

The intended sequence is:

```text
Pass 1 — All Zones
        │
        ▼
     Soak Time
        │
        ▼
Pass 2 — All Zones
        │
        ▼
     Soak Time
        │
        ▼
Additional Passes if Required
```

Checklist:

- [ ] Configure a short cycle runtime for testing.
- [ ] Configure a total runtime greater than the cycle runtime.
- [ ] Confirm the first pass completes across all seven zones.
- [ ] Confirm the soak delay begins.
- [ ] Confirm the next pass begins after the soak period.
- [ ] Confirm the remaining runtime is reduced correctly.
- [ ] Confirm the sequence stops when the required runtime is complete.

---

# 13. ESPHome Safety Timeout Check

Each sprinkler zone includes an independent automatic shut-off timer in the ESPHome firmware.

During commissioning:

- [ ] Confirm the safety timeout is present in the firmware.
- [ ] Confirm normal cycle runtimes remain below the safety timeout.
- [ ] Confirm the Home Assistant automation is not configured to intentionally rely on the safety timeout for normal operation.

The safety timeout is an additional safeguard rather than part of the normal watering schedule.

---

# 14. Physical Irrigation Check

During or immediately after testing, inspect the physical irrigation system.

- [ ] Check for leaks.
- [ ] Check sprinkler head movement.
- [ ] Check spray direction.
- [ ] Check coverage at the edges of the lawn.
- [ ] Check for excessive runoff.
- [ ] Check for areas receiving noticeably less water.
- [ ] Check that sprinklers retract correctly after operation.

If coverage is uneven, investigate sprinkler positioning and nozzle configuration before increasing the watering runtime for the entire garden.

---

# 15. Automation Trace Review

After commissioning tests, review the traces for the three core automations:

```text
Sprinklers - Update Virtual Soil Moisture
Sprinklers - Calculate Water Requirement
Sprinklers - Morning Cycle and Soak
```

Check for:

- [ ] Expected trigger.
- [ ] Conditions passing as expected.
- [ ] Sensible variable values.
- [ ] Expected actions.
- [ ] Successful completion.
- [ ] No unexpected errors.

Automation traces can provide useful confirmation that the system is operating according to its documented logic.

---

# 16. Final Commissioning Checklist

Before returning the system to normal unattended operation:

## Hardware

- [ ] ESP32 is operating correctly.
- [ ] Relay board operates correctly.
- [ ] All seven active sprinkler zones have been tested.
- [ ] No wiring problems are visible.
- [ ] No significant leaks are present.

## Home Assistant

- [ ] All required helpers are available.
- [ ] Weather sensors are available.
- [ ] All core automations are enabled.
- [ ] Automation traces have been reviewed.

## Irrigation Logic

- [ ] Virtual soil moisture updates correctly.
- [ ] Water requirement calculations are sensible.
- [ ] Rain skip logic works correctly.
- [ ] Recovery mode behaves as expected.
- [ ] Cycle and soak logic works correctly.

## Final Safety Check

- [ ] Normal watering runtime remains below the ESPHome safety timeout.
- [ ] Only one sprinkler zone operates at a time.
- [ ] All zones turn off correctly.
- [ ] The master sprinkler enable control is understood.

---

# 17. Return to Automatic Operation

Once commissioning has been completed successfully:

1. Set all helper values to the desired operational settings.
2. Confirm that no temporary testing values remain.
3. Confirm the desired state of recovery mode.
4. Confirm the desired soil moisture model starting value.
5. Turn on automatic sprinkler operation using:

```text
input_boolean.sprinklers_enabled
```

6. Monitor the next full automatic watering cycle where practical.

---

# 18. Commissioning Record

A simple record can be retained after major testing or rebuilding.

| Date | Reason for Test | Result | Notes |
|---|---|---|---|
| | | Pass / Fail | |
| | | Pass / Fail | |
| | | Pass / Fail | |

## Outstanding Issues

```text
Issue:
_____________________________________________

Action Required:
_____________________________________________

Date Resolved:
_____________________________________________
```

---

# 19. Commissioning Principles

A successful test sequence follows the system from the lowest practical layer upwards:

```text
ESP32
  │
  ▼
Relay Board
  │
  ▼
Individual Zones
  │
  ▼
Home Assistant Entities
  │
  ▼
Calculations
  │
  ▼
Automations
  │
  ▼
Full Irrigation Cycle
```

Testing each layer separately makes faults easier to identify and reduces the risk of changing a working component while troubleshooting another part of the system.

---

## Related Documentation

- [Quick Recovery & Rebuild Guide](Quick-Recovery-and-Rebuild-Guide.md)
- [Installation Troubleshooting](Installation-Troubleshooting.md)
- [System Maintenance](System-Maintenance.md)
- [ESPHome Firmware Reference](ESPHome-Firmware-Reference.md)
- [GPIO & ESP32 Pin Reference](GPIO-ESP32-Pin-Reference.md)
- [Irrigation Calibration](Irrigation-Calibration.md)
- [Automation Reference](Automation-Reference.md)
- [Helper Reference](Helper-Reference.md)

---

*Home Assistant Smart Sprinkler System — Appendix L: Testing & Commissioning Checklist*
