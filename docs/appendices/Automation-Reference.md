# Appendix B — Automation Reference

> **Project:** Home Assistant Smart Sprinkler System  
> **Component:** Home Assistant Automations  
> **Status:** Current configuration reference

## Overview

The smart sprinkler system is controlled by three Home Assistant automations working together:

1. **Sprinklers - Update Virtual Soil Moisture** — estimates daily moisture loss.
2. **Sprinklers - Calculate Water Requirement** — calculates whether watering is required and how much water should be applied.
3. **Sprinklers - Morning Cycle and Soak** — physically runs the seven sprinkler zones using cycle-and-soak watering.

The automations form a daily sequence:

```text
03:45  Update Virtual Soil Moisture
   │
   ▼
04:00  Calculate Water Requirement (every two days)
   │
   ▼
Stores required watering runtime
   │
   ▼
Sunrise + 30 minutes
   │
   ▼
Morning Cycle and Soak
   │
   ▼
Seven sprinkler zones run sequentially
```

The calculation and execution stages are deliberately separate. This allows the system to calculate a watering requirement before sunrise, then perform the watering later in the morning.

---

# 1. Sprinklers - Update Virtual Soil Moisture

## Purpose

This automation maintains the system's **virtual soil moisture model**.

At 03:45 each morning, it reads the current virtual soil moisture value and the current Met Office temperature. It then estimates daily moisture loss using a temperature-based scale and subtracts that amount from the virtual soil moisture helper.

The value is never allowed to fall below zero.

## Trigger

| Trigger | Time |
|---|---|
| Time | `03:45:00` |

## Inputs

| Entity | Purpose |
|---|---|
| `input_number.sprinkler_soil_moisture` | Current virtual soil moisture level |
| `sensor.met_office_ashby_de_la_zouch_temperature` | Temperature used to estimate daily moisture loss |

## Temperature-Based Daily Loss

| Temperature | Daily Loss |
|---|---:|
| Below 10°C | 1 |
| 10°C to below 15°C | 3 |
| 15°C to below 20°C | 5 |
| 20°C to below 25°C | 8 |
| 25°C to below 30°C | 12 |
| 30°C and above | 15 |

## Result

The calculated daily loss is subtracted from `input_number.sprinkler_soil_moisture`.

The calculation uses:

```text
maximum(current moisture − daily loss, 0)
```

This prevents the virtual moisture value from becoming negative.

## Complete YAML

```yaml
alias: Sprinklers - Update Virtual Soil Moisture
description: Virtual soil moisture model
triggers:
  - at: "03:45:00"
    trigger: time
actions:
  - variables:
      moisture: "{{ states('input_number.sprinkler_soil_moisture') | float }}"
      temperature: "{{ states('sensor.met_office_ashby_de_la_zouch_temperature') | float }}"
  - variables:
      daily_loss: |
        {% if temperature < 10 %}
          1
        {% elif temperature < 15 %}
          3
        {% elif temperature < 20 %}
          5
        {% elif temperature < 25 %}
          8
        {% elif temperature < 30 %}
          12
        {% else %}
          15
        {% endif %}
  - target:
      entity_id: input_number.sprinkler_soil_moisture
    data:
      value: |
        {{ [moisture - daily_loss, 0] | max }}
    action: input_number.set_value
mode: single
```

---

# 2. Sprinklers - Calculate Water Requirement

## Purpose

This automation decides whether irrigation is required and calculates the amount of watering required.

It runs at 04:00, provided the sprinkler system is enabled, and is configured to run on alternating days using the day number of the year.

The automation considers:

- Current virtual soil moisture.
- Target soil moisture.
- Forecast rainfall during the next 24 hours.
- Rainfall skip threshold.
- Recovery mode.

If sufficient rainfall is forecast, or if the virtual soil moisture is already at or above its target, no watering is scheduled.

Otherwise, the automation calculates the moisture deficit and converts the required water into a sprinkler runtime.

## Trigger

