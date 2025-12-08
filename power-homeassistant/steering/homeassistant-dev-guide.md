# Home Assistant Development Guidelines

This steering file provides comprehensive guidelines for writing Home Assistant YAML configurations, templates, and automations. It is automatically loaded when you're working on Home Assistant configuration files.

**What's covered:**
- YAML syntax and style conventions
- Template notation and best practices
- File organization patterns
- Home Assistant-specific configuration standards
- Automation and sensor design patterns

**When this loads:**
- Creating or modifying automations, scripts, or dashboards
- Writing YAML configuration files
- Working with templates and sensors
- Organizing Home Assistant configuration

## YAML Style Guide

These guidelines are based on the official Home Assistant YAML Style Guide ([developers.home-assistant.io/docs/documenting/yaml-style-guide/](https://developers.home-assistant.io/docs/documenting/yaml-style-guide/)).

### Indentation

- Use **two spaces** for indentation
- Do not use tabs
- Nested elements should be indented two spaces from their parent element

```yaml
# Good
automation:
  - alias: "Good example"
    triggers:
      - trigger: state
        entity_id: binary_sensor.motion
```

### Boolean Values

- Use `true` and `false` for boolean values (lowercase)
- Do not use abbreviations like `yes`/`no` or `on`/`off` (for compatibility with YAML 1.2 specification)

```yaml
# Good
one: true
two: false

# Bad
one: True
two: on
three: yes
```

### Comments

- Align comments with the indentation level
- Comments should preferably be written above the target line
- Comments should start with a capital letter and have a space after `#`

```yaml
# Good
example:
  # Comment
  one: true

# Acceptable, but prefer the above
example:
  one: true # Comment
```

### Sequences (Lists)

- Prefer block-style sequences
- Avoid flow-style (`[1, 2, 3]`) whenever possible

#### Block-Style Sequences

```yaml
# Good
example:
  - 1
  - 2
  - 3

# Bad
example:
- 1
- 2
- 3
```

#### Flow-Style Sequences

When using flow-style, add a space after commas and no spaces before/after opening/closing brackets:

```yaml
# Good
example: [1, 2, 3]

# Bad
example: [ 1,2,3 ]
example: [ 1, 2, 3 ]
```

### Mappings

- Use block-style mappings only
- Do not use flow-style (JSON-like notation)

```yaml
# Good
example:
  one: 1
  two: 2

# Bad
example: { one: 1, two: 2 }
```

### Null Values

- Express null values implicitly
- Avoid explicit null values (`~` or `null`)

```yaml
# Good
example:

# Bad
example: ~
example: null
```

### Strings

- Strings should preferably be enclosed in **double quotes** (`"`)

```yaml
# Good
example: "Hi there!"

# Avoid
example: Hi there!

# Bad
example: 'Hi there!'
```

#### Multi-line Strings

Avoid using line break indicators like `\n` or long single-line strings. Instead, use literal style (preserves line breaks) and folded style (does not preserve line breaks) multi-line strings.

```yaml
# Good
literal_example: |
  This example is an example of literal block scalar style in YAML.
  It allows you to split a string into multiple lines.
folded_example: >
  This example is an example of a folded block scalar style in YAML.
  It allows you to split a string into multi lines, however, it magically
  removes all the new lines placed in your YAML.

# Bad
literal_example: "This example is an example of literal block scalar style in YAML.\nIt allows you to split a string into multiple lines.\n"
folded_example_same_as: "This example is an example of a folded block scalar style in YAML. It allows you to split a string into multi lines, however, it magically removes all the new lines placed in your YAML.\n"
```

Generally, use operators that do not specify trailing newline handling (`|`, `>`). Only use strip operators (`|-`, `>-`: removes trailing newlines) or keep operators (`|+`, `>+`: preserves trailing newlines) when special handling is needed.

## Home Assistant YAML

### Default Values

If a configuration option uses its default value, do not include that option in examples, unless you are explaining that specific option.

For example, the `condition` option in automations is optional and defaults to an empty list `[]`.

```yaml
# Good
- alias: "Test"
  triggers:
    - trigger: state
      entity_id: binary_sensor.motion

# Bad
- alias: "Test"
  triggers:
    - trigger: state
      entity_id: binary_sensor.motion
  condition: []
```

### Strings (Continued)

As mentioned earlier, strings should preferably be enclosed in double quotes, but the following value types are exceptions and quotes can be omitted for better readability:

- Entity IDs (for example, `binary_sensor.motion`)
- Entity attributes (for example, `temperature`)
- Device IDs
- Area IDs
- Platform types (for example, `light`, `switch`)
- Condition types (for example, `numeric_state`, `state`)
- Trigger types (for example, `state`, `time`)
- Action names (for example, `light.turn_on`)
- Device classes (for example, `problem`, `motion`)
- Event names
- Hardcoded values that accept only limited values (for example, automation `mode`)

```yaml
# Good
actions:
  - action: notify.frenck
    data:
      message: "Hi there!"
  - action: light.turn_on
    target:
      entity_id: light.office_desk
      area_id: living_room
    data:
      transition: 10

# Bad
actions:
  - action: "notify.frenck"
    data:
      message: Hi there!
```

### Service Action Targets

When executing a service action on an entity ID, there are three methods:

1. Specify as an action-level property
2. Send as part of the service action call data
3. Specify as an entity in the service action target

Service action targets are the most modern method and the most flexible and recommended approach, as they can target entities, devices, and areas.

```yaml
# Good
actions:
  - action: light.turn_on
    target:
      entity_id: light.living_room
  - action: light.turn_on
    target:
      area_id: living_room
      entity_id: light.office_desk
      device_id: 21349287492398472398

# Bad
actions:
  - action: light.turn_on
    entity_id: light.living_room
  - action: light.turn_on
    data:
      entity_id: light.living_room
```

### Properties Accepting Scalar Values or Lists of Scalar Values

For properties that accept a single value or a list of scalar values, the following rules apply:

- Do not specify multiple values as a single comma-separated string
- Use block-style when using lists
- Do not use a list for a single value
- Using a single scalar value is permitted

```yaml
# Good
entity_id: light.living_room
entity_id:
  - light.living_room
  - light.office

# Bad
entity_id: light.living_room, light.office
entity_id: [light.living_room, light.office]
entity_id:
  - light.living_room
```

### Properties Accepting Mappings or Lists of Mappings

Properties like `condition`, `action`, and `sequence` accept a single mapping or a list of mappings.

In such cases, use a list of mappings even when passing a single mapping. This makes it easier to understand that items can be added and makes it easier to copy and paste single items into your own code.

```yaml
# Good
actions:
  - action: light.turn_on
    target:
      entity_id: light.living_room

# Bad
actions:
  action: light.turn_on
  target:
    entity_id: light.living_room
```

## Templates

Home Assistant templates are powerful but can be confusing or difficult to understand for less experienced users. Therefore, avoid using templates when a pure YAML version is available.

```yaml
# Good
conditions:
  - condition: numeric_state
    entity_id: sun.sun
    attribute: elevation
    below: 4

# Bad
conditions:
  - condition: template
    value_template: "{{ state_attr('sun.sun', 'elevation') < 4 }}"
```

### Quoting Style

Since templates are strings, enclose them in double quotes. As a result, use single quotes within templates.

```yaml
# Good
example: "{{ 'some_value' == some_other_value }}"

# Bad
example: '{{ "some_value" == some_other_value }}'
```

### Template String Length

Avoid long lines in templates and split them into multiple lines to clarify what is happening and improve readability.

```yaml
# Good
value_template: >-
  {{
    is_state('sensor.bedroom_co_status', 'Ok')
    and is_state('sensor.kitchen_co_status', 'Ok')
    and is_state('sensor.wardrobe_co_status', 'Ok')
  }}

# Bad
value_template: "{{ is_state('sensor.bedroom_co_status', 'Ok') and is_state('sensor.kitchen_co_status', 'Ok') and is_state('sensor.wardrobe_co_status', 'Ok') }}"
```

### Short-Style Condition Syntax

Prefer shorthand templates over expressive forms. They provide more concise syntax.

```yaml
# Good
conditions: "{{ some_value == some_other_value }}"

# Bad
conditions:
  - condition: template
    value_template: "{{ some_value == some_other_value }}"
```

### Filters

Spaces are required around the filter pipe marker `|`. If this makes readability unclear, using additional parentheses is recommended.

```yaml
# Good
conditions:
  - "{{ some_value | float }}"
  - "{{ some_value == (some_other_value | some_filter) }}"

# Bad
conditions:
  - "{{ some_value == some_other_value|some_filter }}"
  - "{{ some_value == (some_other_value|some_filter) }}"
```

### Accessing States and State Attributes

Do not use the states object directly when helper methods are available.

For example, use `states('sensor.temperature')` instead of `states.sensor.temperature.state`.

```yaml
# Good
one: "{{ states('sensor.temperature') }}"
two: "{{ state_attr('climate.living_room', 'temperature') }}"

# Bad
one: "{{ states.sensor.temperature.state }}"
two: "{{ states.climate.living_room.attributes.temperature }}"
```

This applies to `states()`, `is_state()`, `state_attr()`, and `is_state_attr()` to avoid errors and error messages when entities are not yet ready (for example, during Home Assistant startup).


### Using Functions and Filters

In Home Assistant templates, prefer functions over filters. Functions are more explicit and improve code readability and maintainability.

#### Recommended Method (Using Functions)

```yaml
# Use function for integer conversion
state: "{{ int(state_attr('sensor.example', 'value')) }}"

# Use function for float conversion
state: "{{ float(states('sensor.temperature')) }}"

# Use function for string replacement
state: "{{ state_attr('sensor.example', 'text') | replace('%', '') }}"
```

#### Method to Avoid (Using Filters Only)

```yaml
# Using filter for integer conversion (avoid)
state: "{{ state_attr('sensor.example', 'value') | int }}"

# Using filter for float conversion (avoid)
state: "{{ states('sensor.temperature') | float }}"
```

### Template Structure

- Use multi-line format (`>-`) for templates exceeding 100 characters
- Use single-line format (`"{{ ... }}"`) for templates not exceeding 100 characters
- Properly indent conditional statements within templates

### Sensor Definitions

- Set `unique_id` for all sensors
- Set appropriate `device_class` and `state_class`
- Set appropriate `unit_of_measurement` for measurements

## Development Best Practices

### Automation Design

1. **Clear Naming**: Use descriptive names that explain the automation's purpose
   ```yaml
   # Good
   - alias: "Porch Light at Sunset"

   # Bad
   - alias: "Auto 1"
   ```

2. **Use Conditions Wisely**: Add conditions to prevent false triggers
   ```yaml
   conditions:
     - condition: state
       entity_id: binary_sensor.someone_home
       state: "on"
   ```

3. **Incremental Building**: Start simple and gradually add complexity
4. **Test Thoroughly**: Use Developer Tools → Services to test actions before adding to automations
5. **Document Complex Logic**: Add comments explaining non-obvious behavior

### Dashboard Management

1. **Organize by Area**: Group cards by room or area for better navigation
2. **Appropriate Card Types**: Choose the best card type for each use case
   - Use `entities` card for simple lists
   - Use `glance` card for quick status overview
   - Use `picture-elements` for custom layouts
3. **Responsive Design**: Ensure layouts work well on mobile devices
4. **Consistent Styling**: Use themes for consistent appearance

### Template Best Practices

1. **Explicit Type Conversion**: Perform explicit type conversion before manipulating values
   ```yaml
   # Good
   state: "{{ int(states('sensor.value')) + 10 }}"

   # Bad
   state: "{{ states('sensor.value') + 10 }}"
   ```

2. **Error Handling**: Set default values for cases where values do not exist
   ```yaml
   state: "{{ float(states('sensor.temperature'), 0) }}"
   ```

3. **Comments**: Add comments to complex templates
4. **Naming Conventions**: Follow consistent naming conventions for sensor names
5. **Reusability**: Design common templates in a reusable form

## Examples

### Good Example

```yaml
- template:
  - sensor:
      - unique_id: sensor.outside_temperature
        name: "Outside Temperature"
        state: "{{ float(states('sensor.weather_temperature')) }}"
        unit_of_measurement: "°C"
        device_class: temperature
        state_class: measurement
```

### Example to Avoid

```yaml
- platform: template
  sensors:
    outside_temperature:
      friendly_name: "Outside Temperature"
      value_template: "{{ states('sensor.weather_temperature') | float(0) }}"
      unit_of_measurement: "°C"
```

## File Organization

### Splitting Files by Component

- Split files by related components
- Organize files by logical groups
- Split large files into smaller, manageable files

```
homeassistant/
  sensors.yaml      # Sensor definitions
  templates.yaml    # Template sensors
  automations.yaml  # Automations
  scripts.yaml      # Scripts
  scenes.yaml       # Scenes
```

These files are loaded from `configuration.yaml` as follows:

```yaml
sensor: !include sensors.yaml
shell_command: !include shell_commands.yaml
switch: !include switches.yaml
template: !include templates.yaml
```

Therefore, split files need to omit the leading key.

#### Good Example

```yaml
- platform: statistics
  unique_id: sensor.balcony_humidity_difference_average
  name: "Balcony Humidity Difference Average"
  entity_id: sensor.balcony_humidity_difference
  state_characteristic: mean
  max_age:
    hours: 51
- platform: statistics
  unique_id: sensor.balcony_temp_average
  name: "Balcony Temperature Average"
  entity_id: sensor.balcony_temperature
  state_characteristic: mean
  max_age:
    hours: 24
```

#### Bad Example

```yaml
sensor:
  - platform: statistics
    unique_id: sensor.balcony_humidity_difference_average
    name: "Balcony Humidity Difference Average"
    entity_id: sensor.balcony_humidity_difference
    state_characteristic: mean
    max_age:
      hours: 51
  - platform: statistics
    unique_id: sensor.balcony_temp_average
    name: "Balcony Temperature Average"
    entity_id: sensor.balcony_temperature
    state_characteristic: mean
    max_age:
      hours: 24
```

### Comments

- Use comments at the beginning of sections to explain content
- Add explanatory comments to complex configurations
- Add a space after `#` in comments

```yaml
# Balcony sensors
- platform: template
  sensors:
    balcony_temperature:
      friendly_name: "Balcony Temperature"
      # Average of two temperature sensors for better accuracy
      value_template: >-
        {{ (float(states('sensor.balcony_temp_1')) +
            float(states('sensor.balcony_temp_2'))) / 2 }}
```

### Naming Conventions

- Use snake_case (lowercase and underscores) for entity IDs
- Use consistent prefixes for related entities
- Names should be specific and descriptive

```yaml
# Good
sensor:
  - platform: template
    sensors:
      living_room_temperature:
        friendly_name: "Living Room Temperature"
      living_room_humidity:
        friendly_name: "Living Room Humidity"
```

## Configuration Best Practices

### Reusable Configuration

- Design common configurations in a reusable form
- Leverage the package feature to group related configurations

### Version Control

- Manage configuration files with a version control system (such as Git)
- Track change history and enable rollback to previous states when issues occur

### Testing

- Use Home Assistant's configuration check feature to validate after configuration changes
- Create backups before making major changes

### Security

- Protect sensitive information (passwords, tokens, and API keys) using `!secret`
- Do not include sensitive information in public repositories

```yaml
# Good
http:
  api_password: !secret http_password
```
