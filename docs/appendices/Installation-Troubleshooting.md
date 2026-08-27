# Appendix I — Installation Troubleshooting

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** Installation, Testing & Troubleshooting Guide  
> **Status:** Current system reference

## Overview

This appendix provides a practical troubleshooting guide for the Home Assistant Smart Sprinkler System.

The system consists of several connected layers:

```text
Home Assistant
      │
      ▼
Automations & Helpers
      │
      ▼
ESPHome Integration
      │
      ▼
ESP32 Controller
      │
      ▼
9-Channel Relay Board
      │
      ▼
Sprinkler Zones & Valves
      │
      ▼
Sprinkler Heads
```

A problem can occur at any point in this chain.

For this reason, troubleshooting should normally begin by identifying the lowest level of the system that is not working correctly.

For example, if a relay can be manually controlled from Home Assistant but does not operate during the morning watering schedule, the hardware is probably working and the problem is more likely to be within the Home Assistant automation or its conditions.

---

# 1. Recommended Troubleshooting Method

When diagnosing a problem, work through the system one layer at a time.

```text
Is Home Assistant working correctly?
        │
        ▼
Is the ESPHome device online?
        │
        ▼
Can the relay be controlled manually?
        │
        ▼
Does the correct sprinkler zone operate?
        │
        ▼
Does water reach the sprinkler?
        │
        ▼
Is the sprinkler providing the expected coverage?
```

Avoid changing several parts of the system at the same time.

A single change followed by a test makes it much easier to identify the cause of a problem.

---

# 2. ESP32 Is Offline

## Symptoms

The sprinkler controller may appear as unavailable in Home Assistant.

The individual sprinkler switch entities may also become unavailable.

## Possible Causes

- ESP32 has lost power.
- Wi-Fi connection has been lost.
- The ESP32 has restarted unexpectedly.
- ESPHome firmware has failed to start correctly.
- The Home Assistant API connection is unavailable.

## Checks

### Check Power

Confirm that the ESP32 is receiving power.

If the onboard LED is expected to operate, note that it only flashes when the configured script is triggered and is not necessarily a permanent power indicator.

### Check ESPHome Logs

Review the ESPHome logs for:

- Startup messages.
- Wi-Fi connection status.
- API connection status.
- Restart loops.
- Configuration errors.

### Check Wi-Fi

Confirm that the ESP32 is within reliable range of the Wi-Fi network.

The sprinkler controller depends on a stable network connection for normal Home Assistant operation.

### Restart the Device

If appropriate, restart the ESP32 and confirm that it reconnects successfully.

After a restart, the sprinkler relay switches are configured to initialise in the off state.

---

# 3. ESP32 Is Online but a Relay Does Not Operate

## Symptoms

The ESPHome device appears online, but switching a sprinkler zone on from Home Assistant does not operate the expected relay.

## Check the GPIO Assignment

Confirm that the relay input is connected to the GPIO configured in ESPHome.

The current GPIO assignments are:

| Sprinkler Zone | ESPHome ID | GPIO |
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

## Check the Relay Board

Confirm that:

- The relay board has power.
- The control wiring is connected securely.
- The correct ESP32 GPIO is connected to the correct relay input.
- The relay board and ESP32 have the required electrical reference connections for the installation.

## Check Active-Low Operation

The relay board is configured as active-low.

Each relay uses:

```yaml
inverted: true
```

If a replacement relay board behaves differently, check whether it is also an active-low design before changing the configuration.

---

# 4. The Wrong Relay Operates

## Symptoms

Turning on Garden Sprinkler 1 causes another relay channel to operate.

## Likely Cause

The physical relay input wiring does not match the GPIO assignments in ESPHome.

For example, the current configuration expects:

```text
GPIO13 → Relay IN1
GPIO14 → Relay IN2
GPIO16 → Relay IN3
GPIO17 → Relay IN4
GPIO18 → Relay IN5
GPIO19 → Relay IN6
GPIO21 → Relay IN7
GPIO22 → Relay IN8
GPIO23 → Relay IN9
```

## Solution

Work through the wiring one channel at a time.

Do not rely on wire colour alone unless the wiring has been clearly documented.

Test each zone manually from Home Assistant and confirm that the expected relay operates before moving to the next channel.

---

# 5. A Relay Turns On but Does Not Turn Off

## Normal Behaviour

Home Assistant should normally turn the relay off after the runtime defined by the irrigation automation.

In addition, every ESPHome relay includes a 31-minute automatic safety timeout.

## Check the Home Assistant Automation

Check whether the automation is still running and whether it has reached the expected `switch.turn_off` action.

Review the automation trace where available.

## Check ESPHome

Each relay contains an automatic safety sequence similar to:

