# Home Assistant Templating Guide

This steering file provides guidance for working with Home Assistant templates using Jinja2.

## Overview

Home Assistant uses Jinja2 templating engine for:
- Formatting outgoing messages (notify, Alexa)
- Processing incoming data (MQTT, REST, command_line sensors)
- Automation templating
- Template sensors and binary sensors

## Template Syntax Basics

### Delimiters

- `{{ ... }}` - Expressions to output values
- `{% ... %}` - Statements (control structures)
- `{# ... #}` - Comments (not included in output)

### YAML Integration Rules

1. Single-line templates MUST be surrounded by quotes (`"` or `'`)
2. Use `default` filter or `is not none` for undefined variables
3. Convert numbers with `float` or `int` filters before comparison
4. Use multiline strings (`>` or `|`) for complex templates

```yaml
# Single-line template (quotes required)
value_template: "{{ states('sensor.temperature') | float }}"

# Multiline template
message: >
  {% if is_state('device_tracker.phone', 'home') %}
    Welcome home!
  {% else %}
    Away from home
  {% endif %}
```

## Home Assistant State Functions

### Core State Functions

```jinja
{# Get state value (recommended) #}
{{ states('sensor.temperature') }}

{# Get state with formatting #}
{{ states('sensor.temperature', with_unit=True) }}
{{ states('sensor.temperature', rounded=True) }}

{# Check state #}
{{ is_state('light.kitchen', 'on') }}
{{ is_state('light.kitchen', ['on', 'unavailable']) }}

{# Get attribute #}
{{ state_attr('light.kitchen', 'brightness') }}

{# Check attribute #}
{{ is_state_attr('light.kitchen', 'brightness', 255) }}

{# Check if entity has valid value #}
{{ has_value('sensor.temperature') }}
```

### Avoid Direct State Access

```jinja
{# BAD - may cause errors during startup #}
{{ states.sensor.temperature.state }}

{# GOOD - handles undefined gracefully #}
{{ states('sensor.temperature') }}
```

## Iterating States

```jinja
{# All entities in a domain #}
{% for state in states.sensor %}
  {{ state.entity_id }}: {{ state.state }}
{% endfor %}

{# Sorted by entity_id #}
{% for state in states.sensor | sort(attribute='entity_id') %}
  {{ state.entity_id }}
{% endfor %}

{# Filter entities that are on #}
{{ ['light.kitchen', 'light.bedroom'] | select('is_state', 'on') | list }}
```

## Time Functions

```jinja
{# Current time #}
{{ now() }}
{{ utcnow() }}

{# Time components #}
{{ now().hour }}
{{ now().minute }}
{{ now().weekday() }}

{# Today at specific time #}
{{ today_at("10:30") }}
{{ now() > today_at("22:00") }}

{# Time calculations #}
{{ now() - timedelta(hours=1, minutes=30) }}

{# Relative time (human readable) #}
{{ relative_time(states.binary_sensor.door.last_changed) }}

{# Time since/until #}
{{ time_since(states.sensor.last_update.last_changed) }}
{{ time_until(as_datetime(states('sensor.next_event'))) }}
```

### Timestamp Conversions

```jinja
{# String to datetime #}
{{ as_datetime('2024-01-15T10:30:00') }}
{{ states('sensor.expires') | as_datetime }}

{# Datetime to timestamp #}
{{ as_timestamp(now()) }}

{# Timestamp to string #}
{{ 1704067200 | timestamp_local }}
{{ 1704067200 | timestamp_custom('%Y-%m-%d %H:%M') }}

{# Parse custom format #}
{{ strptime('2024-01-15', '%Y-%m-%d') }}
```

## Area, Device, and Entity Functions

### Areas

```jinja
{# List all areas #}
{{ areas() }}

{# Get area ID/name #}
{{ area_id('Living Room') }}
{{ area_name('sensor.temperature') }}

{# Get entities/devices in area #}
{{ area_entities('kitchen') }}
{{ area_devices('kitchen') }}
```

### Devices

```jinja
{# Get device info #}
{{ device_id('sensor.temperature') }}
{{ device_name('sensor.temperature') }}
{{ device_attr('sensor.temperature', 'manufacturer') }}

{# Get entities for device #}
{{ device_entities('deadbeefdeadbeef') }}
```

### Floors and Labels

```jinja
{# Floors #}
{{ floors() }}
{{ floor_id('First floor') }}
{{ floor_areas('first_floor') }}

{# Labels #}
{{ labels() }}
{{ label_entities('security') }}
```

## Groups and Expand

```jinja
{# Expand group to individual entities #}
{% for entity in expand('group.all_lights') %}
  {{ entity.entity_id }}: {{ entity.state }}
{% endfor %}

{# Expand with filtering #}
{{ expand('group.all_lights') 
   | selectattr('state', 'eq', 'on') 
   | list }}

{# Multiple sources #}
{{ expand(['group.lights', 'light.kitchen']) }}
```

## Numeric Operations

### Type Conversion

```jinja
{# Convert to float/int #}
{{ states('sensor.temp') | float }}
{{ states('sensor.count') | int }}

{# With default value #}
{{ states('sensor.temp') | float(0) }}
{{ states('sensor.count') | int(default=0) }}

{# Check if number #}
{{ is_number(states('sensor.temp')) }}
{{ states('sensor.temp') | is_number }}
```

