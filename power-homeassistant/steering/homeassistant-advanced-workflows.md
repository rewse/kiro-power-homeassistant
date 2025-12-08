# Advanced Workflows - Extended Examples

This guide provides additional workflow examples for specialized Home Assistant tasks. For basic workflows, see POWER.md.

## Smart Climate System

### Build Smart Climate System (MCP + Steering)

```
User: Create a room-by-room climate system that follows my presence
Agent: Loads homeassistant-smart-climate-guide.md for modular patterns
       Creates helpers:
       - input_select for room modes (Present/Absent/Vent)
       - input_number for target temperatures per mode
       - input_number for computed target temperature
       Uses ha_config_set_input_select and ha_config_set_input_number
       Creates 3 automation chain:
       1. Inputs → Modes (presence + window sensors drive mode)
       2. Modes → Target Temperature (mode determines temperature)
       3. Target Temperature → Climate Device (with 1-hour override reset)
       Uses ha_config_set_automation to deploy each automation
       Confirms system is working with ha_get_entity_state
```

### Add Climate Override (MCP + Steering)

```
User: I want to manually override my thermostat and have it reset after an hour
Agent: Loads homeassistant-smart-climate-guide.md for override pattern
       Explains the pattern:
       - Manual adjustment changes climate device temperature
       - Automation triggers after 1 hour of stable temperature
       - Re-applies mode's target temperature (resets override)
       Creates/updates climate automation with:
       - State trigger on climate entity temperature attribute
       - for: hours: 1 duration
       Uses ha_config_set_automation to deploy
```

### Create Bathroom Boost Mode (MCP + Steering)

```
User: Heat my bathroom automatically when I take a shower
Agent: Loads homeassistant-smart-climate-guide.md for room variations
       Creates helpers:
       - input_select with Boost mode added
       - input_number for boost temperature (e.g., 22°C)
       Updates mode automation with humidity trigger:
       - numeric_state trigger above 70% humidity
       - Sets mode to Boost
       Uses ha_config_set_automation to deploy
       Confirms with ha_get_entity_state on humidity sensor
```

### Set Up Bedroom Sleep Mode (MCP + Steering)

```
User: Lower my bedroom temperature automatically at night when I'm in bed
Agent: Loads homeassistant-smart-climate-guide.md for room variations
       Creates helpers:
       - input_select with Sleep mode added
       - input_number for sleep temperature (e.g., 17°C)
       Updates mode automation with time-based condition:
       - Time between 22:00 and 07:00
       - Presence sensor active
       Uses ha_config_set_automation to deploy
```

### Adapt Climate System for Cooling (MCP + Steering)

```
User: Convert my heating automation to work with air conditioning
Agent: Loads homeassistant-smart-climate-guide.md for cooling adaptation
       Explains temperature adjustments:
       - Present: 24°C (comfortable cooling)
       - Absent: 28°C (allow warmer when away)
       - Vent: Turn off AC when windows open
       Updates automations with cooling-specific logic
       Uses ha_config_set_automation to deploy
```

### Build Adaptive Lighting System (MCP + Steering)

```
User: Adjust my lights color temperature based on time of day and outdoor brightness
Agent: Loads homeassistant-smart-climate-guide.md for modular patterns
       Creates helpers using same Input → Mode → Output chain:
       - input_select for lighting modes (Day/Evening/Night/Bright)
       - input_number for color_temp and brightness per mode
       - input_number for computed color_temp and brightness
       Creates 3 automation chain:
       1. Inputs → Modes (sun elevation + illuminance sensor drive mode)
       2. Modes → Light Settings (mode determines color_temp/brightness)
       3. Light Settings → Bulbs (apply to light entities)
       Uses ha_config_set_automation to deploy each automation
       Confirms with ha_get_entity_state on light entities
```

## Voice Assistant (Assist)

### Extend Voice Assistant with Custom Commands (MCP + Steering)

```
User: Add a custom voice command to raise my standing desk
Agent: Loads homeassistant-tips-and-tricks.md for sentence trigger patterns
       Creates automation with:
       - Sentence trigger with multiple phrases ("Raise the desk", "Desk up")
       - Action to control the desk entity
       Uses ha_config_set_automation to deploy
       Explains how to test with Assist
```

### Chain Voice Commands with Wildcards (MCP + Steering)

```
User: Let me say "Turn on the lights and close the blinds" as one command
Agent: Loads homeassistant-tips-and-tricks.md for wildcard patterns
       Creates automation with:
       - Sentence trigger: "{first_command} and {second_command}"
       - conversation.process actions for each slot
       Explains recursive matching behavior
       Uses ha_config_set_automation to deploy
       Warns about unintentional triggers with simple patterns
```

