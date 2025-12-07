---
name: "homeassistant"
displayName: "Home Assistant"
description: "Control your smart home with natural language. 80+ tools for device control, automation creation, dashboard management, and more"
keywords: ["homeassistant", "home assistant", "smart home", "automation"]
author: "Shibata, Tats"
mcpServers: ["homeassistant"]
---

# Home Assistant Power

## Overview

Control Home Assistant with natural language using Kiro. Access all Home Assistant operations with 80+ tools, from device control to automation creation.

This power uses [@homeassistant-ai/ha-mcp](https://github.com/homeassistant-ai/ha-mcp) to provide full access to Home Assistant capabilities. Spend less time configuring, more time enjoying your smart home.

**Key capabilities:**
- **Search & Discovery**: Fuzzy entity search, deep config search, system overview
- **Control**: Any service call, bulk device control, real-time state retrieval
- **Management**: Automations, scripts, helpers, dashboards, areas, zones, groups, calendars, blueprints
- **Monitoring**: History, statistics, camera snapshots, automation traces, ZHA devices
- **System**: Backup/restore, updates, add-ons, device registry

## When to Use This Power

- Control smart home devices with natural language
- Create and manage automations easily
- Customize dashboards
- Debug and troubleshoot automations
- Manage Home Assistant configuration efficiently
- Create complex scripts and scenes

## Available MCP Servers

### homeassistant
**Package:** `@homeassistant-ai/ha-mcp@latest`
**Connection:** uvx-based MCP server
**Authentication:** Home Assistant long-lived access token

**Configuration Required:**
- `HOMEASSISTANT_URL` - Your Home Assistant URL (e.g., `http://homeassistant.local:8123`)
- `HOMEASSISTANT_TOKEN` - Long-lived access token

**Tools (82 tools):**

**Search & Discovery**
- `ha_search_entities` - Fuzzy entity search
- `ha_deep_search` - Deep config search
- `ha_get_overview` - Get system overview
- `ha_get_state` - Get entity state

**Service & Device Control**
- `ha_call_service` - Call service
- `ha_bulk_control` - Bulk device control
- `ha_get_operation_status` - Get operation status
- `ha_get_bulk_status` - Get bulk status
- `ha_list_services` - List services

**Automations**
- `ha_config_get_automation` - Get automation
- `ha_config_set_automation` - Set automation
- `ha_config_remove_automation` - Remove automation

**Scripts**
- `ha_config_get_script` - Get script
- `ha_config_set_script` - Set script
- `ha_config_remove_script` - Remove script

**Helper Entities**
- `ha_config_list_helpers` - List helpers
- `ha_config_set_helper` - Set helper
- `ha_config_remove_helper` - Remove helper

**Dashboards**
- `ha_config_list_dashboards` - List dashboards
- `ha_config_get_dashboard` - Get dashboard
- `ha_config_set_dashboard` - Set dashboard
- `ha_config_update_dashboard_metadata` - Update dashboard metadata
- `ha_config_delete_dashboard` - Delete dashboard

**Areas & Floors**
- `ha_config_list_areas` - List areas
- `ha_config_set_area` - Set area
- `ha_config_remove_area` - Remove area
- `ha_config_list_floors` - List floors
- `ha_config_set_floor` - Set floor
- `ha_config_remove_floor` - Remove floor

**Zones**
- `ha_list_zones` - List zones
- `ha_create_zone` - Create zone
- `ha_update_zone` - Update zone
- `ha_delete_zone` - Delete zone

**Groups**
- `ha_config_list_groups` - List groups
- `ha_config_set_group` - Set group
- `ha_config_remove_group` - Remove group

**Calendar**
- `ha_config_get_calendar_events` - Get calendar events
- `ha_config_set_calendar_event` - Set calendar event
- `ha_config_remove_calendar_event` - Remove calendar event

**Blueprints**
- `ha_list_blueprints` - List blueprints
- `ha_get_blueprint` - Get blueprint
- `ha_import_blueprint` - Import blueprint

**Device Registry**
- `ha_list_devices` - List devices
- `ha_get_device` - Get device
- `ha_update_device` - Update device
- `ha_remove_device` - Remove device
- `ha_rename_entity` - Rename entity

**History & Statistics**
- `ha_get_history` - Get history
- `ha_get_statistics` - Get statistics

**Automation Traces**
- `ha_get_automation_traces` - Get automation traces

**System & Updates**
- `ha_check_config` - Check config
- `ha_restart` - Restart
- `ha_reload_core` - Reload core
- `ha_get_system_info` - Get system info
- `ha_get_system_health` - Get system health
- `ha_list_updates` - List updates

**Backup & Restore**
- `ha_backup_create` - Create backup
- `ha_backup_restore` - Restore backup

## Onboarding

### Step 1: Install uv

This power uses the `uvx` command, so `uv` MUST be installed:

**macOS / Linux:**

```bash
# Using Homebrew
brew install uv

# Without Homebrew
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows:**

Run in PowerShell or Command Prompt:

```powershell
winget install astral-sh.uv -e
```

### Step 2: Prepare Home Assistant

You MUST generate a long-lived access token in Home Assistant:

1. Log in to Home Assistant
2. Click your profile (username in bottom left)
3. **Security** tab → **Long-lived access tokens**
4. Click "Create token"
5. Enter token name (e.g., Kiro Integration)
6. Copy the generated token (shown only once)

### Step 3: Configure Environment Variables

When installing the power, configure the following environment variables:

- `HOMEASSISTANT_URL` - Your Home Assistant URL (e.g., `http://homeassistant.local:8123`)
- `HOMEASSISTANT_TOKEN` - The generated long-lived access token

### Step 4: Test It

Once the power is installed, test it with:

```
Can you see my Home Assistant?
```

Kiro should respond with a list of entities from your Home Assistant (lights, sensors, switches, etc.).

## What Can You Do With It?

Just talk naturally. Here are real examples:

**Device Control:**
- Turn on the living room lights
- Turn off the bedroom AC
- Turn off all lights

**State Queries:**
- What's the temperature in the living room?
- Which lights are on?

**Automation Management:**
- Create an automation that turns on the porch light at sunset
- Create an automation that turns on the porch light at 8 PM
- Why isn't this automation working?

**Automation Creation Examples:**
- Create an automation that turns on the porch light at sunset → Creates automation with proper triggers and actions
- The motion sensor automation isn't working, debug it → Analyzes execution traces, identifies issues, suggests fixes
- Add the coffee maker to my morning routine → Reads existing automation, adds new action, updates it
- Create a movie mode script: dim lights, close blinds, turn on TV → Creates reusable script with sequence of actions

**Dashboard Customization:**
- Add a weather card to my dashboard → Updates Lovelace dashboard with the new card

Spend less time configuring, more time enjoying your smart home.

## Best Practices

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

### Automation Design

- **Clear Naming**: Use descriptive names that explain the automation's purpose (e.g., "Porch Light at Night")
- **Use Conditions**: Add conditions as needed to prevent false triggers (SHOULD)
- **Check Traces**: When automations don't work, check automation traces to identify issues
- **Incremental Building**: Start simple and gradually add complexity

### Dashboard Management

- **Organize by Area**: Group cards by room or area
- **Appropriate Card Types**: Choose the best card type for each use case
- **Responsive Design**: Ensure layouts work well on mobile devices

### Security

- **Token Management**: Access tokens MUST be treated as sensitive information
- **Use Environment Variables**: Tokens MUST NOT be hardcoded in code or documentation
- **Regular Rotation**: Tokens SHOULD be rotated regularly for security
- **Least Privilege**: Use tokens with minimum required permissions (SHOULD)

## Troubleshooting

### Error: "Connection refused" or "Cannot connect to Home Assistant"
**Cause:** Incorrect Home Assistant URL or Home Assistant is not running
**Solution:**
1. Verify `HOMEASSISTANT_URL` is correct (e.g., `http://homeassistant.local:8123`)
2. Ensure Home Assistant is running
3. Check network connection
4. Verify firewall settings

### Error: "Authentication failed" or "Invalid token"
**Cause:** Incorrect or expired access token
**Solution:**
1. Verify `HOMEASSISTANT_TOKEN` is correct
2. Check token is not expired (long-lived tokens don't expire)
3. Ensure token has required permissions
4. Generate a new token if needed

### Error: "Entity not found"
**Cause:** Incorrect entity ID or entity doesn't exist
**Solution:**
1. Verify entity ID is correct (e.g., `light.living_room`)
2. Check entity is enabled in Home Assistant
3. Ensure entity integration is properly configured
4. Use `ha_search_entities` tool to find correct entity ID

### Error: "uvx: command not found"
**Cause:** `uv` is not installed
**Solution:**
1. Install `uv`:
   - macOS/Linux: `brew install uv` or `curl -LsSf https://astral.sh/uv/install.sh | sh`
   - Windows: `winget install astral-sh.uv -e`
2. Restart terminal to update PATH

### Automation not triggering
**Cause:** Incorrect trigger conditions or conditions not met
**Solution:**
1. Check automation traces with `ha_get_automation_traces` tool
2. Verify trigger conditions (time, state changes, etc.)
3. Ensure conditions are properly configured
4. Verify automation is enabled

## Additional Resources

For detailed troubleshooting and FAQ, see the [official documentation](https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/FAQ.md).

---

**Package:** `@homeassistant-ai/ha-mcp`
**Source:** Home Assistant AI Community
**License:** MIT
