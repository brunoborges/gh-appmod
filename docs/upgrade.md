# Upgrade Guide

For Java upgrade operations, use the [Modernize CLI](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview):

```bash
modernize upgrade
```

## Skills

This extension includes detailed Java upgrade skills that you can install into your repository with `gh appmod add-skills`. These provide version-specific instructions and checklists:

| Skill | Path |
|-------|------|
| JDK 8 → 11 | `.github/skills/java-8-to-11/` |
| JDK 11 → 17 | `.github/skills/java-11-to-17/` |
| JDK 17 → 21 | `.github/skills/java-17-to-21/` |
| JDK 21 → 25 | `.github/skills/java-21-to-25/` |

The Modernize CLI automatically detects these skills when creating upgrade plans.

## Learn More

- [Modernize CLI — Upgrade](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview)
- [Java Upgrade Guide Instructions](https://github.com/brunoborges/java-upgrade-guide-instructions) — Source for version-specific skills
- [Command Reference](commands.md)
