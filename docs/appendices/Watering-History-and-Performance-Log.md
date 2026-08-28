# Appendix P — Watering History & Performance Log

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** Watering History, Performance Monitoring and Seasonal Observations  
> **Status:** Living document

## Overview

This appendix provides a structured way to record how the Home Assistant Smart Sprinkler System performs over time.

The purpose of this log is not simply to record whether the sprinklers operated. It is intended to help compare:

- What the irrigation model calculated.
- What the weather conditions were.
- How much watering was applied.
- How the lawn and garden actually responded.
- Whether changes to the irrigation model improved performance.

The system currently uses a virtual soil moisture model rather than a physical soil moisture sensor. For that reason, regular observations of the actual garden are particularly valuable when refining the system.

The basic feedback loop is:

```text
Weather Conditions
        │
        ▼
Irrigation Model
        │
        ▼
Calculated Watering
        │
        ▼
Actual Garden Response
        │
        ▼
Performance Observation
        │
        ▼
Future Calibration
```

This document should be treated as a practical record rather than something that needs to be completed every day. A short entry after significant watering periods, weather changes or adjustments to the system is usually sufficient.

---

# 1. How to Use This Log

A useful approach is to record observations at three different levels:

## Individual Watering Events

Use these entries when:

- A significant watering cycle occurs.
- The system behaves unexpectedly.
- You are testing a change.
- Weather conditions are unusual.
- You are actively calibrating the system.

## Weekly or Periodic Reviews

Use these to identify trends that may not be obvious from a single watering cycle.

## Seasonal Reviews

Use these to compare the system's behaviour across spring, summer, autumn and winter.

> **Important:** The aim is to identify patterns. One dry patch after one hot day does not necessarily mean the irrigation model is incorrect.

---

# 2. Current Irrigation Model Reference

When reviewing performance, it can be helpful to remember the current system configuration.

## Core Logic

```text
Temperature
     │
     ▼
Virtual Soil Moisture Loss
     │
     ▼
Current Virtual Soil Moisture
     │
     ├──► Moisture Target
     │
     └──► Moisture Deficit
                │
                ▼
          Water Requirement
                │
                ▼
        Convert to Runtime
                │
                ▼
        Cycle & Soak Watering
```

## Key Configuration Values

Record the current values here when beginning a new calibration period.

| Setting | Entity ID | Current Value |
|---|---|---|
| Virtual Soil Moisture | `input_number.sprinkler_soil_moisture` | |
| Moisture Target | `input_number.sprinkler_moisture_target` | |
| Cycle Runtime | `input_number.sprinkler_cycle_runtime` | |
| Soak Time | `input_number.sprinkler_soak_time` | |
| Rain Skip Threshold | `input_number.sprinkler_rain_skip_threshold` | |
| Recovery Mode | `input_boolean.sprinkler_recovery_mode` | On / Off |
| Assumed Irrigation Rate | System calculation | 12 mm/hour |

> Update the table above when starting a new major testing or calibration period.

---

# 3. Individual Watering Event Log

Use the following format to record significant watering events.

## Watering Event: [Date]

**Date:**

```text
YYYY-MM-DD
```

**Reason for Recording:**

```text
Normal watering / Calibration / Unusual weather / Problem investigation / System test
```

### Conditions Before Watering

| Observation | Value / Notes |
|---|---|
| Weather conditions | |
| Approximate temperature | |
| Recent rainfall | |
| Forecast rainfall | |
| Lawn appearance | |
| Soil surface condition | |
| Notable dry areas | |

### Irrigation Model

| Item | Value |
|---|---|
| Virtual soil moisture | |
| Moisture target | |
| Calculated water target | |
| Calculated total runtime | |
| Cycle runtime | |
| Soak time | |
| Recovery mode | On / Off |

### Watering Result

- [ ] Watering completed normally.
- [ ] All expected zones operated.
- [ ] Zones operated in the correct sequence.
- [ ] Cycle and soak behaviour operated as expected.
- [ ] No unexpected relay behaviour occurred.
- [ ] No leaks were observed.
- [ ] No significant runoff was observed.

### Garden Response

Record observations immediately after watering and, where useful, again the following day.

```text
Immediately after watering:

____________________________________________________

The following day:

____________________________________________________
```

