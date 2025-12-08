# Home Assistant Tips and Tricks

This steering file provides practical tips and tricks for Home Assistant power users. It covers advanced techniques, UI customizations, automation patterns, and productivity enhancements that can significantly improve your smart home experience.

**What's covered:**
- Voice assistant (Assist) customization and extensions
- Dashboard organization and navigation patterns
- Automation debugging and testing techniques
- Template sensor patterns for reusable logic
- Script execution patterns
- Zone and area management
- Dynamic scenes and state management

**When this loads:**
- Optimizing Home Assistant workflows
- Creating advanced automations
- Customizing dashboards and UI
- Working with voice assistants
- Debugging and testing configurations

## Voice Assistant (Assist) Tips

### Adding Aliases to Entities and Areas

You can add aliases to entities and areas. Aliases are alternate names that you can use when using Assist, Home Assistant's private voice assistant.

This is useful when:
- You want to use colloquial names for devices
- You have multilingual household members
- You want shorter names for voice commands

```yaml
# Example: Adding aliases via customize.yaml
homeassistant:
  customize:
    light.living_room_main:
      aliases:
        - "main light"
        - "big light"
        - "ceiling light"
```

Note: Aliases also work with Google Assistant, but not with Alexa.

### Extending Assist with Sentence Triggers

You can extend what Assist understands by creating automations with sentence triggers. This allows you to add custom voice commands.

```yaml
automation:
  - alias: "Voice Command - Raise Desk"
    triggers:
      - trigger: conversation
        command:
          - "Raise the desk"
          - "Desk up"
          - "Stand up mode"
    actions:
      - action: cover.open_cover
        target:
          entity_id: cover.standing_desk
```

### Using Wildcards in Sentence Triggers

Wildcards allow you to match parts of a sentence lazily, such as "Add {item} to my shopping list".

```yaml
automation:
  - alias: "Voice Command - Chain Commands"
    triggers:
      - trigger: conversation
        command: "{first_command} and {second_command}"
    actions:
      - action: conversation.process
        data:
          text: "{{ trigger.slots.first_command }}"
      - action: conversation.process
        data:
          text: "{{ trigger.slots.second_command }}"
```

This automation can be recursively called. When you say "1 and 2 and 3 and 4", it matches as "{1} and {2 and 3 and 4}", and processing "2 and 3 and 4" calls the same automation again.

**Caveats:**
- Simple patterns like "{1} and {2}" may trigger unintentionally
- You won't get feedback if some commands are erroneous

### Leveraging Areas for Voice Commands

A good area setup can unlock powerful Assist commands. When areas are properly configured, you can use commands like:
- "Turn off all lights in the living room"
- "Close all blinds in the bedroom"
- "What's the temperature in the kitchen?"

## Dashboard Tips

### Using Subviews for Room Navigation

A subview is a view that does not appear in the dashboard top bar. Use subviews for detailed room views and navigate using Tile cards.

```yaml
views:
  - title: "Home"
    path: home
    cards:
      - type: tile
        entity: light.living_room
        tap_action:
          action: navigate
          navigation_path: /lovelace/living-room
      - type: tile
        entity: light.bedroom
        tap_action:
          action: navigate
          navigation_path: /lovelace/bedroom

  - title: "Living Room"
    path: living-room
    subview: true
    cards:
      - type: entities
        entities:
          - light.living_room_main
          - light.living_room_lamp
          - switch.living_room_fan
```

### Decluttering the Sidebar

Organize your sidebar by long-pressing on "Home Assistant" in the sidebar. Create a navigation card for technical items to keep them out of your daily view.

```yaml
type: grid
columns: 2
cards:
  - type: button
    name: "Developer Tools"
    icon: mdi:hammer-wrench
    tap_action:
      action: navigate
      navigation_path: /developer-tools/state
  - type: button
    name: "Settings"
    icon: mdi:cog
    tap_action:
      action: navigate
      navigation_path: /config/dashboard
  - type: button
    name: "Logs"
    icon: mdi:text-box-outline
    tap_action:
      action: navigate
      navigation_path: /config/logs
```

### Copy, Paste, and Cut on Dashboards

You can copy, cut, and paste cards within a view, across views, and across dashboards. This is useful when:
- Reshuffling views
- Creating similar cards in different dashboards
- Moving cards between views

### Customizing Theme Colors

Most cards use the primary color, while the Tile card can use the accent color. Change the primary/accent colors to personalize your Home Assistant.

```yaml
# In your theme configuration
my_custom_theme:
  primary-color: "#1976D2"
  accent-color: "#FF9800"
```

### Tile Card vs Mushroom Cards

The native Tile card allows you to create a similar look and feel to Mushroom cards. Consider using native Tile cards for:
- Better performance
- No custom card dependencies
- Built-in features like color states

Use Mushroom cards when you need:
- Custom titles and subtitles
- More advanced styling options
- Specific card types not available natively

## Automation Tips

### Testing Automations by Faking State Changes

The Developer Tools can help you simulate literally everything in Home Assistant. Use States tab to change entity states for testing without waiting for real-world conditions.

