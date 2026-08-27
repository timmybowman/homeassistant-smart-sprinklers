# Appendix H — Irrigation Calibration

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** Irrigation Performance & Calibration  
> **Status:** Initial calibration reference

## Overview

This appendix documents the assumptions and observations used to calibrate the Home Assistant Smart Sprinkler System.

The irrigation system uses a calculated watering runtime rather than a permanently fixed timer. To convert a calculated water requirement into a sprinkler runtime, the system needs an estimate of how much water the sprinklers apply to the lawn over time.

The current system uses an assumed precipitation rate of:

```text
12 mm per hour
```

This value is used as a starting point and can be refined as real-world observations and measurements become available.

Calibration is important because the actual amount of water reaching the lawn can be affected by:

- Sprinkler nozzle size.
- Sprinkler sweep angle.
- Water pressure.
- Pipe lengths.
- Flow restrictions.
- Sprinkler spacing.
- Wind.
- Overlap between sprinkler heads.
- Differences between areas of the garden.

The goal of calibration is not necessarily to achieve a perfect laboratory measurement. The practical goal is to establish watering settings that keep the lawn healthy without consistently overwatering.

---

# 1. Current Irrigation Assumption

The Home Assistant watering calculations currently assume a precipitation rate of:

```text
12 mm/hour
```

This means that, as an initial estimate:

| Runtime | Estimated Water Applied |
|---:|---:|
| 5 minutes | 1.0 mm |
| 10 minutes | 2.0 mm |
| 15 minutes | 3.0 mm |
| 20 minutes | 4.0 mm |
| 25 minutes | 5.0 mm |
| 30 minutes | 6.0 mm |

The relationship is calculated using:

```text
Water Applied (mm) = Runtime (minutes) ÷ 60 × Precipitation Rate (mm/hour)
```

The reverse calculation, used when determining watering time, is:

```text
Runtime (minutes) = Water Required (mm) ÷ Precipitation Rate (mm/hour) × 60
```

Using the current assumed rate of 12 mm/hour:

```text
Runtime = Water Required ÷ 12 × 60
```

This is the basis of the runtime calculation used by the sprinkler automation.

---

# 2. Current System Configuration

The garden currently uses:

- Seven active sprinkler zones.
- Rain Bird 5004 sprinkler heads.
- A mixture of full-sweep and half-sweep sprinklers.
- Sequential operation, with one zone operating at a time.
- Cycle-and-soak watering to reduce prolonged continuous watering.
- An ESPHome safety timeout that switches an individual zone off after 31 minutes.

The physical irrigation system is supplied from the mains water supply and uses a combination of:

- Approximately 1 metre of 15 mm copper pipe.
- Approximately 10–15 metres of 32 mm MDPE pipe to the manifold.
- Individual 20 mm MDPE runs from the manifold to the sprinkler heads.

The shortest individual run is approximately 7 metres and the longest is approximately 35 metres.

These differences may affect the water pressure and flow available at different sprinkler locations.

---

# 3. Sprinkler Head Configuration

The system uses Rain Bird 5004 sprinkler heads.

The installation includes a mixture of full-sweep and reduced-sweep coverage.

The full-sweep sprinklers use the largest nozzle configuration, while the sprinklers operating over approximately half the area use a nozzle configuration intended to reduce the flow to approximately 50%.

This arrangement is intended to compensate for the reduced area covered by a half-sweep sprinkler.

The underlying principle is:

```text
Smaller coverage area
        │
        ▼
Reduced water flow
        │
        ▼
Similar application rate across the lawn
```

In practice, the actual application rate should be verified through observation and, if required, measurement.

---

# 4. Why Calibration Matters

A calculated irrigation system can only be as accurate as the assumptions used in its calculations.

For example, if the actual precipitation rate is lower than 12 mm/hour, the system may apply less water than intended.

Conversely, if the actual precipitation rate is higher than 12 mm/hour, the system may apply more water than intended.

```text
Assumed precipitation rate
           │
           ▼
Calculated runtime
           │
           ▼
Actual sprinkler operation
           │
           ▼
Water reaching the lawn
```

The purpose of calibration is to improve the relationship between the calculated runtime and the actual amount of water reaching the grass.

---

# 5. Initial Lawn Observations

The sprinkler system has been observed to provide generally good coverage across the main lawn area.

However, some dry patches have been observed around the edges of the garden.

Possible reasons for this may include:

- Reduced overlap at the outer edges.
- Sprinkler positioning.
- Differences in sprinkler throw.
- Wind.
- Variations in pressure or flow.
- The shape of the lawn extending beyond the main sprinkler coverage pattern.

