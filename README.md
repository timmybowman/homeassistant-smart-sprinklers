<div align="center">

# 🌱 Home Assistant Smart Sprinklers

### Adaptive garden irrigation using Home Assistant, ESPHome and virtual soil moisture modelling

[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Automation-blue?logo=homeassistant)]()
[![ESPHome](https://img.shields.io/badge/ESPHome-ESP32-green)]()
[![Platform](https://img.shields.io/badge/Platform-ESP32-lightgrey)]()
[![Weather](https://img.shields.io/badge/Weather-Met%20Office-orange)]()
[![Status](https://img.shields.io/badge/Status-Production-success)]()

---

*A fully automated irrigation system that calculates how much water the lawn requires using a virtual soil moisture model, local weather data and intelligent cycle & soak watering.*

</div>

---

# 📖 Contents

- Overview
- Features
- How It Works
- System Architecture
- Hardware
- Software
- Irrigation Logic
- Documentation
- Repository Structure
- Future Improvements
- Version History

---

# Overview

This project was developed to provide **adaptive irrigation** for a domestic lawn using Home Assistant and ESPHome.

Unlike a traditional sprinkler timer that waters for the same amount every day regardless of the weather, this system estimates the moisture remaining in the soil, predicts rainfall, and calculates exactly how much irrigation is required before watering begins.

The aim is to:

- reduce unnecessary watering
- encourage deeper root growth
- prevent runoff
- avoid watering before significant rainfall
- minimise manual adjustment throughout the year

The complete system is designed around readily available hardware and is intended to be easy to maintain and expand.

---

# Features

## 🌦 Weather Aware

Uses local weather information to:

- estimate daily evaporation
- monitor forecast rainfall
- skip watering when sufficient rain is expected

---

## 🌱 Virtual Soil Moisture

Rather than using unreliable soil moisture probes, the system maintains a **virtual soil moisture model**.

Every morning the model is updated using estimated evaporation based on ambient temperature.

Rainfall and irrigation replenish the model until the desired moisture target is reached.

---

## 💧 Cycle & Soak Watering

Rather than watering each zone continuously, irrigation is split into multiple passes.

Benefits include:

- improved infiltration
- reduced runoff
- reduced water waste
- healthier root systems

---

## 🛡 Built-in Safety

Safety is implemented at several levels.

### Home Assistant

- Master Enable switch
- Rain skip threshold
- Moisture target
- Runtime calculation

### ESPHome

Every sprinkler output includes:

- Active-Low relay control
- Restore Mode = ALWAYS_OFF
- Automatic 31 minute timeout
- OTA firmware updates

This ensures that a stuck automation cannot leave a valve energised indefinitely.

---

## ⚙ Fully Configurable

Nearly every aspect of the system can be adjusted without editing YAML.

Examples include:

- moisture target
- rain skip threshold
- cycle runtime
- soak time
- recovery mode

---

# How It Works

Every morning the following sequence occurs:

```text
03:45
│
├── Update Virtual Soil Moisture
│
04:00
│
├── Calculate Water Requirement
│
├── Forecast Rain?
│        │
│        ├── Yes → Skip watering
│        │
│        └── No
│
├── Soil Moisture At Target?
│        │
│        ├── Yes → Skip watering
│        │
│        └── No
│
├── Calculate Required Runtime
│
Sunrise +30 minutes
│
└── Execute Cycle & Soak Watering
```

---

# Hardware

Current hardware consists of:

| Component | Description |
|------------|------------|
| Controller | AZ-Delivery ESP32 DevKitC V4 |
| Firmware | ESPHome |
| Relay Board | UCTronics 9 Channel Active-Low Relay Board |
| Sprinklers | Rain Bird 5004 |
| Zones | 7 |
| Communication | Wi-Fi (Home Assistant API) |

---

# Software

## Home Assistant

Responsible for:

- virtual soil moisture
- runtime calculation
- weather integration
- watering schedule
- helper management

---

## ESPHome

Responsible for:

- relay control
- failsafe timeout
- OTA updates
- status LED

---

# Repository Structure

```
homeassistant-smart-sprinklers

docs/
    Hardware.md
    ESP32.md
    Helpers.md
    Irrigation-Logic.md
    Installation.md
    Troubleshooting.md

esphome/
    sprinklers.yaml

homeassistant/
    automations.yaml

images/
    *.svg
```

---

# Documentation

Detailed documentation is provided for:

- Hardware
- Wiring
- ESPHome firmware
- Home Assistant automations
- Helper entities
- Irrigation algorithms
- Troubleshooting
- Maintenance

---

# Design Philosophy

This project deliberately avoids fixed-duration watering.

Instead it attempts to answer a simple question:

> **"How much water does the lawn actually need today?"**

The answer is calculated using:

- estimated evaporation
- rainfall forecast
- target soil moisture
- recovery mode
- precipitation rate
- configurable runtime limits

The result is an irrigation schedule that adapts naturally to seasonal conditions rather than relying on constant manual adjustment.

---

# Future Improvements

Potential future additions include:

- Flow meter monitoring
- Leak detection
- Rain sensor
- Automatic winter shutdown
- Water usage statistics
- Dashboard visualisations
- Multiple irrigation programmes
- Additional sprinkler zones

---

# Version History

## Version 1.0

Initial documented release including:

- ESPHome controller
- Home Assistant automations
- Virtual soil moisture model
- Cycle & soak watering
- Weather-aware irrigation
- Complete rebuild documentation

---

# Licence

This repository is intended as a reference implementation for Home Assistant based irrigation systems.

Adapt it, improve it and make it your own.

---

<div align="center">

**Built using Home Assistant ❤️ ESPHome ❤️ and a lot of experimentation.**

</div>
