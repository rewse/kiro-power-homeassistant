![Cover](cover.jpg)

# Home Assistant Power for Kiro

A Kiro Power that brings Home Assistant expertise directly into your development workflow. Control devices through natural language, write YAML configurations, and build automations with instant access to Home Assistant knowledge and capabilities.

## What is a Kiro Power?

Powers are unified packages that combine MCP tools with framework expertise. Instead of just providing API access, powers give Kiro deep knowledge of Home Assistant patterns, best practices, and workflows. When you mention "homeassistant" or "hass," the power activates—loading relevant tools and context dynamically.

## Features

This power provides comprehensive Home Assistant support:

- **80+ MCP Tools**: Complete Home Assistant API access for device control, automation creation, dashboard management, and system configuration
- **Specialized Knowledge**: YAML automation patterns, best practices, debugging workflows, and Home Assistant conventions
- **Guided Workflows**: Step-by-step assistance for common tasks like creating automations, troubleshooting issues, and optimizing configurations

**MCP tool categories:**

- **Search & Discovery**: Fuzzy entity search, deep config search, system overview
- **Control**: Any service call, bulk device control, real-time state retrieval
- **Management**: Automations, scripts, helpers, dashboards, areas, zones, groups, calendars, blueprints
- **Monitoring**: History, statistics, camera snapshots, automation traces, ZHA devices
- **System**: Backup/restore, updates, add-ons, device registry

## Installation

### From GitHub

1. Open Kiro
2. Open Powers panel
3. Click "Import power from GitHub"
4. Enter: `https://github.com/rewse/kiro-power-homeassistant/tree/main/power-homeassistant`

### From Local Path

1. Clone this repository
2. Open Kiro → Powers panel
3. Click "Import power from a folder"
4. Select the `power-homeassistant` directory

## Prerequisites

This power requires `uv` to be installed:

**macOS / Linux:**
```bash
brew install uv
# or
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows:**
```powershell
winget install astral-sh.uv -e
```

## Configuration

### 1. Generate Access Token

1. Log in to Home Assistant
2. Profile → Security → Long-lived access tokens
3. Click "Create token" and copy it

### 2. Set Environment Variables

When installing the power, configure:
- `HOMEASSISTANT_URL`: Your Home Assistant URL (e.g., `http://homeassistant.local:8123`)
- `HOMEASSISTANT_TOKEN`: The generated long-lived access token

### 3. Test

Ask Kiro: "Can you see my Home Assistant?"

## Usage Examples

### Device Control & State Query

- "Turn on the living room lights"
- "What's the temperature in the bedroom?"
- "Which lights are currently on?"

### Automation and Template Development

- "Create an automation that turns on the porch light at sunset"
- "Why isn't my motion sensor automation working?"
- "Create a template sensor that averages two temperature sensors"

### Dashboard Management

- "Show me my current dashboard configuration"
- "Add a weather card to my dashboard"
- "Create a new view for my climate controls"

### YAML Configuration

- "Can you review my automation YAML for best practices?"
- "How should I split my configuration.yaml?"
- "My template sensor shows 'unavailable' on startup"
- "Update my old automation to use modern syntax"

## Project Structure

```
.
├── power-homeassistant/
│   ├── POWER.md           # Power metadata and documentation
│   ├── mcp.json           # MCP server configuration
│   └── steering/
│       ├── homeassistant-dev-guide.md    # YAML and development patterns
│       └── homeassistant-mcp-tools.md    # MCP tools reference
├── LICENSE.md
└── README.md
```

## How It Works

This power combines:

1. **MCP Tools**: 80+ tools from [ha-mcp](https://github.com/homeassistant-ai/ha-mcp) for direct Home Assistant API access
2. **Framework Expertise**: Built-in knowledge of Home Assistant patterns, YAML syntax, and best practices

When you mention keywords like "homeassistant," "home assistant," "hass," or "lovelace," the power activates—loading relevant tools and context.

## Troubleshooting

### Cannot Connect
- Verify `HOMEASSISTANT_URL` is correct
- Ensure Home Assistant is running
- Check network/firewall settings

### Authentication Error
- Verify `HOMEASSISTANT_TOKEN` is correct
- Generate a new token if needed

### uv Not Found
- Install `uv` and restart terminal

For detailed troubleshooting, see the [official FAQ](https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/FAQ.md).

## Credits

Special thanks to the [Home Assistant AI team](https://github.com/homeassistant-ai/ha-mcp) for their excellent work on the MCP server.
