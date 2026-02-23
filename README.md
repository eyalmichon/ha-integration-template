# Home Assistant Integration Template

A [Copier](https://copier.readthedocs.io/) template for bootstrapping Home Assistant custom integrations with a fully configured development environment.

## What You Get

- **Devcontainer**: Python 3.13, uv, Ruff, Pylance -- opens in VS Code/Cursor and just works
- **CI Workflows**: Ruff lint, Hassfest, HACS validation, CodeQL, automated releases
- **Pre-commit hooks**: Ruff format + lint on every commit
- **Dependabot**: Auto-updates GitHub Actions versions
- **Integration scaffold**: `manifest.json`, config flow, constants, strings, translations
- **Test scaffold**: pytest + pytest-homeassistant-custom-component
- **MCP dev server**: HA lifecycle tools (restart, logs, state, services) + MQTT tools, with auto-discovery for adding integration-specific tools
- **Scripts**: `scripts/setup` (install deps) and `scripts/develop` (run HA locally)

## Quick Start

### Create a New Integration

```bash
# Install copier (one time)
pipx install copier

# Scaffold a new integration
copier copy gh:eyalmichon/ha-integration-template ./ha-my-integration
```

You'll be prompted for the options described below.

### Update an Existing Integration

When you improve this template, pull changes into existing integrations:

```bash
cd ha-my-integration
copier update
```

Copier shows diffs between the template changes and your local modifications, letting you merge like git.

## Template Options Reference

### `domain`

The integration domain identifier. Used as the folder name under `custom_components/`, the `DOMAIN` constant, and in all HA-facing identifiers.

- **Type**: string
- **Example**: `my_device`, `smart_thermostat`, `solar_monitor`
- **Rules**: Lowercase letters, numbers, and underscores only. Must start with a letter.

### `integration_name`

The human-readable name shown in the HA UI, README, HACS listing, and config flow.

- **Type**: string
- **Example**: `My Device`, `Smart Thermostat`, `Solar Monitor`

### `description`

A short description of what the integration does. Used in the generated README and HACS metadata.

- **Type**: string
- **Default**: `A custom Home Assistant integration`
- **Example**: `Monitor and control your Smart Thermostat from Home Assistant`

### `github_user`

Your GitHub username. Used to generate URLs (documentation, issue tracker), CODEOWNERS, and badge links.

- **Type**: string
- **Example**: `eyalmichon`
- **Generates**: `https://github.com/<github_user>/ha-<domain>`

### `iot_class`

How the integration connects to devices/services. This is set in `manifest.json` and shown in the HA integrations page. See the [HA docs on IoT class](https://developers.home-assistant.io/docs/creating_integration_manifest/#iot-class) for details.

- **Type**: choice
- **Default**: `local_polling`
- **Options**:

| Value | When to Use |
|-------|-------------|
| `local_polling` | Polls a local device/service on a regular interval |
| `local_push` | Local device/service pushes updates (e.g. via MQTT, webhooks) |
| `cloud_polling` | Polls a cloud API on a regular interval |
| `cloud_push` | Cloud service pushes updates (e.g. via websocket) |
| `calculated` | Entity values are computed from other data, no external I/O |
| `assumed_state` | State is assumed based on the last command sent (no feedback) |

### `integration_type`

The kind of integration this is. Affects how HA treats config entries. See the [HA docs on integration types](https://developers.home-assistant.io/docs/creating_integration_manifest/#integration-type).

- **Type**: choice
- **Default**: `service`
- **Options**:

| Value | When to Use |
|-------|-------------|
| `hub` | Connects to a hub/bridge that manages multiple devices (e.g. Hue, Z-Wave) |
| `device` | Represents a single device directly (e.g. a smart plug) |
| `service` | Connects to a service/API that isn't a physical device (e.g. weather, calendar) |

### `platforms`

Comma-separated list of HA platforms your integration will expose. Each platform becomes an entry in the `PLATFORMS` list in `const.py`.

- **Type**: string (comma-separated)
- **Default**: `sensor`
- **Example**: `sensor,switch,binary_sensor`
- **Available platforms**:

| Platform | Entity Type |
|----------|------------|
| `sensor` | Numeric/text readings (temperature, energy, count) |
| `binary_sensor` | On/off states (motion, door open, connectivity) |
| `switch` | Toggleable on/off controls |
| `button` | One-shot triggers (restart, sync) |
| `select` | Dropdown selection from a list of options |
| `number` | Numeric input with min/max/step |
| `climate` | HVAC / thermostat control |
| `light` | Light control (brightness, color) |
| `cover` | Blinds, garage doors, shutters |
| `fan` | Fan speed/direction control |
| `lock` | Lock/unlock control |
| `media_player` | Media playback control |
| `camera` | Camera streams/snapshots |

### `has_config_flow`

Whether to include a config flow (the UI-based setup wizard in Settings > Devices & Services). Almost always `true` for new integrations.

- **Type**: boolean
- **Default**: `true`

### `python_version`

Python version for the devcontainer and `pyproject.toml`. Should match the version required by the Home Assistant release you're targeting.

- **Type**: string
- **Default**: `3.13`

## Generated Project Structure

```
ha-my-integration/
  .devcontainer/devcontainer.json     # Dev environment config
  .github/
    workflows/                        # CI: lint, hassfest, HACS, release, CodeQL
    dependabot.yml                    # Auto-update GH Actions
  .cursor/mcp.json                    # MCP server config
  .pre-commit-config.yaml             # Ruff hooks
  custom_components/my_device/        # Integration code
    __init__.py                       # Setup/unload entry
    config_flow.py                    # Config flow
    const.py                          # Domain + platforms
    manifest.json                     # HA manifest
    strings.json                      # UI strings
    translations/en.json              # English translations
  scripts/
    setup                             # Install deps + pre-commit
    develop                           # Run HA locally
    mcp_server/                       # Dev MCP tools
      tools/ha.py                     # HA lifecycle tools
      tools/mqtt.py                   # MQTT tools
  tests/
    conftest.py                       # Test fixtures
  pyproject.toml                      # Python config
  hacs.json                           # HACS metadata
```

## Adding Integration-Specific MCP Tools

The MCP server auto-discovers tool modules. Add a new file in `scripts/mcp_server/tools/`:

```python
# scripts/mcp_server/tools/my_device.py
from typing import Any


def register(mcp) -> None:
    @mcp.tool
    async def my_device_status() -> dict[str, Any]:
        """Check My Device status."""
        return {"status": "ok"}
```

It will be picked up automatically -- no imports or registration needed.
