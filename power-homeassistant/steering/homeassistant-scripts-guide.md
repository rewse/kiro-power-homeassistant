# Home Assistant Scripts Guide

This steering file provides comprehensive guidance on Home Assistant script syntax, actions, and conditions.

## Script Syntax Overview

Scripts are sequences of actions that Home Assistant executes. They can be standalone entities via the Script integration or embedded in automations.

### Basic Structure

```yaml
script:
  example_script:
    sequence:
      - alias: "Turn on ceiling light"
        action: light.turn_on
        target:
          entity_id: light.ceiling
      - alias: "Notify that ceiling light is turned on"
        action: notify.notify
        data:
          message: "Turned on the ceiling light!"
```

All actions support an optional `alias` for documentation and debugging purposes.

---

## Performing Actions

### Basic Action Syntax

```yaml
- alias: "Bedroom lights on"
  action: light.turn_on
  target:
    entity_id: group.bedroom
  data:
    brightness: 100
```

### Targeting Areas and Devices

Use `target` to specify areas, devices, or entities (can be combined):

```yaml
action: light.turn_on
target:
  area_id: living_room
  device_id:
    - ff22a1889a6149c5ab6327a8236ae704
    - 52c050ca1a744e238ad94d170651f96b
  entity_id:
    - light.hallway
    - light.landing
```

### Passing Data to Actions

```yaml
action: light.turn_on
target:
  entity_id: group.living_room
data:
  brightness: 120
  rgb_color: [255, 0, 0]
```

### Templated Actions

Dynamically choose which action to perform:

```yaml
action: >
  {% if states('sensor.temperature') | float > 15 %}
    switch.turn_on
  {% else %}
    switch.turn_off
  {% endif %}
entity_id: switch.ac
```

### Templated Data

```yaml
action: thermostat.set_temperature
target:
  entity_id: >
    {% if is_state('device_tracker.paulus', 'home') %}
      thermostat.upstairs
    {% else %}
      thermostat.downstairs
    {% endif %}
data:
  temperature: "{{ 22 - distance(states.device_tracker.paulus) }}"
```

### Response Variables

Some actions return data that can be used in subsequent actions:

```yaml
action: calendar.get_events
target:
  entity_id: calendar.school
data:
  duration:
    hours: 24
response_variable: agenda

# Use the response in another action
action: notify.gmail_com
data:
  title: "Daily agenda for {{ now().date() }}"
  message: >-
    Your agenda for today:
    {% for event in agenda['calendar.school'].events %}
    {{ event.start}}: {{ event.summary }}
    {% endfor %}
```

### Scene Activation Shortcut

```yaml
- scene: scene.morning_living_room
```

### homeassistant Actions

- `homeassistant.turn_on` - Turns on an entity
- `homeassistant.turn_off` - Turns off an entity
- `homeassistant.toggle` - Toggles an entity
- `homeassistant.update_entity` - Request immediate entity update

---

## Variables

### Setting Variables

```yaml
- alias: "Set variables"
  variables:
    entities:
      - light.kitchen
      - light.living_room
    brightness: 100
- alias: "Control lights"
  action: light.turn_on
  target:
    entity_id: "{{ entities }}"
  data:
    brightness: "{{ brightness }}"
```

### Templated Variables

```yaml
- variables:
    blind_state_message: "The blind is {{ states('cover.blind') }}."
- action: notify.mobile_app_iphone
  data:
    message: "{{ blind_state_message }}"
```

### Variable Scope

Variables assigned in nested blocks (like `if/then`) update the outer scope:

```yaml
sequence:
  - variables:
      people: 0
  - if:
      - condition: state
        entity_id: device_tracker.paulus
        state: "home"
    then:
      - variables:
          people: "{{ people + 1 }}"
          paulus_home: true
  # people is now 1 if condition was true
  - action: notify.notify
    data:
      message: "There are {{ people }} people home"
```

---

## Conditions

Conditions stop script execution when they evaluate to false. Unlike triggers (which are OR), conditions are AND by default.

### Logical Conditions

#### AND Condition

