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

- If inside a Git repository, copies `.github/agents/appmod.agent.md` into the repo
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
| `.github/agents/appmod.agent.md` | Custom Copilot agent for interactive modernization in Copilot Chat |

### Environment Variables

| Variable | Effect |
|----------|--------|
| `COPILOT_HOME` | Override the Copilot config directory (default: `~/.copilot`) |

---

## `gh appmod add-skills`

Install modernization skills into your repository following the [agentskills.io](https://agentskills.io/) specification.

### Usage

```bash
gh appmod add-skills
```

### Behavior

- Must be run from inside a Git repository
- Copies skill files from the extension's `skills/` directory into `.github/skills/` in your repo
- Each skill is a subfolder containing a `SKILL.md` file (and optionally a `checklist.md`)
- Skips files that already exist (safe to re-run)
- Skills are automatically detected by the [Modernize CLI](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview)

### Skills Installed

| Skill | Description |
|-------|-------------|
| `assess` | Modernization posture assessment |
| `upgrade` | General Java upgrade guidance |
| `migrate` | Azure service migration tasks |
| `deploy` | Azure deployment guidance |
| `java-8-to-11` | JDK 8 → 11 upgrade instructions + checklist |
| `java-11-to-17` | JDK 11 → 17 upgrade instructions + checklist |
| `java-17-to-21` | JDK 17 → 21 upgrade instructions + checklist |
| `java-21-to-25` | JDK 21 → 25 upgrade instructions + checklist |

### Files Added

Skills are installed into `.github/skills/{name}/`:

```
.github/skills/
├── assess/SKILL.md
├── upgrade/SKILL.md
├── migrate/SKILL.md
├── deploy/SKILL.md
├── java-8-to-11/SKILL.md
├── java-8-to-11/checklist.md
├── java-11-to-17/SKILL.md
├── java-11-to-17/checklist.md
├── java-17-to-21/SKILL.md
├── java-17-to-21/checklist.md
├── java-21-to-25/SKILL.md
└── java-21-to-25/checklist.md
```

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
