# gh-appmod

A [GitHub CLI](https://cli.github.com/) extension that brings **continuous Java modernization** to your repositories through [GitHub Agentic Workflows](https://github.github.com/gh-aw/). Instead of running one-off upgrades, your repositories continuously assess, upgrade, and harden themselves — producing reviewable pull requests on a schedule.

> **`modernize` does the work. `gh appmod` makes it continuous.**

## Continuous Modernization

Application modernization shouldn't be a periodic, manual effort that falls behind. With `gh appmod`, modernization becomes part of your SDLC:

| Workflow | Schedule | Output |
|----------|----------|--------|
| [`appmod-assess`](.github/workflows/appmod-assess.md) | Weekly | GitHub issue with modernization posture |
| [`appmod-upgrade`](.github/workflows/appmod-upgrade.md) | Monthly | PR with JDK, framework, and dependency upgrades |
| [`appmod-dependencies`](.github/workflows/appmod-dependencies.md) | Weekly | PR with dependency updates |
| [`appmod-security`](.github/workflows/appmod-security.md) | Twice weekly | PR with vulnerability fixes |
| [`appmod-migrate`](.github/workflows/appmod-migrate.md) | On demand | PR migrating services to Azure |

All workflows run in a sandboxed container with read-only access. Changes are proposed through pull requests — the agent never pushes directly.

```bash
# Check your repo's modernization health anytime
gh appmod status
```

📖 **[Agentic Workflows Guide →](docs/agentic-workflows.md)**

## Getting Started

### 1. Install

```bash
gh extension install brunoborges/gh-appmod
```

### 2. Set up your repository

```bash
gh appmod check            # Verify prerequisites
gh appmod init             # Configure MCP server + custom Copilot agent
gh appmod add-skills       # Install Java upgrade skills
gh appmod add-workflows    # Enable continuous modernization workflows
```

### 3. Commit and push

```bash
git add .github/
git commit -m "Enable continuous app modernization"
git push
```

### 4. Monitor

```bash
gh appmod status           # Check workflow health and open PRs
```

📖 **[Full Getting Started Guide →](docs/getting-started.md)**

## What Gets Installed

### Agentic Workflows

Five workflow templates powered by [GitHub Agentic Workflows](https://github.github.com/gh-aw/) and the [App Modernization MCP server](https://www.npmjs.com/package/@microsoft/github-copilot-app-modernization-mcp-server). Each produces reviewable PRs or issues — no manual intervention required.

### Modernization Skills

Eight [agentskills.io](https://agentskills.io/)-compatible skills installed into `.github/skills/`, including version-specific Java upgrade instructions (8→11, 11→17, 17→21, 21→25). These are automatically picked up by both the agentic workflows and the [Modernize CLI](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview).

### Custom Copilot Agent

A [custom Copilot agent](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents) (`.github/agents/appmod.agent.md`) for interactive modernization in Copilot Chat:

> `@appmod` assess this project's modernization posture

## Commands

| Command | Description |
|---------|-------------|
| `gh appmod check` | Verify prerequisites (Node.js, npm) |
| `gh appmod init` | Configure MCP server and add custom Copilot agent |
| `gh appmod add-skills` | Install modernization skills into `.github/skills/` |
| `gh appmod add-workflows` | Add agentic workflows for continuous modernization |
| `gh appmod status` | Show modernization status and workflow health |

📖 **[Command Reference →](docs/commands.md)**

## Prerequisites

- [GitHub CLI](https://cli.github.com/) (`gh`)
- [Node.js](https://nodejs.org/) 22+ and [npm](https://www.npmjs.com/) 10+
- A [GitHub Copilot subscription](https://github.com/features/copilot/plans) (Pro, Pro+, Business, or Enterprise)
- [`gh-aw` extension](https://github.github.com/gh-aw/) (for compiling agentic workflows)
- [Modernize CLI](https://github.com/microsoft/modernize-cli) (for one-off upgrade/migrate/deploy operations)

### Installing the Modernize CLI

```bash
# macOS / Linux
brew tap microsoft/modernize https://github.com/microsoft/modernize-cli
brew install modernize

# Windows
winget install GitHub.Copilot.modernization.agent
```

📖 **[Modernize CLI Quickstart →](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/quickstart)**

## Documentation

| Guide | Description |
|-------|-------------|
| [Getting Started](docs/getting-started.md) | Installation, setup, and first steps |
| [Command Reference](docs/commands.md) | Full details on every command and option |
| [Agentic Workflows](docs/agentic-workflows.md) | Continuous modernization via GitHub Agentic Workflows |
| [Configuration](docs/configuration.md) | MCP server and Copilot CLI configuration |
| [Troubleshooting](docs/troubleshooting.md) | Common issues and solutions |

## Learn More

- [Modernize CLI](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview) — Run individual modernization operations
- [GitHub Agentic Workflows](https://github.github.com/gh-aw/) — The platform powering continuous modernization
- [agentskills.io](https://agentskills.io/) — Skill specification for AI agents
- [GitHub Copilot App Modernization](https://aka.ms/ghcp-appmod) — The broader initiative
- [Feedback](https://aka.ms/ghcp-appmod/feedback)

## License

See [LICENSE](LICENSE) for details.
