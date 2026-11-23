# Octopus Deploy AI Features

This document details the AI-powered capabilities in Octopus Deploy for intelligent DevOps automation.

## Overview

Octopus Deploy integrates AI capabilities to solve previously unsolvable problems in continuous delivery:

- **Log Analysis**: Parse complex deployment logs and diagnose root causes
- **Natural Language Interaction**: Explore your deployment landscape using plain language
- **Agentic Workflows**: Intelligent automation between Octopus and other services
- **Failure Recovery**: AI-assisted rapid recovery from deployment failures

## AI Capabilities

### 1. Octopus AI Assistant

An intelligent companion integrated into the Octopus Deploy interface that provides context-aware guidance and automation.

#### Tier-0 Support

Get instant answers to Octopus Deploy questions without searching documentation.

**How It Works:**
- Draws from complete Octopus Deploy documentation
- Provides natural language responses
- Context-aware based on current page

**Example Prompts:**
- "What is a project in Octopus Deploy?"
- "How do I use runbooks?"
- "Explain the difference between environments and tenants"
- "How do I set up automated deployments?"

**Use Cases:**
- Onboarding new team members
- Quick feature clarification
- Learning best practices

#### Prompt-Based Project Creation

Create fully configured deployment projects from text descriptions.

**How It Works:**
- Describe what you want to deploy
- AI generates complete project configuration
- Follows established best practices

**Example Prompts:**
- "Create an Azure Web App project called 'My Web App'"
- "Generate an AWS Lambda project with QA and Production environments"

**Generated Configuration Includes:**
- Deployment process steps
- Environment configuration
- Variable templates
- Lifecycle settings

#### Deployment Failure Analysis

Immediate analysis of failed deployments with actionable remediation steps.

**How It Works:**
- Gathers deployment context automatically
- Analyzes logs, configuration, and errors
- Provides root cause identification
- Suggests specific resolution steps

**Context Captured:**
- Deployment logs and error messages
- Deployment process configuration
- Script content from deployment steps
- Build information and artifacts
- Environment and target details

**Analysis Output:**
1. **Reason for failure** - Specific step and error that caused failure
2. **What happened** - Detailed breakdown of where it went wrong
3. **Suggestions** - Actionable remediation steps with commands
4. **Next steps** - Recommendations to prevent future failures

**Example Prompt:**
- "Help me understand why the deployment failed. If the deployment didn't fail, say so. Provide suggestions for resolving the issue."

#### Best Practices Adviser

Identify optimization opportunities and maintain healthy configurations.

**Capabilities:**
- Variable analysis (unused, duplicate, exposed secrets)
- Tenant management recommendations
- Resource utilization analysis
- Configuration health checks

**Example Prompts:**
- "Find unused variables in this project"
- "Find duplicate project variables"
- "Help me find project variable values that look like plaintext passwords"
- "Suggest tenant tags to make tenants more manageable"
- "Find unused tenants"
- "Find unused targets"
- "Find unused projects"

#### Custom Prompts

Enhance AI Assistant with organization-specific knowledge and procedures.

**Configuration:**
- Define in Library Variable Sets named "OctoAI Prompts"
- User-facing prompts: `PageName[#].Prompt`
- System prompts (hidden logic): `PageName[#].SystemPrompt`

**Use Cases:**
- Embed internal support processes
- Define team-specific escalation paths
- Include organization-specific troubleshooting steps
- Reference internal documentation

**Example Configuration:**

| Variable | Value |
|----------|-------|
| `Project.Deployment[0].Prompt` | Why did the deployment fail? |
| `Project.Deployment[0].SystemPrompt` | If Azure resource group not found, find responsible team in project description and instruct to create support ticket via team Slack workflow. |

### 2. Octopus Remote MCP Server

Model Context Protocol (MCP) server built into Octopus Server, enabling AI assistants to interact directly with Octopus Deploy at `https://your-octopus-instance.com/mcp`.

#### What is MCP?

- Open standard by Anthropic for connecting AI assistants to external tools
- Enables AI clients (Claude, etc.) to access Octopus Deploy data and operations
- Provides structured tool access rather than raw API

#### Benefits Over AI Assistant

- **Model Choice**: Use with any MCP-compatible AI client
- **Orchestration**: Combine with other MCP servers for complex workflows
- **Flexibility**: Integrate into existing AI workflows

#### Connecting

**Requirements:**
- Octopus Server 2026.3 or later, accessible via HTTPS
- Remote MCP turned on in **Configuration > Settings > MCP Controls** (on by default)
- OAuth sign-in needs Octopus Server 2026.4.9289 or later; otherwise use an Agent API key

**Claude Code with OAuth:**
```bash
claude mcp add --transport http octopus-deploy --scope user https://your-octopus-instance.com/mcp
claude mcp login octopus-deploy
```

Users sign in through the browser and choose **Read-only** or **Full access** on the consent page. Service accounts and the guest account can't use OAuth.

