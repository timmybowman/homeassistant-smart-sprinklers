# Irrigation Logic

| | |
|---|---|
| **Project** | Home Assistant Smart Sprinklers |
| **Document** | 07 – Irrigation Logic |
| **Version** | 1.0 |
| **Platform** | Home Assistant |
| **Status** | Live System |
| **Last Updated** | July 2026 |

---

## Summary

Unlike traditional irrigation controllers that water on fixed timers, this project estimates the amount of water required by the lawn before deciding whether irrigation is necessary.

The system combines weather data, a virtual soil moisture model and configurable irrigation targets to calculate an appropriate watering duration. This approach reduces unnecessary watering while maintaining healthy grass growth.

The goal is not to perfectly simulate soil physics, but to create a practical, predictable and easily adjustable irrigation model suitable for a domestic lawn.

---

# Contents

- Design Philosophy
- Why Virtual Soil Moisture?
- Daily Irrigation Cycle
- Weather Model
- Soil Moisture Model
- Water Requirement Calculation
- Runtime Calculation
- Cycle & Soak Irrigation
- Watering Schedule
- Worked Example
- Design Assumptions
- Future Improvements
- Key Takeaways

---

# Design Philosophy

The irrigation controller is based on a simple principle:

> Only water the lawn when it is likely to benefit from additional moisture.

Rather than watering for a fixed period every day, the system attempts to estimate:

- how much moisture has been lost,
- how much rainfall is expected,
- how much water should be replaced.

This allows watering duration to change automatically as weather conditions change.

---

> [!TIP]
> **Design Goal**
>
> The objective is not to create a scientific soil simulation. The aim is to create a reliable and understandable model that can be adjusted easily as experience with the system grows.

---

# Why Virtual Soil Moisture?

Many smart irrigation systems rely on physical soil moisture sensors.

This project intentionally avoids them.

Reasons include:

- Consumer sensors often become unreliable over time.
- Installation requires burying hardware.
- Different soil types produce inconsistent readings.
- Batteries require maintenance.
- Multiple sensors may be needed for large lawns.

Instead, the system maintains a **virtual soil moisture value** using weather information and irrigation history.

---

# Daily Irrigation Cycle

Every day follows the same sequence.

```text
03:45

↓

Estimate Daily Moisture Loss

↓

04:00

↓

Calculate Water Requirement

↓

Check Forecast Rain

↓

Calculate Runtime

↓

Sunrise +30 minutes

↓

Water Lawn (if required)
```

Each stage prepares data for the next stage.

---

# Weather Model

The irrigation logic currently uses information supplied by the Met Office integration.

The following data influences watering decisions:

| Weather Data | Purpose |
|--------------|---------|
| Ambient Temperature | Estimate evaporation |
| Forecast Rainfall | Skip or reduce watering |

At present, wind speed, humidity and solar radiation are not included in the calculations.

This keeps the system straightforward while still responding to the most significant weather influences.

---

# Soil Moisture Model

The project maintains a helper named:

```
input_number.sprinkler_soil_moisture
```

This value represents the estimated amount of moisture remaining within the root zone.

Each morning the value is reduced according to the estimated evaporation loss.

Current temperature bands are:

| Temperature | Moisture Reduction |
|--------------|------------------:|
| Below 10°C | 1 |
| 10–15°C | 3 |
| 15–20°C | 5 |
| 20–25°C | 8 |
| 25–30°C | 12 |
| Above 30°C | 15 |

These values are intentionally conservative and are expected to evolve as real-world observations are collected.

---

> [!NOTE]
> **Implementation Note**
>
> These values are not intended to represent millimetres of evaporation. They are relative units that provide a simple method of modelling changing soil moisture over time.

---

# Water Requirement Calculation

After estimating the current soil moisture, the system determines whether irrigation is required.

The calculation considers:

- Current soil moisture
- Target soil moisture
- Forecast rainfall
- Recovery mode
- Rain skip threshold

The decision process is:

```text
Forecast Rain

↓

Enough Rain?

├── Yes → Do Not Water

└── No

↓

Soil Moisture

↓

Already at Target?

├── Yes → Do Not Water

└── No

↓

Calculate Moisture Deficit

↓

Determine Water Requirement
```

Only when both conditions fail does irrigation proceed.

---

# Recovery Mode

