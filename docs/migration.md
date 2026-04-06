# Migration Guide

Use `gh appmod migrate` to migrate your Java application's services and dependencies to Azure equivalents.

## Overview

The migration command launches GitHub Copilot CLI with the App Modernization MCP server, which includes a set of **predefined migration tasks** covering common migration scenarios. These tasks encode industry best practices for adopting Azure services.

## How It Works

When you run `gh appmod migrate`, Copilot will:

1. **Analyze** — Scan your project to identify current services, dependencies, and integration points
2. **Match** — Identify applicable predefined migration tasks (or work from your custom prompt)
3. **Plan** — Generate a migration plan with step-by-step changes
4. **Execute** — Apply code changes, update dependencies, and modify configurations
5. **Validate** — Build and test the migrated code
6. **Summarize** — Provide a report of all changes

## Predefined Migration Tasks

The App Modernization MCP server includes predefined tasks for common migration scenarios:

### Message Queue Migrations

| From | To | Example Prompt |
|------|----|---------------|
| RabbitMQ (Spring AMQP/JMS) | Azure Service Bus | `"Migrate from RabbitMQ to Azure Service Bus"` |
| Apache ActiveMQ | Azure Service Bus | `"Migrate from ActiveMQ to Azure Service Bus"` |
| AWS SQS | Azure Service Bus | `"Migrate from AWS SQS to Azure Service Bus"` |

### Storage Migrations

| From | To | Example Prompt |
|------|----|---------------|
| AWS S3 | Azure Blob Storage | `"Migrate from S3 to Azure Blob Storage"` |
| Local file I/O | Azure Storage File Share | `"Migrate local file I/O to Azure Storage File Share mounts"` |

### Database Migrations

| From | To | Example Prompt |
|------|----|---------------|
| Oracle SQL dialect | PostgreSQL | `"Migrate Oracle SQL to PostgreSQL for Azure Database"` |
| Local database auth | Managed Identity auth | `"Migrate database authentication to use Azure Managed Identities"` |

### Security & Identity

| From | To | Example Prompt |
|------|----|---------------|
| Hardcoded secrets / local keystores | Azure Key Vault | `"Migrate secrets and certificates to Azure Key Vault"` |
| AWS Secrets Manager | Azure Key Vault | `"Migrate from AWS Secrets Manager to Azure Key Vault"` |
| Connection string auth | Managed Identity auth | `"Migrate credentials to use Azure Managed Identities"` |
| LDAP authentication | Microsoft Entra ID | `"Migrate user authentication to Microsoft Entra ID"` |

### Communication

| From | To | Example Prompt |
|------|----|---------------|
| Java Mail / SMTP | Azure Communication Services | `"Migrate from Java Mail to Azure Communication Services"` |

### Observability

| From | To | Example Prompt |
|------|----|---------------|
| File-based logging | Console logging (Azure Monitor) | `"Migrate file-based logging to console logging for Azure Monitor"` |

## Usage Examples

### General Migration

Let Copilot analyze your project and suggest migrations:

```bash
gh appmod migrate
```

### Specific Migration Task

Target a specific migration:

```bash
gh appmod migrate "Migrate from S3 to Azure Blob Storage"
```

### Multiple Migrations

Describe everything you want to migrate:

```bash
gh appmod migrate "Migrate from AWS S3 to Azure Blob Storage and from AWS SQS to Azure Service Bus"
```

### Migration with Managed Identity

```bash
gh appmod migrate "Set up Managed Identity authentication for Azure PostgreSQL database"
```

### Security Hardening

```bash
gh appmod migrate "Migrate all hardcoded secrets to Azure Key Vault and switch to Managed Identity for service authentication"
```

## Tips

### Start with Analysis

If you're unsure what needs to be migrated, ask Copilot to analyze first:

```bash
gh appmod migrate "Analyze this project and list all services that should be migrated to Azure equivalents"
```

### Migrate One Service at a Time

For complex applications, migrate services individually:

```bash
# Step 1: Storage
gh appmod migrate "Migrate from S3 to Azure Blob Storage"

# Step 2: Message queue
gh appmod migrate "Migrate from RabbitMQ to Azure Service Bus"

# Step 3: Security
gh appmod migrate "Migrate secrets to Azure Key Vault"
```

### Use a Branch per Migration

```bash
git checkout -b migrate/s3-to-azure-blob
gh appmod migrate "Migrate from S3 to Azure Blob Storage"
git add -A && git commit -m "Migrate from S3 to Azure Blob Storage"
```

### Review Configuration Changes

Migrations often change configuration files. Pay special attention to:

- `application.properties` / `application.yml` — New Azure connection settings
- `pom.xml` / `build.gradle` — Dependency changes (AWS SDKs → Azure SDKs)
- Environment variables — New required variables for Azure services

## What Gets Changed

A typical migration may modify:

| File Type | Changes |
|-----------|---------|
| `pom.xml` / `build.gradle` | Replace vendor SDKs with Azure SDKs |
| `*.java` | Replace service client code, update imports, change API calls |
| `application.properties` / `application.yml` | New Azure service connection config |
| Configuration classes | New Azure service beans and configuration |

## Learn More

- [Predefined Tasks Reference](https://learn.microsoft.com/azure/developer/java/migration/migrate-github-copilot-app-modernization-for-java-predefined-tasks) — Full list of predefined migration tasks
- [Upgrade Guide](upgrade.md) — Upgrade your Java version before migrating
- [Deployment Guide](deployment.md) — Deploy your migrated application
- [Command Reference](commands.md) — Full command details
