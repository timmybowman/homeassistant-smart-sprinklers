# Maintenance

| | |
|---|---|
| **Project** | Home Assistant Smart Sprinklers |
| **Document** | 10 – Maintenance |
| **Version** | 1.0 |
| **Platform** | Home Assistant & ESPHome |
| **Status** | Live System |
| **Last Updated** | July 2026 |

---

## Summary

This document describes the routine maintenance required to keep the Home Assistant Smart Sprinkler system operating reliably.

Unlike traditional irrigation controllers, this project consists of both hardware and software components. Regular maintenance includes checking the sprinkler hardware, reviewing automation behaviour, updating firmware and adjusting watering parameters as the seasons change.

Most maintenance tasks can be completed without modifying the ESPHome firmware or Home Assistant automations.

---

# Contents

- Maintenance Philosophy
- Routine Checks
- Seasonal Adjustments
- Software Updates
- ESPHome Firmware
- Home Assistant Updates
- Backup Strategy
- Hardware Inspection
- System Testing
- Winter Shutdown
- Spring Start-up
- Maintenance Log
- Key Takeaways

---

# Maintenance Philosophy

The system has been designed to require very little routine attention.

Under normal operation the only regular adjustments are:

- Seasonal watering changes
- Firmware updates
- Home Assistant updates
- Occasional hardware inspection

The automations themselves should require little or no modification once they have been tuned for the local environment.

---

> [!TIP]
> **Best Practice**
>
> Make small changes and allow the system to run for several days before making further adjustments. Grass health changes gradually, and over-adjusting settings can make troubleshooting more difficult.

---

# Routine Checks

The following checks are recommended throughout the watering season.

| Frequency | Check |
|-----------|-------|
| Weekly | Confirm all automations are enabled |
| Weekly | Verify weather data is updating |
| Weekly | Inspect sprinkler coverage |
| Monthly | Check helper values |
| Monthly | Review automation traces |
| Monthly | Inspect wiring and enclosure |
| After heavy rain | Confirm watering was skipped if appropriate |

---

# Seasonal Adjustments

As weather conditions change throughout the year, a small number of helper values may require adjustment.

Typical settings include:

| Helper | Reason |
|--------|--------|
| Moisture Target | Adjust for lawn condition |
| Rain Skip Threshold | Increase or decrease sensitivity to forecast rain |
| Cycle Runtime | Match soil infiltration rate |
| Soak Time | Adapt to changing soil conditions |

These adjustments should be made gradually and observed over several watering cycles.

---

# Software Updates

## ESPHome

When updating ESPHome:

1. Review the release notes.
2. Back up the existing firmware YAML.
3. Compile the updated configuration.
4. Upload the firmware over OTA if available.
5. Confirm all relay outputs function correctly.

---

## Home Assistant

When updating Home Assistant:

- Back up the installation before upgrading.
- Review any breaking changes affecting automations or ESPHome.
- Confirm helper entities remain available.
- Verify automation traces after the first scheduled run.

---

> [!IMPORTANT]
> Avoid performing major Home Assistant and ESPHome upgrades at the same time. Updating one component at a time makes it much easier to identify the cause of any unexpected behaviour.

---

# ESPHome Firmware Maintenance

The firmware is intentionally simple and should rarely require changes.

Firmware updates are typically only required when:

- Adding new sprinkler zones.
- Changing GPIO assignments.
- Updating ESPHome.
- Introducing new hardware.

After every firmware update:

- Test each relay output.
- Confirm the onboard LED flashes correctly.
- Verify the 31-minute safety timeout still operates.

---

# Home Assistant Configuration

Routine Home Assistant maintenance may include:

- Adjusting helper values.
- Reviewing automation traces.
- Updating weather entities if required.
- Improving irrigation calculations.
- Adding new dashboard controls.

Any changes to entity IDs should also be reflected in the documentation.

---

# Backup Strategy

Maintaining regular backups is essential.

The following should be backed up whenever significant changes are made.

