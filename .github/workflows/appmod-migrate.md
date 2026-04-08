---
description: "Migrate Java application services to Azure equivalents, producing a pull request with all changes."

on:
  workflow_dispatch:
    inputs:
      task:
        description: "Migration task (e.g., 'Migrate from S3 to Azure Blob Storage')"
        required: true
        type: string

permissions:
  contents: read
  issues: read
  pull-requests: read

imports:
  - shared/mcp/app-modernization.md

tools:
  edit:
  bash:
    - "mvn"
    - "gradle"
    - "java"
    - "cat"
    - "find"
    - "grep"
    - "head"
    - "tail"
    - "ls"
    - "wc"
    - "diff"
    - "git:*"
  github:
    toolsets: [repos, issues, pull_requests]

safe-outputs:
  create-pull-request:
    title-prefix: "[appmod-migrate] "
    labels: [app-modernization, migration, azure]
    base-branch: main
  create-issue:
    title-prefix: "[appmod-migrate] "
    labels: [app-modernization, migration, needs-attention]
---

# Java Application Service Migration

Migrate services in this Java application to Azure equivalents using the App Modernization
MCP server's predefined migration tasks.

## Migration Task

**Requested migration:** ${{ inputs.task }}

## Migration Process

### 0. Check for Custom Skills

If this repository contains custom skills in `.github/skills/`, read them for additional
context about migration targets and patterns. In particular, check the `migrate` skill
for supported migration patterns and Azure service mappings.

### 1. Analyze Current Implementation

Using the App Modernization MCP server tools:

- Scan the project to understand its structure and build system
- Identify the specific services, SDKs, and integration points related to the migration task
- Map out all code locations that reference the source service
- Understand the current configuration (connection strings, credentials, endpoints)

### 2. Match Predefined Migration Tasks

Check if the requested migration matches one of the App Modernization MCP server's predefined
tasks. These encode industry best practices for common migrations:

- **Message queues:** RabbitMQ / ActiveMQ / AWS SQS → Azure Service Bus
- **Storage:** AWS S3 → Azure Blob Storage, local file I/O → Azure Storage File Share
- **Databases:** Oracle SQL → PostgreSQL, password auth → Managed Identity
- **Security:** Hardcoded secrets → Azure Key Vault, AWS Secrets Manager → Key Vault
- **Identity:** LDAP → Microsoft Entra ID, connection strings → Managed Identity
- **Communication:** Java Mail / SMTP → Azure Communication Services
- **Observability:** File-based logging → Console logging for Azure Monitor

If a predefined task matches, use it. Otherwise, perform the migration based on the task description.

### 3. Generate Migration Plan

Create a migration plan that includes:

- All files that need to change
- New dependencies to add and old ones to remove
- Configuration changes required
- Code changes with before/after patterns
- Any new configuration properties or environment variables needed

### 4. Apply Migration Changes

Apply changes systematically:

- Update `pom.xml` / `build.gradle` — replace source SDKs with Azure SDKs
- Update Java source files — replace service client code, update imports, change API calls
- Update configuration files — add Azure service connection properties
- Add new configuration classes or beans as needed
- Remove old service-specific code that is no longer needed

### 5. Build Validation

After applying changes:

- Run `mvn clean compile` or `gradle build` to verify compilation
- Fix any compilation errors and iterate
- Run tests to verify behavior
- Document any tests that need manual updates (e.g., integration tests with service-specific mocks)

### 6. Create Pull Request

Create a pull request with all migration changes. The PR description should include:

- **Migration summary** — what service was migrated from/to
- **Changes by category:**
  - Dependencies added/removed
  - Configuration changes
  - Code modifications
  - New files created
- **Environment variables** — any new variables the application will need at runtime
- **Azure setup required** — what Azure resources need to be provisioned
- **Testing notes** — which tests pass, which need manual attention
- **Rollback guidance** — how to revert if issues arise in production

If the build fails after reasonable remediation attempts, still create the PR but document
the failures clearly. Also create an issue flagging the migration as needing human attention.
