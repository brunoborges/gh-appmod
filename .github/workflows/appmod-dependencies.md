---
description: "Update project dependencies to latest compatible versions, producing a pull request with all changes."

on:
  schedule:
    - cron: "0 9 * * 3" # Every Wednesday at 9:00 UTC
  workflow_dispatch:
    inputs:
      scope:
        description: "Dependency update scope"
        required: false
        type: choice
        options:
          - "all"
          - "direct-only"
          - "security-only"
        default: "all"

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
    title-prefix: "[appmod-dependencies] "
    labels: [app-modernization, dependencies]
    base-branch: main
  create-issue:
    title-prefix: "[appmod-dependencies] "
    labels: [app-modernization, dependencies, needs-attention]
---

# Dependency Update

Update this Java project's dependencies to their latest compatible versions. This is a
lightweight, focused workflow — it updates dependencies without changing the Java runtime
or application framework version.

## Configuration

- **Scope:** ${{ inputs.scope || 'all' }}

## Skills

If this repository contains custom skills in `.github/skills/`, read them for additional
context about the project's modernization goals and constraints.

## Update Process

### 1. Analyze Current Dependencies

Using the App Modernization MCP server tools:

- Identify the build system (Maven/Gradle) and project structure
- List all current dependencies and their versions
- Determine which dependencies have newer versions available
- Identify any dependencies with known vulnerabilities (CVEs)

### 2. Categorize Updates

Group available updates by risk level:

| Category | Description | Examples |
|----------|-------------|---------|
| **Patch** | Bug fixes, no API changes | 3.2.1 → 3.2.5 |
| **Minor** | New features, backward compatible | 3.2.x → 3.3.0 |
| **Major** | Breaking changes possible | 3.x → 4.0.0 |
| **Security** | Fixes known CVEs | Any version with CVE fix |

If scope is `security-only`, only apply security-related updates.
If scope is `direct-only`, skip transitive dependency updates.
If scope is `all`, apply all available updates.

### 3. Apply Updates

Apply dependency updates systematically:

- Update version numbers in `pom.xml` / `build.gradle`
- For Maven, prefer updating properties (`<dependency.version>`) over inline versions
- For multi-module projects, update the parent POM/build first
- Do NOT change the Java version or application framework version
- Do NOT change plugin versions unless required for compatibility

### 4. Build Validation

After applying updates:

- Run `mvn clean compile` or `gradle build` (compilation first)
- If compilation fails, identify the breaking dependency and either:
  - Fix the code if the change is minor
  - Revert that specific dependency update if the fix is non-trivial
- Run tests: `mvn test` or `gradle test`
- Document any test failures

### 5. Create Pull Request

Create a pull request with all dependency changes. The PR description should include:

- **Summary table** of all dependencies updated (name, old version → new version, category)
- **Security fixes** — highlight any CVE fixes prominently
- **Breaking changes** — note any major version bumps and what changed
- **Build status** — whether compilation and tests pass
- **Reverted updates** — any updates that were rolled back due to incompatibility

If some updates cause build failures that cannot be resolved, create the PR with the
successful updates and create a separate issue documenting the problematic dependencies.
