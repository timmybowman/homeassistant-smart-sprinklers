# Home Assistant Automations

| | |
|---|---|
| **Project** | Home Assistant Smart Sprinklers |
| **Document** | 06 – Automations |
| **Version** | 1.0 |
| **Platform** | Home Assistant |
| **Status** | Live System |
| **Last Updated** | July 2026 |

---

## Summary

The Home Assistant automation engine is responsible for calculating irrigation requirements and operating the sprinkler system.

Rather than watering on a fixed schedule, the automations estimate soil moisture, evaluate forecast rainfall and determine how much irrigation is required before activating the sprinkler zones.

The system is intentionally divided into several smaller automations rather than one large automation. This makes the logic easier to understand, simpler to troubleshoot and more reliable.

---

# Contents

- Automation Overview
- Daily Timeline
- Automation Pipeline
- Update Virtual Soil Moisture
- Calculate Water Requirement
- Morning Cycle & Soak
- Safety Features
- Automation Relationships
- Troubleshooting
- Key Takeaways

---

# Automation Overview

The project currently contains three production automations.

| Automation | Purpose |
|------------|---------|
| Sprinklers - Update Virtual Soil Moisture | Models daily evaporation |
| Sprinklers - Calculate Water Requirement | Calculates irrigation requirement |
| Sprinklers - Morning Cycle and Soak | Operates the sprinkler zones |

Each automation performs a single task before passing information to the next stage.

---

> [!TIP]
> **Design Note**
>
> Splitting the project into multiple automations keeps each one focused on a single responsibility.
>
> This makes future maintenance considerably easier than maintaining one very large automation.

---

# Daily Timeline

The irrigation controller follows the same sequence every day.

| Time | Operation |
|------|-----------|
| 03:45 | Update virtual soil moisture |
| 04:00 | Calculate irrigation requirement *(every second day)* |
| Sunrise +30 min | Water the lawn if required |

---

# Automation Pipeline

```
Weather Data
      │
      ▼

Update Virtual Soil Moisture

      │

Soil Moisture Helper

      │

Calculate Water Requirement

      │

Water Target

      │

Total Runtime

      │

Morning Cycle & Soak

      │

ESPHome Controller

      │

Sprinkler Zones
```

Every automation produces data that is consumed by the next automation.

---

# Automation 1

## Sprinklers - Update Virtual Soil Moisture

### Purpose

This automation estimates the amount of water lost from the lawn through evaporation.

Rather than using physical soil moisture sensors, a virtual soil moisture model is maintained.

Every morning the automation subtracts an estimated daily loss based on the current temperature.

---

### Trigger

```
03:45 every morning
```

---

### Inputs

- Current virtual soil moisture
- Ambient temperature

---

### Outputs

Updates:

```
input_number.sprinkler_soil_moisture
```

---

### Daily Loss Table

| Temperature | Moisture Loss |
|------------|--------------:|
| Below 10°C | 1 |
| 10–15°C | 3 |
| 15–20°C | 5 |
| 20–25°C | 8 |
| 25–30°C | 12 |
| Above 30°C | 15 |

Higher temperatures therefore produce greater daily moisture loss.

---

### Result

The automation updates the virtual soil moisture helper ready for the next stage.

---

# Automation 2

## Sprinklers - Calculate Water Requirement

### Purpose

Determines whether irrigation should occur.

This automation combines:

- Virtual soil moisture
- Moisture target
- Forecast rainfall
- Recovery mode
- Rain skip threshold

to calculate today's watering requirement.

---

### Trigger

```
04:00 every second day
```

The automation runs on odd-numbered days of the year.

---

### Inputs

| Helper / Sensor | Purpose |
|----------------|---------|
| Soil Moisture | Current virtual moisture |
| Moisture Target | Desired moisture level |
| Rain Forecast | Expected rainfall |
| Rain Skip Threshold | Skip watering limit |
| Recovery Mode | Increased watering if enabled |

---

### Decision Process

```
Rain Forecast

↓

Enough Rain?

├── Yes → Runtime = 0

└── No

↓

Soil Moisture

↓

Already at Target?

├── Yes → Runtime = 0

└── No

↓

Calculate Deficit

↓

Calculate Water Required

↓

Calculate Runtime
```