```yaml
conditions:
  - condition: and
    conditions:
      - condition: state
        entity_id: "device_tracker.paulus"
        state: "home"
      - condition: numeric_state
        entity_id: "sensor.temperature"
        below: 20
```

Shorthand form:

```yaml
conditions:
  - and:
    - condition: state
      entity_id: "device_tracker.paulus"
      state: "home"
    - condition: numeric_state
      entity_id: "sensor.temperature"
      below: 20
```

#### OR Condition

```yaml
conditions:
  - condition: or
    conditions:
      - condition: state
        entity_id: "device_tracker.paulus"
        state: "home"
      - condition: numeric_state
        entity_id: "sensor.temperature"
        below: 20
```

Shorthand:

```yaml
conditions:
  - or:
    - condition: state
      entity_id: "device_tracker.paulus"
      state: "home"
    - condition: numeric_state
      entity_id: "sensor.temperature"
      below: 20
```

#### NOT Condition

```yaml
conditions:
  - condition: not
    conditions:
      - condition: state
        entity_id: device_tracker.paulus
        state: "home"
```

#### Mixed AND/OR

```yaml
conditions:
  - and:
    - condition: state
      entity_id: "device_tracker.paulus"
      state: "home"
    - or:
      - condition: state
        entity_id: sensor.weather_precip
        state: "rain"
      - condition: numeric_state
        entity_id: "sensor.temperature"
        below: 20
```

### Numeric State Condition

```yaml
conditions:
  - condition: numeric_state
    entity_id: sensor.temperature
    above: 17
    below: 25
```

With value template:

```yaml
conditions:
  - condition: numeric_state
    entity_id: sensor.temperature
    above: 17
    below: 25
    value_template: "{{ float(state.state) + 2 }}"
```

Multiple entities (all must match):

```yaml
conditions:
  - condition: numeric_state
    entity_id:
      - sensor.kitchen_temperature
      - sensor.living_room_temperature
    below: 18
```

Using helper entities for dynamic thresholds:

```yaml
conditions:
  - condition: numeric_state
    entity_id: climate.living_room_thermostat
    attribute: temperature
    above: input_number.temperature_threshold_low
    below: input_number.temperature_threshold_high
```

### State Condition

```yaml
conditions:
  - condition: state
    entity_id: device_tracker.paulus
    state: "not_home"
    for:
      hours: 1
      minutes: 10
```

Multiple entities (all must match):

```yaml
conditions:
  - condition: state
    entity_id:
      - light.kitchen
      - light.living_room
    state: "on"
```

Any entity matches:

```yaml
conditions:
  - condition: state
    entity_id:
      - binary_sensor.motion_sensor_left
      - binary_sensor.motion_sensor_right
    match: any
    state: "on"
```

Multiple possible states:

```yaml
conditions:
  - condition: state
    entity_id: alarm_control_panel.home
    state:
      - "armed_away"
      - "armed_home"
```

Using helper entities:

```yaml
conditions:
  - condition: state
    entity_id: alarm_control_panel.home
    state: input_select.guest_mode
```

### Sun Conditions

Sun state:

```yaml
conditions:
  - condition: state
    entity_id: sun.sun
    state: "above_horizon"  # or "below_horizon"
```

Sunset/sunrise with offset:

```yaml
conditions:
  - condition: sun
    after: sunset
    after_offset: "-01:00:00"
```

When dark:

```yaml
conditions:
  - condition: sun
    after: sunset
    before: sunrise
```

### Template Condition

```yaml
conditions:
  - condition: template
    value_template: "{{ (state_attr('device_tracker.iphone', 'battery_level')|int) > 50 }}"
```

Shorthand notation:

```yaml
conditions: "{{ (state_attr('device_tracker.iphone', 'battery_level')|int) > 50 }}"
```

Or in a list:

```yaml
conditions:
  - "{{ (state_attr('device_tracker.iphone', 'battery_level')|int) > 50 }}"
  - condition: state
    entity_id: alarm_control_panel.home
    state: armed_away
```

### Time Condition