| Item | Location |
|------|----------|
| ESPHome Firmware | `esphome/sprinklers.yaml` |
| Automations | `homeassistant/automations.yaml` |
| Helper Definitions | `homeassistant/helpers.yaml` *(if used)* |
| Template Sensors | `homeassistant/template_sensors.yaml` *(if used)* |
| Documentation | `docs/` |
| Git Repository | GitHub |

---

## Version Control

This project is maintained using Git.

Recommended workflow:

1. Make a change.
2. Test the system.
3. Commit the change.
4. Push to GitHub.
5. Update the documentation if required.

Keeping code and documentation synchronised makes future maintenance much easier.

---

# Hardware Inspection

Regularly inspect the physical installation.

Check:

- Enclosure seals.
- Cable glands.
- Relay terminals.
- Transformer connections.
- Signs of corrosion.
- Insect ingress.
- Water ingress.

Outdoor electrical equipment should always remain clean and dry.

---

# Sprinkler Inspection

At least once per month inspect the sprinkler system itself.

Confirm:

- Heads rise fully.
- Nozzles are unobstructed.
- Spray patterns are even.
- Rotation is correct.
- Coverage remains consistent.
- No leaks are present.

Damaged or blocked sprinklers should be repaired before adjusting irrigation settings.

---

# System Testing

A full functional test should be performed periodically.

## Relay Test

Activate each sprinkler zone manually.

Verify:

- Correct relay operates.
- Correct valve opens.
- Correct sprinkler head activates.

---

## Automation Test

Run each automation manually from Home Assistant.

Confirm:

- No errors are reported.
- Variables calculate correctly.
- Expected helper values are updated.

---

## Safety Test

Confirm that:

- Turning off **Sprinklers Enabled** prevents watering.
- Rain skip logic still functions.
- ESPHome safety timeout remains operational.

---

# Winter Shutdown

Where irrigation is not required during winter:

- Turn off **Sprinklers Enabled**.
- Isolate the irrigation water supply.
- Drain pipework where necessary.
- Shut down the irrigation transformer if appropriate.
- Inspect and clean the enclosure.

Winter maintenance helps reduce the likelihood of frost damage and prolongs the life of the system.

---

# Spring Start-up

Before the first watering cycle of the season:

- Restore the water supply.
- Inspect all valves.
- Check for leaks.
- Verify Wi-Fi connectivity.
- Confirm weather data is updating.
- Test all sprinkler zones.
- Enable **Sprinklers Enabled**.

Run a complete manual watering cycle before relying on automatic operation.

---

# Maintenance Log

Maintaining a simple log can help identify long-term trends.

Suggested entries include:

| Date | Activity | Notes |
|------|----------|------|
| YYYY-MM-DD | Firmware Updated | ESPHome 2026.x |
| YYYY-MM-DD | Sprinkler Adjusted | Replaced nozzle |
| YYYY-MM-DD | Moisture Target Changed | Increased to 78 |
| YYYY-MM-DD | Weather Integration Updated | Entity names unchanged |

---

> [!TIP]
> Recording configuration changes makes it much easier to understand why the system behaves differently several months later.

---

# Key Takeaways

- Inspect the system regularly rather than waiting for faults to appear.
- Adjust helper values gradually.
- Keep firmware and Home Assistant up to date.
- Test the hardware after every software update.
- Back up configuration files before making significant changes.
- Use Git to track both code and documentation changes.

---

## Related Files

| File | Description |
|------|-------------|
| `docs/09-Installation.md` | Installation guide |
| `docs/11-Troubleshooting.md` | Fault diagnosis |
| `docs/06-Automations.md` | Automation reference |
| `esphome/sprinklers.yaml` | ESPHome firmware |
| `homeassistant/automations.yaml` | Automation configuration |

---

## Navigation

⬅️ Previous: [09 – Installation & Setup](09-Installation.md)

➡️ Next: [11 – Troubleshooting](11-Troubleshooting.md)