**Steps:**
1. Go to Developer Tools → States
2. Find the entity you want to modify
3. Change the state or attributes
4. Your automation will trigger as if the real change occurred

**Warning:** Make sure to restore real values once you are done testing.

### Using Trigger IDs with Choose Action

Combine "Trigger IDs" and the "choose" action to group simple automations by functional packages.

```yaml
automation:
  - alias: "Light Control Based on Sun"
    triggers:
      - trigger: sun
        event: sunset
        id: sunset
      - trigger: sun
        event: sunrise
        id: sunrise
    actions:
      - choose:
          - conditions:
              - condition: trigger
                id: sunset
            sequence:
              - action: light.turn_on
                target:
                  entity_id: light.porch
          - conditions:
              - condition: trigger
                id: sunrise
            sequence:
              - action: light.turn_off
                target:
                  entity_id: light.porch
```

### Viewing Trigger Data While Building

Triggers convey a lot of data, and you can see the data while building an automation. Fire the trigger and click on the "Triggered" banner to see all available data.

This is especially useful for:
- Wildcard triggers (to see captured values)
- Event triggers (to see event data)
- State triggers (to see old and new states)

### Keeping Automations Instant

Automations are not meant to run forever. They reset after a restart. If you need to wait hours to perform an action, use an `input_datetime` helper and a second automation triggered by it.

```yaml
# Automation 1: Set the timer
automation:
  - alias: "Schedule Delayed Action"
    triggers:
      - trigger: state
        entity_id: binary_sensor.motion
        to: "on"
    actions:
      - action: input_datetime.set_datetime
        target:
          entity_id: input_datetime.delayed_action_time
        data:
          datetime: "{{ now() + timedelta(hours=2) }}"

# Automation 2: Execute when timer fires
automation:
  - alias: "Execute Delayed Action"
    triggers:
      - trigger: time
        at: input_datetime.delayed_action_time
    actions:
      - action: light.turn_off
        target:
          entity_id: light.living_room
```

### Using Input Boolean as Automation Master Switch

Create an `input_boolean` helper and add it as a condition to every automation in a functional group. This gives you a single button that controls the complete functionality.

```yaml
# Helper definition
input_boolean:
  smart_cleaning_enabled:
    name: "Smart Cleaning Enabled"
    icon: mdi:robot-vacuum

# In each related automation
automation:
  - alias: "Smart Cleaning - Start When Away"
    conditions:
      - condition: state
        entity_id: input_boolean.smart_cleaning_enabled
        state: "on"
    # ... rest of automation
```

### Using the Logbook for Debugging

The logbook holds valuable information for debugging. You can find:
- Which automation triggered a change
- What trigger caused the automation to run
- What conditions were evaluated
- Which user initially caused the chain of events

## Script Tips

### Running vs Turning On a Script

**Running a script** blocks subsequent actions until the script completes.

**Turning on a script** is an immediate action that does not wait for the script to finish.

```yaml
# Blocking execution - waits for script to complete
actions:
  - action: script.turn_on
    target:
      entity_id: script.long_running_task
  - action: light.turn_on  # This runs immediately, doesn't wait
    target:
      entity_id: light.indicator

# Non-blocking execution - waits for script to complete
actions:
  - action: script.long_running_task  # Blocks until complete
  - action: light.turn_on  # This waits for script to finish
    target:
      entity_id: light.indicator
```

## Template Sensor Tips

### Creating Reusable Logic with Template Sensors

If you keep creating the same complex trigger/condition, simplify your automations by creating a template sensor. This keeps the logic in one place.

```yaml
template:
  - sensor:
      - unique_id: home_occupancy_status
        name: "Home Occupancy"
        state: >-
          {% set people_home = states.person | selectattr('state', 'eq', 'home') | list | count %}
          {% set total_people = states.person | list | count %}
          {% if people_home == 0 %}
            empty
          {% elif people_home == total_people %}
            full
          {% else %}
            partial
          {% endif %}
```

Now you can use this sensor in multiple automations:

```yaml
automation:
  - alias: "Turn Off Lights When Home Empty"
    triggers:
      - trigger: state
        entity_id: sensor.home_occupancy
        to: "empty"
    actions:
      - action: light.turn_off
        target:
          entity_id: all
```

### Using expand() for Dynamic Entity Lists

`expand()` is an advanced templating feature that lets you expand specific parts of your state machine.

```yaml
template:
  - sensor:
      - unique_id: pending_updates_count
        name: "Pending Updates"
        state: >-
          {{ expand(states.update)
             | selectattr('state', 'eq', 'on')
             | list
             | count }}
        attributes:
          entities: >-
            {{ expand(states.update)
               | selectattr('state', 'eq', 'on')
               | map(attribute='name')
               | list
               | sort }}
```

This sensor gives you the number of pending updates and their names, which you can use to create notifications.

## Zone Tips

### Using Zone State for Occupancy

The state of a zone represents the number of persons present in it. You can create triggers and conditions based on zone occupancy.

