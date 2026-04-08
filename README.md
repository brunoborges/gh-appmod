# gh-appmod

A [GitHub CLI](https://cli.github.com/) extension that sets up your repository for [GitHub Copilot App Modernization for Java](https://aka.ms/ghcp-appmod). It configures the MCP server, adds a custom Copilot agent, installs modernization skills, and sets up agentic workflows — so you can focus on modernizing with the [Modernize CLI](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview).

> **Note:** For running upgrade, migrate, and deploy operations, use the [Modernize CLI](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview) (`modernize` command). This extension handles setup and repo scaffolding.

## Prerequisites

- [GitHub CLI](https://cli.github.com/) (`gh`)
- [Node.js](https://nodejs.org/) 22+ and [npm](https://www.npmjs.com/) 10+
- A [GitHub Copilot subscription](https://github.com/features/copilot/plans) (Pro, Pro+, Business, or Enterprise)

## Getting Started

### 1. Install

```bash
gh extension install brunoborges/gh-appmod
```

### 2. Check prerequisites

```bash
gh appmod check
```

### 3. Configure MCP server and agent (one-time)

```bash
gh appmod init
```

### 4. Add modernization skills to your repo

```bash
gh appmod add-skills
```

This installs [agentskills.io](https://agentskills.io/)-compatible skills into `.github/skills/` — including Java upgrade instructions (8→11, 11→17, 17→21, 21→25), migration tasks, assessment, and deployment guidance.

### 5. Modernize your Java app

Use the [Modernize CLI](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview) to run modernization operations:

```bash
modernize assess
modernize upgrade
modernize migrate
modernize deploy
```

Or interact with the custom Copilot agent in Copilot Chat:

> `@appmod` assess this project's modernization posture

📖 **[Full Getting Started Guide →](docs/getting-started.md)**

## Commands

| Command | Description |
|---------|-------------|
| `gh appmod check` | Verify prerequisites (Node.js, npm, etc.) |
| `gh appmod init` | Configure MCP server and add custom Copilot agent |
| `gh appmod add-skills` | Install modernization skills into `.github/skills/` |
| `gh appmod add-workflows` | Add agentic workflows for continuous modernization |

📖 **[Command Reference →](docs/commands.md)**

## What This Extension Sets Up

### MCP Server Configuration

`gh appmod init` adds the [App Modernization MCP server](https://www.npmjs.com/package/@microsoft/github-copilot-app-modernization-mcp-server) to `~/.copilot/mcp-config.json`, making it available to Copilot CLI and Copilot Chat.

### Custom Copilot Agent

`gh appmod init` adds a [custom Copilot agent](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents) (`.github/agents/appmod.agent.md`) to your repository. This lets you interact with the App Modernization MCP server **conversationally** in VS Code, JetBrains, or on github.com.

### Modernization Skills

`gh appmod add-skills` installs skills following the [agentskills.io](https://agentskills.io/) specification into `.github/skills/`. These skills are automatically picked up by the Modernize CLI when creating upgrade and migration plans.

Available skills:

| Skill | Description |
|-------|-------------|
| `assess` | Modernization posture assessment |
| `upgrade` | General Java upgrade guidance |
| `migrate` | Azure service migration tasks |
| `deploy` | Azure deployment guidance |
| `java-8-to-11` | JDK 8 → 11 instructions + checklist |
| `java-11-to-17` | JDK 11 → 17 instructions + checklist |
| `java-17-to-21` | JDK 17 → 21 instructions + checklist |
| `java-21-to-25` | JDK 21 → 25 instructions + checklist |

### Agentic Workflows (Continuous Modernization)

`gh appmod add-workflows` adds [GitHub Agentic Workflows](https://github.github.com/gh-aw/) — modernization runs **automatically on GitHub.com**, on a schedule or triggered by events, producing reviewable pull requests.

| Workflow | Trigger | Output |
|----------|---------|--------|
| [`appmod-assess`](.github/workflows/appmod-assess.md) | Weekly schedule / manual | GitHub issue with modernization findings |
| [`appmod-upgrade`](.github/workflows/appmod-upgrade.md) | Monthly schedule / manual | PR with JDK, framework, and dependency upgrades |
| [`appmod-migrate`](.github/workflows/appmod-migrate.md) | Manual | PR migrating services to Azure equivalents |

📖 **[Agentic Workflows Proposal →](docs/agentic-workflows.md)**

## Documentation

| Guide | Description |
|-------|-------------|
| [Getting Started](docs/getting-started.md) | Installation, setup, and first steps |
| [Command Reference](docs/commands.md) | Full details on every command and option |
| [Configuration](docs/configuration.md) | MCP server and Copilot CLI configuration |
| [Agentic Workflows](docs/agentic-workflows.md) | Continuous modernization via GitHub Agentic Workflows |
| [Troubleshooting](docs/troubleshooting.md) | Common issues and solutions |

## Learn More

- [Modernize CLI](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview) — The CLI for running modernization operations
- [GitHub Copilot App Modernization for Java](https://learn.microsoft.com/azure/developer/java/migration/github-copilot-app-modernization-for-java-copilot-cli)
- [Predefined Migration Tasks](https://learn.microsoft.com/azure/developer/java/migration/migrate-github-copilot-app-modernization-for-java-predefined-tasks)
- [agentskills.io](https://agentskills.io/) — Skill specification for AI agents
- [Feedback](https://aka.ms/ghcp-appmod/feedback)

## License

See [LICENSE](LICENSE) for details.