### Overall Assessment

```text
Too little water / About right / Too much water / Inconclusive
```

### Notes

```text
____________________________________________________

____________________________________________________

____________________________________________________
```

---

# 4. Quick Watering Log

For ordinary watering periods where a full report is unnecessary, use the following table.

| Date | Weather | Runtime | Result | Lawn Condition | Notes |
|---|---|---:|---|---|---|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |

Suggested result descriptions include:

```text
No watering required
Watered normally
Watering skipped for rain
Watering interrupted
Manual watering required
Calibration test
```

---

# 5. Lawn Condition Assessment

A simple and consistent assessment system can make observations easier to compare over time.

## Overall Lawn Condition

| Rating | Description |
|---|---|
| 5 | Excellent — healthy colour and good resilience |
| 4 | Good — generally healthy with minor dry areas |
| 3 | Acceptable — some visible stress or unevenness |
| 2 | Poor — noticeable dry or stressed areas |
| 1 | Very poor — significant drought stress or damage |

## Moisture Appearance

| Assessment | Description |
|---|---|
| Wet | Surface remains visibly wet or soft |
| Damp | Clearly moist without standing water |
| Normal | Appears healthy and appropriately moist |
| Dry | Surface or grass shows signs of drying |
| Very dry | Significant visible drought stress |

The exact descriptions do not need to be scientifically precise. The most important thing is to use the same general standards over time.

---

# 6. Zone Performance Review

Because different areas of a garden can receive different amounts of water, it can be useful to assess zones individually.

| Zone | Coverage | Lawn Condition | Dry Areas | Runoff | Notes |
|---|---|---|---|---|---|
| Zone 1 | Good / Fair / Poor | 1–5 | None / Minor / Significant | None / Minor / Significant | |
| Zone 2 | Good / Fair / Poor | 1–5 | None / Minor / Significant | None / Minor / Significant | |
| Zone 3 | Good / Fair / Poor | 1–5 | None / Minor / Significant | None / Minor / Significant | |
| Zone 4 | Good / Fair / Poor | 1–5 | None / Minor / Significant | None / Minor / Significant | |
| Zone 5 | Good / Fair / Poor | 1–5 | None / Minor / Significant | None / Minor / Significant | |
| Zone 6 | Good / Fair / Poor | 1–5 | None / Minor / Significant | None / Minor / Significant | |
| Zone 7 | Good / Fair / Poor | 1–5 | None / Minor / Significant | None / Minor / Significant | |

> **Note:** If one area repeatedly performs differently from the rest of the garden, investigate physical coverage and sprinkler calibration before changing the watering requirement for every zone.

---

# 7. Weekly or Periodic Performance Review

A periodic review can help identify trends.

## Review Period

**From:**

```text
YYYY-MM-DD
```

**To:**

```text
YYYY-MM-DD
```

### Weather Summary

| Condition | Observation |
|---|---|
| General weather | |
| Rainfall pattern | |
| Temperature pattern | |
| Unusually hot days | |
| Prolonged dry periods | |

### System Activity

| Item | Observation |
|---|---|
| Number of watering cycles | |
| Number of rain skips | |
| Manual watering required | |
| System problems | |
| Changes made | |

### Lawn Performance

```text
Overall lawn condition:

____________________________________________________

Best-performing areas:

____________________________________________________

Areas requiring attention:

____________________________________________________
```

### Irrigation Assessment

```text
The system appears to be:

[ ] Applying too much water
[ ] Applying approximately the right amount of water
[ ] Applying too little water
[ ] Performing inconsistently
[ ] Too early to determine
```

### Action for the Next Period

```text
No changes required / Continue monitoring / Adjust calibration / Investigate hardware
```

---

# 8. Spring Performance Review

## Spring Conditions

Spring can involve rapidly changing conditions, including:

- Cool temperatures.
- Periods of significant rainfall.
- Increasing daylight.
- Gradually increasing lawn growth.
- Occasional warm and dry periods.

The irrigation model may require less watering during consistently wet periods but should still be monitored as temperatures begin to increase.

### Spring Checklist

- [ ] Check the irrigation system after winter.
- [ ] Inspect sprinkler heads.
- [ ] Check for leaks or damaged pipework.
- [ ] Confirm all zones operate correctly.
- [ ] Review the virtual soil moisture starting value.
- [ ] Monitor the effect of increasing temperatures.
- [ ] Check whether rain skip logic is behaving sensibly.