```yaml
conditions:
  - condition: time
    after: "15:00:00"
    before: "02:00:00"
    weekday:
      - mon
      - wed
      - fri
```

Using helper entities:

```yaml
conditions:
  - condition: time
    after: input_datetime.house_silent_hours_start
    before: input_datetime.house_silent_hours_end
```

### Trigger Condition

```yaml
conditions:
  - condition: trigger
    id: event_trigger
```

Multiple triggers:

```yaml
conditions:
  - condition: trigger
    id:
      - event_1_trigger
      - event_2_trigger
```

### Zone Condition

```yaml
conditions:
  - condition: zone
    entity_id: device_tracker.paulus
    zone: zone.home
```

### Disabling Conditions

```yaml
conditions:
  - enabled: false
    condition: state
    entity_id: sun.sun
    state: "above_horizon"
```

---

## Delay

```yaml
# Seconds
- delay: 5

# HH:MM
- delay: "01:00"

# HH:MM:SS
- delay: "00:01:30"

# Named units
- delay:
    minutes: 1

# Templated
- delay: "{{ states('input_number.minute_delay') | multiply(60) | int }}"
```

---

## Wait Actions

### Wait for Template

```yaml
- wait_template: "{{ is_state('media_player.floor', 'stop') }}"
```

### Wait for Trigger

```yaml
- wait_for_trigger:
    - trigger: event
      event_type: MY_EVENT
    - trigger: state
      entity_id: light.LIGHT
      to: "on"
      for: 10
```

### Wait Timeout

```yaml
- wait_template: "{{ is_state('binary_sensor.entrance', 'on') }}"
  timeout: "00:01:00"
```

Abort on timeout:

```yaml
- wait_for_trigger:
    - trigger: event
      event_type: ifttt_webhook_received
  timeout:
    minutes: "{{ timeout_minutes }}"
  continue_on_timeout: false
```

### Wait Variable

After wait completes, `wait` variable contains:
- `wait.completed` - true if condition was met
- `wait.remaining` - timeout remaining, or none
- `wait.trigger` - trigger info (for wait_for_trigger only)

```yaml
- wait_template: "{{ is_state('binary_sensor.door', 'on') }}"
  timeout: 10
- if:
    - "{{ not wait.completed }}"
  then:
    - action: script.door_did_not_open
```

---

## Fire an Event

```yaml
- event: LOGBOOK_ENTRY
  event_data:
    name: Paulus
    message: is waking up
    entity_id: device_tracker.paulus
    domain: light
```

Custom events with templates:

```yaml
- event: MY_EVENT
  event_data:
    name: myEvent
    customData: "{{ myCustomVariable }}"
```

---

## Repeat Actions

### Counted Repeat

```yaml
repeat:
  count: "{{ count|int * 2 - 1 }}"
  sequence:
    - delay: 2
    - action: light.toggle
      target:
        entity_id: "light.{{ light }}"
```

### For Each

```yaml
repeat:
  for_each:
    - "living_room"
    - "kitchen"
    - "office"
  sequence:
    - action: light.turn_off
      target:
        entity_id: "light.{{ repeat.item }}"
```

With mappings:

```yaml
repeat:
  for_each:
    - language: English
      message: Hello World
    - language: Dutch
      message: Hallo Wereld
  sequence:
    - action: notify.phone
      data:
        title: "Message in {{ repeat.item.language }}"
        message: "{{ repeat.item.message }}!"
```

### While Loop

```yaml
repeat:
  while:
    - condition: state
      entity_id: input_boolean.do_something
      state: "on"
    - condition: template
      value_template: "{{ repeat.index <= 20 }}"
  sequence:
    - action: script.something
```

Shorthand:

```yaml
repeat:
  while: "{{ is_state('sensor.mode', 'Home') and repeat.index < 10 }}"
  sequence:
    - ...
```

### Repeat Until

```yaml
repeat:
  sequence:
    - action: shell_command.turn_something_on
    - delay:
        milliseconds: 200
  until:
    - condition: state
      entity_id: binary_sensor.something
      state: "on"
```