Recovery Mode is intended for periods of prolonged heat or drought.

When enabled:

- the calculated deficit is refilled more aggressively,
- the lawn receives additional irrigation,
- normal operation can be restored by disabling the helper.

This avoids permanently increasing irrigation for the entire season.

---

# Runtime Calculation

Once the water requirement has been calculated, it is converted into a sprinkler runtime.

The project currently assumes an average precipitation rate of:

```
12 mm/hour
```

This figure is based on the installed Rain Bird 5004 sprinkler layout and provides the relationship between required water depth and watering duration.

The result is stored in:

```
input_number.sprinkler_total_runtime
```

This represents the runtime required **for each sprinkler zone**.

---

# Why Cycle & Soak?

Applying all of the required water in one continuous watering cycle is not always efficient.

Long watering periods may result in:

- surface runoff,
- water pooling,
- reduced soil infiltration.

Instead, the irrigation controller divides watering into several shorter passes separated by soak periods.

Example:

```text
40 minutes required

↓

20 minute watering pass

↓

45 minute soak

↓

20 minute watering pass

↓

Finished
```

The soak period allows water to move deeper into the soil before the next watering cycle begins.

---

> [!TIP]
> **Design Note**
>
> The soak period is configurable using `input_number.sprinkler_soak_time`, allowing the behaviour to be adjusted for different soil types.

---

# Why Water After Sunrise?

The project waters approximately 30 minutes after sunrise.

This timing was chosen to balance several factors.

### Advantages

- Reduced fungal risk compared with overnight watering.
- Cooler temperatures than midday.
- Lower evaporation than afternoon watering.
- Easier visual inspection of the system.

Although many irrigation systems water during the night, this project intentionally avoids extended overnight leaf wetness where possible.

---

# Why Water Every Two Days?

Rather than watering every day, the calculation automation runs every second day.

Benefits include:

- Encourages deeper root growth.
- Reduces unnecessary irrigation.
- Better matches the storage capacity of most lawn soils.
- Allows forecast rainfall to contribute naturally before irrigation occurs.

The interval can be adjusted in future if required.

---

# Worked Example

The following example demonstrates a typical watering calculation.

| Parameter | Value |
|-----------|------:|
| Soil Moisture | 60 |
| Moisture Target | 75 |
| Moisture Deficit | 15 |
| Recovery Mode | Off |
| Forecast Rain | 1 mm |
| Rain Skip Threshold | 5 mm |

Result:

```text
Moisture Deficit

↓

Water Requirement

↓

Total Runtime

↓

Morning Cycle & Soak

↓

Sprinklers Operate
```

The exact runtime depends on the configured refill factor and the assumed sprinkler precipitation rate.

---

# Design Assumptions

The current irrigation model is based on several assumptions.

- The lawn receives reasonably uniform sprinkler coverage.
- The sprinkler precipitation rate averages approximately 12 mm/hour.
- Temperature is an acceptable indicator of relative evaporation.
- Forecast rainfall is sufficiently accurate for short-term irrigation decisions.
- Soil conditions remain broadly consistent across the irrigated area.

These assumptions can be refined over time as more operational experience is gained.

---

# Future Improvements

Possible future enhancements include:

- Seasonal adjustment factors.
- Wind speed compensation.
- Relative humidity compensation.
- Solar radiation modelling.
- Actual evapotranspiration (ET₀) calculations.
- Flow meter integration.
- Rain sensor integration.
- Automatic leak detection.
- Historical irrigation statistics.
- Water consumption reporting.

The current design intentionally favours simplicity over complexity while providing a solid foundation for future development.

---

# Key Takeaways

- The system uses a virtual soil moisture model rather than physical sensors.
- Weather data is used to estimate changing soil conditions.
- Irrigation is calculated rather than scheduled using fixed timers.
- Cycle & soak watering improves water infiltration.
- Every irrigation decision is based on configurable helper values.
- The logic is designed to be practical, understandable and easy to refine.

---

## Related Files

| File | Description |
|------|-------------|
| `docs/06-Automations.md` | Automation reference |
| `docs/05-Helpers.md` | Helper reference |
| `homeassistant/automations.yaml` | Live automation configuration |

---

## Navigation

⬅️ Previous: [06 – Automations](06-Automations.md)

➡️ Next: [08 – Weather Integration](08-Weather-Integration.md)