## Dashboard Organization

### Organize Dashboard with Subviews (MCP + Steering)

```
User: Create a dashboard with room navigation using tiles
Agent: Loads homeassistant-tips-and-tricks.md for subview patterns
       Uses ha_config_get_dashboard to read current config
       Creates structure:
       - Main view with Tile cards for navigation
       - Subviews for each room (subview: true)
       - Tile tap_action with navigate to subview path
       Uses ha_config_set_dashboard to update
```

## Automation Patterns

### Test Automation with Fake State Changes (Steering only)

```
User: How can I test my temperature-based automation without waiting for weather?
Agent: Loads homeassistant-tips-and-tricks.md for testing techniques
       Explains Developer Tools → States workflow:
       - Find the entity to modify
       - Change state or attributes temporarily
       - Automation triggers as if real change occurred
       Warns to restore real values after testing
```

### Create Dynamic Scene for Temporary State Capture (MCP + Steering)

```
User: Save my current light settings before movie mode and restore after
Agent: Loads homeassistant-tips-and-tricks.md for dynamic scene patterns
       Creates automation with:
       - scene.create action to capture current state
       - Movie mode settings applied
       - Second automation to restore with scene.turn_on
       Notes that scene.create scenes don't survive restart
       Uses ha_config_set_automation to deploy
```

### Group Automations with Master Switch (MCP + Steering)

```
User: Create a single switch to enable/disable all my vacation automations
Agent: Loads homeassistant-tips-and-tricks.md for master switch pattern
       Creates input_boolean helper for the group
       Uses ha_config_set_input_boolean to create
       Updates each related automation to add condition:
       - state condition checking input_boolean is "on"
       Uses ha_config_set_automation to update each automation
       Confirms with ha_get_entity_state
```

### Set Up Zone-Based Occupancy Detection (MCP + Steering)

```
User: Trigger automations when everyone leaves home
Agent: Loads homeassistant-tips-and-tricks.md for zone state patterns
       Explains zone.home state represents person count
       Creates automation with:
       - numeric_state trigger on zone.home below 1
       Uses ha_config_set_automation to deploy
       Shows variations: home occupied, home full
```

### Schedule Delayed Action with input_datetime (MCP + Steering)

```
User: Turn off the lights 2 hours after motion stops
Agent: Loads homeassistant-tips-and-tricks.md for delayed action pattern
       Creates input_datetime helper with ha_config_set_input_datetime
       Creates two automations:
       1. Motion stops → Set datetime to now + 2 hours
       2. Time trigger on input_datetime → Turn off lights
       Explains why this survives restarts unlike delay action
       Uses ha_config_set_automation to deploy both
```

## Script Patterns

### Implement Parallel Actions in Script (MCP + Steering)

```
User: Send notifications to multiple people at the same time
Agent: Loads homeassistant-scripts-guide.md for parallel actions
       Creates script with parallel block:
       - Multiple notify actions execute simultaneously
       - No order guarantee for completion
       Explains caveats (variable conflicts, independent failures)
       Uses ha_config_set_script to deploy
```

### Use Response Variables from Actions (MCP + Steering)

```
User: Get calendar events and send them as a daily digest notification
Agent: Loads homeassistant-scripts-guide.md for response variables
       Creates automation with:
       - calendar.get_events action with response_variable
       - Template to format events in notification message
       Uses ha_config_set_automation to deploy
       Shows how to access response data in subsequent actions
```

### Create Repeat Loop for Multiple Rooms (MCP + Steering)

```
User: Turn off lights in each room with a 5-second delay between them
Agent: Loads homeassistant-scripts-guide.md for repeat/for_each
       Creates script with:
       - repeat.for_each with room list
       - delay action between iterations
       - repeat.item to access current room
       Uses ha_config_set_script to deploy
       Explains repeat.index and repeat.first/last variables
```

## Mobile App Integration

### Set Up iOS Actions for Apple Watch and CarPlay (MCP + Steering)

```
User: Create an action button for my Apple Watch to arm the alarm
Agent: Loads homeassistant-companion-app-guide.md for iOS Actions
       Explains two setup methods:
       - In-app: Name, icon, colors, show_in_watch setting
       - configuration.yaml: ios.actions section
       Creates automation with ios.action_fired event trigger
       Uses ha_config_set_automation to deploy
       Confirms with instructions to sync Apple Watch
```

### Configure Android Quick Settings Tiles (Steering only)

```
User: Add a tile to my Android notification panel to toggle my office lights
Agent: Loads homeassistant-companion-app-guide.md for Quick Settings
       Explains setup process:
       - Navigate to Settings > Companion App > Manage Tiles
       - Set label and select entity
       - Edit quick settings panel to add tile
       Lists supported domains and options (custom icon, vibrate, require unlock)
```

