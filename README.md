![Cover](cover.jpg)

# Home Assistant Power for Kiro

Control Home Assistant with natural language using Kiro.

## Features

Access all Home Assistant operations with 80+ tools:

- **Search & Discovery**: Fuzzy entity search, deep config search, system overview
- **Control**: Any service call, bulk device control, real-time state retrieval
- **Management**: Automations, scripts, helpers, dashboards, areas, zones, groups, calendars, blueprints
- **Monitoring**: History, statistics, camera snapshots, automation traces, ZHA devices
- **System**: Backup/restore, updates, add-ons, device registry

## Project Structure

```
.
├── power-homeassistant/   # Power files directory
│   ├── POWER.md           # Kiro Power metadata and documentation
│   └── mcp.json           # MCP server configuration
├── LICENSE.md             # License file
└── README.md              # User documentation
```

## Installation

### 1. Install Directly in Kiro

1. Open Kiro
2. Open Powers panel
3. Click "Import power from GitHub"
4. Enter `https://github.com/rewse/kiro-power-homeassistant/tree/main/power-homeassistant`

### 2. Install from Local Path

1. Clone this repository
2. Open Kiro
3. Open Powers panel
4. Click "Import power from a folder"
5. Select the `power-homeassistant` directory

## Prerequisites

This power requires `uv` to be installed:

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

## Configuration

### Generate Access Token in Home Assistant

1. Log in to Home Assistant
2. Profile → Security → Long-lived access tokens
3. Click "Create token"
4. Enter token name (e.g., Kiro Integration)
5. Copy the generated token

### Set Environment Variables

When installing the power, configure the following environment variables:

- `HOMEASSISTANT_URL`: Your Home Assistant URL (e.g., `http://homeassistant.local:8123`)
- `HOMEASSISTANT_TOKEN`: The generated long-lived access token

## Usage Examples

Just talk naturally:

- **"Create an automation that turns on the porch light at sunset"**
  - Creates automation with proper triggers and actions
- **"Add a weather card to my dashboard"**
  - Updates Lovelace dashboard with the new card
- **"The motion sensor automation isn't working, debug it"**
  - Analyzes execution traces, identifies issues, suggests fixes
- **"Add the coffee maker to my morning routine"**
  - Reads existing automation, adds new action, updates it
- **"Create a movie mode script"**
  - Creates reusable script with sequence of actions

### More Examples

- Turn on the living room lights
- Turn off all lights
- What's the temperature in the living room?
- Which lights are on?
- Run the "goodnight" scene

## MCP Server

- [@homeassistant-ai/ha-mcp](https://github.com/homeassistant-ai/ha-mcp).

Special thanks to the Home Assistant AI team for their excellent work on this MCP server.

## Troubleshooting

### Cannot Connect

- Verify `HOMEASSISTANT_URL` is correct (e.g., `http://homeassistant.local:8123`)
- Ensure Home Assistant is running
- Check network connection
- Verify firewall settings

### Authentication Error

- Verify `HOMEASSISTANT_TOKEN` is correct
- Check token is not expired
- Ensure token has required permissions

### uv Not Found

- Verify `uv` is installed correctly
- Restart terminal to update PATH

For detailed troubleshooting, see the [official FAQ](https://github.com/homeassistant-ai/ha-mcp/blob/master/docs/FAQ.md).
