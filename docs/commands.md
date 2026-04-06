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

## `gh appmod init`

Configure the App Modernization MCP server and add the custom Copilot agent to your repository.

### Usage

```bash
gh appmod init
```

### Behavior

**Step 1: MCP Server Configuration**

- If `~/.copilot/mcp-config.json` doesn't exist, creates it with the `app-modernization` server
- If the file exists but doesn't have the server, merges the entry into the existing config (preserving other MCP servers)
- If already configured, reports success and continues
- Uses Node.js for safe JSON manipulation when merging

**Step 2: Custom Copilot Agent**

- If inside a Git repository, copies `.github/agents/app-modernization.agent.md` into the repo
- The agent enables interactive modernization in Copilot Chat (VS Code, JetBrains, github.com)
- Skips if the agent file already exists (safe to re-run)
- Skips if not inside a Git repository

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

### Files Added

| File | Description |
|------|-------------|
| `.github/agents/app-modernization.agent.md` | Custom Copilot agent for interactive modernization in Copilot Chat |

### Environment Variables

| Variable | Effect |
|----------|--------|
| `COPILOT_HOME` | Override the Copilot config directory (default: `~/.copilot`) |

---

## `gh appmod add-workflows`

Add GitHub Agentic Workflows for continuous app modernization to the current repository.

### Usage

```bash
gh appmod add-workflows [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--no-compile` | Skip compiling workflows with `gh aw` |
| `-h`, `--help` | Show command help |

### Behavior

- Must be run from inside a Git repository
- Copies agentic workflow files into `.github/workflows/`
- Skips files that already exist (safe to re-run)
- Attempts to compile with `gh aw compile` if `gh-aw` is installed
- Reports next steps for committing and triggering workflows

### Files Added

| File | Description |
|------|-------------|
| `.github/workflows/shared/mcp/app-modernization.md` | Shared MCP server config for the App Modernization server |
| `.github/workflows/appmod-assess.md` | Weekly assessment — reports modernization posture as a GitHub issue |
| `.github/workflows/appmod-upgrade.md` | Upgrade workflow — produces PRs with JDK/framework/dependency upgrades |
| `.github/workflows/appmod-migrate.md` | Migration workflow — produces PRs migrating services to Azure |

### Prerequisites

- [GitHub CLI](https://cli.github.com/) (`gh`)
- [`gh-aw` extension](https://github.github.com/gh-aw/) (for compiling workflows)

### Examples

```bash
# Add workflows and compile
gh appmod add-workflows

# Add workflows without compiling
gh appmod add-workflows --no-compile
```

See the [Agentic Workflows documentation](agentic-workflows.md) for details on each workflow, the security model, and customization options.

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
