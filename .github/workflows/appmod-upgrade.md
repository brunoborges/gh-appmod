---
description: "Upgrade this Java application's runtime and framework versions, producing a pull request with all changes."

on:
  workflow_dispatch:
    inputs:
      target-jdk:
        description: "Target JDK version (e.g., 21, 25). Leave empty for latest LTS."
        required: false
        type: string
      target-framework:
        description: "Target framework version (e.g., 'Spring Boot 3.4'). Leave empty for auto-detect."
        required: false
        type: string
      scope:
        description: "Upgrade scope"
        required: false
        type: choice
        options:
          - "full"
          - "jdk-only"
          - "framework-only"
          - "dependencies-only"
        default: "full"
  schedule:
    - cron: "0 9 1 * *" # First day of every month at 9:00 UTC

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
    - "javac"
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
    title-prefix: "[appmod-upgrade] "
    labels: [app-modernization, upgrade]
    base-branch: main
  create-issue:
    title-prefix: "[appmod-upgrade] "
    labels: [app-modernization, upgrade, needs-attention]
---

# Java Application Upgrade

Upgrade this Java application's runtime version, framework, and dependencies. Produce a
pull request with all necessary changes.

## Configuration

- **Target JDK:** ${{ inputs.target-jdk || 'latest LTS' }}
- **Target Framework:** ${{ inputs.target-framework || 'latest stable version' }}
- **Scope:** ${{ inputs.scope || 'full' }}

## Upgrade Process

### 1. Analyze Current State

Using the App Modernization MCP server tools:

- Determine the current Java version, framework, and dependency versions
- Identify the build system (Maven/Gradle) and project structure
- Understand the current state before making any changes

### 2. Generate Upgrade Plan

Create a detailed upgrade plan using the App Modernization MCP server:

- Map out all required changes for the target version(s)
- Identify breaking changes that will require code remediation
- Order changes to minimize risk (dependencies → runtime → framework → code)

If the scope is `jdk-only`, focus exclusively on Java version changes.
If the scope is `framework-only`, focus on the application framework.
If the scope is `dependencies-only`, update dependencies to latest compatible versions.
If the scope is `full`, perform all upgrades.

### 3. Apply Changes

Apply the upgrade changes systematically:

- Update build configuration (`pom.xml`, `build.gradle`, etc.)
- Update Java source/target compatibility settings
- Replace deprecated API calls with their modern equivalents
- Update import statements for moved/renamed packages
- Fix breaking changes identified in the upgrade plan
- Update configuration files (`application.properties`, `application.yml`, etc.)

### 4. Build Validation

After applying changes, validate the build:

- Run `mvn clean compile` or `gradle build` (without tests first)
- If compilation fails, fix the issues and try again
- Once compilation succeeds, run `mvn test` or `gradle test`
- If tests fail, analyze failures and apply fixes
- Iterate until the build passes or document remaining issues

### 5. Create Pull Request

Create a pull request with all changes. The PR description should include:

- **Summary** of what was upgraded (from version X → Y)
- **Changes made** — categorized list of all modifications
- **Build status** — whether the build and tests pass
- **Known issues** — any remaining items that need manual attention
- **Migration notes** — anything reviewers should be aware of

If the build fails after reasonable attempts at remediation, still create the PR but clearly
document the failures. Also create an issue flagging the upgrade as needing human attention.

### 6. Post-Upgrade Checks

- Verify no secrets or credentials were accidentally introduced
- Confirm dependency versions are consistent across modules (for multi-module projects)
- Check that no test files were accidentally deleted or disabled
