---
name: upgrade
description: Upgrade Java runtime versions, frameworks, and dependencies to their latest compatible releases
---

# Upgrade

Upgrade Java runtime versions, frameworks, and dependencies.

## Version-Specific Upgrade Guides

For detailed, JEP-by-JEP upgrade instructions, see the version-specific skills:

- **Java 8 → 11** — Module system, removed Java EE modules, `var`, HTTP Client
- **Java 11 → 17** — Records, Sealed Classes, Pattern Matching, Text Blocks
- **Java 17 → 21** — Virtual Threads, Pattern Matching for switch, Record Patterns
- **Java 21 → 25** — Flexible Constructors, Stream Gatherers, Class-File API

## Steps

1. **Analyze first** — Scan the project structure, build files (`pom.xml`, `build.gradle`), and source code to determine current Java version, framework versions, and dependency versions.
2. **Generate an upgrade plan** — Produce a plan detailing what will be upgraded and what breaking changes to expect. Present the plan for review.
3. **Apply changes incrementally** — Make changes in logical groups:
   - Build configuration (Java version, plugin versions)
   - Dependencies (framework BOMs, library versions)
   - Source code (API changes, deprecated method replacements)
   - Configuration files (property renames, format changes)
4. **Validate after each group** — Run the build (`mvn clean compile` or `gradle build`) to confirm changes compile successfully. Run tests if available.
5. **Check for vulnerabilities** — After upgrading, scan for known vulnerabilities in the updated dependencies.

## Guidelines

- Prefer incremental upgrades over large version jumps when the gap is significant.
- Always explain what changes are being made and why.
- If a build fails after changes, analyze the error and attempt to fix it before reporting.
- When multiple upgrade paths exist, present options and let the developer decide.
