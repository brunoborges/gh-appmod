# Configuration

Details on how `gh appmod` configures the Copilot CLI and MCP server.

## MCP Server Configuration

The App Modernization MCP server is what enables Copilot CLI to perform Java modernization tasks. It's configured in the Copilot CLI configuration file.

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

Run `gh appmod setup` to automatically configure the MCP server:

```bash
gh appmod setup
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

Or use the Copilot CLI's built-in command:

```bash
copilot
# Then inside the interactive session:
/mcp add app-modernization
```

### Session-Only Configuration

If you don't run `gh appmod setup`, the extension automatically injects the MCP server configuration for each session using Copilot CLI's `--additional-mcp-config` flag. This means:

- The MCP server is available for that session only
- No permanent changes are made to your configuration
- You'll see an informational message each time

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `COPILOT_HOME` | Override the Copilot CLI configuration directory | `~/.copilot` |

## Copilot CLI Pass-Through Options

All action commands (`upgrade`, `migrate`, `deploy`) pass unrecognized flags through to `copilot`. Useful options include:

### Model Selection

```bash
# Use a specific model
gh appmod upgrade --model gpt-5.2 "Upgrade to JDK 21"
```

### Reasoning Effort

```bash
# Use higher reasoning effort for complex tasks
gh appmod migrate --effort xhigh "Migrate Oracle SQL dialect to PostgreSQL"
```

### Permissions

```bash
# Auto-approve everything
gh appmod upgrade --yolo

# Or be more selective
gh appmod upgrade --allow-all-tools
```

### Additional Directories

```bash
# Allow Copilot to access files outside the current directory
gh appmod upgrade --add-dir /path/to/shared/libs "Upgrade to JDK 21"
```

## Verifying Configuration

### Check MCP Server Status

After setup, verify in Copilot CLI:

```bash
copilot
# Then:
/mcp show
```

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

## Next Steps

- [Getting Started](getting-started.md) — Initial setup walkthrough
- [Command Reference](commands.md) — Full command details
- [Troubleshooting](troubleshooting.md) — Common issues and solutions
