# Getting Started

This guide walks you through installing and using `gh appmod` for the first time.

## Step 1: Install Prerequisites

Before using `gh appmod`, make sure you have the following installed:

| Tool | Minimum Version | Install |
|------|----------------|---------|
| [GitHub CLI](https://cli.github.com/) (`gh`) | Any recent version | [Installation guide](https://github.com/cli/cli#installation) |
| [GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli) (`copilot`) | Latest | [Installation guide](https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli) |
| [Node.js](https://nodejs.org/) | 22+ | [Download](https://nodejs.org/) |
| [npm](https://www.npmjs.com/) | 10+ | Bundled with Node.js |

You also need a [GitHub Copilot subscription](https://github.com/features/copilot/plans) (Pro, Pro+, Business, or Enterprise).

> **Note:** If you receive Copilot through an organization, the **Copilot CLI policy** must be enabled in your organization's settings.

## Step 2: Install the Extension

Install `gh appmod` from GitHub:

```bash
gh extension install <owner>/gh-appmod
```

Or, to install from a local clone:

```bash
git clone <repo-url>
cd gh-appmod
gh extension install .
```

Verify the installation:

```bash
gh appmod --version
```

## Step 3: Check Prerequisites

Run the built-in check to make sure everything is properly installed:

```bash
gh appmod check
```

You should see output like:

```
GitHub Copilot App Modernization for Java
──────────────────────────────────────────

Checking prerequisites...

✓ GitHub CLI (gh version 2.x.x)
✓ Node.js v22.x.x
✓ npm v10.x.x
✓ Copilot CLI GitHub Copilot CLI x.x.x

⚠ MCP server 'app-modernization' not configured. Run 'gh appmod setup' to add it.

✓ All prerequisites met!
```

## Step 4: Configure the MCP Server

The App Modernization MCP server powers the upgrade, migration, and deployment capabilities. Set it up with:

```bash
gh appmod setup
```

This adds the `app-modernization` MCP server to your Copilot CLI configuration at `~/.copilot/mcp-config.json`. This is a one-time setup — the server will be available in all future Copilot CLI sessions.

> **Tip:** If you skip this step, `gh appmod` will still work — it automatically injects the MCP server configuration for each session. Running `setup` just makes it permanent.

## Step 5: Start Modernizing

Navigate to your Java project directory, then run one of the modernization commands:

### Upgrade your Java application

```bash
# Use the default upgrade prompt
gh appmod upgrade

# Or provide a specific prompt
gh appmod upgrade "Upgrade to JDK 21 and Spring Boot 3.2"
```

### Migrate to Azure

```bash
# Use the default migration prompt
gh appmod migrate

# Or describe a specific migration
gh appmod migrate "Migrate from S3 to Azure Blob Storage"
```

### Deploy to Azure

```bash
# Use the default deployment prompt
gh appmod deploy

# Or specify a target
gh appmod deploy "Deploy to Azure App Service"
```

Each command launches the GitHub Copilot CLI in interactive mode with the App Modernization MCP server, so you can collaborate with Copilot on the modernization task step by step.

## What Happens Next

When a modernization command runs, Copilot CLI will:

1. **Analyze your project** — Scan the codebase to understand the current state
2. **Generate a plan** — Create a step-by-step modernization plan
3. **Execute changes** — Perform code modifications with your approval
4. **Validate** — Build the project and check for issues
5. **Summarize** — Show a summary of what was changed

You remain in control throughout — Copilot asks for approval before modifying files or running commands.

> **Tip:** Use `--yolo` to auto-approve all actions if you're comfortable:
> ```bash
> gh appmod upgrade --yolo "Upgrade to JDK 21"
> ```

## Next Steps

- [Command Reference](commands.md) — Full details on every command and option
- [Upgrade Guide](upgrade.md) — Deep dive into Java upgrade scenarios
- [Migration Guide](migration.md) — Detailed migration scenarios and predefined tasks
- [Deployment Guide](deployment.md) — Deploying to Azure services
- [Agentic Workflows](agentic-workflows.md) — Enable continuous modernization via GitHub Agentic Workflows
- [Configuration](configuration.md) — MCP server and Copilot CLI configuration
- [Troubleshooting](troubleshooting.md) — Common issues and solutions
