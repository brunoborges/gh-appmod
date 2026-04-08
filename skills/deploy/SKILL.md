---
name: deploy
description: Deploy Java applications to Azure services including App Service, Container Apps, AKS, and Spring Apps
---

# Deploy

Deploy Java applications to Azure services.

## Azure Deployment Targets

| Target | Best For |
|--------|----------|
| Azure App Service | Traditional web apps, REST APIs |
| Azure Container Apps | Containerized microservices, event-driven apps |
| Azure Kubernetes Service (AKS) | Complex microservice architectures, existing K8s workloads |
| Azure Spring Apps | Spring Boot / Spring Cloud applications |

## Steps

1. **Analyze the project** — Determine the application type, framework, build tool, and runtime requirements.
2. **Recommend a deployment target** — Based on the analysis, suggest the most appropriate Azure service. Explain the reasoning.
3. **Generate artifacts** — Create the necessary deployment configuration files (Dockerfile, deployment YAML, GitHub Actions workflow, etc.).
4. **Guide through deployment** — Provide step-by-step instructions for deploying to the chosen service, including any Azure CLI commands needed.
5. **Validate** — Confirm the deployment configuration is correct and the application can be built for deployment.

## Guidelines

- Always recommend the simplest deployment option that meets the application's needs.
- Explain trade-offs between deployment targets when multiple options are viable.
- Generate production-ready configurations (health checks, resource limits, logging).
- Never include secrets or credentials in deployment artifacts — use Azure Key Vault or Managed Identity.
