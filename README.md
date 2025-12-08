![Cover](cover.jpg)

# Home Assistant Power for Kiro

A Kiro Power that brings Home Assistant expertise directly into your development workflow. Control devices through natural language, write YAML configurations, and build automations with instant access to Home Assistant knowledge and capabilities.

## What is a Kiro Power?

Powers are unified packages that combine MCP tools with framework expertise. Instead of just providing API access, powers give Kiro deep knowledge of Home Assistant patterns, best practices, and workflows. When you mention "automation" or "YAML," the power activates—loading relevant tools and context dynamically.

## Features

This power provides comprehensive Home Assistant support through 82 MCP tools and specialized knowledge:

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

### Natural Language Control

- "Turn on the living room lights"
- "What's the temperature in the bedroom?"
- "Which lights are currently on?"

### Automation Development

- "Create an automation that turns on the porch light at sunset"
- "The motion sensor automation isn't working, debug it"
- "Add the coffee maker to my morning routine"

### Dashboard Management

- "Add a weather card to my dashboard"
- "Create a custom button card for my lights"

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

1. **MCP Tools**: 82 tools from [ha-mcp](https://github.com/homeassistant-ai/ha-mcp) for direct Home Assistant API access
2. **Framework Expertise**: Built-in knowledge of Home Assistant patterns, YAML syntax, and best practices

When you mention keywords like "automation," "YAML," or "dashboard," the power activates dynamically—loading only relevant tools and context.

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