| Trigger | Time |
|---|---|
| Time | `04:00:00` |

## Conditions

### Sprinklers Enabled

The master helper must be switched on:

```text
input_boolean.sprinklers_enabled = on
```

### Alternating-Day Schedule

The automation uses the day number of the year and only runs when the day number is odd:

```jinja
{{ (now().strftime('%j') | int) % 2 == 1 }}
```

This provides the current every-two-days calculation schedule.

## Inputs

| Entity | Purpose |
|---|---|
| `input_number.sprinkler_soil_moisture` | Current virtual soil moisture |
| `input_number.sprinkler_moisture_target` | Desired soil moisture target |
| `sensor.met_office_ashby_de_la_zouch_temperature` | Current temperature value |
| `sensor.sprinkler_forecast_rain_24h` | Forecast rainfall |
| `input_number.sprinkler_rain_skip_threshold` | Rainfall amount that causes watering to be skipped |
| `input_boolean.sprinkler_recovery_mode` | Enables the higher recovery watering factor |

> **Note:** The current YAML reads the temperature into a variable, although that variable is not subsequently used in the calculation.

## Decision Process

### 1. Rain Forecast Check

If forecast rainfall is greater than or equal to the configured rainfall limit:

```text
forecast_rain >= rain_limit
```

The automation sets both of the following to zero:

- `input_number.sprinkler_water_target`
- `input_number.sprinkler_total_runtime`

No watering is scheduled.

### 2. Moisture Target Check

If the current virtual soil moisture is already at or above the target:

```text
moisture >= target
```

The automation also sets both outputs to zero.

### 3. Calculate the Moisture Deficit

If neither skip condition applies:

```text
deficit = target − moisture
```

### 4. Calculate Water Required

The deficit is multiplied by a refill factor.

| Mode | Refill Factor |
|---|---:|
| Normal | 30% |
| Recovery Mode | 45% |

The result is stored in:

```text
input_number.sprinkler_water_target
```

### 5. Convert Water Requirement to Runtime

The system currently uses an assumed precipitation rate of **12 mm per hour**.

The calculated water requirement is converted into minutes:

```text
total_runtime = (water_needed_mm ÷ 12) × 60
```

The runtime is rounded and capped at 180 minutes before being stored in:

```text
input_number.sprinkler_total_runtime
```

The separate `input_number.sprinkler_runtime` helper is also updated to the lower of the configured cycle runtime or 30 minutes.

## Outputs

| Entity | Purpose |
|---|---|
| `input_number.sprinkler_water_target` | Calculated water requirement |
| `input_number.sprinkler_total_runtime` | Total runtime required for each zone |
| `input_number.sprinkler_runtime` | Runtime value capped at 30 minutes |

## Complete YAML

```yaml
alias: Sprinklers - Calculate Water Requirement
description: >-
  Calculates irrigation from virtual soil moisture and 24 hour rainfall forecast
  (Runs every 2 days)
triggers:
  - trigger: time
    at: "04:00:00"
conditions:
  - condition: state
    entity_id: input_boolean.sprinklers_enabled
    state: "on"
  - condition: template
    value_template: "{{ (now().strftime('%j') | int) % 2 == 1 }}"
actions:
  - variables:
      moisture: |
        {{ states('input_number.sprinkler_soil_moisture') | float }}
      target: |
        {{ states('input_number.sprinkler_moisture_target') | float }}
      temperature: |
        {{ states('sensor.met_office_ashby_de_la_zouch_temperature') | float }}
      forecast_rain: |
        {{ states('sensor.sprinkler_forecast_rain_24h') | float }}
      rain_limit: |
        {{ states('input_number.sprinkler_rain_skip_threshold') | float }}
      recovery: |
        {{ is_state('input_boolean.sprinkler_recovery_mode','on') }}
  - choose:
      - conditions:
          - condition: template
            value_template: |
              {{ forecast_rain >= rain_limit }}
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.sprinkler_water_target
            data:
              value: 0
          - action: input_number.set_value
            target:
              entity_id: input_number.sprinkler_total_runtime
            data:
              value: 0
      - conditions:
          - condition: template
            value_template: |
              {{ moisture >= target }}
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.sprinkler_water_target
            data:
              value: 0
          - action: input_number.set_value
            target:
              entity_id: input_number.sprinkler_total_runtime
            data:
              value: 0
    default:
      - variables:
          deficit: |
            {{ target - moisture }}
          water_needed_mm: |
            {% if recovery %}
              {{ deficit * 0.45 }}
            {% else %}
              {{ deficit * 0.30 }}
            {% endif %}
          total_runtime: |
            {{ ((water_needed_mm / 12) * 60) | round(0) }}
      - action: input_number.set_value
        target:
          entity_id: input_number.sprinkler_water_target
        data:
          value: |
            {{ water_needed_mm }}
      - action: input_number.set_value
        target:
          entity_id: input_number.sprinkler_total_runtime
        data:
          value: |
            {{ [total_runtime,180] | min }}
      - action: input_number.set_value
        target:
          entity_id: input_number.sprinkler_runtime
        data:
          value: >
            {{ [states('input_number.sprinkler_cycle_runtime') | int, 30] | min
            }}
mode: single
```