```yaml
# Trigger when home becomes empty
triggers:
  - trigger: numeric_state
    entity_id: zone.home
    below: 1

# Trigger when home is occupied
triggers:
  - trigger: numeric_state
    entity_id: zone.home
    above: 0

# Trigger when everyone is home
triggers:
  - trigger: template
    value_template: >-
      {{ states('zone.home') | int == states.person | list | count }}
```

## Scene Tips

### Creating Dynamic Scenes

You can capture the state of your devices to restore them later with `scene.create`. This is useful for temporary state changes.

```yaml
automation:
  - alias: "Movie Mode with State Restore"
    triggers:
      - trigger: state
        entity_id: media_player.tv
        to: "playing"
    actions:
      # Save current state
      - action: scene.create
        data:
          scene_id: before_movie
          snapshot_entities:
            - light.living_room
            - cover.living_room_blinds
      # Apply movie settings
      - action: light.turn_off
        target:
          entity_id: light.living_room
      - action: cover.close_cover
        target:
          entity_id: cover.living_room_blinds

  - alias: "Restore After Movie"
    triggers:
      - trigger: state
        entity_id: media_player.tv
        from: "playing"
    actions:
      - action: scene.turn_on
        target:
          entity_id: scene.before_movie
```

**Note:** Scenes created with `scene.create` do not survive a restart.

## UI Tips

### Favorite Colors for Lights

Lights have favorite colors that you can customize. Long press on a light entity to edit and add/remove favorite colors. This makes it quick to set commonly used colors.

### Absolute vs Relative Time Display

In the more-info screen for lights, covers, switches, alarms, fans, sirens, and locks, you can change how the last state change is displayed by clicking on it. Toggle between:
- Relative time: "2 hours ago"
- Absolute time: "14:30:25"

### Display Precision for Numerical Sensors

Change the display precision of numerical sensors directly in the UI without modifying the sensor itself. This only affects display, not the actual sensor value.

### Device Class for Binary Sensors

Edit the device class of binary sensors directly in the UI to change how they are displayed (icon and state labels). For example:
- `door`: Shows "Open" / "Closed"
- `motion`: Shows "Detected" / "Clear"
- `problem`: Shows "Problem" / "OK"

## Backup Tips

### Setting Up Remote Backups

Home Assistant supports network storage for backups. If you have a NAS, setting up daily backups is straightforward:

1. Go to Settings → System → Storage
2. Add network storage (SMB/NFS)
3. Configure automatic backups to use the network location

**Best practices:**
- Schedule daily backups
- Keep at least 7 days of backups
- Test restore periodically
- Store backups off-site if possible

## Media Player Tips

### Universal Media Player

If you have multiple media sources (TV, Cast, speakers, Plex, Android TV remote), use the Universal Media Player integration to create a unified media player.

```yaml
media_player:
  - platform: universal
    name: "Living Room Media Center"
    children:
      - media_player.living_room_tv
      - media_player.living_room_chromecast
      - media_player.living_room_speaker
    commands:
      turn_on:
        action: remote.turn_on
        target:
          entity_id: remote.living_room_tv
      turn_off:
        action: remote.turn_off
        target:
          entity_id: remote.living_room_tv
      volume_up:
        action: media_player.volume_up
        target:
          entity_id: media_player.living_room_speaker
      volume_down:
        action: media_player.volume_down
        target:
          entity_id: media_player.living_room_speaker
    attributes:
      state: media_player.living_room_tv
      volume_level: media_player.living_room_speaker|volume_level
```

## Companion App Tips

### macOS Companion App Sensors

The macOS Companion App exposes many sensors from your machine, including:
- Camera in use (useful for meeting detection)
- Microphone in use
- Active app
- Battery level
- Screen state

```yaml
automation:
  - alias: "Do Not Disturb During Meetings"
    triggers:
      - trigger: state
        entity_id: binary_sensor.macbook_camera_in_use
        to: "on"
    actions:
      - action: input_boolean.turn_on
        target:
          entity_id: input_boolean.do_not_disturb
```

### Geocoded Location Sensor

The Companion App provides a Geocoded Location sensor that gives you the city (and country if abroad) where a person is located. This can be combined with person entities for rich presence information.

## Local Voice Assistant Setup

### Running Assist Locally

For complete local voice assistant setup, you need:
- **Whisper**: Speech-to-text (STT)
- **Piper**: Text-to-speech (TTS)

**Requirements:**
- Sufficient CPU power (not recommended for Raspberry Pi)
- Install Whisper and Piper add-ons

**Alternative:** Use Home Assistant Cloud (Nabu Casa) for one-click setup with cloud-based STT/TTS, which also supports Home Assistant development.

## Resources

- [Home Assistant Companion Docs](https://companion.home-assistant.io/)
- [Universal Media Player Integration](https://home-assistant.io/integrations/universal/)
- [Whisper Add-on](https://github.com/home-assistant/addons/tree/master/whisper)
- [Piper Add-on](https://github.com/home-assistant/addons/tree/master/piper)