### Repeat Loop Variable

Available inside repeat:
- `repeat.first` - true during first iteration
- `repeat.index` - iteration number (1, 2, 3, ...)
- `repeat.last` - true during last iteration (counted loops only)
- `repeat.item` - current item (for_each only)

---

## If-Then-Else

```yaml
- if:
    - condition: state
      entity_id: zone.home
      state: 0
  then:
    - action: vacuum.start
      target:
        area_id: living_room
  else:
    - action: notify.notify
      data:
        message: "Skipped cleaning, someone is home!"
```

---

## Choose Action

```yaml
- choose:
    # IF morning
    - conditions:
        - condition: template
          value_template: "{{ now().hour < 9 }}"
      sequence:
        - action: script.sim_morning
    # ELIF day
    - conditions:
        - condition: template
          value_template: "{{ now().hour < 18 }}"
      sequence:
        - action: script.sim_day
  # ELSE night
  default:
    - action: light.turn_off
      target:
        entity_id: all
```

With template shorthand:

```yaml
- choose:
    - conditions: "{{ trigger.to_state.state == 'Home' }}"
      sequence:
        - action: script.arrive_home
```

---

## Grouping Actions (Sequence)

```yaml
- alias: "Turn on devices"
  sequence:
    - action: light.turn_on
      target:
        entity_id: light.ceiling
    - action: siren.turn_on
      target:
        entity_id: siren.noise_maker
```

---

## Parallel Actions

```yaml
- parallel:
    - action: notify.person1
      data:
        message: "These messages are sent at the same time!"
    - action: notify.person2
      data:
        message: "These messages are sent at the same time!"
```

With nested sequences:

```yaml
- parallel:
    - sequence:
        - wait_for_trigger:
            - trigger: state
              entity_id: binary_sensor.motion
              to: "on"
        - action: notify.person1
          data:
            message: "This message awaited the motion trigger"
    - action: notify.person2
      data:
        message: "I am sent immediately!"
```

**Caveats:**
- No order guarantee for completion
- If one action fails, others continue running
- Variables can conflict between parallel actions

---

## Stopping Scripts

```yaml
- stop: "Stop running the rest of the sequence"
```

With response:

```yaml
- stop: "Stop running the rest of the sequence"
  response_variable: "my_response_variable"
```

With error:

```yaml
- stop: "Well, that was unexpected!"
  error: true
```

---

## Continue on Error

```yaml
- alias: "If this one fails..."
  continue_on_error: true
  action: notify.super_unreliable_service_provider
  data:
    message: "I'm going to error out..."

- alias: "This one will still run!"
  action: persistent_notification.create
  data:
    title: "Hi there!"
    message: "I'm fine..."
```

---

## Disabling Actions

```yaml
- enabled: false
  alias: "This action will not run"
  action: notify.notify
  data:
    message: "Turning on the ceiling light!"
```

---

## Conversation Response

For voice assistant integrations:

```yaml
- variables:
    my_var: "123"
- set_conversation_response: "{{ 'Testing ' + my_var }}"
```

Clear response:

```yaml
- set_conversation_response: ~
```

---

## Best Practices

1. **Use aliases** - Add descriptive aliases to all actions for better debugging and traces
2. **Template conditions** - Use shorthand template conditions for simple checks
3. **Response variables** - Leverage response_variable for actions that return data
4. **Error handling** - Use `continue_on_error` for non-critical actions
5. **Parallel wisely** - Only use parallel when order doesn't matter and actions are independent
6. **Variable scope** - Be aware that variables in nested blocks update outer scope
7. **Wait timeouts** - Always consider adding timeouts to wait actions
8. **Condition placement** - Place conditions early in sequences to avoid unnecessary action execution

---

## References

- [Scripts](https://www.home-assistant.io/docs/scripts/) - Official Home Assistant script syntax documentation
- [Perform Actions](https://www.home-assistant.io/docs/scripts/perform-actions/) - Guide for performing actions in scripts and automations
- [Conditions](https://www.home-assistant.io/docs/scripts/conditions/) - Complete reference for all condition types
