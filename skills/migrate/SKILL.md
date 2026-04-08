---
name: migrate
description: Migrate Java application services and dependencies to Azure equivalents using predefined migration tasks
---

# Migrate

Migrate Java application services and dependencies to Azure equivalents.

## Predefined Migration Tasks

The following common migration patterns are supported:

- **Message queues:** RabbitMQ, ActiveMQ, AWS SQS → Azure Service Bus
- **Storage:** AWS S3 → Azure Blob Storage, local file I/O → Azure Storage File Share
- **Databases:** Oracle SQL dialect → PostgreSQL, password auth → Managed Identity
- **Security:** Hardcoded secrets → Azure Key Vault, AWS Secrets Manager → Key Vault
- **Identity:** LDAP → Microsoft Entra ID, connection strings → Managed Identity
- **Communication:** Java Mail / SMTP → Azure Communication Services
- **Observability:** File-based logging → Console logging for Azure Monitor

## Steps

1. **Scan the project** — Identify current dependencies, services, and integration patterns.
2. **Match against migration tasks** — Determine which predefined migration tasks apply.
3. **Present a migration plan** — Show what will be migrated, to which Azure service, and what changes are needed.
4. **Apply migrations incrementally** — One service migration at a time. Validate the build after each.
5. **Update configuration** — Ensure `application.properties`/`application.yml` and other config files reflect the new services.
6. **Validate** — Run the build and tests to confirm the migration compiles and passes.

## Guidelines

- Migrate one service at a time to keep changes reviewable.
- Always explain the Azure service being adopted and why it maps to the current dependency.
- Prefer Managed Identity over connection strings and secrets where possible.
- If a migration task doesn't exist for a service, explain the manual steps needed.