---

# 3. Sprinklers - Morning Cycle and Soak

## Purpose

This automation performs the physical watering sequence.

Rather than running each sprinkler zone for the entire calculated runtime in one continuous session, the system divides watering into shorter cycles. Each zone receives one cycle of water, the system then allows the soil to soak, and the process repeats until the required total runtime has been delivered.

This approach is designed to improve water absorption and reduce runoff.

## Trigger

| Trigger | Time |
|---|---|
| Sunrise | +30 minutes |

## Conditions

The automation only runs when:

1. `input_boolean.sprinklers_enabled` is `on`.
2. `input_number.sprinkler_total_runtime` is greater than zero.

## Inputs

| Entity | Purpose |
|---|---|
| `input_number.sprinkler_total_runtime` | Total watering time required |
| `input_number.sprinkler_cycle_runtime` | Maximum duration of one watering cycle |
| `input_number.sprinkler_soak_time` | Delay between watering passes |
| `input_boolean.sprinklers_enabled` | Master enable switch |

## Sprinkler Zones

The automation currently controls seven zones in the following order:

1. `switch.sprinklers_garden_sprinkler_1`
2. `switch.sprinklers_garden_sprinkler_2`
3. `switch.sprinklers_garden_sprinkler_3`
4. `switch.sprinklers_garden_sprinkler_4`
5. `switch.sprinklers_garden_sprinkler_5`
6. `switch.sprinklers_garden_sprinkler_6`
7. `switch.sprinklers_garden_sprinkler_7`

## Cycle and Soak Process

The automation repeats the following process while there is still runtime remaining:

### 1. Determine the Current Cycle Length

The runtime for the current pass is the smaller of:

- The configured cycle runtime.
- The remaining total runtime.

```text
run_time = minimum(cycle_runtime, total_runtime)
```

### 2. Water All Seven Zones

Each zone runs sequentially for `run_time` minutes.

Only one zone is operated at a time.

### 3. Reduce Remaining Runtime

After all seven zones have completed a pass:

```text
remaining runtime = total_runtime − run_time
```

### 4. Soak

If further watering is required, the automation waits for the configured soak time before beginning the next pass.

This repeats until the total runtime has been delivered.

## Example

With:

- Total runtime: 10 minutes
- Cycle runtime: 5 minutes
- Soak time: 45 minutes

The sequence would be:

```text
Pass 1
Zone 1 → 5 minutes
Zone 2 → 5 minutes
Zone 3 → 5 minutes
...
Zone 7 → 5 minutes

45-minute soak period

Pass 2
Zone 1 → 5 minutes
Zone 2 → 5 minutes
Zone 3 → 5 minutes
...
Zone 7 → 5 minutes
```

