# gh-appmod

A [GitHub CLI](https://cli.github.com/) extension for [GitHub Copilot App Modernization for Java](https://aka.ms/ghcp-appmod). Helps developers upgrade, migrate, and deploy Java applications using GitHub Copilot CLI and the App Modernization MCP server — directly from the terminal.

## Prerequisites

- [GitHub CLI](https://cli.github.com/) (`gh`)
- [GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli) (`copilot`)
- [Node.js](https://nodejs.org/) 22+
- [npm](https://www.npmjs.com/) 10+
- A [GitHub Copilot subscription](https://github.com/features/copilot/plans)

## Installation

```bash
gh extension install <owner>/gh-appmod
```

Or install locally for development:

```bash
git clone <repo-url>
cd gh-appmod
gh extension install .
```

## Quick Start

```bash
# 1. Verify prerequisites
gh appmod check

# 2. Configure the App Modernization MCP server
gh appmod setup

# 3. Start modernizing!
gh appmod upgrade
```

## Commands

### `gh appmod check`

Verifies that all prerequisites are installed and properly configured.

```bash
gh appmod check
```

### `gh appmod setup`

Configures the App Modernization MCP server in your Copilot CLI config (`~/.copilot/mcp-config.json`). This is a one-time setup.

```bash
gh appmod setup
```

### `gh appmod upgrade [prompt]`

Launches Copilot CLI to upgrade your Java application. Provide a custom prompt or use the default.

```bash
gh appmod upgrade
gh appmod upgrade "Upgrade to JDK 21 and Spring Boot 3.2"
gh appmod upgrade --yolo "Upgrade to latest Java LTS"
```

### `gh appmod migrate [prompt]`

Launches Copilot CLI to migrate your Java application to Azure.

```bash
gh appmod migrate
gh appmod migrate "Migrate from S3 to Azure Blob Storage"
gh appmod migrate "Migrate from Oracle DB to Azure PostgreSQL"
```

### `gh appmod deploy [prompt]`

Launches Copilot CLI to deploy your Java application to Azure.

```bash
gh appmod deploy
gh appmod deploy "Deploy to Azure App Service"
gh appmod deploy "Deploy to Azure Kubernetes Service"
```

### Common Options

All action commands (`upgrade`, `migrate`, `deploy`) accept:

| Option | Description |
|--------|-------------|
| `--yolo` | Enable all Copilot permissions (no confirmations) |
| `--model <name>` | Use a specific AI model |
| `-h`, `--help` | Show command help |

Extra flags are passed through directly to the `copilot` CLI.

## How It Works

This extension wraps the [GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli) with the [App Modernization MCP server](https://www.npmjs.com/package/@microsoft/github-copilot-app-modernization-mcp-server) (`@microsoft/github-copilot-app-modernization-mcp-server`).

When you run a command like `gh appmod upgrade`, the extension:

1. Checks if the MCP server is configured in `~/.copilot/mcp-config.json`
2. If not, injects it inline via `--additional-mcp-config` for the session
3. Launches `copilot` in interactive mode with your prompt (`-i`)

## Learn More

- [GitHub Copilot App Modernization for Java](https://learn.microsoft.com/azure/developer/java/migration/github-copilot-app-modernization-for-java-copilot-cli)
- [Predefined Migration Tasks](https://learn.microsoft.com/azure/developer/java/migration/migrate-github-copilot-app-modernization-for-java-predefined-tasks)
- [GitHub Copilot CLI Documentation](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli)
- [Feedback](https://aka.ms/ghcp-appmod/feedback)

## License

See [LICENSE](LICENSE) for details.
