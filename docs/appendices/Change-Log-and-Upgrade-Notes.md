# Appendix N — Change Log & Upgrade Notes

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** Change History and Future Upgrade Reference  
> **Status:** Living document

## Overview

This document provides a central place to record significant changes made to the Home Assistant Smart Sprinkler System.

It is intended to help answer questions such as:

- What changed?
- When did it change?
- Why was the change made?
- Which parts of the system were affected?
- What should be checked after an upgrade?
- What future improvements are being considered?

Keeping a record of changes is particularly useful when troubleshooting. A problem that appears after a modification can often be traced back more easily when the system's configuration history is documented.

> **Recommendation:** Update this document whenever a significant change is made to the ESPHome firmware, Home Assistant automations, helpers, irrigation hardware or irrigation logic.

---

# 1. Current System Baseline

The current documented system consists of:

```text
Home Assistant
      │
      ├── Weather Data
      ├── Helper Entities
      ├── Virtual Soil Moisture Model
      └── Sprinkler Automations
                │
                ▼
           ESPHome API
                │
                ▼
             ESP32
                │
                ▼
        Active-Low Relay Board
                │
                ▼
        Irrigation Zones 1–7
```

The current core automations are:

```text
Sprinklers - Update Virtual Soil Moisture
Sprinklers - Calculate Water Requirement
Sprinklers - Morning Cycle and Soak
```

The current controller is an ESP32 running ESPHome, with nine relay channels configured, of which seven are currently used by the automatic irrigation system.

---

# 2. Change Log

## Change Log Format

Each significant change should record:

- **Date** — When the change was made.
- **Version** — Optional version or milestone number.
- **Component** — The part of the system affected.
- **Change** — What was changed.
- **Reason** — Why the change was made.
- **Testing** — How the change was checked.

---

## Current Documented Baseline

| Date | Component | Change | Notes |
|---|---|---|---|
| Current baseline | ESPHome | ESP32 sprinkler controller configured with 9 relay channels | Zones 1–7 currently used |
| Current baseline | ESPHome | Relay outputs configured as active-low | Uses `inverted: true` |
| Current baseline | ESPHome | Outputs configured to restore off | Uses `restore_mode: ALWAYS_OFF` |
| Current baseline | ESPHome | 31-minute independent safety timeout added to each zone | Additional protection against a zone remaining on |
| Current baseline | Home Assistant | Virtual soil moisture model implemented | Uses temperature-based estimated daily loss |
| Current baseline | Home Assistant | Water requirement calculation implemented | Uses moisture deficit and forecast rainfall |
| Current baseline | Home Assistant | Cycle and soak watering implemented | Watering is divided into multiple passes where required |
| Current baseline | Weather | Met Office temperature sensor used | Supports the virtual soil moisture model |
| Current baseline | Weather | 24-hour rainfall forecast sensor used | Supports rain skip logic |

> **Note:** The table above records the documented configuration baseline rather than attempting to recreate an exact historical timeline.

---

# 3. Adding a New Change

Copy the following section when recording a significant modification.

## Change: [Short Description]

**Date:** `YYYY-MM-DD`

**Component:**

```text
Home Assistant / ESPHome / Hardware / Irrigation / Weather
```

**Changed by:**

```text
Name or description
```

### What Changed?

Describe the configuration, hardware or logic that was changed.

### Why Was It Changed?

Describe the reason for making the change.

### Components Affected

List any affected items, for example:

```text
Automation:
Sprinklers - Morning Cycle and Soak

Helper:
input_number.sprinkler_cycle_runtime

ESPHome:
zone_1
```

### Testing Performed

Record how the change was tested.

- [ ] Configuration validated.
- [ ] Individual component tested.
- [ ] Automation trace reviewed.
- [ ] Full irrigation sequence tested where appropriate.
- [ ] No unexpected behaviour observed.

### Result

```text
Successful / Requires Further Work / Reverted
```

### Additional Notes

Add any useful observations, limitations or follow-up actions.

---

# 4. Upgrade Principles

Before making a significant change, follow this general approach:

```text
Document Current State
        │
        ▼
Create Backup
        │
        ▼
Make One Change
        │
        ▼
Test the Change
        │
        ▼
Review Results
        │
        ▼
Update Documentation
```

Where practical, avoid changing several unrelated parts of the system at the same time.

Making changes incrementally makes troubleshooting considerably easier.

---

# 5. Home Assistant Upgrade Notes

Home Assistant updates may affect:

- Automation YAML syntax.
- Integration behaviour.
- Template syntax.
- Entity availability.
- Helper behaviour.
- ESPHome integration behaviour.

## Before Updating Home Assistant