```yaml
on_turn_on:
  - delay: 31min
  - switch.turn_off: zone_1
```

If the relay remains on beyond the expected safety period, investigate the ESPHome configuration and device logs.

> **Safety note:** If a sprinkler valve remains energised unexpectedly, turn the zone off manually and, if necessary, isolate the irrigation system before continuing troubleshooting.

---

# 6. Sprinklers Run for Exactly 31 Minutes

## Likely Cause

The ESPHome safety timeout has been reached.

Each relay is configured to automatically turn off after 31 minutes.

This is a failsafe and should not normally be responsible for ending a standard watering cycle.

## Check

Review the Home Assistant automation and confirm that it is sending its normal switch-off command before the ESPHome safety timeout is reached.

The normal cycle runtime should remain below the safety limit.

---

# 7. A Sprinkler Zone Does Not Run During the Morning Automation

## First Check the Automation Conditions

The `Sprinklers - Morning Cycle and Soak` automation only runs when its conditions are satisfied.

The relevant conditions include:

```text
Sprinklers Enabled = ON

AND

Sprinkler Total Runtime > 0
```

Check:

```text
input_boolean.sprinklers_enabled
```

and:

```text
input_number.sprinkler_total_runtime
```

If either condition is preventing the automation from running, the hardware may be functioning correctly.

## Check the Automation Trace

Review the trace for:

```text
Sprinklers - Morning Cycle and Soak
```

The trace can show whether:

- The automation was triggered.
- A condition failed.
- A zone was switched on.
- The automation encountered an error.
- The automation was delayed during a watering or soak period.

---

# 8. The Morning Watering Automation Did Not Start

## Check Sunrise Triggering

The automation is configured to trigger at:

```text
Sunrise + 30 minutes
```

Check that Home Assistant has the correct location and time configuration.

Also check that the automation is enabled.

## Check the Master Switch

Confirm that:

```text
input_boolean.sprinklers_enabled
```

is turned on.

## Check Total Runtime

The morning automation requires:

```text
input_number.sprinkler_total_runtime > 0
```

If the calculated total runtime is zero, the system will correctly skip watering.

This may be intentional if:

- Rainfall is forecast.
- The soil moisture model is at or above the target.
- The water requirement calculation has determined that irrigation is not required.

---

# 9. No Watering Runtime Is Being Calculated

The automation responsible for calculating the irrigation requirement is:

```text
Sprinklers - Calculate Water Requirement
```

It is configured to run at:

```text
04:00
```

and is designed to run every two days according to the current day-of-year calculation.

## Check the Master Enable Switch

Confirm:

```text
input_boolean.sprinklers_enabled
```

is on.

## Check the Every-Two-Day Condition

The automation currently uses:

```jinja
{{ (now().strftime('%j') | int) % 2 == 1 }}
```

This means the calculation only runs on alternating days based on the day number within the year.

If the automation does not run on a particular day, this may be expected behaviour rather than a fault.

## Check the Rain Forecast

The calculation uses:

```text
sensor.sprinkler_forecast_rain_24h
```

and compares it with:

```text
input_number.sprinkler_rain_skip_threshold
```

If forecast rain meets or exceeds the configured threshold, the automation intentionally sets the watering requirement and total runtime to zero.

## Check Virtual Soil Moisture

The calculation also checks:

```text
input_number.sprinkler_soil_moisture
```

against:

```text
input_number.sprinkler_moisture_target
```

If the virtual soil moisture is already at or above the target, watering is intentionally skipped.

---

# 10. Virtual Soil Moisture Is Not Changing

The virtual soil moisture model is updated by:

```text
Sprinklers - Update Virtual Soil Moisture
```

The automation is scheduled for:

```text
03:45
```

## Check Temperature Sensor

The automation uses:

```text
sensor.met_office_ashby_de_la_zouch_temperature
```

Confirm that this sensor has a valid numeric value.

The temperature is used to estimate daily moisture loss.

## Check Automation Status

Confirm that the automation is enabled and has run at the expected time.

## Check the Helper

Confirm that:

```text
input_number.sprinkler_soil_moisture
```

is available and has not been removed or recreated with a different entity ID.

---

# 11. Rain Forecast Appears Incorrect

The irrigation system uses the template sensor:

```text
sensor.sprinkler_forecast_rain_24h
```

If this value appears incorrect, the problem may be within the template sensor or the weather information it uses.

Check:

1. Whether the source weather information is available.
2. Whether the template sensor is updating.
3. Whether the value is numeric.
4. Whether the forecast represents the expected time period.

If the rain forecast sensor is unavailable or returns an unexpected value, the watering calculation may not behave as intended.

---

# 12. Cycle and Soak Behaviour Appears Too Slow

The cycle-and-soak system works through all seven zones sequentially.