Dry edges do not necessarily mean that the overall runtime should immediately be increased.

Increasing the runtime for every zone may overwater areas that are already receiving sufficient water.

Where possible, the cause of local dry areas should be identified before increasing the overall system runtime.

---

# 6. Practical Calibration Method

A simple calibration method can be carried out using several identical, shallow containers.

Examples include:

- Straight-sided measuring containers.
- Rain gauges.
- Small identical cups with a wide opening.

The containers should be placed at different locations across the lawn.

A useful distribution includes:

- Areas close to sprinkler heads.
- Areas between sprinkler heads.
- The centre of the lawn.
- The outer edges.
- Areas where dry patches have previously been observed.

Run the irrigation system for a known period and measure the water collected in each container.

For example:

```text
Known sprinkler runtime: 20 minutes

Container 1: 4 mm
Container 2: 3 mm
Container 3: 5 mm
Container 4: 2 mm
Container 5: 4 mm
```

The results can be used to estimate the average application rate and identify areas receiving significantly less water.

---

# 7. Calculating the Measured Precipitation Rate

If a measured amount of water is collected during a known runtime, the estimated hourly precipitation rate can be calculated as:

```text
Measured Water (mm) ÷ Runtime (minutes) × 60
```

For example:

```text
Average collected water: 4 mm
Runtime: 20 minutes

4 ÷ 20 × 60 = 12 mm/hour
```

In this example, the measurement supports the current 12 mm/hour assumption.

Another example:

```text
Average collected water: 5 mm
Runtime: 20 minutes

5 ÷ 20 × 60 = 15 mm/hour
```

This would indicate that the actual application rate may be closer to 15 mm/hour.

---

# 8. Using an Average Measurement

Individual collection points may vary because sprinkler coverage is not perfectly uniform.

For this reason, it is generally more useful to consider the average across several locations.

For example:

| Collection Point | Water Collected |
|---|---:|
| 1 | 3.5 mm |
| 2 | 4.0 mm |
| 3 | 4.5 mm |
| 4 | 3.0 mm |
| 5 | 5.0 mm |
| **Average** | **4.0 mm** |

If the sprinklers operated for 20 minutes:

```text
4.0 mm ÷ 20 minutes × 60
= 12 mm/hour
```

The average application rate can then be used as the starting point for the Home Assistant runtime calculation.

---

# 9. Uniformity Is as Important as the Average

A good average result does not necessarily mean that every part of the lawn is receiving the correct amount of water.

For example:

```text
Location A: 6 mm
Location B: 6 mm
Location C: 1 mm
Location D: 1 mm
```

The average may appear acceptable, but half of the measured locations are receiving significantly less water.

Calibration should therefore consider both:

1. The average amount of water applied.
2. The consistency of coverage across the lawn.

If large differences are found, possible improvements include:

- Adjusting sprinkler arcs.
- Checking nozzle sizes.
- Adjusting sprinkler positions.
- Checking for obstructions.
- Improving overlap.
- Creating separate irrigation zones in future.

---

# 10. Relationship with Cycle and Soak Watering

The system uses cycle-and-soak watering.

Rather than applying the entire watering requirement in one continuous run, the system can divide the runtime into smaller cycles.

For example:

```text
Total watering requirement: 20 minutes per zone
Cycle runtime: 5 minutes
Soak time: 45 minutes
```

The sequence becomes:

```text
Cycle 1
Zone 1 → Zone 7, 5 minutes each
        │
        ▼
45-minute soak period
        │
        ▼
Cycle 2
Zone 1 → Zone 7, 5 minutes each
        │
        ▼
45-minute soak period
        │
        ▼
Further cycles as required
```

The purpose of soaking is to give water time to move into the soil before additional water is applied.

Because the zones are operated sequentially, the actual time between repeated watering of an individual zone may already be significantly longer than the configured soak period.

This should be considered when adjusting the cycle-and-soak settings.

---

# 11. Understanding Zone Timing

The Home Assistant automation currently works through all seven zones sequentially during each watering pass.

For example, with a five-minute cycle:

```text
Zone 1 → 5 minutes
Zone 2 → 5 minutes
Zone 3 → 5 minutes
Zone 4 → 5 minutes
Zone 5 → 5 minutes
Zone 6 → 5 minutes
Zone 7 → 5 minutes
```

By the time the system returns to Zone 1 for another pass, Zone 1 has already had time for the first application of water to soak into the ground.

The configured soak time adds an additional delay between complete passes.

This means the effective soaking period for an individual zone is influenced by:

```text
Time spent watering the remaining zones
              +
Configured soak time
```

