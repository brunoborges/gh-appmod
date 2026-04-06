---
mcp-servers:
  app-modernization:
    command: "npx"
    args: ["-y", "@microsoft/github-copilot-app-modernization-mcp-server"]
    allowed: ["*"]

runtimes:
  java:
    version: "21"
  node:
    version: "22"

network:
  allowed:
    - defaults
    - registry.npmjs.org
    - repo.maven.apache.org
    - plugins.gradle.org
    - repo1.maven.org
---
