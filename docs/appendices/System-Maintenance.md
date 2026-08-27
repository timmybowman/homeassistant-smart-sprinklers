# Appendix J — System Maintenance

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** Routine Maintenance & Preventative Care  
> **Status:** Current system reference

## Overview

This appendix provides a routine maintenance guide for the Home Assistant Smart Sprinkler System.

Although much of the system is automated, regular checks help ensure that both the electronic control system and the physical irrigation system continue to operate reliably.

The system consists of several areas that benefit from occasional inspection:

```text
Home Assistant Configuration
          │
          ▼
ESPHome Firmware & ESP32 Controller
          │
          ▼
Relay Board & Electrical Connections
          │
          ▼
Sprinkler Valves & Pipework
          │
          ▼
Sprinkler Heads & Lawn Coverage
```

Preventative maintenance is generally easier than diagnosing a failure during a period of hot or dry weather.

---

# 1. Maintenance Philosophy

The aim of routine maintenance is not to make frequent changes to a working system.

Instead, maintenance should focus on:

- Checking that the system is operating as expected.
- Identifying physical wear or damage.
- Confirming that sensors and helpers contain sensible values.
- Keeping backups of known-working configurations.
- Making gradual adjustments when the garden or system changes.

A useful principle is:

```text
Observe
   │
   ▼
Check
   │
   ▼
Record
   │
   ▼
Change only when necessary
```

Avoid making unnecessary changes to automations or firmware simply because a newer version is available.

A stable, working irrigation system is generally more valuable than frequent changes without a clear benefit.

---

# 2. Regular Maintenance Schedule

The following schedule is intended as a practical guide rather than a strict requirement.

| Frequency | Recommended Checks |
|---|---|
| Weekly during watering season | Check lawn coverage and sprinkler operation |
| Monthly | Check sprinkler heads, pipework and controller status |
| Before the main watering season | Test all zones and review configuration |
| After major system changes | Test the complete system in stages |
| Before winter | Consider frost protection and shutdown requirements |
| After winter | Inspect the system before returning it to regular use |

---

# 3. Weekly Checks During the Watering Season

During periods when the system is actively watering, occasional visual checks are useful.

## Check Lawn Appearance

Look for:

- Dry patches.
- Areas that remain excessively wet.
- Changes in grass colour.
- Uneven growth.
- Water pooling.
- Runoff from the lawn.

A localised dry area does not necessarily mean that the entire system needs a longer runtime.

Check sprinkler coverage first.

## Observe a Watering Cycle

When convenient, occasionally watch part of a watering cycle.

Confirm that:

- Only the intended zone operates.
- Sprinklers rise and retract correctly.
- Water is being distributed across the expected area.
- No significant leaks are visible.
- The automation moves between zones correctly.

This can identify developing problems before they become more significant.

---

# 4. Monthly ESP32 and ESPHome Checks

The ESP32 is the physical controller responsible for operating the relay outputs.

## Check Device Availability

Confirm that the ESPHome device is online in Home Assistant.

The device should normally remain available and responsive.

## Review ESPHome Logs When Necessary

There is no need to constantly monitor logs on a working system.

However, logs are useful if:

- The device has become unavailable.
- A sprinkler has behaved unexpectedly.
- The ESP32 has restarted.
- A firmware update has recently been installed.

Look for:

- Wi-Fi connection problems.
- Repeated restarts.
- API connection failures.
- Unexpected errors.

## Check the Safety Timeout

Each sprinkler relay is configured with a 31-minute automatic safety timeout.

This is an important independent safeguard.

The safety timeout should remain in place unless there is a clear and carefully considered reason to change it.

The normal irrigation automation should complete each individual watering cycle before this limit is reached.

---

# 5. Relay Board and Electrical Maintenance

The relay board should be inspected periodically, particularly if it is installed in an environment exposed to temperature or humidity changes.

## Check for Physical Issues

Inspect for:

- Loose connections.
- Damaged wiring.
- Signs of corrosion.
- Moisture ingress.
- Excessive heat.
- Damaged relay terminals.

If any electrical work is required, ensure that power is safely isolated before working on the controller or relay connections.

## Confirm Correct Zone Operation

Occasionally test the sprinkler zones individually from Home Assistant.

The current active garden zones are:

| Zone | Home Assistant Entity |
|---|---|
| Garden Sprinkler 1 | `switch.sprinklers_garden_sprinkler_1` |
| Garden Sprinkler 2 | `switch.sprinklers_garden_sprinkler_2` |
| Garden Sprinkler 3 | `switch.sprinklers_garden_sprinkler_3` |
| Garden Sprinkler 4 | `switch.sprinklers_garden_sprinkler_4` |
| Garden Sprinkler 5 | `switch.sprinklers_garden_sprinkler_5` |
| Garden Sprinkler 6 | `switch.sprinklers_garden_sprinkler_6` |
| Garden Sprinkler 7 | `switch.sprinklers_garden_sprinkler_7` |

