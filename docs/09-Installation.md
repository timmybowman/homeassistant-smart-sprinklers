# Installation & Setup

| | |
|---|---|
| **Project** | Home Assistant Smart Sprinklers |
| **Document** | 09 – Installation & Setup |
| **Version** | 1.0 |
| **Platform** | Home Assistant & ESPHome |
| **Status** | Live System |
| **Last Updated** | July 2026 |

---

## Summary

This guide explains how to build, install and configure the Home Assistant Smart Sprinkler Controller from scratch.

The project is designed around an ESP32 running ESPHome, connected to a 9-channel active-low relay board. Home Assistant performs all irrigation calculations, while ESPHome safely operates the sprinkler valves.

By following this guide, a new system can be recreated using the configuration files included in this repository.

---

# Contents

- Prerequisites
- Hardware Installation
- ESPHome Installation
- Home Assistant Configuration
- Weather Integration
- Uploading the Firmware
- Testing
- Initial Configuration
- Commissioning
- Related Files

---

# Prerequisites

Before beginning, ensure the following are available.

## Hardware

- ESP32 Development Board (ESP32-WROOM-32 DevKit)
- 9-Channel UCTronics Active-Low Relay Board
- 24VAC Irrigation Solenoids
- 24VAC Irrigation Transformer
- Suitable waterproof enclosure
- Wiring and terminals
- Reliable Wi-Fi coverage

---

## Software

- Home Assistant
- ESPHome Add-on
- File Editor or Studio Code Server
- Met Office Integration
- Git (optional)
- GitHub account (optional)

---

# Hardware Installation

## Install the ESP32

Mount the ESP32 securely inside the enclosure.

Ensure that:

- USB access remains available.
- Wi-Fi reception is adequate.
- The board is protected from moisture.

---

## Install the Relay Board

Connect the ESP32 GPIO outputs to the relay inputs as documented in:

```
docs/02-Hardware.md
```

The relay board used in this project is active-low.

ESPHome handles this automatically using:

```yaml
inverted: true
```

No changes are required within Home Assistant.

---

## Connect the Irrigation Valves

Each sprinkler valve should be connected to its assigned relay output.

Current configuration:

| Zone | Relay |
|------|------|
| Zone 1 | Relay 1 |
| Zone 2 | Relay 2 |
| Zone 3 | Relay 3 |
| Zone 4 | Relay 4 |
| Zone 5 | Relay 5 |
| Zone 6 | Relay 6 |
| Zone 7 | Relay 7 |

Relays 8 and 9 remain available for future expansion.

---

> [!IMPORTANT]
> Always disconnect mains power before working on the relay board or transformer wiring.

---

# Install ESPHome

Install the ESPHome add-on using the Home Assistant Add-on Store.

Create a new ESPHome device.

Replace the generated YAML with:

```
esphome/sprinklers.yaml
```

Update:

- Wi-Fi SSID
- Wi-Fi Password
- Secrets file (if used)

Compile and upload the firmware.

---

# Upload Firmware

The first upload should normally be completed using a USB connection.

Once connected:

1. Compile the firmware.
2. Flash the ESP32.
3. Verify successful boot.
4. Confirm Wi-Fi connectivity.
5. Adopt the device in Home Assistant.

Future updates can normally be performed over-the-air (OTA).

---

# Install Home Assistant Configuration

Copy the supplied configuration files into Home Assistant.

Repository location:

```
homeassistant/
```

Files include:

- automations.yaml
- helpers.yaml
- template_sensors.yaml

Restart Home Assistant after importing the configuration.

---

# Create Helper Entities

If helper entities are not imported automatically, create them manually using the values documented in:

```
docs/05-Helpers.md
```

Verify that each helper:

- exists,
- has the correct entity ID,
- uses the expected range,
- has the correct default value.

---

# Configure the Weather Integration

Install the Met Office integration.

Confirm that the required weather entities are available.

The project currently expects:

```
sensor.met_office_ashby_de_la_zouch_temperature
```

and

```
sensor.sprinkler_forecast_rain_24h
```

If different entity names are used, update the automation YAML accordingly.

---

# Enable the Automations

After importing the configuration, enable:

- Sprinklers - Update Virtual Soil Moisture
- Sprinklers - Calculate Water Requirement
- Sprinklers - Morning Cycle and Soak

Verify that all three automations are enabled.

---

# Initial Configuration

Recommended starting values:

| Helper | Suggested Value |
|---------|----------------:|
| Moisture Target | 75 |
| Rain Skip Threshold | 5 mm |
| Cycle Runtime | 20 minutes |
| Soak Time | 45 minutes |
| Recovery Mode | Off |

These values provide a conservative starting point and can be adjusted after observing the system over several weeks.

---

# Commissioning

Before relying on automatic irrigation, perform the following checks.

## Relay Test

Turn each sprinkler zone on manually.

Confirm:

- Correct relay activates.
- Correct valve opens.
- Correct sprinkler operates.
- Relay switches off correctly.

---

## ESPHome Safety Test

Run a sprinkler zone continuously.

Confirm that the relay switches off automatically after approximately 31 minutes.

This confirms the hardware safety timeout is functioning correctly.

---

## Weather Test

Verify:

- Temperature updates correctly.
- Rain forecast sensor produces valid values.
- Automations complete successfully.

---

## Watering Test

Temporarily reduce the calculated runtime.

Run the Morning Cycle & Soak automation manually.

Confirm:

- All seven zones operate.
- Runtime is correct.
- Soak delay occurs.
- Automation completes normally.

---

> [!TIP]
> Perform the first full irrigation cycle while observing the system. This allows valve sequencing, sprinkler coverage and soak timing to be verified before unattended operation.

---

# Recommended Adjustments

After several weeks of operation, review:

- Moisture Target
- Rain Skip Threshold
- Cycle Runtime
- Soak Time

Small adjustments are usually sufficient to fine-tune watering for local soil conditions.

---

# Updating the System

Future updates generally involve one of three tasks:

### ESPHome Firmware

Replace:

```
esphome/sprinklers.yaml
```

Compile and upload OTA.

---

### Home Assistant Automations

Replace:

```
homeassistant/automations.yaml
```

Reload automations.

---

### Documentation

Update the relevant document within:

```
docs/
```

Documentation version numbers should be incremented whenever significant changes are made.

---

# Related Files

| File | Description |
|------|-------------|
| `docs/02-Hardware.md` | Hardware wiring |
| `docs/03-ESPHome.md` | ESPHome firmware |
| `docs/05-Helpers.md` | Helper configuration |
| `docs/06-Automations.md` | Automation documentation |
| `esphome/sprinklers.yaml` | ESPHome firmware |
| `homeassistant/automations.yaml` | Home Assistant automations |

---

# Key Takeaways

- Install the hardware before configuring Home Assistant.
- Upload the ESPHome firmware before enabling automations.
- Verify weather sensors before relying on automatic watering.
- Test each sprinkler zone individually.
- Confirm the ESPHome safety timeout is operating correctly.
- Observe the first automatic watering cycle before leaving the system unattended.

---

## Navigation

⬅️ Previous: [08 – Weather Integration](08-Weather-Integration.md)

➡️ Next: [10 – Maintenance](10-Maintenance.md)