### Set Up Siri Shortcuts for Home Assistant (MCP + Steering)

```
User: Create a Siri shortcut to run my morning routine script
Agent: Loads homeassistant-companion-app-guide.md for Siri Shortcuts
       Explains available actions:
       - Call Service, Fire Event, Perform Action
       Lists launch methods (voice, widget, Back Tap, notification)
       Creates script if needed with ha_config_set_script
       Provides notification trigger example for shortcut activation
```

### Troubleshoot Location Tracking Issues (MCP + Steering)

```
User: My phone location isn't updating in Home Assistant
Agent: Loads homeassistant-companion-app-guide.md for troubleshooting
       Checks device_tracker entity with ha_get_entity_state
       Provides platform-specific checklist:
       - iOS: Location permission "Always", check sensor.last_update_trigger
       - Android: Background access, battery optimization disabled
       Explains location history debugging in Companion App settings
```

### Group and Replace Notifications with Tags (MCP + Steering)

```
User: Send notifications that update instead of creating duplicates
Agent: Loads homeassistant-companion-app-guide.md for notification tags
       Creates automation with:
       - data.tag for notification replacement
       - data.group for notification grouping
       Shows clear_notification pattern for removing specific notifications
       Uses ha_config_set_automation to deploy
```

### Create Critical Alert for Emergency Notifications (MCP + Steering)

```
User: Send a notification that bypasses Do Not Disturb when smoke is detected
Agent: Loads homeassistant-companion-app-guide.md for critical notifications
       Creates automation with platform-specific syntax:
       - iOS: push.sound.critical or interruption-level: critical
       - Android: ttl: 0, priority: high, channel: alarm_stream
       Uses ha_config_set_automation to deploy
       Warns about using sparingly for true emergencies
```

## Template Patterns

### Create Reusable Template Macros (Steering only)

```
User: How can I reuse the same temperature formatting across multiple sensors?
Agent: Loads homeassistant-templating-guide.md for custom templates
       Explains custom_templates directory setup
       Creates macro example:
       - Define macro in config/custom_templates/helpers.jinja
       - Import with {% from 'helpers.jinja' import macro_name %}
       Provides format_temp macro example with unit conversion
```

### Aggregate Power Consumption from Multiple Sensors (MCP + Steering)

```
User: Create a sensor that sums power from all my smart plugs
Agent: Loads homeassistant-templating-guide.md for aggregation patterns
       Creates template sensor with:
       - List of power sensor entity IDs
       - map('states') | map('float', default=0) | sum pattern
       - Proper unit_of_measurement and device_class
       Uses ha_config_set_template to deploy (or provides YAML)
```

### Calculate Distance Between Entities (MCP + Steering)

```
User: Create a sensor showing distance from my phone to home
Agent: Loads homeassistant-templating-guide.md for distance functions
       Creates template sensor with:
       - distance() function with device_tracker and zone
       - Proper unit_of_measurement (km or mi)
       Shows closest() function for finding nearest entity
       Uses ha_config_set_template to deploy (or provides YAML)
```

### Create Dynamic Entity List with expand() (MCP + Steering)

```
User: Create a sensor counting all pending updates
Agent: Loads homeassistant-tips-and-tricks.md for expand() patterns
       Creates template sensor with:
       - expand(states.update) to get all update entities
       - selectattr('state', 'eq', 'on') filter
       - Attribute listing entity names
       Uses ha_config_set_template to deploy (or provides YAML)
```

## Configuration Management

### Configure Universal Media Player (MCP + Steering)

```
User: Combine my TV, Chromecast, and speakers into one media player
Agent: Loads homeassistant-tips-and-tricks.md for Universal Media Player
       Provides YAML configuration with:
       - children: list of media player entities
       - commands: override for turn_on, volume, etc.
       - attributes: state and volume_level sources
       Explains configuration.yaml integration
```

### Manage Secrets in configuration.yaml (Steering only)

```
User: How do I keep my API keys and passwords secure?
Agent: Loads homeassistant-dev-guide.md for security best practices
       Explains secrets.yaml pattern:
       - Create secrets.yaml in config directory
       - Reference with !secret key_name
       - Add secrets.yaml to .gitignore
       Provides example for API keys, passwords, tokens
       Warns against committing secrets to version control
```

### Use Packages for Configuration Organization (Steering only)

```
User: How can I organize all my vacuum-related configs in one file?
Agent: Loads homeassistant-dev-guide.md for file organization
       Explains packages feature:
       - Create packages directory
       - Group related automations, scripts, sensors, helpers
       - Reference in configuration.yaml with packages: !include_dir_named packages
       Provides vacuum package example with all related configs
```