### Math Functions

```jinja
{# Basic math #}
{{ states('sensor.temp') | float + 5 }}
{{ (states('sensor.temp') | float * 1.8) + 32 }}

{# Rounding #}
{{ states('sensor.temp') | float | round(1) }}
{{ states('sensor.temp') | float | round(0, 'floor') }}
{{ states('sensor.temp') | float | round(0, 'ceil') }}

{# Min/Max/Average #}
{{ [1, 2, 3, 4, 5] | max }}
{{ [1, 2, 3, 4, 5] | min }}
{{ [1, 2, 3, 4, 5] | average }}

{# Clamp value to range #}
{{ value | clamp(0, 100) }}

{# Trigonometry #}
{{ sin(1.5708) }}
{{ cos(0) }}
{{ sqrt(16) }}
```

## String Operations

```jinja
{# Case conversion #}
{{ 'hello' | upper }}
{{ 'HELLO' | lower }}
{{ 'hello world' | title }}
{{ 'hello world' | capitalize }}

{# Trimming #}
{{ '  hello  ' | trim }}

{# Replace #}
{{ 'hello world' | replace('world', 'home') }}

{# Split/Join #}
{{ 'a,b,c' | split(',') }}
{{ ['a', 'b', 'c'] | join(', ') }}

{# Regex #}
{{ 'sensor_123' | regex_search('\\d+') }}
{{ 'hello123' | regex_replace('\\d+', '') }}
```

## Conditional Logic

### If Statements

```jinja
{% if is_state('light.kitchen', 'on') %}
  Kitchen light is on
{% elif is_state('light.kitchen', 'off') %}
  Kitchen light is off
{% else %}
  Kitchen light is unavailable
{% endif %}
```

### Immediate If (iif)

```jinja
{# Simple ternary #}
{{ iif(is_state('light.kitchen', 'on'), 'On', 'Off') }}

{# As filter #}
{{ is_state('light.kitchen', 'on') | iif('On', 'Off') }}

{# With none handling #}
{{ iif(condition, 'true_val', 'false_val', 'none_val') }}
```

### Default Values

```jinja
{# Provide default for undefined #}
{{ states('sensor.unknown') | default('N/A') }}

{# Check if defined #}
{% if variable is defined %}
  {{ variable }}
{% endif %}
```

## JSON Handling

```jinja
{# Object to JSON string #}
{{ {'temp': 25, 'unit': '°C'} | to_json }}
{{ {'temp': 25} | to_json(pretty_print=True) }}

{# JSON string to object #}
{% set data = '{"temp": 25}' | from_json %}
{{ data.temp }}
```

## Distance and Location

```jinja
{# Distance from home #}
{{ distance('device_tracker.phone') }}

{# Distance between entities #}
{{ distance('device_tracker.phone', 'zone.work') }}

{# Closest entity #}
{{ closest(states.device_tracker) }}
{{ closest('zone.home', states.device_tracker) }}
```

## Reusable Templates

Create reusable templates in `config/custom_templates/`:

```jinja
{# custom_templates/helpers.jinja #}
{% macro format_temp(entity_id) %}
  {{ states(entity_id) | float | round(1) }}°C
{% endmacro %}
```

Use in automations:

```jinja
{% from 'helpers.jinja' import format_temp %}
Temperature: {{ format_temp('sensor.temperature') }}
```

## Common Patterns

### Safe State Access

```jinja
{% set temp = states('sensor.temperature') %}
{% if temp not in ['unknown', 'unavailable'] and is_number(temp) %}
  {{ temp | float | round(1) }}°C
{% else %}
  Temperature unavailable
{% endif %}
```

### Entity Filtering

```jinja
{# Lights that are on #}
{% set on_lights = states.light 
   | selectattr('state', 'eq', 'on') 
   | list %}
{{ on_lights | length }} lights are on

{# Sensors with specific attribute #}
{{ states.sensor 
   | selectattr('attributes.device_class', 'defined')
   | selectattr('attributes.device_class', 'eq', 'temperature')
   | list }}
```

### Aggregating Values

```jinja
{# Sum of power sensors #}
{% set power_sensors = [
  'sensor.plug1_power',
  'sensor.plug2_power',
  'sensor.plug3_power'
] %}
{{ power_sensors 
   | map('states') 
   | map('float', default=0) 
   | sum | round(1) }} W
```

## Template Editor

Use Developer Tools > Template to test templates:
1. Navigate to Settings > Developer Tools > Template
2. Write template in editor
3. See real-time results on the right

## Best Practices

1. Always use `states()` function instead of `states.domain.entity.state`
2. Handle `unknown` and `unavailable` states explicitly
3. Use `| float(0)` or `| int(0)` with default values
4. Test templates in Developer Tools before using in automations
5. Use multiline YAML syntax for complex templates
6. Create reusable macros for repeated patterns
7. Use `has_value()` to check entity availability

## Resources

- [Home Assistant Templating Documentation](https://www.home-assistant.io/docs/configuration/templating/)
- [Jinja2 Template Designer Documentation](https://jinja.palletsprojects.com/en/stable/templates/)
- [Template Editor](https://my.home-assistant.io/redirect/developer_template)
