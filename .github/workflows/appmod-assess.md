---
description: "Assess the modernization posture of this Java application and report findings as a GitHub issue."

on:
  schedule:
    - cron: "0 9 * * 1" # Every Monday at 9:00 UTC
  workflow_dispatch:

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
  github:
    toolsets: [repos, issues, pull_requests]

safe-outputs:
  create-issue:
    title-prefix: "[appmod-assess] "
    labels: [app-modernization, assessment]
    close-older-issues: true
    expires: 30d
---

# Java Application Modernization Assessment

Perform a comprehensive modernization assessment of this Java application. Your goal is to
produce an actionable report that helps the team understand their current modernization posture
and prioritize next steps.

## Analysis Steps

### 1. Project Structure Discovery

- Identify the build system (Maven, Gradle, or both)
- Find the main `pom.xml` or `build.gradle` / `build.gradle.kts` files
- Determine if this is a multi-module project

### 2. Runtime and Framework Versions

Using the App Modernization MCP server tools, analyze the project to determine:

- **Current Java version** (source/target compatibility, toolchain, CI config)
- **Current framework** and version (Spring Boot, Jakarta EE, Quarkus, Micronaut, etc.)
- **Latest available LTS Java version** and how far behind the project is
- **Latest stable framework version** and the upgrade distance

### 3. Dependency Health

- Identify dependencies with known vulnerabilities (CVEs)
- Flag dependencies that are significantly outdated (2+ major versions behind)
- Note any dependencies that are end-of-life or unmaintained

### 4. Cloud Readiness

- Identify hardcoded configuration (secrets, connection strings, file paths)
- Check for cloud-unfriendly patterns (local file I/O, in-memory sessions, etc.)
- Note any vendor-specific SDK usage (AWS, GCP) that could be migrated to Azure

### 5. Modernization Recommendations

Based on the analysis, provide:

- A **priority-ordered list** of recommended modernization actions
- **Estimated complexity** for each action (Low / Medium / High)
- **Risk assessment** for each action
- Suggested order of operations (what to upgrade first)

## Output Format

Create a single GitHub issue with the assessment report. Structure the issue body with clear
markdown sections, tables for version comparisons, and actionable recommendations. Include a
summary table at the top:

| Area | Current | Target | Gap | Priority |
|------|---------|--------|-----|----------|
| Java | ... | ... | ... | ... |
| Framework | ... | ... | ... | ... |
| Dependencies | ... | ... | ... | ... |

End the report with concrete next steps the team can take, referencing the `appmod-upgrade`
and `appmod-migrate` workflows where applicable.
