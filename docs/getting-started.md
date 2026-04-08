# Getting Started

This guide walks you through installing and setting up `gh appmod` for the first time. This extension configures your environment and repository for Java app modernization — the actual upgrade, migrate, and deploy operations are handled by the [Modernize CLI](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview).

## Step 1: Install Prerequisites

Before using `gh appmod`, make sure you have the following installed:

| Tool | Minimum Version | Install |
|------|----------------|---------|
| [GitHub CLI](https://cli.github.com/) (`gh`) | Any recent version | [Installation guide](https://github.com/cli/cli#installation) |
| [Node.js](https://nodejs.org/) | 22+ | [Download](https://nodejs.org/) |
| [npm](https://www.npmjs.com/) | 10+ | Bundled with Node.js |
| [Modernize CLI](https://github.com/microsoft/modernize-cli) | Latest | See below |

You also need a [GitHub Copilot subscription](https://github.com/features/copilot/plans) (Pro, Pro+, Business, or Enterprise).

> **Note:** If you receive Copilot through an organization, the **Copilot CLI policy** must be enabled in your organization's settings.

### Installing the Modernize CLI

The [Modernize CLI](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview) handles the actual upgrade, migrate, and deploy operations. Install it with:

```bash
# macOS / Linux
brew tap microsoft/modernize https://github.com/microsoft/modernize-cli
brew install modernize

# Windows
winget install GitHub.Copilot.modernization.agent
```

📖 **[Modernize CLI Quickstart →](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/quickstart)**

## Step 2: Install the Extension

Install `gh appmod` from GitHub:

```bash
gh extension install brunoborges/gh-appmod
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

⚠ MCP server 'app-modernization' not configured. Run 'gh appmod init' to add it.

✓ All prerequisites met!
```

## Step 4: Configure MCP Server and Agent

Set up the MCP server and custom Copilot agent:

```bash
gh appmod init
```

This does two things:

1. **MCP server** — Adds the `app-modernization` MCP server to `~/.copilot/mcp-config.json` (one-time, global)
2. **Custom agent** — Adds `.github/agents/appmod.agent.md` to your repo for Copilot Chat integration

## Step 5: Add Modernization Skills

Install the bundled skills into your repository:

```bash
gh appmod add-skills
```

This copies skill files into `.github/skills/` following the [agentskills.io](https://agentskills.io/) specification. Skills include Java version-specific upgrade instructions (8→11, 11→17, 17→21, 21→25), migration tasks, assessment, and deployment guidance.

These skills are automatically detected by both the agentic workflows and the Modernize CLI when creating plans.

## Step 6: Enable Continuous Modernization

This is the main event. Add agentic workflows to your repository:

```bash
gh appmod add-workflows
```

This installs five workflow templates that will continuously assess, upgrade, and harden your Java application:

| Workflow | Schedule | Output |
|----------|----------|--------|
| `appmod-assess` | Weekly | Issue with modernization posture |
| `appmod-upgrade` | Monthly | PR with JDK/framework upgrades |
| `appmod-dependencies` | Weekly | PR with dependency updates |
| `appmod-security` | Twice weekly | PR with vulnerability fixes |
| `appmod-migrate` | On demand | PR migrating services to Azure |

## Step 7: Commit, Push, and Monitor

```bash
git add .github/
git commit -m "Enable continuous app modernization"
git push
```

Check your modernization health anytime:

```bash
gh appmod status
```

## One-Off Modernization

For individual upgrade, migrate, or deploy operations, use the [Modernize CLI](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview):

```bash
modernize assess
modernize upgrade
modernize migrate
modernize deploy
```

Or interact with the custom Copilot agent in Copilot Chat (VS Code, JetBrains, github.com):

> `@appmod` assess this project's modernization posture

## Next Steps

- [Command Reference](commands.md) — Full details on every command and option
- [Configuration](configuration.md) — MCP server and Copilot CLI configuration
- [Agentic Workflows](agentic-workflows.md) — Enable continuous modernization via GitHub Agentic Workflows
- [Troubleshooting](troubleshooting.md) — Common issues and solutions
