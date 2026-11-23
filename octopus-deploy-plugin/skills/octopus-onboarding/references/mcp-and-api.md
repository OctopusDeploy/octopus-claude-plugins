# MCP tools and REST equivalents

This reference accompanies `SKILL.md` (the `octopus-onboarding` skill). It tells the agent which Remote MCP Server tools to prefer for each onboarding step, which REST endpoints they map to, and which writes need REST.

Read this file once at the start of an onboarding session.

## Detect transport

The user either has the Octopus Remote MCP Server connected, or they don't.

- **MCP connected** — your tool list includes `find_spaces`, `find_projects`, `search_contracts`, `describe_contract`, `execute_query`, and `execute_command`. The server is named `octopus-deploy-mcp` when it comes from this plugin, and is typically `octopus-deploy` when the user added it themselves.
- **MCP not connected** — fall back to direct REST against `{serverUrl}` with `X-Octopus-ApiKey: <api-key>` in every request header.

Default when in doubt: probe for `find_spaces` in your available tools. If it's there, MCP is live. If it's not, REST it is.

## The rule

When MCP is connected, **every read flow in this skill goes through MCP**. Writes go through MCP when a command contract exists for them. A few onboarding writes have no contract (see *Writes with no contract*); those use REST with an API key, even when MCP is connected.

When MCP is not connected, use the REST endpoints listed below.

## How the Remote MCP Server is organized

- **Dedicated tools** appear directly in your tool list: `find_spaces`, `find_projects`, `find_environments`, `find_releases`, `find_deployments`, `find_server_tasks`, `find_runbooks`, `find_tenants`, `find_deployment_targets`, `find_certificates`, `find_accounts`, `find_events`, `get_current_user`, `get_deployment_process`, `get_variables`, `get_branches`, `create_release`, `deploy_release`, `deploy_release_for_tenants`, `run_runbook_in_database`, `run_runbook_in_git`.
- **Contracts** cover the rest of the API. Find one with `search_contracts` (keyword search, optionally by `Category`), read its input schema with `describe_contract`, then run it with `execute_query` (reads) or `execute_command` (writes), passing the contract name in `Name` and its parameters as a JSON object in `Arguments`.
- Contracts flagged `IsComplex` are full replacements, not patches: omitted fields are cleared. Read the current state first and resubmit every value with only your changes applied.
- Contract and parameter names change between Octopus versions. Treat the names below as starting points, and confirm with `describe_contract` before calling one.

## Tool ↔ REST cross reference

### Reads — core and discovery

| MCP | Equivalent REST | Notes |
|---|---|---|
| `find_spaces` | `GET /api/spaces/all` | Pick a space ID once at session start. |
| `get_current_user` | `GET /api/users/me` | Confirms credentials and surfaces permission scope. |
| `find_environments` | `GET /api/{sp}/environments/all` | |
| `find_projects` | `GET /api/{sp}/projects/all` (paged) | |
| `get_deployment_process` | `GET /api/{sp}/projects/{id}/deploymentprocesses` | Set `GitObject` for Config-as-Code projects. |
| `get_variables` | `GET /api/{sp}/variables/{variableSetId}` | |
| `get_branches` | `GET /api/{sp}/projects/{id}/git/branches` | Version-controlled projects only. |
| `find_releases` | `GET /api/{sp}/projects/{id}/releases` | |
| `find_runbooks` | `GET /api/{sp}/runbooks` | |
| `find_tenants` | `GET /api/{sp}/tenants` | |
| `find_deployment_targets` | `GET /api/{sp}/machines` | |
| `find_certificates` | `GET /api/{sp}/certificates` | |
| `find_accounts` | `GET /api/{sp}/accounts` | |
| `execute_query` `find_feeds` | `GET /api/{sp}/feeds` | |
| `execute_query` `find_package_versions` | `GET /api/{sp}/feeds/{id}/packages/versions?packageId=...` | |
| `execute_query` `find_lifecycles` | `GET /api/{sp}/lifecycles` | |
| `execute_query` `find_tag_sets` | `GET /api/{sp}/tagsets` | |
| `execute_query` `find_interruptions` | `GET /api/{sp}/interruptions` | Set `PendingOnly`, or `Regarding` to a task ID. |

### Tasks and logs

| MCP | Equivalent REST | Notes |
|---|---|---|
| `find_server_tasks` | `GET /api/tasks` | Filter by `States`, `Project`, `Environment`, `Active`. |
| `find_deployments` | `GET /api/{sp}/deployments` | |
| `execute_query` `get_server_task_log` | `GET /api/{sp}/tasks/{id}/details` then client-side search | Defaults to a failure-focused summary. Set `Mode` to `Search` with `SearchPattern` (regex) and `ContextLines` to search the log. Task logs can be megabytes; **always prefer `get_server_task_log` over fetching the whole log**. |
| `execute_query` `get_child_server_tasks` | — | For "Deploy a Release" child deployments. |

To get a task ID from an Octopus URL, read it from the URL path (for example `.../tasks/ServerTasks-123`, or the deployment ID in `.../deployments/Deployments-45` passed to `find_deployments`).

