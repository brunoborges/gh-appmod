# Migration Guide

For Azure migration operations, use the [Modernize CLI](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview):

```bash
modernize migrate
```

## Skills

This extension includes an Azure migration skill that you can install into your repository with `gh appmod add-skills`. The skill covers common migration patterns:

- S3 → Azure Blob Storage
- RabbitMQ / ActiveMQ → Azure Service Bus
- Oracle SQL → PostgreSQL
- Hardcoded secrets → Azure Key Vault
- LDAP → Microsoft Entra ID
- SMTP → Azure Communication Services
- File logging → Console logging (Azure Monitor)

The Modernize CLI automatically detects these skills when creating migration plans.

## Learn More

- [Modernize CLI — Migrate](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview)
- [Predefined Migration Tasks](https://learn.microsoft.com/azure/developer/java/migration/migrate-github-copilot-app-modernization-for-java-predefined-tasks)
- [Command Reference](commands.md)
