---
inclusion: manual
description: Guide for building complex smart climate systems (heating/cooling) with Home Assistant using modular automation patterns
---

# Smart Climate System Guide

This guide explains how to build a complex, room-by-room smart climate system in Home Assistant. The approach breaks down complexity into simple, manageable automation chains. While the examples focus on heating, the same patterns apply to cooling systems, heat pumps, and any climate entity.

## Core Principles

### 1. Plan Before You Build
Define what you want to achieve before writing any automation:
- What modes do you need per room?
- What inputs will drive those modes?
- What temperatures correspond to each mode?

### 2. Break Down Complexity
Split the automation into independent pieces:
- **Inputs → Modes**: Sensors determine the climate mode
- **Modes → Target Temperature**: Each mode has a specific temperature
- **Target Temperature → Climate Device**: Apply temperature to the actual device

### 3. Use Helpers Extensively
Helpers act as intermediate state holders that "bound" complexity:
- `input_select` for modes
- `input_number` for temperatures
- Each helper represents one piece of the chain

## Recommended Climate Modes

### Standard Room Modes
- `Present`: Room is occupied (19°C for heating, 24°C for cooling recommended)
- `Absent`: Room is unoccupied (16°C for heating, 28°C for cooling)
- `Vent`: Windows are open (7°C frost protection for heating, off for cooling)

### Special Modes (Room-Specific)
- `Boost`: Temporary high/low temp (e.g., bathroom during shower, quick cool-down)
- `Sleep`: Nighttime temperature (e.g., bedroom)
- `Away`: Extended absence (lower/higher than Absent)
- `Eco`: Energy-saving mode with wider temperature tolerance

## Input Triggers

### Presence Detection
- Use mmWave presence sensors for accurate detection
- Add 5-minute duration to avoid false positives
- Combine with home-level presence to filter pet movements

```yaml
# Example: Presence trigger with duration
- trigger: state
  entity_id: binary_sensor.office_presence
  for:
    minutes: 5
```

### Window/Door Sensors
- Contact sensors detect open windows
- Add 5-minute duration before switching to Vent mode
- Prevents mode changes during brief openings

```yaml
# Example: Window open trigger
- trigger: state
  entity_id: binary_sensor.office_window
  for:
    minutes: 5
  to: "on"
```

### Home Presence
- Use a home-level mode (Occupied/Away/Vacation)
- Override room presence when home is empty
- Prevents climate control from pet movements when away

## Implementation Pattern

### Step 1: Create Mode Helper

Create an `input_select` for each room's climate modes:

```yaml
# Example: Office climate modes
input_select:
  office_climate_modes:
    name: Office Climate Modes
    options:
      - Present
      - Absent
      - Vent
    initial: Absent
```

### Step 2: Create Temperature Helpers

Create `input_number` helpers for each mode's target temperature:

```yaml
# Example: Office temperature helpers
input_number:
  office_target_temp_present:
    name: Office Target Temp (Present)
    min: 5
    max: 30
    step: 0.5
    initial: 19
    unit_of_measurement: "°C"
  
  office_target_temp_absent:
    name: Office Target Temp (Absent)
    min: 5
    max: 30
    step: 0.5
    initial: 16
    unit_of_measurement: "°C"
  
  office_target_temp_vent:
    name: Office Target Temp (Vent)
    min: 5
    max: 30
    step: 0.5
    initial: 7
    unit_of_measurement: "°C"
  
  office_target_temp:
    name: Office Target Temperature
    min: 5
    max: 30
    step: 0.5
    unit_of_measurement: "°C"
```

### Step 3: Automation Chain

#### Automation 1: Inputs → Modes

```yaml
alias: Compute Office Climate Modes
triggers:
  - trigger: state
    entity_id:
      - input_select.home_modes
  - trigger: state
    entity_id: binary_sensor.office_window
    for:
      minutes: 5
  - trigger: state
    entity_id: binary_sensor.office_presence
    for:
      minutes: 5
  - trigger: homeassistant
    event: start
actions:
  - choose:
      # Priority 1: Vent mode if window open
      - conditions:
          - condition: state
            entity_id: binary_sensor.office_window
            state: "on"
            for:
              minutes: 5
        sequence:
          - action: input_select.select_option
            target:
              entity_id: input_select.office_climate_modes
            data:
              option: Vent
      # Priority 2: Present if home occupied AND room presence
      - conditions:
          - condition: state
            entity_id: binary_sensor.office_presence
            state: "on"
            for:
              minutes: 5
          - condition: state
            entity_id: input_select.home_modes
            state: Occupied
        sequence:
          - action: input_select.select_option
            target:
              entity_id: input_select.office_climate_modes
            data:
              option: Present
    # Default: Absent
    default:
      - action: input_select.select_option
        target:
          entity_id: input_select.office_climate_modes
        data:
          option: Absent
mode: single
```

#### Automation 2: Modes → Target Temperature