### Spring Seasonal Record

**Year:**

```text
__________
```

| Observation | Notes |
|---|---|
| Overall spring weather | |
| Rainfall | |
| Lawn recovery after winter | |
| First significant irrigation period | |
| System performance | |
| Changes made | |

### Spring Conclusion

```text
What worked well?

____________________________________________________

What should be adjusted next spring?

____________________________________________________
```

---

# 9. Summer Performance Review

## Summer Conditions

Summer is likely to be the most demanding period for the irrigation system.

Monitor for:

- Extended dry periods.
- High temperatures.
- Rapid moisture loss.
- Localised dry patches.
- Runoff during watering.
- Increased differences between shaded and exposed areas.

The current virtual soil moisture model uses temperature as an indicator of estimated moisture loss, making summer observations particularly useful for calibration.

### Summer Checklist

- [ ] Monitor lawn condition regularly during dry periods.
- [ ] Compare calculated watering against actual lawn response.
- [ ] Check for dry edges or uneven coverage.
- [ ] Watch for runoff.
- [ ] Review whether cycle and soak periods are appropriate.
- [ ] Record periods where recovery mode is required.
- [ ] Check whether the assumed irrigation rate remains realistic.

### Summer Seasonal Record

**Year:**

```text
__________
```

| Observation | Notes |
|---|---|
| Hottest period | |
| Longest dry period | |
| Lawn condition at peak summer | |
| Number of significant watering periods | |
| Recovery mode used | Yes / No |
| Areas of repeated stress | |
| Overall system performance | |

### Summer Conclusion

```text
Did the lawn receive enough water?

____________________________________________________

Was any area consistently over-watered?

____________________________________________________

Did the irrigation model respond appropriately to hot weather?

____________________________________________________
```

---

# 10. Autumn Performance Review

## Autumn Conditions

Autumn can be a useful period for reducing irrigation and observing how quickly the lawn recovers from summer stress.

Conditions may include:

- Lower temperatures.
- Increased rainfall.
- Reduced evaporation.
- Continued grass growth.
- Occasional dry early-autumn periods.

### Autumn Checklist

- [ ] Monitor whether the system begins watering less frequently.
- [ ] Check that rain skip logic is preventing unnecessary watering.
- [ ] Observe lawn recovery following summer.
- [ ] Inspect sprinklers after heavy leaf fall where applicable.
- [ ] Record whether any summer calibration changes remain appropriate.

### Autumn Seasonal Record

**Year:**

```text
__________
```

| Observation | Notes |
|---|---|
| Overall autumn weather | |
| Rainfall pattern | |
| Reduction in watering | |
| Lawn recovery | |
| System performance | |
| Preparation for winter | |

### Autumn Conclusion

```text
What changes should be carried forward into the following year?

____________________________________________________
```

---

# 11. Winter Performance Review

## Winter Conditions

The sprinkler system may require little or no automatic irrigation during prolonged wet or cold periods.

Winter is also a useful time for:

- Reviewing system performance.
- Updating documentation.
- Planning improvements.
- Inspecting hardware.
- Preparing for the next growing season.

### Winter Checklist

- [ ] Review whether automatic watering should remain enabled.
- [ ] Inspect accessible irrigation hardware.
- [ ] Check the ESP32 and relay enclosure.
- [ ] Review the year's watering records.
- [ ] Identify calibration improvements.
- [ ] Back up the Home Assistant and ESPHome configuration.
- [ ] Update the GitHub documentation.

### Winter Seasonal Record

**Year:**

```text
__________
```

| Observation | Notes |
|---|---|
| Winter weather | |
| Automatic watering activity | |
| Hardware issues | |
| Maintenance completed | |
| Documentation updates | |
| Planned improvements | |

---

# 12. Annual Performance Summary

At the end of each full growing year, complete a short annual review.

## Year

```text
__________
```

### Overall System Performance

| Area | Rating / Assessment |
|---|---|
| Reliability | Excellent / Good / Fair / Poor |
| Watering accuracy | Excellent / Good / Fair / Poor |
| Lawn condition | Excellent / Good / Fair / Poor |
| Rain skip performance | Excellent / Good / Fair / Poor |
| Cycle and soak performance | Excellent / Good / Fair / Poor |
| Ease of maintenance | Excellent / Good / Fair / Poor |

