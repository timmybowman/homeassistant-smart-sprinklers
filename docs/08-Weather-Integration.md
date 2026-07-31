# Weather Integration

| | |
|---|---|
| **Project** | Home Assistant Smart Sprinklers |
| **Document** | 08 – Weather Integration |
| **Version** | 1.0 |
| **Platform** | Home Assistant |
| **Status** | Live System |
| **Last Updated** | July 2026 |

---

## Summary

The irrigation controller uses weather data to make informed watering decisions rather than relying on fixed schedules.

Current weather conditions and short-term forecasts are obtained through the **Met Office Integration** for Home Assistant. These values are used to estimate moisture loss, determine whether rainfall is expected, and calculate how much irrigation is required.

The weather integration is intentionally separated from the irrigation logic. This allows the weather provider to be changed in the future with minimal changes to the automation logic.

---

# Contents

- Design Philosophy
- Weather Provider
- Weather Entities
- Forecast Rainfall
- Temperature Modelling
- Daily Weather Workflow
- Weather Decision Tree
- Failure Handling
- Future Improvements
- Key Takeaways

---

# Design Philosophy

Rather than watering according to a fixed timetable, the system attempts to answer three simple questions every morning:

1. **How much moisture has probably been lost?**
2. **Is nature likely to replace that moisture today?**
3. **If not, how much water should be applied?**

Weather information provides the data required to answer these questions.

---

> [!TIP]
> **Design Goal**
>
> Weather data should influence irrigation decisions without making the system overly complex or difficult to maintain.

---

# Weather Provider

The project currently uses the **Met Office** integration supplied by Home Assistant.

Reasons for choosing the Met Office include:

- High-quality UK weather forecasts.
- Reliable temperature data.
- Forecast rainfall information.
- Native Home Assistant integration.
- No additional hardware required.
- Suitable for domestic irrigation projects.

Because the system uses Home Assistant entities rather than directly querying an online service, a different weather provider could be substituted later with only minor changes to the automations.

---

# Weather Entities

The following weather entities are currently used.

| Entity | Purpose |
|---------|---------|
| `sensor.met_office_ashby_de_la_zouch_temperature` | Current ambient temperature |
| `sensor.sprinkler_forecast_rain_24h` | Calculated rainfall expected during the next 24 hours |

---

## Ambient Temperature

Temperature is used as a simple indicator of evaporation.

Higher temperatures generally increase the rate at which moisture is lost from the soil.

The current automation groups temperatures into bands rather than attempting to calculate exact evapotranspiration.

| Temperature | Estimated Daily Moisture Loss |
|-------------|------------------------------:|
| Below 10°C | 1 |
| 10–15°C | 3 |
| 15–20°C | 5 |
| 20–25°C | 8 |
| 25–30°C | 12 |
| Above 30°C | 15 |

These values are intentionally conservative and can be adjusted as operational experience is gained.

---

## Forecast Rainfall

Forecast rainfall is represented by the template sensor:

```
sensor.sprinkler_forecast_rain_24h
```

This sensor calculates the total rainfall expected during the next 24 hours using forecast data supplied by the weather integration.

The value is compared against:

```
input_number.sprinkler_rain_skip_threshold
```

If the forecast rainfall meets or exceeds the configured threshold, irrigation is skipped for that day.

---

> [!NOTE]
> **Implementation Note**
>
> The irrigation controller uses forecast rainfall rather than waiting for rainfall to occur. This helps avoid unnecessary watering immediately before significant rain is expected.

---

# Daily Weather Workflow

Each morning the weather data is processed in the following order.

```text
Weather Integration

↓

Current Temperature

↓

Estimate Daily Moisture Loss

↓

Forecast Rainfall

↓

Rain Skip Threshold

↓

Calculate Irrigation Requirement

↓

Store Runtime
```

This sequence ensures that both recent conditions and expected rainfall influence the final watering decision.

---

# Weather Decision Tree

The rainfall decision is intentionally straightforward.

```text
Forecast Rain

↓

Forecast ≥ Rain Skip Threshold?

├── Yes
│
│   Runtime = 0
│
└── No
    │
    ▼
Continue Moisture Calculation
```

This conservative approach reduces unnecessary irrigation while still allowing the lawn to receive additional water during dry periods.

---

# Why Forecasts Instead of Rain Sensors?

Some irrigation systems rely on physical rain sensors.

This project deliberately avoids additional hardware where practical.

Advantages of forecast-based decisions include:

- No external wiring.
- No batteries to replace.
- No roof-mounted equipment.
- Fewer maintenance requirements.
- Easy installation.

Although forecasts are never perfect, they provide sufficient accuracy for a domestic lawn while greatly simplifying the overall system.

---

# Failure Handling

The automations are designed to fail safely if weather data becomes unavailable.

Possible causes include:

- Internet connectivity issues.
- Weather provider outages.
- Home Assistant integration failures.
- Missing sensor entities.

If weather information cannot be obtained, the calculated runtime may be reduced or set to zero depending on the automation logic.

When investigating unexpected behaviour, the Home Assistant **Automation Trace** view should be checked first.

---

# Future Improvements

The current weather model intentionally focuses on the two factors that have the greatest influence on a domestic lawn: temperature and rainfall.

Potential future enhancements include:

- Relative humidity.
- Wind speed.
- Solar radiation.
- UV index.
- Atmospheric pressure.
- Actual evapotranspiration (ET₀) calculations.
- Seasonal adjustment factors.
- Historical rainfall accumulation.
- Local weather station integration.
- Automatic rain gauge support.

These additions could improve accuracy but would also increase system complexity.

---

> [!TIP]
> **Design Philosophy**
>
> Additional data does not always produce better irrigation decisions.
>
> The current design intentionally favours simplicity, reliability and maintainability over scientific precision.

---

# Key Takeaways

- Weather data is central to the irrigation calculations.
- The Met Office integration provides temperature and forecast information.
- Temperature estimates daily moisture loss.
- Forecast rainfall helps avoid unnecessary watering.
- The weather provider can be replaced without redesigning the system.
- The design balances useful weather information with long-term reliability.

---

## Related Files

| File | Description |
|------|-------------|
| `docs/07-Irrigation-Logic.md` | Irrigation decision model |
| `docs/06-Automations.md` | Automation reference |
| `homeassistant/template_sensors.yaml` | Forecast rainfall template sensor *(if used)* |

---

## Navigation

⬅️ Previous: [07 – Irrigation Logic](07-Irrigation-Logic.md)

➡️ Next: [09 – Installation & Setup](09-Installation.md)