The ESPHome firmware also contains outputs for Garden Sprinklers 8 and 9, although the current automatic irrigation sequence uses the first seven zones.

---

# 6. Sprinkler Head Maintenance

Sprinkler heads are exposed to soil, grass, weather and garden debris.

They should be checked periodically for:

- Dirt or debris around the sprinkler.
- Obstructed movement.
- Incorrect spray direction.
- Damaged nozzles.
- Reduced throw distance.
- Water leaking around the sprinkler body.

## Check Sprinkler Arcs

If a dry patch develops, check the sprinkler arc before increasing the watering runtime.

A sprinkler may have been moved slightly by:

- Lawn maintenance.
- Ground movement.
- Accidental contact.
- Frost or seasonal soil changes.

Small adjustments to sprinkler direction can sometimes solve a coverage problem without changing the automation.

---

# 7. Pipework and Water Supply

Periodically inspect accessible pipework and fittings for leaks.

Pay particular attention to:

- Connections near the manifold.
- Pipe joints.
- Connections near sprinkler heads.
- Any areas where pipework has been disturbed by gardening work.

A leak can reduce pressure and affect sprinkler performance without necessarily being immediately obvious from the lawn surface.

If the coverage of several sprinklers appears to reduce at the same time, investigate the water supply and possible leaks before changing sprinkler settings.

---

# 8. Home Assistant Helper Maintenance

The system uses helpers to store configuration values and calculated results.

The most important helpers should be checked occasionally to ensure that they still contain sensible values.

## Configuration Helpers

These include:

```text
input_boolean.sprinklers_enabled
input_boolean.sprinkler_recovery_mode

input_number.sprinkler_cycle_runtime
input_number.sprinkler_moisture_target
input_number.sprinkler_rain_skip_threshold
input_number.sprinkler_runtime
input_number.sprinkler_soak_time
```

## Calculated and Model Helpers

These include:

```text
input_number.sprinkler_soil_moisture
input_number.sprinkler_total_runtime
input_number.sprinkler_water_target
```

When reviewing these values, ask whether they make sense in relation to recent weather and watering.

For example:

- A very high total runtime during wet weather may indicate a calculation issue.
- A soil moisture value that never changes may indicate that the update automation is not running.
- A permanently zero runtime may be intentional, but should be understood.

---

# 9. Weather and Forecast Checks

The irrigation calculations depend on weather-related information.

The current system uses:

```text
sensor.met_office_ashby_de_la_zouch_temperature
```

and:

```text
sensor.sprinkler_forecast_rain_24h
```

These values should be checked if the irrigation decisions begin to appear unreasonable.

## Temperature

The temperature value contributes to the virtual soil moisture depletion model.

Unexpected or unavailable temperature data may affect the estimated daily moisture loss.

## Rain Forecast

The rainfall forecast is used to decide whether watering should be skipped.

If the system unexpectedly waters before significant forecast rain, or repeatedly skips watering when conditions are dry, review the forecast sensor and its source data.

---

# 10. Automation Maintenance

The system currently relies on three core automations:

1. `Sprinklers - Update Virtual Soil Moisture`
2. `Sprinklers - Calculate Water Requirement`
3. `Sprinklers - Morning Cycle and Soak`

These automations should normally be left alone when working correctly.

After changes to Home Assistant, ESPHome or related entities, it is worth confirming that:

- The automations remain enabled.
- Entity IDs have not changed.
- Required helpers still exist.
- Weather sensors remain available.
- Automation traces show expected behaviour.

## Review Automation Traces

Automation traces are particularly useful after a system change.

They can help confirm:

```text
Trigger received
      │
      ▼
Conditions passed
      │
      ▼
Variables calculated
      │
      ▼
Actions completed
```

There is generally no need to review every trace on a working system, but they are valuable when investigating unexpected behaviour.

---

# 11. Seasonal Review

At the beginning and end of the main watering season, review the system settings.

Consider:

- Has the lawn changed?
- Have any sprinkler heads been moved?
- Has garden landscaping altered coverage?
- Are the current runtime assumptions still reasonable?
- Have dry patches developed in new areas?

The assumed precipitation rate and virtual soil moisture model can be refined gradually based on real-world observations.

Refer to the **Irrigation Calibration** appendix when reviewing watering performance.

---

# 12. Winter and Frost Considerations

The appropriate winter procedure depends on the physical construction of the irrigation system and which components are exposed to freezing temperatures.

Where relevant, consider:

