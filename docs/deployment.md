# Deployment Guide

For Azure deployment operations, use the [Modernize CLI](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview):

```bash
modernize deploy
```

## Skills

This extension includes an Azure deployment skill that you can install into your repository with `gh appmod add-skills`. The skill covers deployment to:

- Azure App Service
- Azure Container Apps
- Azure Kubernetes Service (AKS)
- Azure Spring Apps
- Azure Functions

The Modernize CLI automatically detects these skills when creating deployment plans.

## Learn More

- [Modernize CLI — Deploy](https://learn.microsoft.com/azure/developer/github-copilot-app-modernization/modernization-agent/overview)
- [Command Reference](commands.md)
