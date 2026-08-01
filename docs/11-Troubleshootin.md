# Troubleshooting

| | |
|---|---|
| **Project** | Home Assistant Smart Sprinklers |
| **Document** | 11 – Troubleshooting |
| **Version** | 1.0 |
| **Platform** | Home Assistant & ESPHome |
| **Status** | Live System |
| **Last Updated** | July 2026 |

---

## Summary

This document provides a structured approach to diagnosing faults within the Home Assistant Smart Sprinkler system.

The irrigation controller consists of multiple independent components:

- Home Assistant
- ESPHome
- ESP32 Hardware
- Relay Board
- Irrigation Valves
- Weather Integration
- Home Network

Understanding which component is responsible for a fault is often the quickest route to a solution.

---

# Contents

- Diagnostic Strategy
- System Health Checklist
- Common Problems
- ESPHome Issues
- Home Assistant Issues
- Automation Issues
- Weather Integration
- Hardware Issues
- Wi-Fi Problems
- Irrigation Problems
- Frequently Asked Questions
- Maintenance Tips

---

# Diagnostic Strategy

Whenever possible, begin troubleshooting at the highest level before investigating individual components.

```
Problem Detected

↓

Home Assistant Running?

↓

ESPHome Online?

↓

Device Available?

↓

Automation Triggered?

↓

Relay Activated?

↓

Valve Opened?

↓

Sprinkler Operating?
```

This simple workflow usually identifies the faulty component within a few minutes.

---

# System Health Checklist

Before investigating a fault, verify the following:

| Check | Expected |
|---------|----------|
| Home Assistant running | ✅ |
| ESPHome device online | ✅ |
| Wi-Fi connected | ✅ |
| Weather entities updating | ✅ |
| Helpers available | ✅ |
| Automations enabled | ✅ |
| Relay board powered | ✅ |
| Irrigation transformer powered | ✅ |

---

# Common Problems

---

## Nothing Waters

### Possible Causes

- Sprinklers Enabled is OFF
- Total Runtime is zero
- Rain Skip Threshold triggered
- ESPHome offline
- Relay board unpowered
- Irrigation transformer off

### Checks

Verify:

```
input_boolean.sprinklers_enabled
```

is ON.

Then inspect:

```
input_number.sprinkler_total_runtime
```

If this is zero, review the Calculate Water Requirement automation.

---

## Only One Zone Works

Possible causes include:

- Incorrect relay wiring
- Damaged valve
- Incorrect GPIO assignment
- Faulty relay channel

Test each sprinkler manually from Home Assistant.

---

## Wrong Zone Activates

Usually caused by wiring.

Check:

- Relay wiring
- ESPHome GPIO assignments
- Solenoid wiring

Compare the physical installation against:

```
docs/02-Hardware.md
```

---

## Watering Never Stops

The ESPHome firmware contains a hardware timeout.

Each relay automatically switches OFF after approximately:

```
31 minutes
```

If watering continues beyond this period, inspect the relay board and valve wiring.

---

## Watering Stops Too Soon

Possible causes:

- Runtime calculated incorrectly
- ESPHome safety timeout reached
- Automation interrupted
- Manual stop

Review the Automation Trace for the Morning Cycle & Soak automation.

---

# ESPHome Issues

---

## Device Offline

Check:

- Power supply
- USB cable
- Wi-Fi
- ESPHome logs

If necessary, reconnect using USB and upload the firmware again.

---

## Cannot Upload OTA

Possible causes:

- ESP32 offline
- Wi-Fi disconnected
- Device renamed
- OTA password mismatch

Verify that the ESPHome device appears online before attempting an OTA update.

---

## Relays Do Not Click

Possible causes:

- Incorrect GPIO
- Relay board not powered
- Active-low configuration missing
- Wiring fault

Confirm every relay configuration includes:

```yaml
inverted: true
```

---

## Onboard LED Does Not Flash

The firmware flashes the onboard LED whenever a relay changes state.

If no flashing occurs:

- firmware may not be running,
- script may have been modified,
- GPIO2 may be unavailable.

---

# Home Assistant Issues

---

## Automation Never Runs

Check:

- Automation enabled
- Trigger time correct
- Conditions satisfied

Use the **Trace** feature to determine why execution stopped.

---

## Helper Missing

If a helper has been deleted:

Recreate it using:

```
docs/05-Helpers.md
```

