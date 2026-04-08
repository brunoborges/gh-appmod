# Agentic Workflows for Continuous App Modernization

## Vision

Application modernization is not a one-time project — it is a **continuous discipline** within the Software Development Lifecycle (SDLC). Frameworks evolve, runtimes release new LTS versions, dependencies accumulate vulnerabilities, and cloud platforms introduce better services. Teams that treat modernization as a periodic, manual effort inevitably fall behind, accruing technical debt that compounds over time.

**GitHub Agentic Workflows** make it possible to embed app modernization directly into the SDLC — running alongside CI/CD as a **Continuous Modernization** practice. Instead of a developer remembering to run `gh appmod upgrade` locally, the repository itself continuously monitors, assesses, and proposes modernization changes through reviewable pull requests.

## Why Agentic Workflows

### The Problem with One-Off Modernization

Traditional app modernization follows a pattern:

1. A team recognizes their Java 11 / Spring Boot 2.x application is behind
2. Someone allocates a sprint (or several) to perform the upgrade
3. The upgrade is done, and the team moves on — until the next major version arrives
4. Repeat, each time with more accumulated drift

This pattern has several problems:

- **Reactive, not proactive** — Teams upgrade only when forced (EOL, CVEs, compliance)
- **High cost per event** — Large version jumps require more remediation
- **Knowledge silos** — The developer who did the last upgrade may have left
- **No guardrails** — Nothing prevents the codebase from drifting further behind
- **Inconsistency across repos** — In an organization with dozens of Java services, each is at a different version

### Continuous Modernization as an SDLC Discipline

By integrating app modernization into GitHub Agentic Workflows, modernization becomes:

| Traditional | Continuous |
|-------------|------------|
| Manual, developer-initiated | Automated, event-driven |
| Periodic (yearly, quarterly) | Continuous (weekly, on-demand) |
| Large, risky upgrades | Small, incremental changes |
| One repo at a time | Organization-wide |
| Requires local toolchain setup | Runs on GitHub Actions runners |
| Results in uncommitted local changes | Results in reviewable pull requests |

### Why GitHub Agentic Workflows Specifically

GitHub Agentic Workflows (`gh-aw`) provide the ideal foundation because:

1. **Security-first architecture** — The AI agent runs with read-only permissions in a sandboxed container. Code changes are proposed via `create-pull-request` safe outputs, never pushed directly. A threat detection layer scans all proposed changes before they reach the repository.

2. **Natural language workflows** — Modernization tasks are expressed as markdown instructions, not complex YAML pipelines. Domain experts can author and maintain workflows without CI/CD expertise.

