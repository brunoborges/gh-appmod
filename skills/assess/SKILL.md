---
name: assess
description: Assess a Java project's modernization posture — Java version, framework versions, dependency health, vulnerabilities, and migration opportunities
---

# Assess

Evaluate a Java project's modernization posture and produce actionable findings.

## Assessment Areas

| Area | What's Checked |
|------|---------------|
| Java version | Current JDK version vs. latest LTS |
| Framework versions | Spring Boot, Jakarta EE, etc. vs. latest stable |
| Dependencies | Outdated libraries, known CVEs, deprecated artifacts |
| Migration candidates | Services with Azure equivalents (storage, messaging, identity, etc.) |
| Code patterns | Deprecated API usage, hardcoded secrets, non-portable patterns |
| Build health | Build tool version, plugin versions, build success |

## Steps

1. **Scan the project** — Analyze `pom.xml`/`build.gradle`, source code, and configuration files.
2. **Identify findings** — Categorize by area (Java version, frameworks, dependencies, migrations, code patterns).
3. **Prioritize** — Rank findings by impact and effort (quick wins first, then larger efforts).
4. **Report** — Present a structured summary with:
   - Current state (versions, dependencies, services)
   - Recommended actions (ordered by priority)
   - Estimated complexity for each action (low / medium / high)

## Guidelines

- Assessment is read-only — do not modify any project files.
- Be specific about version numbers and CVE IDs when reporting vulnerabilities.
- Distinguish between critical issues (security, EOL runtime) and improvements (newer framework features).
- When multiple upgrade paths exist, mention all viable options.
