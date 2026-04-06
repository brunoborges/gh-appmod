# Command Reference

Complete reference for all `gh appmod` commands.

## Global Options

| Option | Description |
|--------|-------------|
| `-h`, `--help` | Show help for the extension or a specific command |
| `-v`, `--version` | Show the extension version |

---

## `gh appmod check`

Verify that all prerequisites are installed and properly configured.

### Usage

```bash
gh appmod check
```

### What It Checks

| Check | Requirement |
|-------|-------------|
| GitHub CLI (`gh`) | Installed |
| Node.js | Version 22 or later |
| npm | Version 10 or later |
| Copilot CLI (`copilot`) | Installed |
| MCP server config | `app-modernization` entry in `~/.copilot/mcp-config.json` |

### Exit Codes

| Code | Meaning |
|------|---------|
| `0` | All prerequisites met |
| `1` | One or more prerequisites are missing |

---

## `gh appmod setup`

Configure the App Modernization MCP server in your Copilot CLI configuration.

### Usage

```bash
gh appmod setup
```

### Behavior

- If `~/.copilot/mcp-config.json` doesn't exist, creates it with the `app-modernization` server
- If the file exists but doesn't have the server, merges the entry into the existing config (preserving other MCP servers)
- If already configured, reports success and exits
- Uses Node.js for safe JSON manipulation when merging

### Configuration Added

The following entry is added to `~/.copilot/mcp-config.json`:

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

### Environment Variables

| Variable | Effect |
|----------|--------|
| `COPILOT_HOME` | Override the Copilot config directory (default: `~/.copilot`) |

---

## `gh appmod upgrade [prompt]`

Launch Copilot CLI to upgrade your Java application.

### Usage

```bash
gh appmod upgrade [options] [prompt]
```

### Arguments

| Argument | Description |
|----------|-------------|
| `prompt` | Custom upgrade prompt. If omitted, a default prompt is used. |

### Options

| Option | Description |
|--------|-------------|
| `--yolo` | Enable all Copilot permissions (no confirmations) |
| `--model <name>` | Use a specific AI model |
| `--effort <level>` | Set reasoning effort (`low`, `medium`, `high`, `xhigh`) |
| `--autopilot` | Enable autopilot continuation |
| `--add-dir <path>` | Allow Copilot access to an additional directory |
| `-h`, `--help` | Show command help |

All unrecognized flags are passed through to the `copilot` CLI.

### Default Prompt

When no prompt is provided:

> Upgrade this Java project to the latest Java LTS version and latest compatible framework versions. Analyze the project first, then generate an upgrade plan, perform code remediation, build the project, and check for vulnerabilities.

### Examples

```bash
# Upgrade with defaults
gh appmod upgrade

# Specific upgrade target
gh appmod upgrade "Upgrade to JDK 21 and Spring Boot 3.2"

# Auto-approve everything
gh appmod upgrade --yolo "Upgrade to latest Java LTS"

# Use a specific AI model
gh appmod upgrade --model gpt-5.2 "Upgrade to JDK 21"
```

---

## `gh appmod migrate [prompt]`

Launch Copilot CLI to migrate your Java application to Azure.

### Usage

```bash
gh appmod migrate [options] [prompt]
```

### Arguments

| Argument | Description |
|----------|-------------|
| `prompt` | Custom migration prompt. If omitted, a default prompt is used. |

### Options

Same as [`gh appmod upgrade`](#gh-appmod-upgrade-prompt).

### Default Prompt

When no prompt is provided:

> Analyze this Java project and help me migrate it to Azure. Identify the current dependencies and services, suggest appropriate Azure equivalents, and perform the migration step by step.

### Examples

```bash
# Migrate with defaults
gh appmod migrate

# Specific migration scenario
gh appmod migrate "Migrate from S3 to Azure Blob Storage"

# Database migration
gh appmod migrate "Migrate from Oracle DB to Azure PostgreSQL"

# Message queue migration with auto-approval
gh appmod migrate --yolo "Migrate from RabbitMQ to Azure Service Bus"
```

---

## `gh appmod deploy [prompt]`

Launch Copilot CLI to deploy your Java application to Azure.

### Usage

```bash
gh appmod deploy [options] [prompt]
```

### Arguments

| Argument | Description |
|----------|-------------|
| `prompt` | Custom deployment prompt. If omitted, a default prompt is used. |

### Options

Same as [`gh appmod upgrade`](#gh-appmod-upgrade-prompt).

### Default Prompt

When no prompt is provided:

> Deploy this Java application to Azure. Analyze the project, recommend the best Azure service for deployment, and guide me through the deployment process.

### Examples

```bash
# Deploy with defaults
gh appmod deploy

# Deploy to a specific service
gh appmod deploy "Deploy to Azure App Service"

# Deploy to Kubernetes
gh appmod deploy "Deploy to Azure Kubernetes Service"

# Deploy with auto-approval
gh appmod deploy --yolo "Deploy to Azure Container Apps"
```

---

## Flag Pass-Through

All action commands (`upgrade`, `migrate`, `deploy`) pass unrecognized flags through to the `copilot` CLI. This means you can use any Copilot CLI option. Some useful ones:

| Copilot CLI Flag | Description |
|-----------------|-------------|
| `--yolo` | Auto-approve all tool usage and file access |
| `--model <name>` | Choose a specific AI model |
| `--effort <level>` | Set reasoning effort level |
| `--autopilot` | Enable autopilot continuation |
| `--add-dir <path>` | Allow access to additional directories |
| `--allow-tool <tool>` | Pre-approve specific tools |
| `--deny-tool <tool>` | Block specific tools |

For the full list, run `copilot --help`.
