# Troubleshooting

Common issues and solutions when using `gh appmod`.

## Prerequisite Issues

### "Node.js is not installed" or "Version X found, but 22+ is required"

Install Node.js 22 or later:

- **macOS (Homebrew):** `brew install node@22`
- **All platforms:** Download from [nodejs.org](https://nodejs.org/)
- **Version manager (nvm):** `nvm install 22 && nvm use 22`

Verify:

```bash
node --version   # Should be v22.x.x or later
npm --version    # Should be v10.x.x or later
```

### "GitHub Copilot CLI is not installed"

Install the Copilot CLI following the [official guide](https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli):

```bash
# Install via npm
npm install -g @githubnext/github-copilot-cli
```

Or check the [latest installation instructions](https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli) as the install method may change.

### "GitHub CLI (gh) is not installed"

Install the GitHub CLI:

- **macOS:** `brew install gh`
- **Windows:** `winget install GitHub.cli`
- **Linux:** See [installation instructions](https://github.com/cli/cli#installation)

## Init Issues

### `gh appmod init` says "already configured" but MCP server doesn't work

The MCP server entry might be in your config but malformed. Check the config:

```bash
cat ~/.copilot/mcp-config.json
```

Verify the `app-modernization` entry looks like:

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

If it's malformed, remove the entry and run `gh appmod init` again.

### Custom `COPILOT_HOME` location

If you've set `COPILOT_HOME`, the config file is at `$COPILOT_HOME/mcp-config.json` instead of `~/.copilot/mcp-config.json`:

```bash
echo $COPILOT_HOME
cat "$COPILOT_HOME/mcp-config.json"
```

## Runtime Issues

### MCP server fails to start

The MCP server is downloaded via `npx` on first use. Common issues:

1. **Network issues** — Ensure you can reach the npm registry:
   ```bash
   npm ping
   ```

2. **npm cache issues** — Clear the cache:
   ```bash
   npm cache clean --force
   ```

3. **Test manually** — Try running the MCP server directly:
   ```bash
   npx -y @microsoft/github-copilot-app-modernization-mcp-server --help
   ```

### Copilot asks to trust the directory

This is normal Copilot CLI behavior. Choose one of:
- **Yes, proceed** — Trust for this session only
- **Yes, and remember** — Trust permanently for this directory

### Copilot doesn't use the App Modernization tools

Verify the MCP server is loaded:

1. Start Copilot CLI: `copilot`
2. Run `/mcp show` to see configured MCP servers
3. The `app-modernization` server should be listed

If not, run `gh appmod init` and try again.

### "Permission denied" errors

Copilot CLI asks for approval before modifying files. If you want to skip confirmations:

```bash
gh appmod upgrade --yolo "Upgrade to JDK 21"
```

### Session interrupted or lost

Copilot CLI supports session resumption:

```bash
# Resume the most recent session
copilot --continue

# Or pick from a list
copilot --resume
```

## Authentication Issues

### Not logged in to GitHub

If Copilot prompts you to log in:

```bash
# Log in via GitHub CLI
gh auth login

# Or inside Copilot CLI
/login
```

### Copilot subscription not found

Ensure you have an active [GitHub Copilot subscription](https://github.com/features/copilot/plans). If you receive Copilot through an organization, the **Copilot CLI policy** must be enabled in your organization's settings.

## Getting Help

### Check your setup

```bash
gh appmod check
```

### View extension version

```bash
gh appmod --version
```

### View Copilot CLI version

```bash
copilot --version
```

### Report an issue

If you encounter a bug or have feedback:

- [File an issue](https://github.com/microsoft/github-copilot-appmod/issues/new?template=feedback-template.yml)
- Use `/feedback` inside a Copilot CLI session

## Next Steps

- [Getting Started](getting-started.md) — Setup from scratch
- [Configuration](configuration.md) — Advanced configuration
- [Command Reference](commands.md) — Full command details
