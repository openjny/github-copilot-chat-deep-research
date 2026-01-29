# GitHub Copilot Deep Research for Microsoft

A collection of GitHub Copilot custom agents and prompts designed for conducting thorough research on Microsoft technologies using official documentation and trusted sources.

## Overview

This repository provides custom agent definitions and prompts for GitHub Copilot Chat that enable deep research capabilities focused on Microsoft ecosystem. The agents are optimized to search Microsoft Learn documentation, Azure docs, and web sources, compile findings into structured research reports, and ensure credibility through proper citation.

## Features

- **Multi-Agent Research Workflow**: Orchestrated agents that handle different aspects of the research process
- **Microsoft Learn Integration**: Direct search and retrieval from official Microsoft Learn documentation
- **Azure Documentation Support**: Specialized for Azure services, features, and best practices research
- **Web Search Support**: Extended research capabilities through web search for broader context
- **Automated Review**: Built-in review agent to identify gaps and weak evidence in reports
- **Critical Thinking Analysis**: Prompt for analyzing responses with a skeptical, detail-oriented approach
- **Fact-Checking**: Agent for adding evidence footnotes with links to official Microsoft documentation

## Use Cases

- Researching Azure service comparisons and architecture decisions
- Investigating Microsoft 365 features and integration options
- Exploring .NET, Visual Studio, and developer tools documentation
- Compiling reference materials for Microsoft certifications
- Creating technical documentation with verified Microsoft sources

## Requirements

- GitHub Copilot Chat
- VS Code with Copilot extension
- [Microsoft Docs MCP server](https://github.com/nicobailey/microsoft-docs-mcp) (recommended, for enhanced MS Learn integration)
- [Web Search for Copilot](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-websearchforcopilot) extension (optional, for web search capabilities)

## Usage

### Quick Search

Use `/mshelp.ask` to quickly search Microsoft Learn documentation and web sources:

```
/mshelp.ask Azure Functions pricing tiers
```

### Deep Research

Use `/mshelp.research` for comprehensive research with MS Learn and web sources:

```
/mshelp.research Azure Container Apps vs Azure Kubernetes Service comparison
```

Or use `/mshelp.websearch` for web-focused research:

```
/mshelp.websearch Latest trends in serverless computing on Azure
```

### Critical Thinking

After receiving a response, use the critical thinking prompt to analyze it:

```
/criticalthink
```

### Fact Checking

Add evidence footnotes to existing research reports:

```
/factcheck
```

## Research Report Conventions

- File names follow the format `YYYY-MM-DD-<slug>.md`
- Reports contain facts only, no inferences or opinions
- All claims must include footnote citations with URLs (preferably to docs.microsoft.com or learn.microsoft.com)
- Research results are structured for data consumption

## License

MIT