This is important when choosing a soak duration.

A long soak setting may not always be necessary if the sequential operation of the other zones already provides sufficient time between watering passes.

---

# 12. Calibration of Cycle Runtime

The cycle runtime determines how long an individual sprinkler zone operates before the system moves on to the next zone.

The current system includes a separate helper for this:

```text
input_number.sprinkler_cycle_runtime
```

The cycle runtime should remain comfortably below the ESPHome 31-minute safety timeout.

A shorter cycle may be beneficial if:

- Water begins to run off the surface.
- The soil absorbs water slowly.
- The lawn is on a slope.
- Water is visibly pooling.

A longer cycle may be appropriate if:

- The soil absorbs water easily.
- There is little or no runoff.
- The system requires fewer watering passes.

The best setting is the longest cycle that provides good absorption without significant runoff.

---

# 13. Calibration of Soak Time

The soak period is controlled by:

```text
input_number.sprinkler_soak_time
```

The soak time allows water to absorb into the soil before another watering pass begins.

The appropriate setting depends on:

- Soil type.
- Soil compaction.
- Weather conditions.
- Water application rate.
- Cycle runtime.
- The amount of time spent watering the other zones.

Because the seven zones operate sequentially, the system naturally provides a period of rest for each zone while the other zones are being watered.

When adjusting the soak time, consider the complete time between two watering cycles for the same zone rather than looking only at the configured soak helper.

---

# 14. Gradual Adjustment Strategy

Calibration changes should ideally be made gradually.

A sensible approach is:

1. Establish a starting application rate.
2. Observe the lawn over several watering cycles.
3. Record areas that remain dry or appear overwatered.
4. Adjust one setting at a time where possible.
5. Allow time to observe the effect of the change.
6. Record the result.

Avoid changing several major settings simultaneously, as this makes it difficult to determine which change improved or worsened the results.

---

# 15. Suggested Calibration Record

The following table can be copied into a garden maintenance record when testing the system.

| Date | Weather | Runtime | Cycle Runtime | Soak Time | Measured Water | Lawn Observation |
|---|---|---:|---:|---:|---:|---|
| | | | | | | |
| | | | | | | |
| | | | | | | |
| | | | | | | |

Useful observations include:

- Grass colour.
- Dry patches.
- Water runoff.
- Standing water.
- Edge coverage.
- Areas receiving excessive water.

Over time, this record can help identify the most effective settings for the particular garden.

---

# 16. Current Calibration Values

The following values represent the current documented starting point for the system.

| Setting | Current Value | Purpose |
|---|---:|---|
| Assumed precipitation rate | 12 mm/hour | Converts water requirement into runtime |
| Active zones | 7 | Number of zones used by automatic watering |
| ESPHome safety timeout | 31 minutes | Independent maximum continuous relay operation |
| Sprinkler type | Rain Bird 5004 | Lawn sprinkler heads |
| Cycle runtime | Controlled by helper | Maximum runtime for each watering pass |
| Soak time | Controlled by helper | Delay between complete watering passes |

The cycle runtime and soak time are deliberately controlled through Home Assistant helpers so they can be refined without modifying the ESPHome firmware.

---

# 17. Future Improvements

The irrigation calibration can be improved over time without changing the basic architecture of the system.

Possible future refinements include:

- Measuring the actual precipitation rate across the garden.
- Recording the application rate for individual zones.
- Adjusting sprinkler nozzles or arcs.
- Improving dry edge coverage.
- Using different runtime calculations for zones with significantly different application rates.
- Recording seasonal lawn performance.
- Refining the virtual soil moisture model using real-world observations.

The current system is designed to allow these improvements to be introduced gradually.

---

# 18. Calibration Principles

The overall calibration approach can be summarised as:

```text
Measure
   │
   ▼
Observe
   │
   ▼
Adjust one variable
   │
   ▼
Run the system
   │
   ▼
Observe the lawn
   │
   ▼
Repeat if required
```

The aim is to develop a system that responds appropriately to the garden's real-world conditions rather than relying indefinitely on theoretical values.

---

## Related Documentation

- [Hardware Specification](Hardware-Specification.md)
- [GPIO & ESP32 Pin Reference](GPIO-ESP32-Pin-Reference.md)
- [ESPHome Firmware Reference](ESPHome-Firmware-Reference.md)
- [Irrigation Logic](../docs/07-Irrigation-Logic.md)
- [Automation Reference](Automation-Reference.md)
- [Helper Reference](Helper-Reference.md)

---

*Home Assistant Smart Sprinkler System — Appendix H: Irrigation Calibration*