**Claude Code with an Agent API key:**
```bash
claude mcp add --transport http octopus-deploy https://your-octopus-instance.com/mcp --header "X-Octopus-ApiKey: ${OCTOPUS_API_KEY}"
```

**Tools:** dedicated tools such as `find_spaces`, `find_projects`, `find_deployments`, `create_release`, and `deploy_release`, plus contract tools (`search_contracts`, `describe_contract`, `execute_query`, `execute_command`) that reach the rest of the AI-enabled API.

The local `@octopusdeploy/mcp-server` npm package is deprecated in favor of the Remote MCP Server. See the [Remote MCP Server documentation](https://octopus.com/docs/octopus-ai/mcp/remote).

#### MCP Use Cases

**Change Management:**
Understand what changes are deployed where.

*Example Prompt:*
> "Customer X submitted a support ticket about a bug. Can you tell me what release they are on, when it was deployed, and if there were any issues?"

**Troubleshooting:**
Identify and diagnose failures and unhealthy resources.

*Example Prompt:*
> "Check health of the PaymentService in the Production space and report any issues. Check Kubernetes service status for a comprehensive report."

**Administration & Compliance:**
Ensure optimal instance health and compliance.

*Example Prompt:*
> "Find certificates soon set to expire in the Production space"

**Cross-Service Orchestration:**
Combine with other MCP servers for complex workflows.

*Example Prompt:*
> "Check accounts in Production space, find the preproduction Azure account, then check available resources in that subscription using the Azure MCP"

### 3. Octopus Recovery Agent

AI-powered rapid recovery from deployment failures (Octopus Cloud only).

#### Purpose

Recovery Agent helps achieve short mean time to recovery (MTTR) - a hallmark of elite software delivery. It uses AI to:

1. Analyze deployment failures
2. Diagnose root causes
3. Suggest remediation steps
4. (Future) Execute remediation actions

#### How It Works

- Automatically analyzes failed deployments
- Provides specific, contextual recovery suggestions
- Maintains user control while handling investigation
- Integrates directly into deployment UI

#### Availability

- **Currently**: Octopus Cloud customers only
- **Future**: Self-hosted availability planned (no specific date)

#### Features

- AI-powered failure analysis
- Root cause identification
- Suggested remediation steps
- Context-aware recommendations
- Audit trail of analysis and suggestions

## Security and Privacy

### Data Protection

- **No Training on Customer Data**: Customer data never used to train AI models
- **Azure OpenAI Platform**: Models hosted in Azure with strict data handling
- **Stateless Interactions**: No history retention, data not stored by models
- **GDPR Compliance**: EU instances use EU-hosted models (Sweden)

### Sensitive Data Handling

- Sensitive variables never exposed to AI
- API never returns sensitive values
- Existing permission boundaries respected
- All interactions logged and auditable

### Regional Data Processing

- US instances: US-based model processing
- EU instances: EU-based model processing (Sweden)
- All data transmission encrypted via HTTPS

### Open Source Transparency

- AI Assistant backend: [GitHub - OctopusCopilot](https://github.com/OctopusSolutionsEngineering/OctopusCopilot)
- Remote MCP Server: [Documentation](https://octopus.com/docs/octopus-ai/mcp/remote)
- Independent security audit available via [Trust Center](https://trust.octopus.com/)

## Pricing

- **AI Assistant**: Currently free (pricing may change)
- **MCP Server**: Open source, free
- **Recovery Agent**: Currently free for Cloud customers (pricing may change)

## Getting Started

### AI Assistant
1. Install Chrome extension
2. Navigate to Octopus Deploy instance
3. Open AI Assistant from interface
4. Start with suggested prompts

### Remote MCP Server
1. Confirm Remote MCP is turned on in **Configuration > Settings > MCP Controls**
2. Add `https://your-octopus-instance.com/mcp` to your MCP client
3. Sign in with OAuth, or send an Agent API key in the `X-Octopus-ApiKey` header

### Recovery Agent (Cloud)
1. Enabled by default for Cloud customers
2. Appears automatically on failed deployments
3. Contact support to opt out if desired

## AI Assistant Cookbook Examples

Common tasks the AI Assistant can help with:

**Deployment & Release Management:**
- Investigate production deployment failures
- Generate deployment rollback plans
- List failed deployments
- Evaluate deployment frequency

**Variable Management:**
- Detect unused variables
- Detect overlapping variable names
- Fix variable binding errors
- Recommend variable scoping

**Infrastructure Analysis:**
- Summarize worker pool health
- Audit target role distribution
- Review runbook usage
- Summarize tenant tag coverage

**Compliance & Auditing:**
- Audit PCI deployments
- Security best practices check
- Check retention policy consistency
- Audit environment naming and counts

**Optimization:**
- Improve multi-tenant deployments
- Speed up lifecycle phases
- Resolve rolling deployment timeouts
- Analyze step template usage