3. **MCP server integration** — The [App Modernization MCP server](https://www.npmjs.com/package/@microsoft/github-copilot-app-modernization-mcp-server) provides specialized Java modernization tools (upgrade planning, code remediation, vulnerability scanning) that the AI agent can leverage during workflow execution.

4. **Safe outputs** — Every change the agent proposes goes through structured output validation, threat detection, and permission-controlled write jobs. The agent cannot modify the repository directly.

5. **Event-driven and scheduled triggers** — Workflows can run on a schedule (weekly upgrade checks), on demand (`workflow_dispatch`), or in response to events (new issue labeled `modernize`, new Dependabot alert).

6. **Organization-wide reach** — Using `dispatch_repository` and cross-repo operations, a single orchestrator workflow can trigger modernization across every Java repository in an organization.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Repository                         │
│                                                              │
│  .github/agents/                                             │
│  └── appmod.agent.md               (custom Copilot agent)    │
│                                                              │
│  .github/workflows/                                          │
│  ├── shared/mcp/app-modernization.md   (MCP server config)   │
│  ├── appmod-upgrade.md                 (upgrade workflow)     │
│  ├── appmod-assess.md                  (health check)         │
│  └── appmod-migrate.md                 (migration workflow)   │
└──────────────────────┬──────────────────────────────────────┘
                       │ triggers (schedule, dispatch, events)
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              GitHub Actions Runner (Sandboxed)                │
│                                                              │
│  ┌─────────────┐  ┌──────────────────┐  ┌───────────────┐   │
│  │  AI Agent    │  │  App Mod MCP     │  │  GitHub MCP   │   │
│  │  (Copilot /  │◄─┤  Server          │  │  Server       │   │
│  │   Claude /   │  │  (upgrade plan,  │  │  (repo read)  │   │
│  │   Codex)     │  │   remediation,   │  │               │   │
│  │             │◄─┤   vuln scan)     │  │               │   │
│  │  Read-only   │  └──────────────────┘  └───────────────┘   │
│  │  token only  │                                             │
│  └──────┬───────┘                                             │
│         │ proposed changes (artifact)                         │
│         ▼                                                     │
│  ┌──────────────┐  ┌────────────────┐  ┌─────────────────┐   │
│  │  Threat      │─▶│  Safe Outputs   │─▶│  Pull Request   │   │
│  │  Detection   │  │  (validated)    │  │  (reviewable)   │   │
│  └──────────────┘  └────────────────┘  └─────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Workflow Catalog

### Custom Copilot Agent (`appmod.agent.md`)

A [custom Copilot agent](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents) that enables interactive modernization through Copilot Chat — in VS Code, JetBrains, or on github.com. Developers can ask the agent to assess, upgrade, or migrate their Java application conversationally, with the App Modernization MCP server providing specialized tools.

Example interactions:
- *"Assess the modernization posture of this project"*
- *"Upgrade to JDK 21 and Spring Boot 3.4"*
- *"Migrate from S3 to Azure Blob Storage"*

### 1. Upgrade Assessment (`appmod-assess.md`)

**Trigger:** Weekly schedule + manual dispatch

Performs a non-destructive assessment of the repository's modernization posture. Creates a GitHub issue summarizing:

- Current Java version vs. latest LTS
- Current framework versions vs. latest stable
- Known vulnerabilities in dependencies
- Recommended upgrade path
- Estimated complexity

This workflow **does not modify code** — it produces an informational issue that helps teams plan and prioritize.

### 2. Java Upgrade (`appmod-upgrade.md`)

**Trigger:** Manual dispatch (with configurable target versions) + scheduled

Performs an actual upgrade of the Java application:

- Upgrades JDK version
- Updates framework dependencies
- Applies code remediation for breaking changes
- Validates the build
- Creates a pull request with all changes

The PR includes a detailed summary of what was changed and why, making code review straightforward.

### 3. Service Migration (`appmod-migrate.md`)

**Trigger:** Manual dispatch with migration task description

Migrates application services to Azure equivalents using the predefined migration tasks in the App Modernization MCP server:

- Message queues (RabbitMQ/ActiveMQ/SQS → Azure Service Bus)
- Storage (S3 → Azure Blob Storage)
- Databases (Oracle → PostgreSQL)
- Security (hardcoded secrets → Azure Key Vault)
- Identity (LDAP → Microsoft Entra ID)

### 4. Dependency Updates (`appmod-dependencies.md`)

**Trigger:** Weekly schedule (Wednesday) + manual dispatch

A lightweight, focused workflow that updates dependencies without changing the Java runtime or application framework version:

- Scans for available dependency updates (patch, minor, major)
- Categorizes updates by risk level
- Applies updates and validates the build
- Creates a PR with a summary table of all changes

Supports configurable scope: `all` (default), `direct-only`, or `security-only`.

This workflow complements the full upgrade workflow — it handles the routine dependency hygiene between major upgrades.

### 5. Security Scan (`appmod-security.md`)

**Trigger:** Twice weekly (Monday, Thursday) + manual dispatch

Scans dependencies for known vulnerabilities (CVEs) and optionally applies fixes:

- Generates a complete dependency tree (direct and transitive)
- Identifies CVEs with severity filtering (critical, high, medium, low)
- In `remediate` mode: upgrades to patched versions and creates a PR
- In `report-only` mode: creates an issue with findings and recommendations
- Prefers patch version bumps over major version jumps for safety

Configurable inputs: minimum severity threshold and action mode (remediate vs. report-only).

## Getting Started

### Prerequisites

- [GitHub CLI](https://cli.github.com/) with the `gh-aw` extension installed
- The `gh-appmod` extension installed
- A GitHub repository containing a Java application
- A [GitHub Copilot subscription](https://github.com/features/copilot/plans) (for the Copilot engine)

### 1. Install the gh-aw CLI extension

```bash
gh extension install github/gh-aw
```

### 2. Add the workflows to your repository

From your Java project root:

```bash
gh appmod add-workflows
```

This copies the agentic workflow files into `.github/workflows/` and compiles them with `gh aw compile` (if available).

### 3. Commit and push

```bash
git add .github/workflows/
git commit -m "Add continuous app modernization workflows"
git push
```

### 4. Trigger your first run

```bash
# Run a modernization assessment
gh workflow run appmod-assess.lock.yml

# Or trigger an upgrade
gh workflow run appmod-upgrade.lock.yml -f target-jdk=21 -f target-framework="Spring Boot 3.4"
```

## Security Considerations

All workflows inherit the security guarantees of GitHub Agentic Workflows:

- **Read-only agent token** — The AI agent cannot push code, create branches, or modify repository settings
- **Sandboxed execution** — The agent runs in an isolated container with a network firewall
- **Threat detection** — AI-powered scanning of all proposed changes before they are applied
- **Safe outputs** — Code changes land as pull requests, requiring human review before merge
- **No secrets in the agent** — Write tokens and API keys exist only in post-agent jobs

### Network Access

The workflows configure network allowlists for domains the agent needs:

- `registry.npmjs.org` — To resolve the App Modernization MCP server package
- `repo.maven.apache.org` — For Maven dependency resolution during build validation
- `plugins.gradle.org` — For Gradle plugin resolution (if applicable)

## Future Directions

- **Organization-wide orchestration** — A meta-workflow that dispatches modernization across all Java repos in an org
- **Dependabot integration** — Trigger upgrade workflows when Dependabot detects outdated Java/framework versions
- **PR feedback loop** — If a modernization PR's CI fails, dispatch a follow-up agent to diagnose and fix
- **Modernization dashboard** — Aggregate assessment results across repos into a GitHub Project board
- **Custom migration tasks** — Allow teams to define their own migration recipes as MCP server extensions

## Learn More

- [GitHub Agentic Workflows](https://github.github.com/gh-aw/) — Official documentation
- [App Modernization MCP Server](https://www.npmjs.com/package/@microsoft/github-copilot-app-modernization-mcp-server) — The MCP server powering these workflows
- [GitHub Copilot App Modernization for Java](https://aka.ms/ghcp-appmod) — The broader app modernization initiative
- [Continuous AI by GitHub Next](https://githubnext.com/projects/continuous-ai) — The vision behind agentic CI/CD
