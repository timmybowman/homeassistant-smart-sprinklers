Home Assistant Smart Sprinklers
Version: 1.0
Document: 01 – Introduction
Last Updated: July 2026

# Introduction

> **Project Name:** Home Assistant Smart Sprinklers
>
> **Version:** 1.0
>
> **Status:** Production
>
> **Controller:** AZ-Delivery ESP32 DevKitC V4
>
> **Firmware:** ESPHome
>
> **Automation Platform:** Home Assistant

---

# Overview

This project provides an adaptive irrigation controller built around Home Assistant and ESPHome. Rather than watering the garden on a fixed timer, the system estimates how much moisture has been lost from the soil, predicts incoming rainfall and calculates how much irrigation is required before any watering begins.

The primary goal is to apply only the amount of water the lawn actually requires while avoiding unnecessary watering, reducing waste and encouraging healthier root growth.

Although designed for a domestic garden, the principles used throughout the project are equally applicable to larger irrigation systems.

---

# Design Objectives

The project was designed around a number of key objectives.

## Reliability

The system should continue operating safely without requiring frequent intervention.

Multiple layers of protection have therefore been included, including:

- Automatic relay timeout inside ESPHome
- Home Assistant master enable switch
- Rain forecast checking
- Virtual soil moisture model
- Runtime limits
- Automatic relay reset after reboot

---

## Simplicity

Although the internal calculations are relatively sophisticated, day-to-day operation should remain simple.

The entire irrigation system can normally be controlled using only a handful of Home Assistant helpers.

Most adjustments can therefore be made without modifying YAML.

---

## Adaptability

Every garden behaves differently.

Rather than hard-coding watering durations, this project allows the following values to be adjusted at any time:

- Soil moisture target
- Rain skip threshold
- Cycle runtime
- Soak duration
- Recovery mode
- Water application target

This makes seasonal adjustment straightforward without changing any automation logic.

---

## Safety

Water valves should never be allowed to remain energised indefinitely.

For this reason every relay output contains a firmware-based safety timeout.

Whenever a sprinkler zone is switched on:

- The onboard LED flashes twice.
- The relay activates.
- A 31-minute timer begins.
- The relay automatically switches off.
- The LED flashes twice again.

This protection exists independently of Home Assistant and remains active even if an automation fails unexpectedly.

---

# Why Home Assistant?

Home Assistant provides an ideal platform for irrigation control because it combines automation, weather data, helper entities and scheduling within a single ecosystem.

Advantages include:

- Local operation
- Excellent automation engine
- Native ESPHome integration
- Flexible dashboards
- Extensive weather integrations
- Easy expansion
- Excellent backup facilities

Using Home Assistant also allows the irrigation system to integrate naturally with the rest of the smart home.

---

# Why ESPHome?

ESPHome was selected because it provides a robust and reliable interface between Home Assistant and the relay hardware.

Benefits include:

- Native Home Assistant API
- OTA firmware updates
- Simple YAML configuration
- Reliable GPIO control
- Built-in logging
- Hardware automation
- Firmware-level safety

Keeping the relay control inside ESPHome means that hardware safety features continue operating even if Home Assistant becomes unavailable.

---

# Why Virtual Soil Moisture?

Many commercial irrigation systems rely on inexpensive soil moisture probes.

Whilst attractive in principle, these sensors often suffer from several disadvantages:

- Poor long-term stability
- Corrosion
- Installation difficulties
- Inconsistent readings
- Localised measurements
- Cable failures

Instead, this project estimates the amount of moisture remaining in the soil using environmental data.

Each day the model is updated according to estimated evaporation.

Rainfall and irrigation then replenish the virtual soil moisture level.

Although simplified compared with professional evapotranspiration models, this approach has proven sufficiently accurate for domestic lawn irrigation while avoiding the maintenance associated with buried sensors.

---

# Irrigation Philosophy

Traditional sprinkler timers generally answer the question:

> "How long should the sprinklers run today?"

This project instead asks:

> **"How much water does the lawn actually need today?"**

Only after calculating the required amount of water is a runtime determined.

This distinction is fundamental to the design philosophy of the system.

---

# Cycle & Soak Watering

Applying a large amount of water in one continuous run often exceeds the infiltration rate of the soil.

The result is:

- Surface runoff
- Water waste
- Reduced root depth
- Uneven watering

Instead, watering is divided into multiple shorter cycles separated by soak periods.

This allows water to penetrate deeper into the soil before the next watering pass begins.

The cycle duration and soak period are both configurable through Home Assistant helpers.

---

# Weather Awareness

Local weather data forms an important part of the irrigation calculations.

The system currently uses the Met Office integration to obtain:

- Ambient temperature
- Forecast rainfall

These values influence:

- Daily moisture depletion
- Water requirement calculations
- Rain skip decisions

As a result, irrigation adapts naturally to changing weather conditions.

---

# Current Hardware

The current installation consists of:

| Component | Description |
|-----------|-------------|
| Controller | AZ-Delivery ESP32 DevKitC V4 |
| Firmware | ESPHome |
| Relay Board | UCTronics 9-Channel Active-Low Relay Board |
| Sprinkler Heads | Rain Bird 5004 Rotors |
| Active Zones | 7 |
| Spare Outputs | 2 |
| Weather Source | Met Office Integration |
| Automation Platform | Home Assistant |

---

# Daily Operating Sequence

Every day the system follows the same sequence:

1. Update the virtual soil moisture model.
2. Calculate the irrigation requirement.
3. Check forecast rainfall.
4. Skip watering if appropriate.
5. Calculate the required runtime.
6. Wait until Sunrise +30 minutes.
7. Execute cycle & soak watering.
8. Automatically stop all sprinkler zones.

This process requires no manual intervention under normal operating conditions.

---

# Intended Audience

This repository has been written for three different audiences.

## Home Assistant Users

Anyone wishing to build a similar irrigation controller.

---

## Future Maintenance

A complete record of the hardware, software and configuration is maintained so the system can be rebuilt if required.

---

## Future Development

The documentation has been structured so additional features can be incorporated without rewriting the existing documentation.

Examples include:

- Rain sensors
- Flow meters
- Leak detection
- Additional irrigation zones
- Water usage monitoring
- Weather station integration

---

# Repository Structure

```
docs/
    Introduction
    Hardware
    ESPHome
    Home Assistant
    Helpers
    Automations
    Irrigation Logic
    Weather Integration
    Installation
    Maintenance
    Troubleshooting
    Future Improvements
    Appendix

esphome/
    sprinklers.yaml

homeassistant/
    automations.yaml

images/
    SVG diagrams
```

---

# Summary

This project demonstrates how Home Assistant and ESPHome can be combined to create an intelligent irrigation controller that adapts watering to environmental conditions rather than relying on fixed schedules.

By combining weather forecasting, virtual soil moisture modelling and configurable watering cycles, the system aims to provide efficient irrigation while remaining simple to maintain and easy to expand.

The following documents describe every aspect of the system in sufficient detail for it to be rebuilt from scratch if required.

---

**Next Document:** `02-Hardware.md`
← Previous Document              Next Document →