### Key Achievements

```text
____________________________________________________

____________________________________________________
```

### Problems Encountered

```text
____________________________________________________

____________________________________________________
```

### Calibration Lessons

```text
____________________________________________________

____________________________________________________
```

### Changes Planned for Next Year

```text
____________________________________________________

____________________________________________________
```

---

# 13. Calibration Change Record

Whenever a value is changed as a result of real-world observations, record it here.

| Date | Setting Changed | Previous Value | New Value | Reason | Result |
|---|---|---:|---:|---|---|
| | | | | | |
| | | | | | |
| | | | | | |

Examples of settings that may be adjusted include:

```text
Sprinkler Cycle Runtime
Sprinkler Soak Time
Sprinkler Moisture Target
Sprinkler Rain Skip Threshold
Virtual Soil Moisture Starting Value
Assumed Irrigation Rate
```

> Avoid changing several calibration values simultaneously where possible. Changing one variable at a time makes it easier to understand which adjustment produced an improvement.

---

# 14. Weather Extremes and Special Events

Record unusual conditions that may help explain unexpected irrigation behaviour.

Examples include:

- Heatwaves.
- Extended drought.
- Unusually heavy rainfall.
- Long periods of cloudy weather.
- Unexpected late frosts.
- Water supply interruptions.
- Exceptional lawn stress.

## Special Event Record

**Date or Period:**

```text
____________________________
```

**Event:**

```text
____________________________
```

**Effect on Garden:**

```text
____________________________________________________
```

**System Response:**

```text
____________________________________________________
```

**Lesson or Action Taken:**

```text
____________________________________________________
```

---

# 15. Observations Worth Watching

Over time, the following questions can help determine whether the system is improving.

## Watering Accuracy

- Does the lawn generally look healthy without frequent manual intervention?
- Does the system appear to water only when required?
- Are calculated runtimes broadly sensible?

## Rain Skip Behaviour

- Does forecast rain reliably prevent unnecessary watering?
- Are there occasions where watering was skipped but rain did not arrive?
- Are there occasions where watering occurred shortly before significant rainfall?

## Virtual Soil Moisture Model

- Does the calculated soil moisture trend generally match the appearance of the garden?
- Does the model react sufficiently during hot weather?
- Does it recover appropriately during wetter periods?

## Physical Irrigation

- Are some areas consistently drier than others?
- Are there areas with excessive runoff?
- Has sprinkler coverage changed over time?

These observations can help distinguish between a **calculation problem** and a **physical irrigation problem**.

---

# 16. Performance Trend Summary

Use this table to build a simple long-term record.

| Season | Year | Lawn Condition | Watering Performance | Main Issue | Main Improvement |
|---|---:|---|---|---|---|
| Spring | | | | | |
| Summer | | | | | |
| Autumn | | | | | |
| Winter | | | | | |

After several seasons, this table should provide a useful overview of how the system and garden have changed over time.

---

# 17. Long-Term Goal

The purpose of recording performance is to gradually improve confidence in the irrigation model.

The ideal progression is:

```text
Initial Assumptions
        │
        ▼
Observe Real Garden Performance
        │
        ▼
Identify Consistent Patterns
        │
        ▼
Make Small Calibration Changes
        │
        ▼
Observe Again
        │
        ▼
Improve the Model Over Time
```

The system does not need to be perfectly calibrated immediately.

A gradual approach based on real observations is likely to produce a more useful system than repeatedly making large changes based on a single dry or wet period.

---

# 18. Related Documentation

- [System Architecture Overview](System-Architecture-Overview.md)
- [Irrigation Calibration](Irrigation-Calibration.md)
- [Automation Reference](Automation-Reference.md)
- [Template Sensor Reference](Template-Sensor-Reference.md)
- [Testing & Commissioning Checklist](Testing-and-Commissioning-Checklist.md)
- [System Maintenance](System-Maintenance.md)
- [Change Log & Upgrade Notes](Change-Log-and-Upgrade-Notes.md)

---

*Home Assistant Smart Sprinkler System — Appendix P: Watering History & Performance Log*