---

### Runtime Calculation

The automation calculates:

```
Moisture Deficit

↓

Water Needed (mm)

↓

Runtime (minutes)

↓

Store Total Runtime
```

The calculated runtime is limited to a maximum of **180 minutes**.

---

### Outputs

Updates:

- Water Target
- Total Runtime
- Runtime Helper

---

> [!IMPORTANT]
> **Recovery Mode**
>
> When enabled, Recovery Mode increases the refill factor, allowing the lawn to recover more quickly after periods of drought or unusually hot weather.

---

# Automation 3

## Sprinklers - Morning Cycle and Soak

### Purpose

Operates all sprinkler zones using the runtime calculated earlier in the morning.

Rather than watering continuously, the total runtime is divided into multiple passes.

---

### Trigger

```
Sunrise +30 minutes
```

---

### Preconditions

The automation only runs if:

- Sprinklers Enabled is ON
- Total Runtime is greater than zero

---

### Inputs

- Total Runtime
- Cycle Runtime
- Soak Time

---

### Cycle & Soak

Example:

```
Total Runtime

40 minutes

↓

20 minute cycle

↓

20 minute soak

↓

20 minute cycle

↓

Complete
```

The soak period allows water to infiltrate before the next watering pass begins.

---

### Zone Sequence

```
Zone 1

↓

Zone 2

↓

Zone 3

↓

Zone 4

↓

Zone 5

↓

Zone 6

↓

Zone 7
```

Each zone receives exactly the same runtime during each watering pass.

---

### Multi-Pass Operation

```
Pass 1

↓

Soak

↓

Pass 2

↓

Soak

↓

Pass 3

↓

Finished
```

The process repeats until the calculated runtime reaches zero.

---

# Safety Features

The automation layer includes several safeguards.

| Feature | Purpose |
|---------|---------|
| Master Enable | Disable the entire irrigation system |
| Rain Skip Threshold | Prevent unnecessary watering |
| Moisture Target | Prevent overwatering |
| Runtime Limit | Prevent excessive watering |
| Cycle Runtime | Prevent long continuous watering |
| ESPHome Timeout | Hardware protection after 31 minutes |

These software safeguards are complemented by the hardware protections implemented within ESPHome.

---

# Automation Relationships

```
Update Virtual Soil Moisture

↓

Soil Moisture Helper

↓

Calculate Water Requirement

↓

Total Runtime Helper

↓

Morning Cycle & Soak

↓

ESPHome

↓

Relay Outputs

↓

Sprinklers
```

Each automation depends on the successful completion of the previous stage.

---

# Troubleshooting

## Lawn Not Watering

Check:

- Sprinklers Enabled
- Rain Skip Threshold
- Forecast Rain
- Total Runtime
- Automation traces

---

## Runtime Always Zero

Check:

- Soil Moisture Target
- Forecast Rain
- Recovery Mode
- Temperature sensor

---

## Watering Stops Unexpectedly

If watering stops after approximately 31 minutes, this is normal behaviour.

The ESPHome firmware automatically switches every relay off after 31 minutes as a hardware safety feature.

---

## Automation Traces

When diagnosing unexpected behaviour, Home Assistant's **Automation Trace** feature is the recommended starting point.

The traces show:

- Trigger
- Conditions
- Variables
- Branch selection
- Actions executed

This makes it possible to determine why an automation behaved in a particular way.

---

# Key Takeaways

- The irrigation system uses three independent automations.
- Each automation has a single responsibility.
- Virtual soil moisture replaces physical soil moisture sensors.
- Weather forecasts influence watering decisions.
- Cycle & soak irrigation improves water infiltration.
- Hardware safety remains the responsibility of ESPHome.

---

## Related Files

| File | Description |
|------|-------------|
| `homeassistant/automations.yaml` | Live automation configuration |
| `docs/05-Helpers.md` | Helper reference |
| `docs/07-Irrigation-Logic.md` | Irrigation calculation model |

---

## Navigation

⬅️ Previous: [05 – Helper Entities](05-Helpers.md)

➡️ Next: [07 – Irrigation Logic](07-Irrigation-Logic.md)