- [ ] Record the current Home Assistant version.
- [ ] Back up the Home Assistant installation.
- [ ] Review any important configuration changes in the sprinkler system.
- [ ] Ensure the sprinkler system can be manually disabled.
- [ ] Avoid starting an upgrade immediately before an unattended watering cycle.

## After Updating Home Assistant

- [ ] Confirm Home Assistant starts correctly.
- [ ] Confirm the ESPHome device is online.
- [ ] Confirm sprinkler switches are available.
- [ ] Confirm helper entities are available.
- [ ] Confirm weather entities are available.
- [ ] Review the sprinkler automations for errors.
- [ ] Check recent automation traces.
- [ ] Perform a controlled zone test before relying on automatic watering.

---

# 6. ESPHome Firmware Upgrade Notes

The ESP32 controller is an important safety-critical component of the irrigation system because it directly controls the relay outputs.

## Before Updating ESPHome Firmware

- [ ] Save a copy of the working ESPHome YAML configuration.
- [ ] Confirm the current firmware is operating correctly.
- [ ] Review proposed configuration changes.
- [ ] Ensure all sprinkler zones are off before beginning.
- [ ] Keep a known working configuration available for recovery.

## After Updating ESPHome Firmware

- [ ] Confirm the ESP32 reconnects to Wi-Fi.
- [ ] Confirm the device reconnects to Home Assistant.
- [ ] Confirm all sprinkler switches are available.
- [ ] Confirm no sprinkler zone activates unexpectedly.
- [ ] Confirm outputs restore to off after a restart.
- [ ] Test each active zone individually.
- [ ] Confirm the safety timeout remains configured.

---

# 7. Automation Upgrade Notes

The three core automations should be treated as a connected system.

```text
Virtual Soil Moisture
        │
        ▼
Water Requirement
        │
        ▼
Morning Cycle & Soak
```

A change to one automation may affect the behaviour of another.

## Before Changing an Automation

- [ ] Save a copy of the current YAML.
- [ ] Identify which helpers and sensors are used.
- [ ] Consider any downstream automations affected by the change.
- [ ] Make one logical change at a time where practical.

## After Changing an Automation

- [ ] Confirm the YAML validates successfully.
- [ ] Confirm the automation can be enabled.
- [ ] Review a test run or automation trace.
- [ ] Confirm helper values are sensible.
- [ ] Confirm the resulting sprinkler behaviour is as expected.

---

# 8. Helper Entity Changes

Helper entity IDs are referenced directly by automations.

Changing or deleting a helper can therefore cause an automation to fail.

Before modifying a helper:

- [ ] Check which automations reference the entity.
- [ ] Record the existing entity ID.
- [ ] Record the existing value and configuration.
- [ ] Update affected automations if the entity ID changes.
- [ ] Test the complete logic after making the change.

> **Important:** Renaming a helper's display name is not necessarily the same as changing its entity ID. Always check the actual entity ID before editing automation YAML.

---

# 9. Irrigation Hardware Upgrade Notes

Hardware changes may include:

- Replacing a sprinkler head.
- Changing a nozzle.
- Adding a valve or zone.
- Replacing the relay board.
- Replacing the ESP32.
- Changing pipework.
- Modifying water pressure or flow.

Any significant hardware change may affect the irrigation rate or coverage.

After modifying irrigation hardware:

- [ ] Check for leaks.
- [ ] Test affected zones individually.
- [ ] Check spray coverage.
- [ ] Check for runoff.
- [ ] Reassess irrigation calibration if necessary.
- [ ] Update relevant documentation and diagrams.

---

# 10. Future Upgrade Ideas

The following areas may be considered for future development. They are ideas for consideration rather than planned or implemented features.

## Weather and Irrigation Improvements

Possible areas for future investigation:

- More detailed evapotranspiration modelling.
- Improved rainfall forecasting.
- Historical rainfall tracking.
- Seasonal irrigation adjustments.
- Automatic adjustment based on longer-term weather conditions.

## Irrigation Calibration

The current runtime calculations depend on an assumed irrigation rate.

Future calibration could include:

- Measuring actual precipitation rate.
- Comparing watering results against observed lawn condition.
- Adjusting the assumed rate used in calculations.
- Recording seasonal performance.

## Additional Zones

The ESPHome configuration currently defines nine relay channels, while seven are used by the documented automatic watering sequence.

Future expansion could consider:

```text
Zone 8
Zone 9
```

Any additional zones would require updates to:

- ESPHome documentation.
- Home Assistant automation zone lists.
- Hardware diagrams.
- Testing procedures.

## Monitoring and Diagnostics

Potential future additions could include:

