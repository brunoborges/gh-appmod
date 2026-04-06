# gh-appmod

A [GitHub CLI](https://cli.github.com/) extension for [GitHub Copilot App Modernization for Java](https://aka.ms/ghcp-appmod). Upgrade, migrate, and deploy Java applications using GitHub Copilot CLI and the App Modernization MCP server — directly from the terminal.

## Prerequisites

- [GitHub CLI](https://cli.github.com/) (`gh`)
- [GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli) (`copilot`)
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

### 3. Set up (one-time)

```bash
gh appmod setup
```

This configures the local MCP server for CLI use and, when run from a Git repository, adds agentic workflow files for continuous modernization.

### 4. Modernize your Java app

Navigate to your Java project folder, then:

```bash
# Upgrade Java version and framework
gh appmod upgrade "Upgrade to JDK 21 and Spring Boot 3.2"

# Migrate services to Azure
gh appmod migrate "Migrate from S3 to Azure Blob Storage"

# Deploy to Azure
gh appmod deploy "Deploy to Azure App Service"
```

Each command launches Copilot CLI in interactive mode — you collaborate with Copilot step by step, approving changes as you go.

> **Tip:** Add `--yolo` to auto-approve all actions: `gh appmod upgrade --yolo`

📖 **[Full Getting Started Guide →](docs/getting-started.md)**

## Commands

| Command | Description |
|---------|-------------|
| `gh appmod check` | Verify prerequisites (Node.js, npm, Copilot CLI) |
| `gh appmod setup` | Set up local MCP server and agentic workflows |
| `gh appmod upgrade [prompt]` | Upgrade Java version and framework |
| `gh appmod migrate [prompt]` | Migrate services to Azure |
| `gh appmod deploy [prompt]` | Deploy to Azure |

All action commands accept `--yolo`, `--model <name>`, and other [Copilot CLI flags](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli) as pass-through options.

📖 **[Command Reference →](docs/commands.md)**

## How It Works

This extension wraps the [GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli) with the [App Modernization MCP server](https://www.npmjs.com/package/@microsoft/github-copilot-app-modernization-mcp-server). When you run a command:

1. Ensures the MCP server is available (from config or injected inline)
2. Launches `copilot` in interactive mode with your prompt
3. Copilot analyzes your project, generates a plan, applies changes, and validates

## Agentic Workflows (Continuous Modernization)

Turn app modernization into a continuous SDLC practice — not a one-off task. Using [GitHub Agentic Workflows](https://github.github.com/gh-aw/), modernization runs automatically on GitHub.com, producing reviewable pull requests.

| Workflow | Trigger | Description |
|----------|---------|-------------|
| [`appmod-assess`](.github/workflows/appmod-assess.md) | Weekly schedule / manual | Assesses modernization posture, creates an issue with findings |
| [`appmod-upgrade`](.github/workflows/appmod-upgrade.md) | Monthly schedule / manual | Upgrades Java version, framework, and dependencies via PR |
| [`appmod-migrate`](.github/workflows/appmod-migrate.md) | Manual | Migrates services to Azure equivalents via PR |

📖 **[Agentic Workflows Proposal →](docs/agentic-workflows.md)**

## Documentation

| Guide | Description |
|-------|-------------|
| [Getting Started](docs/getting-started.md) | Installation, setup, and first steps |
| [Command Reference](docs/commands.md) | Full details on every command and option |
| [Upgrade Guide](docs/upgrade.md) | Java and framework upgrade scenarios |
| [Migration Guide](docs/migration.md) | Migration scenarios and predefined tasks |
| [Deployment Guide](docs/deployment.md) | Deploying to Azure services |
| [Configuration](docs/configuration.md) | MCP server and Copilot CLI configuration |
| [Agentic Workflows](docs/agentic-workflows.md) | Continuous modernization via GitHub Agentic Workflows |
| [Troubleshooting](docs/troubleshooting.md) | Common issues and solutions |

## Learn More

- [GitHub Copilot App Modernization for Java](https://learn.microsoft.com/azure/developer/java/migration/github-copilot-app-modernization-for-java-copilot-cli)
- [Predefined Migration Tasks](https://learn.microsoft.com/azure/developer/java/migration/migrate-github-copilot-app-modernization-for-java-predefined-tasks)
- [GitHub Copilot CLI Documentation](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli)
- [Feedback](https://aka.ms/ghcp-appmod/feedback)

## License

See [LICENSE](LICENSE) for details.
