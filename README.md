<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github.com/octopusdeploy/mcp-server/blob/main/images/OctopusDeploy_Logo_DarkMode.png?raw=true">
  <source media="(prefers-color-scheme: light)" srcset="https://github.com/octopusdeploy/mcp-server/blob/main/images/OctopusDeploy_Logo_LightMode.png?raw=true">
  <img alt="Octopus Deploy Logo" src="https://github.com/octopusdeploy/mcp-server/blob/main/images/OctopusDeploy_Logo_LightMode.png?raw=true" />
</picture>

# Claude Code Plugin for Octopus Deploy

[Octopus](https://octopus.com) makes it easy to deliver software to Kubernetes, multi-cloud, on-prem infrastructure, and anywhere else. Automate the release, deployment, and operations of your software and AI workloads with a tool that can handle CD at scale in ways no other tool can.

This plugin combines the power of the [Octopus Remote MCP Server](https://octopus.com/docs/octopus-ai/mcp/remote) and a specialized set of skills to enable [Claude Code](https://claude.com/claude-code) to interact with Octopus Deploy in an automated fashion.

The MCP connects directly to your Octopus instance and can perform most actions that a regular user can:
- Create projects and deployment processes
- Create releases
- Execute runbooks
- Configure project variables
- Find any interruptions requiring human attention
- Inspect audit log
- And much more

The plugin comes with a set of skills to help with:
- Connecting a repository to Octopus
- Writing OCL ([Octopus Configuration Language](https://octopus.com/docs/projects/version-control/ocl-file-format))
- Diagnosing deployment failures and fixing them
- Producing an SOC2 audit report

## Getting Started

### 1. Add the marketplace and install the plugin

In Claude Code:

```bash
/plugin marketplace add OctopusDeploy/octopus-claude-plugins
/plugin install octopus-deploy-devops
```

### 2. Requirements

- Octopus Server 2026.3 or later, reachable over HTTPS. OAuth sign-in needs Octopus Server 2026.4.9289 or later.
- Remote MCP turned on in **Configuration > Settings > MCP Controls** (on by default).

### 3. Set your Octopus Server URL

The plugin connects to the Remote MCP Server at `$OCTOPUS_SERVER_URL/mcp`.

**Unix/Mac (bash/zsh):**
```bash
export OCTOPUS_SERVER_URL="https://your-instance.octopus.app"
```

**Windows (PowerShell):**
```powershell
$env:OCTOPUS_SERVER_URL = "https://your-instance.octopus.app"
```

Don't include a trailing slash. Restart Claude Code after setting the variable.

### 4. Sign in

Run `/mcp` in Claude Code, select `plugin:octopus-deploy-devops:octopus-deploy-mcp`, and sign in through the browser with your Octopus user account. On the consent page, choose **Read-only** or **Full access**. To change the access level later, disconnect and sign in again. Sessions last up to 20 days.

### Using an API key instead of OAuth

Service accounts, the guest account, and Octopus Server versions without OAuth sign-in need an API key. Disable the plugin's MCP server in `/mcp`, then add the Remote MCP Server yourself with an [Agent API key](https://octopus.com/docs/octopus-ai/mcp/remote):

```bash
export OCTOPUS_API_KEY="API-XXXXXXXXXXXXX"
claude mcp add --transport http octopus-deploy "$OCTOPUS_SERVER_URL/mcp" --header "X-Octopus-ApiKey: ${OCTOPUS_API_KEY}"
```

The skills work with either connection.

### Migrating from the local MCP server

Earlier versions of this plugin ran the deprecated [`@octopusdeploy/mcp-server`](https://github.com/OctopusDeploy/mcp-server) package with `OCTOPUS_API_KEY`. The plugin no longer reads `OCTOPUS_API_KEY`; you can remove it unless you use the API key setup above or the REST fallbacks in the onboarding skill.

### Octopus Server older than 2026.3

Older versions don't have the Remote MCP Server. Check your version with `curl "$OCTOPUS_SERVER_URL/api"` (the `Version` field). On an older server, disable the plugin's MCP server in `/mcp`, then add the deprecated local MCP server yourself. It needs Node.js 20 or later.

```bash
claude mcp add octopus-deploy \
  --env OCTOPUS_SERVER_URL="https://your-instance.example.com" \
  --env OCTOPUS_API_KEY="API-XXXXXXXXXXXXX" \
  -- npx -y @octopusdeploy/mcp-server
```

The skills are written for the Remote MCP Server's tools. The local server's tools have different names (for example `list_spaces` instead of `find_spaces`), so the onboarding skill doesn't detect it and uses REST with your API key instead.

## Examples

Here is an example of what you can do with this plugin:
- 📊 **Show me the deployment status of my project, and when it was last deployed to tenant X**
- 🔍 **Investigate and diagnose why a deployment has failed**
- 🔧 **Use the appropriate runbook to fix the failure**
- 🚀 **Redeploy to tenant**

## Resources

- [Octopus Deploy Documentation](https://octopus.com/docs)
- [Octopus Remote MCP Server](https://octopus.com/docs/octopus-ai/mcp/remote)

## License

This plugin is maintained by Octopus Deploy.

## Support

For issues or questions:
- Plugin issues: [GitHub Issues](https://github.com/OctopusDeploy/octopus-claude-plugins/issues)
- Octopus Deploy support: [Octopus Support](https://octopus.com/support)
