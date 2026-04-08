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
| `.github/workflows/appmod-dependencies.md` | Dependency updates — weekly PRs with latest compatible versions |
| `.github/workflows/appmod-security.md` | Security scan — vulnerability detection and remediation PRs |

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

## `gh appmod status`

Show the modernization status and workflow health for the current repository.

### Usage

```bash
gh appmod status
```

### Behavior

- Must be run from inside a Git repository with a GitHub remote
- Uses `gh api` to query GitHub Actions for workflow run history
- Shows the latest run status for each `appmod-*` workflow
- Lists open pull requests and issues with the `app-modernization` label
- Reports which workflows, skills, and agent are installed locally

### Output Sections

| Section | Description |
|---------|-------------|
| **Agentic Workflows** | Latest run status (✓ success, ✗ failure, ⟳ in progress) and date for each workflow |
| **Open Pull Requests** | PRs created by workflows (labeled `app-modernization`) |
| **Open Issues** | Issues created by workflows (labeled `app-modernization`) |
| **Repository Setup** | Whether workflows, skills, and agent are installed locally |

### Example Output

```
GitHub Copilot App Modernization for Java
──────────────────────────────────────────

Repository: myorg/my-java-app

Agentic Workflows

  ✓ appmod-assess          success      2026-04-07 09:00
  ✗ appmod-upgrade         failure      2026-04-01 09:00
  ✓ appmod-dependencies    success      2026-04-03 09:00
  ✓ appmod-security        success      2026-04-04 09:00
    appmod-migrate         not installed or never run

Open Pull Requests

  #42 [appmod-dependencies] Update 3 dependencies
  #39 [appmod-security] Fix CVE-2026-1234 in jackson-databind

Open Issues

  #41 [appmod-assess] Modernization Assessment — April 2026

Repository Setup

✓ 5 workflow file(s) in .github/workflows/
✓ 8 skill(s) in .github/skills/
✓ Custom agent installed
```