### Tenant variables

| MCP | Equivalent REST |
|---|---|
| `execute_query` `get_tenant_project_variables` / `get_tenant_common_variables` | `GET /api/{sp}/tenants/{id}/projectvariables`, `/commonvariables` |
| `execute_query` `get_tenants_missing_variables` | `GET /api/{sp}/tenants/variables-missing` |

### API catalog

The Remote MCP Server has no API catalog tool. For REST endpoint shapes, step types, and the `Octopus.Action.*` property catalog, use `GET {serverUrl}/api/experimental/llms.txt` with the API key header. The catalog is around 350 KB; save it to a file and search it rather than reading it whole.

## Writes

The OAuth consent page sets the session's access level. A **Read-only** session rejects every write; ask the user to reconnect with **Full access** rather than working around it.

### Writes with a dedicated tool or contract

| MCP | Effect | REST equivalent |
|---|---|---|
| `execute_command` `create_environment` | Additive. | `POST /api/{sp}/environments` |
| `execute_command` `create_project` | Additive. | `POST /api/{sp}/projects` |
| `execute_command` `create_lifecycle` | Additive. | `POST /api/{sp}/lifecycles` |
| `execute_command` `create_tag_set` | Additive. | `POST /api/{sp}/tagsets` |
| `execute_command` `create_tenant` / `modify_tenant` | Additive / replace. | `POST` / `PUT /api/{sp}/tenants` |
| `execute_command` `modify_tenant_project_variables` | Replace. | `POST /api/{sp}/tenants/{id}/projectvariables` |
| `execute_command` `modify_project_variables_in_database` / `_in_git` | Replace. | `PUT /api/{sp}/variables/{id}` |
| `execute_command` `create_project_trigger` | Additive. | `POST /api/{sp}/projecttriggers` |
| `create_release` | Additive — creates a new release record. | `POST /api/{sp}/releases/create/v1` |
| `deploy_release` / `deploy_release_for_tenants` | **Destructive** — mutates live state. | `POST /api/{sp}/deployments` |
| `run_runbook_in_database` / `run_runbook_in_git` | **Destructive** — runs operational automation. | `POST /api/{sp}/runbookruns` |

### Writes with no contract

These onboarding writes have no Remote MCP equivalent. Use REST with an API key, even when MCP is connected. Ask the user for an API key (Profile → My API Keys) the first time one of these is needed, and tell them which step needs it.

| Write | REST |
|---|---|
| Register a deployment target | `POST /api/{sp}/machines` |
| Create a feed | `POST /api/{sp}/feeds` |
| Create a cloud account | `POST /api/{sp}/accounts` |
| Update the deployment process | `PUT /api/{sp}/projects/{id}/deploymentprocesses` |
| Turn on multi-tenancy | `PUT /api/{sp}/featuresconfiguration` |
| Push build information | `POST /api/{sp}/build-information` |

Before using REST for one of these, run `search_contracts` for it once; newer Octopus versions may have added a contract.

## Announcing writes

Per the *Announcing progress to the user* convention in `SKILL.md`, every write call (MCP or REST) must be followed by a one-line confirmation. Reads stay silent. Use the `✓` form for mutations and `→` for output-only handoffs (snippets the user pastes / commands the user runs).

| Call | Announcement |
|---|---|
| `create_environment` succeeds | `✓ Created environment '<Name>': <Id>` |
| `create_project` succeeds | `✓ Created project '<Name>': Projects-<id>` |
| `PUT .../deploymentprocesses` succeeds | `✓ Updated deployment process for '<project>' (<N> steps)` |
| `POST .../machines` succeeds | `✓ Registered target '<Name>': Machines-<id>` |
| `create_release` succeeds | `✓ Created release <Version> for '<Project>': Releases-<id>` |
| `deploy_release` invoked but *output-only* (snippet handed to user) | `→ Deploy command ready (waiting for you to run it)` |
| `run_runbook_in_database` succeeds | `✓ Ran runbook '<Name>': RunbookRuns-<id>` |
| Pre-existing object detected (matched by name in §0.0 resume) | `✓ Reused existing <resource> '<Name>': <Id>` |
| 409 collision on retry of a known-name resource | `✓ Reused existing <resource> '<Name>': <Id>` (look up the ID first; don't surface the 409 as an error) |

Announce what a destructive call will do in one line before making it, then print the `✓` line after it succeeds.

## Practical guidance

- **Reads first, writes last.** Most onboarding is reads (find spaces, find projects, get deployment process). Stay in MCP for everything read-shaped.
- **Use `get_server_task_log` for failure diagnosis.** Use `Mode: Search` instead of fetching full task details.
- **Look up contracts instead of guessing.** `search_contracts` with a few keywords (for example `create feed`) is cheaper than a failed call.
- **REST fallback parity.** Every flow in `SKILL.md` works via raw REST too. If the MCP server isn't connected, the same `POST /api/{sp}/...` call sequences from `SKILL.md` apply.