- Turning off automatic watering.
- Isolating the water supply.
- Draining vulnerable pipework.
- Protecting exposed valves and equipment.
- Ensuring sprinkler heads and pipework are not left vulnerable to frost damage.

Before shutting the system down for an extended period, record the current Home Assistant helper settings if seasonal changes are likely to be made.

## Winter Shutdown Checklist

- [ ] Turn off `input_boolean.sprinklers_enabled`.
- [ ] Confirm all sprinkler switches are off.
- [ ] Isolate the irrigation water supply if required.
- [ ] Drain or protect vulnerable pipework where appropriate.
- [ ] Check the ESP32 and relay enclosure for moisture protection.
- [ ] Save a backup of the current configuration.

---

# 13. Spring Startup Checklist

Before returning the system to regular automatic operation:

- [ ] Inspect accessible pipework and fittings.
- [ ] Check sprinkler heads for damage or obstruction.
- [ ] Confirm the water supply is available.
- [ ] Check the ESP32 is online.
- [ ] Confirm all relay outputs initialise in the off state.
- [ ] Test each sprinkler zone individually.
- [ ] Check Home Assistant helpers.
- [ ] Confirm weather sensors are available.
- [ ] Test the calculation automations.
- [ ] Run a short test watering cycle.
- [ ] Inspect lawn coverage.
- [ ] Re-enable automatic watering when satisfied.

A short manual test is preferable to discovering a problem during the first unattended watering cycle of the season.

---

# 14. Software and Firmware Updates

Home Assistant and ESPHome are actively developed platforms, and updates may occasionally affect integrations, configuration formats or behaviour.

Before applying significant updates:

1. Ensure the current system is working.
2. Save copies of the ESPHome and Home Assistant configuration.
3. Read relevant update notes where practical.
4. Apply changes carefully.
5. Test the sprinkler system after updating.

It is generally preferable not to perform major updates immediately before leaving home or during a period when reliable irrigation is particularly important.

---

# 15. Backup Maintenance

A backup is most useful when it represents a configuration that is known to work.

After making a successful change to the system, consider updating the project documentation and configuration backups.

Important items to preserve include:

```text
ESPHome YAML configuration
Home Assistant automation YAML
Template sensor configuration
Helper configuration and values
GPIO wiring reference
Hardware documentation
```

The GitHub repository provides a useful central location for maintaining this documentation and configuration history.

---

# 16. Suggested Maintenance Log

A simple maintenance record can help identify recurring issues.

| Date | Area Checked | Observation | Action Taken |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

Examples of useful entries include:

- Adjusted sprinkler arc.
- Removed debris from sprinkler head.
- Tested all seven zones.
- Updated ESPHome firmware.
- Changed cycle runtime.
- Investigated dry patch.
- Confirmed rainfall forecast sensor operation.

The record does not need to be detailed. Its main purpose is to provide useful history when a problem develops later.

---

# 17. System Health Checklist

The following checklist provides a quick overall health check.

## Home Assistant

- [ ] Core automations are enabled.
- [ ] Required helpers are available.
- [ ] Weather sensors have valid values.

## ESPHome

- [ ] ESP32 is online.
- [ ] Home Assistant API connection is working.
- [ ] No unexpected restart pattern is visible.

## Relay System

- [ ] Relays respond to manual testing.
- [ ] Correct zones operate.
- [ ] No visible wiring damage is present.

## Irrigation

- [ ] No obvious leaks are visible.
- [ ] Sprinklers provide expected coverage.
- [ ] No significant runoff is occurring.
- [ ] Lawn condition is generally healthy.

## Documentation

- [ ] Current configuration is backed up.
- [ ] Significant changes have been documented.
- [ ] The repository reflects the current working system.

---

# 18. Maintenance Principles

The long-term maintenance strategy for the sprinkler system can be summarised simply:

```text
Keep a working backup
        │
        ▼
Check the physical system
        │
        ▼
Observe the lawn
        │
        ▼
Review unusual behaviour
        │
        ▼
Make gradual changes
        │
        ▼
Update the documentation
```

The system is designed to be understandable and maintainable rather than completely dependent on a single fixed configuration.

Regular observation of both the system and the lawn will help improve the irrigation logic over time.

---

## Related Documentation

- [Installation Troubleshooting](Installation-Troubleshooting.md)
- [Irrigation Calibration](Irrigation-Calibration.md)
- [Hardware Specification](Hardware-Specification.md)
- [GPIO & ESP32 Pin Reference](GPIO-ESP32-Pin-Reference.md)
- [ESPHome Firmware Reference](ESPHome-Firmware-Reference.md)
- [Automation Reference](Automation-Reference.md)
- [Helper Reference](Helper-Reference.md)

---

*Home Assistant Smart Sprinkler System — Appendix J: System Maintenance*
