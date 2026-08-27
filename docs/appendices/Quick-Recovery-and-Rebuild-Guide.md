# Appendix K — Quick Recovery & Rebuild Guide

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** Disaster Recovery & System Rebuild  
> **Status:** Current system reference

## Overview

This appendix provides a practical recovery procedure for rebuilding the Home Assistant Smart Sprinkler System following a hardware failure, configuration loss, Home Assistant rebuild or ESP32 replacement.

The system has been designed around separate components:

```text
Home Assistant
      │
      ├── Helpers
      ├── Weather & Template Sensors
      └── Automations
              │
              ▼
         ESPHome API
              │
              ▼
        ESP32 Controller
              │
              ▼
        Active-Low Relay Board
              │
              ▼
         Sprinkler Valves
```

This separation means that the system can normally be rebuilt one layer at a time.

> **Important:** Do not attempt to restore the complete system in one step. Test each layer before moving on to the next.

---

# 1. Recovery Priorities

If the system has failed, the first priority is to ensure that no sprinkler zone is unintentionally running.

The recommended recovery order is:

```text
1. Make the irrigation system safe
              │
              ▼
2. Restore Home Assistant
              │
              ▼
3. Restore ESPHome / ESP32
              │
              ▼
4. Confirm relay operation
              │
              ▼
5. Restore helpers and sensors
              │
              ▼
6. Restore automations
              │
              ▼
7. Test watering sequence
              │
              ▼
8. Re-enable automatic watering
```

Do not re-enable unattended automatic watering until the complete system has been tested.

---

# 2. Make the System Safe

Before beginning any rebuild work:

- [ ] Turn off the irrigation water supply if necessary.
- [ ] Confirm that all sprinkler zones are off.
- [ ] Turn off `input_boolean.sprinklers_enabled`.
- [ ] Check that no relay is energised.
- [ ] Isolate electrical power before changing wiring.

The ESPHome firmware uses `restore_mode: ALWAYS_OFF`, which is intended to ensure that sprinkler outputs initialise in the off state after a restart.

However, the physical state of the irrigation system should always be checked rather than assumed.

---

# 3. Identify What Needs Rebuilding

Not every failure requires a complete rebuild.

Use the following guide to identify the affected layer.

| Problem | Likely Recovery Area |
|---|---|
| Home Assistant unavailable | Home Assistant |
| Helpers missing | Home Assistant helpers |
| Automations missing or disabled | Home Assistant automations |
| ESP32 unavailable | Power, Wi-Fi or ESPHome |
| ESP32 has failed completely | ESP32 replacement and ESPHome restore |
| Relay does not operate | Wiring, relay board or ESPHome GPIO configuration |
| One zone does not operate | Individual relay channel, wiring or valve |
| Watering logic is incorrect | Sensors, helpers or automations |
| Entire configuration lost | Complete rebuild |

Where possible, restore only the affected component and test it before making wider changes.

---

# 4. Essential Backup Files

The most important files for rebuilding the system are:

```text
ESPHome firmware configuration
Home Assistant automation YAML
Template sensor configuration
Helper entity configuration
GPIO and wiring documentation
Hardware specification
```

The GitHub repository should act as the central reference for these files.

Before making significant changes to the live system, ensure that the repository contains a copy of the current known-working configuration.

---

# 5. Rebuilding the ESP32 Controller

If the ESP32 needs to be replaced or completely re-flashed, restore the ESPHome configuration first.

The current ESPHome device is configured with:

```text
Name: esphome-web-fd4824
Friendly Name: Sprinklers
Platform: ESP32
Framework: ESP-IDF
```

The firmware defines nine relay outputs, with the first seven currently used by the automatic irrigation system.

## GPIO Assignments

| Sprinkler | ESPHome ID | GPIO |
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

The relay board is configured as active-low, so the ESPHome GPIO switches use:

```yaml
inverted: true
```

Each zone also includes:

```yaml
restore_mode: ALWAYS_OFF
```

and an independent 31-minute safety timeout.

> **Safety feature:** Keep the safety timeout in the restored configuration. The Home Assistant automation is intended to control normal runtimes, while the ESPHome timeout provides an independent safeguard.

Refer to the complete firmware backup in:

```text
ESPHome Firmware Reference
```

or the stored ESPHome YAML configuration in the repository.

---

# 6. Test the ESP32 Before Restoring Automation

Do not immediately restore automatic watering after flashing or replacing the ESP32.

First confirm:

- [ ] ESP32 powers on correctly.
- [ ] ESPHome starts successfully.
- [ ] Wi-Fi connects successfully.
- [ ] Home Assistant can communicate with the device.
- [ ] All sprinkler switches appear in Home Assistant.

The expected active switch entities are:

```text
switch.sprinklers_garden_sprinkler_1
switch.sprinklers_garden_sprinkler_2
switch.sprinklers_garden_sprinkler_3
switch.sprinklers_garden_sprinkler_4
switch.sprinklers_garden_sprinkler_5
switch.sprinklers_garden_sprinkler_6
switch.sprinklers_garden_sprinkler_7
```

Test each zone individually before proceeding.

---

# 7. Restore and Test the Relay Wiring

Once the ESP32 is online, confirm that the physical relay wiring matches the documented GPIO assignments.

Test one relay at a time.

```text
Home Assistant Switch
        │
        ▼
ESP32 GPIO Output
        │
        ▼
Relay Channel
        │
        ▼
Sprinkler Valve
```

For each zone:

1. Turn the zone on manually.
2. Confirm that the expected relay operates.
3. Confirm that the correct valve opens.
4. Confirm that the expected sprinkler zone receives water.
5. Turn the zone off.
6. Confirm that the relay and valve turn off.

Complete this test for all active zones before restoring unattended automation.

---

# 8. Restore Home Assistant Helpers

The sprinkler system relies on the following helper entities.

## Input Booleans

```text
input_boolean.sprinklers_enabled
input_boolean.sprinkler_recovery_mode
```

## Input Numbers

```text
input_number.sprinkler_cycle_runtime
input_number.sprinkler_moisture_target
input_number.sprinkler_rain_skip_threshold
input_number.sprinkler_runtime
input_number.sprinkler_soak_time
input_number.sprinkler_soil_moisture
input_number.sprinkler_total_runtime
input_number.sprinkler_water_target
```

When recreating helpers, use the documented entity IDs consistently.

Changing an entity ID can prevent automations from finding the helper they expect.

Refer to:

```text
Helper Reference
```

for the complete helper details and documented purposes.

---

# 9. Restore Weather and Template Sensors

The irrigation system uses weather information as part of its virtual soil moisture and watering calculations.

The documented temperature sensor is:

```text
sensor.met_office_ashby_de_la_zouch_temperature
```

The calculated rainfall forecast sensor is:

```text
sensor.sprinkler_forecast_rain_24h
```

Before restoring the irrigation automations, confirm that these sensors are available and returning sensible values.

A missing or unavailable sensor can cause calculations to behave unexpectedly.

---

# 10. Restore the Core Automations

The system currently uses three core automations:

### 1. Virtual Soil Moisture Update

```text
Sprinklers - Update Virtual Soil Moisture
```

Runs at:

```text
03:45
```

Its purpose is to estimate daily moisture loss using the temperature sensor and update the virtual soil moisture helper.

---

### 2. Water Requirement Calculation

```text
Sprinklers - Calculate Water Requirement
```

Runs at:

```text
04:00
```

The current logic checks:

- Whether sprinklers are enabled.
- Whether the current day matches the every-two-day schedule.
- Virtual soil moisture.
- Moisture target.
- Forecast rainfall.
- Rain skip threshold.
- Recovery mode.

It then calculates:

```text
Water required
      │
      ▼
Water target in mm
      │
      ▼
Total runtime
```

---

### 3. Morning Cycle and Soak

```text
Sprinklers - Morning Cycle and Soak
```

Triggers at:

```text
Sunrise + 30 minutes
```

The automation:

1. Checks that sprinklers are enabled.
2. Checks that total runtime is greater than zero.
3. Runs the seven zones sequentially.
4. Limits each pass using the cycle runtime.
5. Applies a soak delay between complete passes when further watering is required.

Restore the automations from the complete YAML backups rather than attempting to recreate them from memory.

---

# 11. Restore the System in a Safe State

After rebuilding the helpers and automations, do not immediately enable automatic watering.

Start with:

```text
Sprinklers Enabled: OFF
```

Check that all helpers have reasonable values.

In particular, inspect:

```text
input_number.sprinkler_soil_moisture
input_number.sprinkler_total_runtime
input_number.sprinkler_water_target
```

A restored helper may initially contain a default value rather than the value that existed before the failure.

Decide whether those values should be restored from a record or allowed to rebuild naturally over subsequent calculation cycles.

---

# 12. Recommended Functional Test

Before returning the system to normal operation, perform a short controlled test.

## Step 1 — Test a Single Zone

Turn on one sprinkler manually for a short period.

Confirm:

