---
description: "Scan for security vulnerabilities in dependencies and apply fixes, producing a pull request with remediation changes."

on:
  schedule:
    - cron: "0 9 * * 1,4" # Monday and Thursday at 9:00 UTC
  workflow_dispatch:
    inputs:
      severity:
        description: "Minimum severity to remediate"
        required: false
        type: choice
        options:
          - "critical"
          - "high"
          - "medium"
          - "low"
        default: "high"
      mode:
        description: "Action mode"
        required: false
        type: choice
        options:
          - "remediate"
          - "report-only"
        default: "remediate"

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
    title-prefix: "[appmod-security] "
    labels: [app-modernization, security, vulnerability]
    base-branch: main
  create-issue:
    title-prefix: "[appmod-security] "
    labels: [app-modernization, security, vulnerability]
    close-older-issues: true
    expires: 14d
---

# Security Vulnerability Scan and Remediation

Scan this Java project for known security vulnerabilities in dependencies and, if configured,
automatically apply fixes by upgrading to patched versions.

## Configuration

- **Minimum severity:** ${{ inputs.severity || 'high' }}
- **Mode:** ${{ inputs.mode || 'remediate' }}

## Skills

If this repository contains custom skills in `.github/skills/`, read them for additional
context about the project's security requirements and constraints.

## Scan Process

### 1. Dependency Inventory

Using the App Modernization MCP server tools:

- Identify the build system and project structure
- Generate a complete dependency tree (direct and transitive)
- Record all dependency coordinates (groupId, artifactId, version)

### 2. Vulnerability Scan

Scan dependencies for known vulnerabilities:

- Use `mvn org.owasp:dependency-check-maven:check` or equivalent Gradle plugin
- Alternatively, analyze the dependency tree against the App Modernization MCP server's vulnerability database
- For each vulnerability found, record:
  - **CVE ID** and description
  - **Severity** (Critical / High / Medium / Low)
  - **Affected dependency** and version
  - **Fixed version** (if available)
  - **CVSS score**

### 3. Filter by Severity

Apply the minimum severity filter:

- `critical` — Only CVEs with CVSS ≥ 9.0
- `high` — CVEs with CVSS ≥ 7.0
- `medium` — CVEs with CVSS ≥ 4.0
- `low` — All CVEs

### 4. Remediation (if mode is "remediate")

For each vulnerability with an available fix:

- Update the dependency to the minimum version that resolves the CVE
- Prefer patch version bumps over major version jumps
- If a fix requires a major version change:
  - Apply the update
  - Run compilation to check for breakage
  - If breakage occurs, document it and move on (don't block other fixes)
- For transitive dependency vulnerabilities:
  - Add explicit version management in the parent POM or dependency management section
  - Use `<dependencyManagement>` (Maven) or `constraints` (Gradle) to pin safe versions

After applying fixes:

- Run `mvn clean compile` or `gradle build`
- Run tests
- Document any failures

### 5. Create Output

**If mode is "remediate" and fixes were applied:**

Create a pull request with:

- **Security summary** — number of vulnerabilities found, fixed, and remaining
- **Vulnerability table:**

  | CVE | Severity | Dependency | Old Version | Fixed Version | Status |
  |-----|----------|-----------|-------------|---------------|--------|
  | ... | ... | ... | ... | ... | ✅ Fixed / ⚠️ Breaking |

- **Build status** — compilation and test results
- **Manual attention** — vulnerabilities that could not be auto-fixed
- **Risk assessment** — what happens if unfixed vulnerabilities are left

**If mode is "report-only" or no fixes are available:**

Create a GitHub issue with:

- **Vulnerability report** — full table of findings
- **Recommended actions** — prioritized list of remediation steps
- **Dependencies to watch** — transitive dependencies introducing risk
- **Next steps** — suggest running in `remediate` mode if fixes are available

**If no vulnerabilities are found:**

Create a brief issue confirming the project is clean at the current severity threshold.
Close any older security assessment issues.
