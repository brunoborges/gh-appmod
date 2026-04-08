# Configuration

Details on how `gh appmod` configures the MCP server and repository.

## MCP Server Configuration

The App Modernization MCP server enables Copilot to perform Java modernization tasks. It's configured in the Copilot CLI configuration file.

### Configuration File Location

The MCP server configuration lives in:

```
~/.copilot/mcp-config.json
```

This path can be changed by setting the `COPILOT_HOME` environment variable:

```bash
export COPILOT_HOME=/custom/path
# Config will be at /custom/path/mcp-config.json
```

### Automatic Setup

Run `gh appmod init` to automatically configure the MCP server:

```bash
gh appmod init
```

This adds the following entry to your `mcp-config.json`:

```json
{
  "mcpServers": {
    "app-modernization": {
      "type": "local",
      "command": "npx",
      "tools": ["*"],
      "args": ["-y", "@microsoft/github-copilot-app-modernization-mcp-server"]
    }
  }
}
```

### Manual Setup

You can also configure the MCP server manually:

1. Open `~/.copilot/mcp-config.json` (create it if it doesn't exist)
2. Add the `app-modernization` entry as shown above
3. Verify with `gh appmod check`

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `COPILOT_HOME` | Override the Copilot CLI configuration directory | `~/.copilot` |

## Verifying Configuration

### Check All Prerequisites

```bash
gh appmod check
```

### View Configuration File

```bash
cat ~/.copilot/mcp-config.json
```

## Removing the MCP Server

To remove the App Modernization MCP server from your config:

1. Edit `~/.copilot/mcp-config.json`
2. Remove the `"app-modernization"` entry from the `mcpServers` object
3. Save the file

Or delete the entire config to start fresh:

```bash
rm ~/.copilot/mcp-config.json
```

## Modernization Skills

The extension bundles modular skill definitions following the [agentskills.io](https://agentskills.io/) specification. Use `gh appmod add-skills` to install them into your repository at `.github/skills/`.

### Available Skills

| Skill | Description |
|-------|-------------|
| `assess` | Modernization posture assessment |
| `upgrade` | General Java upgrade guidance |
| `migrate` | Azure service migration tasks |
| `deploy` | Azure deployment guidance |
| `java-8-to-11` | JDK 8 → 11 upgrade instructions + checklist |
| `java-11-to-17` | JDK 11 → 17 upgrade instructions + checklist |
| `java-17-to-21` | JDK 17 → 21 upgrade instructions + checklist |
| `java-21-to-25` | JDK 21 → 25 upgrade instructions + checklist |

### How Skills Are Used

- **Modernize CLI** — Automatically detects skills in `.github/skills/` when creating upgrade and migration plans
- **Custom agent** — The `appmod.agent.md` agent is self-contained but also reads skills from `.github/skills/` when available
- **Agentic workflows** — Workflows can reference skills for detailed instructions

## Next Steps

- [Getting Started](getting-started.md) — Initial setup walkthrough
- [Command Reference](commands.md) — Full command details
- [Troubleshooting](troubleshooting.md) — Common issues and solutions