- The correct relay operates.
- The correct valve opens.
- Water reaches the correct sprinkler.
- The sprinkler turns off correctly.

## Step 2 — Test All Zones

Repeat the test for all seven active zones.

## Step 3 — Test Calculation Logic

Run or trigger the calculation automation in a controlled way and confirm that:

- The automation completes successfully.
- The calculated values are sensible.
- The total runtime is updated.

## Step 4 — Test Cycle and Soak

Use a short test runtime.

Confirm that:

```text
Zone 1
   ▼
Zone 2
   ▼
Zone 3
   ▼
...
   ▼
Zone 7
```

operates in the expected sequence.

If more than one watering pass is required, confirm that the soak delay occurs before the next pass.

---

# 13. Re-enable Automatic Watering

Only after successful testing should automatic watering be re-enabled.

Recommended final checks:

- [ ] ESP32 is online.
- [ ] All seven active zones operate correctly.
- [ ] All zones turn off correctly.
- [ ] Weather sensors are available.
- [ ] Helpers have sensible values.
- [ ] Automations are enabled.
- [ ] Automation traces show no errors.
- [ ] The cycle runtime remains below the ESPHome safety timeout.
- [ ] `input_boolean.sprinklers_enabled` is intentionally turned on.

Once these checks are complete, the system can return to normal automatic operation.

---

# 14. Full Rebuild Checklist

Use this checklist if the entire system needs to be rebuilt.

## Hardware

- [ ] Install or reconnect ESP32.
- [ ] Install or reconnect relay board.
- [ ] Confirm power supply.
- [ ] Confirm GPIO wiring.
- [ ] Confirm sprinkler valve connections.

## ESPHome

- [ ] Restore ESPHome YAML.
- [ ] Configure Wi-Fi secrets.
- [ ] Install firmware.
- [ ] Confirm device is online.
- [ ] Test all relay outputs.

## Home Assistant

- [ ] Restore ESPHome integration.
- [ ] Recreate helpers.
- [ ] Restore weather integration.
- [ ] Restore template sensors.
- [ ] Restore automations.

## Testing

- [ ] Test each zone manually.
- [ ] Test the water requirement calculation.
- [ ] Test the cycle-and-soak sequence.
- [ ] Check automation traces.
- [ ] Check ESPHome logs.

## Return to Service

- [ ] Confirm all settings.
- [ ] Enable sprinklers.
- [ ] Monitor the first automatic watering cycle.
- [ ] Update documentation if anything changed.

---

# 15. Recovery Troubleshooting

If a rebuild does not work immediately, avoid making random changes to several components.

Return to the lowest layer that can be tested successfully.

For example:

```text
Can Home Assistant see the ESP32?
        │
       No ───► Check power / Wi-Fi / ESPHome
        │
       Yes
        ▼
Can a relay be operated manually?
        │
       No ───► Check GPIO / relay wiring
        │
       Yes
        ▼
Does the correct sprinkler operate?
        │
       No ───► Check valve / zone plumbing
        │
       Yes
        ▼
Does the automation control it?
        │
       No ───► Check helpers / conditions / automation
        │
       Yes
        ▼
System recovered
```

This approach avoids changing a working layer while attempting to repair a problem elsewhere.

---

# 16. After a Successful Recovery

Once the system is working again:

1. Save the confirmed working configuration.
2. Update the GitHub repository if anything changed.
3. Record any changes to GPIO wiring or hardware.
4. Update the relevant documentation.
5. Keep the previous backup until the restored system has operated reliably.

A recovery is also a useful opportunity to improve the documentation if anything was difficult to remember or unclear during the rebuild.

---

# 17. Recovery Principles

The most important principle is:

```text
Restore
   │
   ▼
Test
   │
   ▼
Confirm
   │
   ▼
Move to the next layer
```

The sprinkler system does not need to be rebuilt from memory.

The purpose of this repository and its appendices is to provide a reliable reference for rebuilding the system in a controlled and understandable way.

---

## Related Documentation

- [Installation Troubleshooting](Installation-Troubleshooting.md)
- [System Maintenance](System-Maintenance.md)
- [ESPHome Firmware Reference](ESPHome-Firmware-Reference.md)
- [GPIO & ESP32 Pin Reference](GPIO-ESP32-Pin-Reference.md)
- [Hardware Specification](Hardware-Specification.md)
- [Automation Reference](Automation-Reference.md)
- [Helper Reference](Helper-Reference.md)
- [Irrigation Calibration](Irrigation-Calibration.md)

---

*Home Assistant Smart Sprinkler System — Appendix K: Quick Recovery & Rebuild Guide*
