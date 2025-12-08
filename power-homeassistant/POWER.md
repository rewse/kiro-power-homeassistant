---
name: "homeassistant"
displayName: "Home Assistant"
description: "Control your smart home with natural language. 80+ tools for device control, automation creation, dashboard management, and more"
keywords: ["homeassistant", "home assistant", "hass", "ha-mcp", "lovelace"]
author: "Shibata, Tats"
---

# Home Assistant Power

Control Home Assistant with natural language using Kiro. This power combines MCP tools with Home Assistant expertise to help you manage your smart home effectively.

**What this power provides:**

- **80+ MCP Tools**: Complete Home Assistant API access for device control, automation creation, dashboard management, and system configuration
- **Specialized Knowledge**: YAML automation patterns, best practices, debugging workflows, and Home Assistant conventions via steering files
- **Guided Workflows**: Step-by-step assistance for common tasks like creating automations, troubleshooting issues, and optimizing configurations

**MCP tool categories:**

- **Search & Discovery**: Fuzzy entity search, deep config search, system overview
- **Control**: Any service call, bulk device control, real-time state retrieval
- **Management**: Automations, scripts, helpers, dashboards, areas, zones, groups, calendars, blueprints
- **Monitoring**: History, statistics, camera snapshots, automation traces, ZHA devices
- **System**: Backup/restore, updates, add-ons, device registry

# Onboarding

## Step 1: Validate dependencies

Before using Home Assistant Power, ensure the following are installed:

- **uv package manager**: Required to run the MCP server
  - Verify with: `uvx --version`
  - **CRITICAL**: If `uvx` command is not found, DO NOT proceed. Install uv first.

## Step 2: Install uv (if not installed)

- macOS/Linux: `brew install uv` or `curl -LsSf https://astral.sh/uv/install.sh | sh`
- Windows: `winget install astral-sh.uv -e`

After installation, restart your terminal to update PATH.

## Step 3: Generate Home Assistant token

1. Open Home Assistant web interface
2. Navigate to: Profile → Security → Long-lived access tokens
3. Click "Create token" and copy the generated token
4. Keep this token secure - you'll need it when installing the power

## Step 4: Configure environment variables

When installing this power, configure:
- `HOMEASSISTANT_URL` - Your Home Assistant URL (e.g., `http://homeassistant.local:8123`)
- `HOMEASSISTANT_TOKEN` - The generated long-lived access token

## Step 5: Verify connection

Test the connection by asking: "Can you see my Home Assistant?"

# When to Load Steering Files

- Writing YAML automations or scripts → `homeassistant-dev-guide.md`
- Creating complex automation triggers or conditions → `homeassistant-dev-guide.md`
- Looking up specific MCP tool usage → `homeassistant-mcp-tools.md`
- Debugging automation issues → `homeassistant-dev-guide.md`
- Working with dashboards or Lovelace cards → `homeassistant-dev-guide.md`

# Best Practices

## Natural Language First

**Start with natural language** for common operations:
- Device control: "Turn on the living room lights"
- State queries: "What's the temperature in the bedroom?"
- Simple automations: "Create an automation to turn off lights at midnight"

## Security

- **Token Management**: Access tokens MUST be treated as sensitive information
- **Use Environment Variables**: Tokens MUST NOT be hardcoded in code or documentation
- **Regular Rotation**: Tokens SHOULD be rotated regularly for security

## Do's and Don'ts

### ✅ Do:

- Use natural language for device control and state queries
- Use native YAML conditions instead of templates when available
- Use service action targets for entity control
- Check automation traces when debugging issues
- Store tokens in environment variables
- Use descriptive automation names (e.g., "Porch Light at Sunset")
- Test automations after creation

### ❌ Don't:

- Hardcode tokens in code or documentation
- Use template conditions when native conditions work
- Use legacy entity_id syntax in service calls
- Skip testing after creating automations
- Use abbreviations for automation names
- Ignore automation traces when debugging

# Troubleshooting

## "Connection refused" or "Cannot connect to Home Assistant"

**Cause:** Incorrect Home Assistant URL or Home Assistant is not running

**Solution:**
1. Verify `HOMEASSISTANT_URL` is correct (e.g., `http://homeassistant.local:8123`)
2. Ensure Home Assistant is running
3. Check network connection
4. Verify firewall settings

## "Authentication failed" or "Invalid token"

**Cause:** Incorrect or expired access token

**Solution:**
1. Verify `HOMEASSISTANT_TOKEN` is correct
2. Check token is not expired (long-lived tokens don't expire)
3. Ensure token has required permissions
4. Generate a new token if needed

## "Entity not found"

**Cause:** Incorrect entity ID or entity doesn't exist

**Solution:**
1. Verify entity ID is correct (e.g., `light.living_room`)
2. Check entity is enabled in Home Assistant
3. Ensure entity integration is properly configured
4. Use `ha_search_entities` tool to find correct entity ID

## "uvx: command not found"

**Cause:** `uv` is not installed

**Solution:**
1. Install `uv`:
   - macOS/Linux: `brew install uv` or `curl -LsSf https://astral.sh/uv/install.sh | sh`
   - Windows: `winget install astral-sh.uv -e`
2. Restart terminal to update PATH

## Automation not triggering

**Cause:** Incorrect trigger conditions or conditions not met

**Solution:**
1. Check automation traces with `ha_get_automation_traces` tool
2. Verify trigger conditions (time, state changes, etc.)
3. Ensure conditions are properly configured
4. Verify automation is enabled

# Tips

1. **Start with natural language** - Describe what you want in plain language
2. **Use fuzzy search** - `ha_search_entities` finds entities even with partial names
3. **Check traces for debugging** - `ha_get_automation_traces` shows why automations triggered or didn't
4. **Leverage steering files** - Complex YAML work loads `homeassistant-dev-guide.md` automatically
5. **Test incrementally** - Create simple automations first, then add complexity
6. **Use bulk control** - `ha_bulk_control` for multiple devices at once
7. **Monitor system health** - `ha_get_system_health` for troubleshooting
8. **Keep tokens secure** - Never commit tokens to version control

# Resources

### Home Assistant MCP Server

- [Home Assistant MCP Server](https://github.com/homeassistant-ai/ha-mcp)
- [FAQ & Troubleshooting](https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/FAQ.md)
- [macOS uv Installation](https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/macOS-uv-guide.md)
- [Windows uv Installation](https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/Windows-uv-guide.md)

### Home Assistant Developer Guide

- [Standards](https://developers.home-assistant.io/docs/documenting/standards)
- [General style guide](https://developers.home-assistant.io/docs/documenting/general-style-guide)
- [YAML Style Guide](https://developers.home-assistant.io/docs/documenting/yaml-style-guide/)

---

**Package:** `ha-mcp`
**Source:** [Home Assistant AI Community](https://github.com/homeassistant-ai/ha-mcp)
**License:** MIT