Each zone therefore receives the full calculated 10 minutes, split into two five-minute applications.

Because the zones run sequentially, each zone naturally receives a substantial additional soak period while the other zones are being watered.

## Safety Relationship with ESPHome

The ESPHome firmware contains an independent 31-minute automatic switch-off safety timeout for each relay.

The cycle-and-soak automation should therefore use cycle lengths comfortably below that limit. The ESPHome timeout remains a hardware-level safety backstop rather than part of the normal watering schedule.

## Complete YAML

```yaml
alias: Sprinklers - Morning Cycle and Soak
description: Adaptive sprinkler watering with cycle and soak
triggers:
  - event: sunrise
    offset: "00:30:00"
    trigger: sun
conditions:
  - condition: state
    entity_id: input_boolean.sprinklers_enabled
    state: "on"
  - condition: numeric_state
    entity_id: input_number.sprinkler_total_runtime
    above: 0
actions:
  - variables:
      total_runtime: |
        {{ states('input_number.sprinkler_total_runtime') | int }}
      cycle_runtime: |
        {{ states('input_number.sprinkler_cycle_runtime') | int }}
      soak_time: |
        {{ states('input_number.sprinkler_soak_time') | int }}
      zones:
        - switch.sprinklers_garden_sprinkler_1
        - switch.sprinklers_garden_sprinkler_2
        - switch.sprinklers_garden_sprinkler_3
        - switch.sprinklers_garden_sprinkler_4
        - switch.sprinklers_garden_sprinkler_5
        - switch.sprinklers_garden_sprinkler_6
        - switch.sprinklers_garden_sprinkler_7
  - repeat:
      while:
        - condition: template
          value_template: |
            {{ total_runtime > 0 }}
      sequence:
        - variables:
            run_time: |
              {{ [cycle_runtime, total_runtime] | min }}
        - repeat:
            count: 7
            sequence:
              - action: switch.turn_on
                target:
                  entity_id: "{{ zones[repeat.index-1] }}"
              - delay:
                  minutes: "{{ run_time }}"
              - action: switch.turn_off
                target:
                  entity_id: "{{ zones[repeat.index-1] }}"
        - variables:
            total_runtime: |
              {{ total_runtime - run_time }}
        - if:
            - condition: template
              value_template: |
                {{ total_runtime > 0 }}
          then:
            - delay:
                minutes: "{{ soak_time }}"
mode: single
```

---

# Automation Dependencies

The three automations depend on one another through the helper entities used to store the system's virtual state and calculated requirements.

```text
┌─────────────────────────────────────────────┐
│ Update Virtual Soil Moisture                │
│ 03:45                                       │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
          input_number.sprinkler_soil_moisture
                       │
                       ▼
┌─────────────────────────────────────────────┐
│ Calculate Water Requirement                 │
│ 04:00 — alternating days                    │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
       sprinkler_water_target / total_runtime
                       │
                       ▼
┌─────────────────────────────────────────────┐
│ Morning Cycle and Soak                      │
│ Sunrise + 30 minutes                        │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
             Seven physical sprinkler zones
```

---

# Rebuild Checklist

If the automations ever need to be recreated:

- [ ] Recreate the three automations using the YAML above.
- [ ] Confirm the entity IDs match the helpers and switches in the system.
- [ ] Confirm the Met Office temperature entity is available.
- [ ] Confirm `sensor.sprinkler_forecast_rain_24h` is available.
- [ ] Test each automation manually before relying on scheduled execution.
- [ ] Confirm calculated runtimes are sensible before enabling automatic watering.
- [ ] Confirm the cycle runtime remains below the ESPHome safety timeout.

---

## Related Documentation

- [ESPHome Firmware Reference](ESPHome-Firmware-Reference.md)
- [Helper Reference](Helper-Reference.md)
- [Template Sensor Reference](Template-Sensor-Reference.md)
- [GPIO Reference](GPIO-Reference.md)

---

*Home Assistant Smart Sprinkler System — Appendix B: Automation Reference*
