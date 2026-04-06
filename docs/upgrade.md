# Upgrade Guide

Use `gh appmod upgrade` to modernize your Java application to newer runtime and framework versions.

## Overview

The upgrade command launches GitHub Copilot CLI with the App Modernization MCP server to help you:

- Upgrade your Java (JDK) version
- Update framework versions (Spring Boot, Jakarta EE, etc.)
- Remediate breaking changes in code
- Update dependencies to compatible versions
- Build and validate the upgraded project

## How It Works

When you run `gh appmod upgrade`, Copilot will:

1. **Analyze** — Scan your project to determine the current Java version, framework versions, and dependencies
2. **Plan** — Generate an upgrade plan listing all required changes
3. **Remediate** — Apply code changes to address API differences, deprecated methods, and breaking changes
4. **Build** — Compile the project to verify the changes
5. **Validate** — Check for vulnerabilities and remaining issues
6. **Summarize** — Provide a report of all changes made

## Common Upgrade Scenarios

### Upgrade to a Specific JDK Version

```bash
gh appmod upgrade "Upgrade this project to JDK 21"
```

### Upgrade JDK and Spring Boot Together

```bash
gh appmod upgrade "Upgrade to JDK 21 and Spring Boot 3.2"
```

### Upgrade to the Latest LTS

```bash
gh appmod upgrade "Upgrade to the latest Java LTS version"
```

### Upgrade Framework Only

```bash
gh appmod upgrade "Upgrade Spring Boot to version 3.4"
```

### Upgrade with Vulnerability Check

```bash
gh appmod upgrade "Upgrade to JDK 21, update all dependencies to latest stable versions, and check for known vulnerabilities"
```

## Tips

### Use Auto-Approval for Batch Upgrades

If you're confident in the upgrade process and want to speed it up:

```bash
gh appmod upgrade --yolo "Upgrade to JDK 21 and Spring Boot 3.2"
```

### Review Changes Before Committing

After the upgrade completes, review the changes:

```bash
git diff
```

If everything looks good, commit:

```bash
git add -A && git commit -m "Upgrade to JDK 21 and Spring Boot 3.2"
```

If something doesn't look right, reset:

```bash
git checkout .
```

### Work on a Branch

It's a good idea to create a branch before upgrading:

```bash
git checkout -b upgrade/jdk21-spring-boot-3.2
gh appmod upgrade "Upgrade to JDK 21 and Spring Boot 3.2"
```

### Upgrade Incrementally

For large projects, consider upgrading incrementally rather than jumping multiple major versions at once:

```bash
# Step 1: Upgrade JDK first
gh appmod upgrade "Upgrade to JDK 17"

# Step 2: Then upgrade the framework
gh appmod upgrade "Upgrade Spring Boot to 3.2"

# Step 3: Then modernize dependencies
gh appmod upgrade "Update all dependencies to their latest compatible versions"
```

## What Gets Changed

A typical upgrade may modify:

| File Type | Changes |
|-----------|---------|
| `pom.xml` / `build.gradle` | Java version, dependency versions, plugin versions |
| `*.java` | Deprecated API replacements, new API usage, import changes |
| `application.properties` / `application.yml` | Updated configuration keys |
| `Dockerfile` | Base image version |
| Other configs | Framework-specific configuration updates |

## Next Steps

- [Migration Guide](migration.md) — Migrate services to Azure
- [Deployment Guide](deployment.md) — Deploy your upgraded application
- [Command Reference](commands.md) — Full command details