- System status dashboard.
- Last watering timestamp.
- Last calculated irrigation requirement.
- Automation failure notifications.
- Watering history.

---

# 11. Upgrade Checklist

Use this checklist for significant upgrades.

## Before the Change

- [ ] Documentation reviewed.
- [ ] Current configuration backed up.
- [ ] Relevant YAML saved.
- [ ] Current system operation confirmed.
- [ ] Sprinklers can be manually disabled.
- [ ] A recovery plan is available.

## During the Change

- [ ] Only the intended components are modified.
- [ ] Entity IDs are checked carefully.
- [ ] Safety settings are retained.
- [ ] Changes are documented.

## After the Change

- [ ] System starts normally.
- [ ] ESPHome device is online.
- [ ] Helper entities are available.
- [ ] Weather sensors are available.
- [ ] Automations are enabled.
- [ ] Individual zones tested.
- [ ] Automation traces reviewed.
- [ ] Documentation updated.

---

# 12. Versioning Suggestions

Formal version numbers are optional, but simple milestones can make the project easier to manage.

One possible format is:

```text
v1.0 — Initial documented working system
v1.1 — Minor configuration improvements
v1.2 — Irrigation calibration update
v2.0 — Significant redesign or architecture change
```

A version number does not need to change for every small adjustment.

Consider recording a new version when a change significantly affects:

- Irrigation behaviour.
- System architecture.
- Hardware.
- Safety behaviour.
- Calculation logic.

---

# 13. Rollback and Recovery

If an upgrade causes unexpected behaviour, the safest approach is generally:

```text
Stop Automatic Watering
        │
        ▼
Identify the Last Change
        │
        ▼
Restore the Previous Working Configuration
        │
        ▼
Test Individual Components
        │
        ▼
Return to Automatic Operation
```

The master control can be used to prevent normal automatic watering:

```text
input_boolean.sprinklers_enabled
```

Where a configuration change is suspected, avoid making further unrelated changes until the system is understood or restored to a known working state.

For detailed recovery procedures, refer to:

[Quick Recovery & Rebuild Guide](Quick-Recovery-and-Rebuild-Guide.md)

---

# 14. Documentation Update Checklist

After making a significant change, consider whether the following documents need updating:

- [ ] Main project README.
- [ ] Hardware specification.
- [ ] GPIO & ESP32 pin reference.
- [ ] ESPHome firmware reference.
- [ ] Automation reference.
- [ ] Helper reference.
- [ ] Template sensor reference.
- [ ] Irrigation calibration.
- [ ] Installation troubleshooting.
- [ ] System maintenance.
- [ ] Testing and commissioning checklist.
- [ ] Glossary & entity reference.
- [ ] Relevant diagrams.

Keeping documentation current means the repository remains useful as a recovery resource rather than simply becoming a historical snapshot.

---

# 15. Planned Work and Ideas

Use this section to record ideas before they become active changes.

| Idea | Reason | Priority | Status |
|---|---|---|---|
| | | Low / Medium / High | Idea |
| | | Low / Medium / High | Researching |
| | | Low / Medium / High | Planned |
| | | Low / Medium / High | Implemented |

> **Tip:** Keeping ideas separate from completed changes makes it easier to distinguish between what the system currently does and what might be added in the future.

---

# 16. Change History Summary

The sprinkler system should be considered a living project.

Its development can be summarised as:

```text
Working System
      │
      ▼
Document Current Configuration
      │
      ▼
Back Up Known Working State
      │
      ▼
Make Controlled Improvements
      │
      ▼
Test Thoroughly
      │
      ▼
Update Documentation
      │
      └───────────────► Repeat as Required
```

The most important principle is to retain a **known working configuration** before making significant changes.

A well-maintained repository, together with backups of the Home Assistant and ESPHome configuration, provides a practical way to recover from mistakes and experiment with improvements safely.

---

## Related Documentation

- [System Architecture Overview](System-Architecture-Overview.md)
- [Automation Reference](Automation-Reference.md)
- [Helper Reference](Helper-Reference.md)
- [Hardware Specification](Hardware-Specification.md)
- [ESPHome Firmware Reference](ESPHome-Firmware-Reference.md)
- [Installation Troubleshooting](Installation-Troubleshooting.md)
- [Quick Recovery & Rebuild Guide](Quick-Recovery-and-Rebuild-Guide.md)
- [Testing & Commissioning Checklist](Testing-and-Commissioning-Checklist.md)
- [Glossary & Entity Reference](Glossary-and-Entity-Reference.md)

---

*Home Assistant Smart Sprinkler System — Appendix N: Change Log & Upgrade Notes*