This means a complete watering pass can take a significant amount of time.

For example, with:

```text
Cycle Runtime: 5 minutes
Seven Zones: 7
```

A single pass requires approximately:

```text
5 × 7 = 35 minutes
```

before any configured soak period is added.

This is expected behaviour.

Remember that each individual zone is already resting while the remaining zones are being watered.

When considering the soak setting, take into account both:

```text
Time spent watering other zones
+
Configured soak time
```

---

# 13. One Area of the Lawn Is Dry

Before increasing the overall sprinkler runtime, check whether the problem is localised.

Possible causes include:

- Sprinkler arc adjustment.
- Nozzle selection.
- Sprinkler obstruction.
- Reduced pressure.
- Edge coverage.
- Wind.
- Poor overlap between sprinkler heads.

If the main lawn is healthy but the edges are dry, increasing the runtime for all zones may result in overwatering the areas that are already receiving sufficient water.

Refer to the Irrigation Calibration appendix when investigating uneven coverage.

---

# 14. Water Is Running Off or Pooling

If water is running off the lawn or pooling, consider reducing:

```text
input_number.sprinkler_cycle_runtime
```

A shorter cycle allows the system to apply water in smaller amounts.

The soak period can then allow time for the water to absorb before the next pass.

The cycle-and-soak approach is particularly useful when the soil cannot absorb water as quickly as the sprinkler applies it.

---

# 15. ESP32 Restarted During Watering

The ESPHome configuration uses:

```yaml
restore_mode: ALWAYS_OFF
```

This means that following a restart, the sprinkler outputs are intended to initialise in the off state.

If a restart occurs during watering:

1. The currently active zone should stop.
2. Home Assistant may continue or interrupt the automation depending on its state and the timing of the restart.
3. The automation trace should be reviewed before manually restarting the watering schedule.

Avoid assuming that a partially completed watering cycle will automatically restart correctly.

---

# 16. Testing After Changes

Whenever changes are made to the ESPHome firmware, wiring, helpers or automations, test the system in stages.

## Recommended Test Order

### Stage 1 — ESP32

- Confirm the device is online.
- Confirm the ESPHome API is connected.

### Stage 2 — Relay Board

- Test each relay individually.
- Confirm the correct relay operates.

### Stage 3 — Sprinkler Zones

- Confirm each valve and sprinkler zone operates correctly.

### Stage 4 — Home Assistant Entities

- Confirm all helpers and sensors have valid values.

### Stage 5 — Water Requirement Calculation

- Run or test the calculation automation.
- Confirm the calculated values are reasonable.

### Stage 6 — Morning Watering

- Test the cycle-and-soak sequence with a short runtime.
- Confirm zones run sequentially.
- Confirm zones turn off correctly.
- Confirm the soak delay behaves as expected.

Testing in this order helps prevent a software problem being mistaken for a hardware problem, or vice versa.

---

# 17. Quick Diagnostic Reference

| Symptom | First Place to Check |
|---|---|
| ESP32 unavailable | Power, Wi-Fi, ESPHome logs |
| Relay does not operate | GPIO wiring and relay power |
| Wrong relay operates | GPIO-to-relay wiring |
| Relay stays on | Home Assistant automation and ESPHome timeout |
| Zone skipped | Automation conditions and trace |
| No watering scheduled | Water requirement automation |
| Runtime is zero | Rain forecast and soil moisture values |
| Soil moisture unchanged | Update automation and temperature sensor |
| Lawn edges dry | Sprinkler coverage and calibration |
| Water running off | Cycle runtime and soak settings |

---

# 18. Before Making Major Changes

Before changing the sprinkler system configuration:

- [ ] Save a copy of the current ESPHome YAML.
- [ ] Save a copy of the Home Assistant automations.
- [ ] Record the current helper values.
- [ ] Make one logical change at a time.
- [ ] Test the affected part of the system.
- [ ] Check automation traces and ESPHome logs.
- [ ] Keep the 31-minute ESPHome safety timeout in place.
- [ ] Confirm all sprinkler zones are off before leaving the system unattended.

A working configuration is valuable. Keeping a known-good backup makes experimentation much safer.

---

## Related Documentation

- [Hardware Specification](Hardware-Specification.md)
- [GPIO & ESP32 Pin Reference](GPIO-ESP32-Pin-Reference.md)
- [ESPHome Firmware Reference](ESPHome-Firmware-Reference.md)
- [Irrigation Calibration](Irrigation-Calibration.md)
- [Automation Reference](Automation-Reference.md)
- [Helper Reference](Helper-Reference.md)
- [Installation & Setup](../docs/09-Installation-and-Setup.md)

---

*Home Assistant Smart Sprinkler System — Appendix I: Installation Troubleshooting*