Ensure the entity ID matches the documentation exactly.

---

## Entity Not Found

Usually indicates:

- renamed helper
- renamed ESPHome device
- deleted entity

Search Developer Tools → States.

---

# Automation Issues

---

## Runtime Always Zero

Possible causes:

- Forecast rain above threshold
- Soil moisture already at target
- Automation failed
- Weather entity unavailable

Inspect:

- Water Target
- Soil Moisture
- Forecast Rain

---

## Soil Moisture Never Changes

Check:

```
Sprinklers - Update Virtual Soil Moisture
```

Verify:

- automation enabled
- trigger time reached
- temperature sensor available

---

## Cycle & Soak Does Not Repeat

Check:

```
input_number.sprinkler_cycle_runtime
```

and

```
input_number.sprinkler_total_runtime
```

If Total Runtime is smaller than Cycle Runtime only one watering pass will occur.

---

# Weather Integration

---

## Temperature Not Updating

Verify:

- Met Office integration loaded
- Internet connection available
- Entity ID unchanged

---

## Rain Forecast Missing

Inspect:

```
sensor.sprinkler_forecast_rain_24h
```

If unavailable:

- review template sensor,
- check forecast entities,
- restart Home Assistant.

---

# Hardware Issues

---

## Relay Clicks But Valve Does Not Open

Likely causes:

- transformer fault
- valve wiring fault
- failed solenoid
- broken cable

Measure the voltage reaching the valve while the relay is active.

---

## Valve Opens But No Water

Check:

- irrigation isolation valve
- water pressure
- blocked filter
- blocked sprinkler

---

## Sprinkler Coverage Poor

Inspect:

- nozzle blockage
- incorrect nozzle
- low pressure
- damaged head
- rotation adjustment

---

# Wi-Fi Problems

Poor Wi-Fi may prevent OTA updates and Home Assistant communication.

Recommendations:

- Install the controller within good Wi-Fi coverage.
- Avoid mounting inside metal enclosures.
- Consider adding a wireless access point if signal strength is poor.

---

# Frequently Asked Questions

---

### Why doesn't the system water every day?

The irrigation calculation currently runs every second day.

This encourages deeper root growth while reducing unnecessary watering.

---

### Why is watering skipped when rain is forecast?

Forecast rainfall above the configured threshold is assumed to provide sufficient irrigation.

This reduces water consumption.

---

### Why does watering pause between cycles?

The soak period allows water to infiltrate into the soil before additional irrigation is applied.

---

### Why is there a 31-minute limit?

This is a hardware safety feature implemented within the ESPHome firmware.

It prevents a relay remaining energised indefinitely should Home Assistant fail.

---

### Why use virtual soil moisture instead of sensors?

Virtual soil moisture avoids:

- buried electronics,
- batteries,
- calibration,
- sensor drift,
- unreliable consumer moisture sensors.

---

# Useful Home Assistant Tools

Developer Tools

Useful for:

- checking entity states,
- testing services,
- inspecting helpers.

---

Automation Trace

Useful for:

- viewing variables,
- checking conditions,
- following execution paths.

---

ESPHome Logs

Useful for:

- firmware debugging,
- Wi-Fi diagnostics,
- device boot information.

---

# Before Requesting Support

Before investigating a new issue, collect the following information:

- Home Assistant version
- ESPHome version
- Recent configuration changes
- Automation Trace
- ESPHome logs
- Relevant screenshots
- Current helper values

This information significantly reduces the time required to identify the cause of a problem.

---

> [!TIP]
> **Golden Rule**
>
> Change only one thing at a time.
>
> If multiple settings are altered simultaneously it becomes much harder to determine which change caused the problem.

---

# Related Files

| File | Description |
|------|-------------|
| `docs/10-Maintenance.md` | Routine maintenance |
| `docs/06-Automations.md` | Automation reference |
| `docs/03-ESPHome.md` | Firmware documentation |
| `docs/02-Hardware.md` | Wiring reference |

---

# Key Takeaways

- Follow a structured diagnostic process.
- Start with Home Assistant before investigating hardware.
- Use Automation Traces whenever possible.
- Keep firmware and documentation up to date.
- Test hardware methodically.
- Make one change at a time.

---

## Navigation

⬅️ Previous: [10 – Maintenance](10-Maintenance.md)

➡️ Next: [12 – Future Improvements](12-Future-Improvements.md)
