# Deployment Guide

Use `gh appmod deploy` to deploy your Java application to Azure services.

## Overview

The deploy command launches GitHub Copilot CLI with the App Modernization MCP server to help you deploy your Java application to Azure. Copilot will analyze your project, recommend an appropriate Azure service, and guide you through the deployment process.

## How It Works

When you run `gh appmod deploy`, Copilot will:

1. **Analyze** — Scan your project to understand the application type, dependencies, and requirements
2. **Recommend** — Suggest the most appropriate Azure service for your application
3. **Configure** — Set up deployment configurations, Dockerfiles, or CI/CD pipelines as needed
4. **Deploy** — Execute the deployment to Azure
5. **Verify** — Confirm the deployment was successful
6. **Summarize** — Show the deployed application URL and details

## Azure Deployment Targets

Depending on your application, Copilot may recommend:

| Azure Service | Best For |
|--------------|----------|
| [Azure App Service](https://azure.microsoft.com/products/app-service/) | Web apps and APIs with managed infrastructure |
| [Azure Container Apps](https://azure.microsoft.com/products/container-apps/) | Containerized microservices with auto-scaling |
| [Azure Kubernetes Service (AKS)](https://azure.microsoft.com/products/kubernetes-service/) | Complex microservices requiring full Kubernetes control |
| [Azure Spring Apps](https://azure.microsoft.com/products/spring-apps/) | Spring Boot and Spring Cloud applications |
| [Azure Functions](https://azure.microsoft.com/products/functions/) | Event-driven, serverless workloads |

## Usage Examples

### Default Deployment

Let Copilot analyze and recommend the best target:

```bash
gh appmod deploy
```

### Deploy to a Specific Service

```bash
gh appmod deploy "Deploy to Azure App Service"
```

### Deploy to Containers

```bash
gh appmod deploy "Containerize this application and deploy to Azure Container Apps"
```

### Deploy to Kubernetes

```bash
gh appmod deploy "Deploy to Azure Kubernetes Service with Helm charts"
```

### Deploy Spring Apps

```bash
gh appmod deploy "Deploy this Spring Boot app to Azure Spring Apps"
```

### Full Pipeline

```bash
gh appmod deploy "Set up a GitHub Actions CI/CD pipeline and deploy to Azure App Service"
```

## Prerequisites for Deployment

Before deploying, make sure you have:

1. **Azure CLI** installed and authenticated (`az login`)
2. **An Azure subscription** with appropriate permissions
3. **A resource group** (or Copilot can create one for you)

Copilot will guide you through any missing prerequisites during the deployment process.

## Tips

### Upgrade and Migrate First

For the best results, upgrade and migrate your application before deploying:

```bash
# 1. Upgrade to modern Java
gh appmod upgrade "Upgrade to JDK 21 and Spring Boot 3.2"

# 2. Migrate services to Azure
gh appmod migrate "Migrate from S3 to Azure Blob Storage"

# 3. Deploy
gh appmod deploy "Deploy to Azure App Service"
```

### Review Generated Configurations

Copilot may generate deployment configurations. Review them before deploying:

- `Dockerfile` — Container build configuration
- `.github/workflows/*.yml` — CI/CD pipeline definitions
- `azure-deploy.yml` / `deploy.sh` — Deployment scripts
- ARM templates or Bicep files — Infrastructure as code

### Test Locally First

After Copilot generates deployment configurations (like a `Dockerfile`), test locally before deploying:

```bash
docker build -t myapp .
docker run -p 8080:8080 myapp
```

### Use Auto-Approval Carefully

The `--yolo` flag auto-approves all actions, including Azure resource creation. Be mindful of costs:

```bash
# This will create Azure resources without asking
gh appmod deploy --yolo "Deploy to Azure App Service"
```

## What Gets Created

A typical deployment may create or modify:

| File / Resource | Description |
|----------------|-------------|
| `Dockerfile` | Container image build instructions |
| `.github/workflows/deploy.yml` | CI/CD pipeline for automated deployment |
| Azure Resource Group | Container for Azure resources |
| Azure App Service / Container App | The compute service hosting your app |
| Azure Container Registry | Docker image registry (if containerized) |

## Next Steps

- [Command Reference](commands.md) — Full command details
- [Configuration](configuration.md) — Advanced configuration options
- [Troubleshooting](troubleshooting.md) — Deployment troubleshooting