```yaml
alias: Compute Office Target Temperature
triggers:
  - trigger: state
    entity_id:
      - input_select.office_climate_modes
      - input_number.office_target_temp_absent
      - input_number.office_target_temp_present
      - input_number.office_target_temp_vent
  - trigger: homeassistant
    event: start
actions:
  - choose:
      - conditions:
          - condition: state
            entity_id: input_select.office_climate_modes
            state: Present
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.office_target_temp
            data:
              value: "{{ states('input_number.office_target_temp_present') }}"
      - conditions:
          - condition: state
            entity_id: input_select.office_climate_modes
            state: Absent
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.office_target_temp
            data:
              value: "{{ states('input_number.office_target_temp_absent') }}"
      - conditions:
          - condition: state
            entity_id: input_select.office_climate_modes
            state: Vent
        sequence:
          - action: input_number.set_value
            target:
              entity_id: input_number.office_target_temp
            data:
              value: "{{ states('input_number.office_target_temp_vent') }}"
mode: single
```

#### Automation 3: Target Temperature → Climate Device (with Override)

```yaml
alias: Drive Office Climate
triggers:
  # Immediate: when target temp changes
  - trigger: state
    entity_id: input_number.office_target_temp
  # Override reset: after 1 hour of manual change
  - trigger: state
    entity_id: climate.office
    attribute: temperature
    for:
      hours: 1
  - trigger: homeassistant
    event: start
actions:
  - action: climate.set_temperature
    target:
      entity_id: climate.office
    data:
      temperature: "{{ states('input_number.office_target_temp') }}"
mode: single
```

## Manual Override Pattern

The override mechanism allows manual control without fighting the automation:

1. User manually adjusts thermostat/valve
2. Temperature changes but mode stays the same
3. After 1 hour, automation re-applies the mode's target temperature
4. If no manual change occurred, re-applying has no effect

This approach is "helpful but never in the way" - users can override when needed, and the system self-corrects after a timeout.

## Room-Specific Variations

### Bathroom with Boost Mode

Add humidity-triggered boost for shower time:

```yaml
# Additional mode
input_select:
  bathroom_climate_modes:
    options:
      - Present
      - Absent
      - Boost  # Triggered by humidity

# Boost trigger in mode automation
- conditions:
    - condition: numeric_state
      entity_id: sensor.bathroom_humidity
      above: 70
  sequence:
    - action: input_select.select_option
      target:
        entity_id: input_select.bathroom_climate_modes
      data:
        option: Boost
```

### Bedroom with Sleep Mode

Add time-based sleep mode:

```yaml
# Additional mode
input_select:
  bedroom_climate_modes:
    options:
      - Present
      - Absent
      - Sleep  # Lower/higher temp at night

# Sleep trigger
- conditions:
    - condition: time
      after: "22:00:00"
      before: "07:00:00"
    - condition: state
      entity_id: binary_sensor.bedroom_presence
      state: "on"
  sequence:
    - action: input_select.select_option
      target:
        entity_id: input_select.bedroom_climate_modes
      data:
        option: Sleep
```

### Cooling System Adaptation

For air conditioning or heat pumps in cooling mode:

```yaml
# Cooling-specific temperatures
input_number:
  office_target_temp_present:
    initial: 24  # Comfortable cooling temp
  office_target_temp_absent:
    initial: 28  # Allow warmer when away
  office_target_temp_vent:
    initial: 30  # Effectively off when windows open

# Turn off cooling when windows open instead of frost protection
- conditions:
    - condition: state
      entity_id: binary_sensor.office_window
      state: "on"
  sequence:
    - action: climate.turn_off
      target:
        entity_id: climate.office_ac
```

## Best Practices

### Always Add Startup Trigger
Include `homeassistant.start` trigger in all automations to ensure correct state after restart:

```yaml
- trigger: homeassistant
  event: start
```

### Use Duration for Stability
Add `for` duration to presence and window triggers to prevent rapid mode switching:
- 5 minutes is a good default
- Adjust based on your usage patterns

### Combine Home and Room Presence
Filter room presence with home-level presence to avoid climate control for pets:

```yaml
- condition: state
  entity_id: input_select.home_modes
  state: Occupied
```

### Keep Automations Single-Purpose
Each automation should do one thing:
- One for computing modes
- One for computing target temperature
- One for driving the climate device

### Use Descriptive Names
Name helpers and automations clearly:
- `office_climate_modes` not `office_modes`
- `Compute Office Target Temperature` not `Office Temp`

## Scaling to Multiple Rooms

The pattern scales by duplication:
1. Create the same helpers for each room
2. Create the same 3 automations for each room
3. Adjust inputs and modes per room's needs

Typical setup for a home:
- 26 helpers (modes + temperatures per room)
- 20 automations (3 per room + home-level)

## Troubleshooting

### Mode Not Changing
- Check if duration conditions are met
- Verify sensor states in Developer Tools
- Check automation traces for condition failures

### Temperature Not Applying
- Verify climate entity supports `set_temperature`
- Check if device is in correct HVAC mode
- Look for entity unavailable states

### Override Not Working
- Ensure climate entity exposes temperature attribute
- Verify 1-hour duration trigger is configured
- Check if automation is enabled

## Hardware Compatibility

This pattern works with any climate entity:
- Smart thermostats (Netatmo, Ecobee, Nest)
- Thermostatic radiator valves (TRVs)
- Heat pumps (heating and cooling)
- Air conditioners
- Electric heaters with smart plugs + temperature sensors
- Mini-split systems

Requirements:
- Climate entity with `set_temperature` support
- Presence sensors (mmWave recommended)
- Contact sensors for windows/doors

## References

- [Original Blog Post by JLo](https://blog.jlpouffier.fr/a-complex-smart-heating-system-build-simply/)
- [JLo's Home Assistant Config](https://github.com/jlpouffier/home-assistant-config)
