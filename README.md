# Orcha MCP server

Orcha is a unified context layer for AI tools and agents. It stores, indexes, and exposes organizational, team, or individual knowledge. Files, structured databases, and connected sources live in one workspace, and agents reach them through Model Context Protocol (MCP), CLI with virtual file system, or REST API, with permissions, citations, and provenance.

This repository is the public connection guide and registry listing for Orcha's hosted MCP server. The service runs at:

```
https://app.tryorcha.com/mcp
```

Transport is Streamable HTTP. The application source code is private; nothing needs to be installed or built from this repository to use the server.

## What the server provides

28 tools spanning:

- Usage guides the agent can read for itself (`get_usage_guide`), covering tool choice, search tuning, citations, writing back, and troubleshooting
- Ranked search over all indexed workspace content (`search_context`) with fast, balanced, and fresh routing modes, plus a citation on every result
- File and folder reading, creation, update, organization, and deletion, scoped to what the connection is allowed to touch
- Curated memories with usage guidance (`list_memories`, `get_memory`), including agent-side curation (`create_memory`, `update_memory`) covering files, folders, synced documents, and database views
- Structured databases with typed queries (`list_databases`, `query_database`, record CRUD)
- Direct access to connected read-only sources (`query_source`, `fetch_source_document`, `list_source_documents`)
- Filesystem-shaped browsing (`browse_context`): ls, tree, find, cat, grep, and stat over a virtual tree of the workspace

Every connection is isolated to the organization, workspace, permissions, and optional folder scope it was granted.

The server also serves its usage guides as Agent Skills under the draft
`io.modelcontextprotocol/skills` extension: `skills/list` enumerates them and
each file is readable at `skill://orcha/<name>/<path>`. Clients without skills
support get the same content from the `get_usage_guide` tool.

## Connect with OAuth (recommended)

Add the server URL to any OAuth-capable MCP client. The client discovers Orcha's authorization server automatically and walks you through consent; the grant is scoped to the organization and workspace you approve.

Claude Code:

```bash
claude mcp add --transport http orcha "https://app.tryorcha.com/mcp"
```

claude.ai and Claude Desktop: Settings, Connectors, Add custom connector, then enter `https://app.tryorcha.com/mcp`.

Codex takes TOML rather than JSON. Add this to `~/.codex/config.toml`, or `.codex/config.toml` in a trusted project:

```toml
[mcp_servers.orcha]
url = "https://app.tryorcha.com/mcp"
auth = "oauth"
```

Codex also reads Agent Skills from `.agents/skills/` in a repository or `~/.agents/skills/` globally, so Orcha's four usage guides can be installed there. They are available with no installation through `get_usage_guide`.

Cursor and other OAuth-capable clients: add a remote MCP server with the same URL and follow the sign-in prompt.

Claude Code users can instead install the plugin below, which does the same
connection and adds the usage guides as skills.

## Connect with a bearer token (headless and CI clients)

1. Create an Orcha account and workspace at [app.tryorcha.com/signup](https://app.tryorcha.com/signup).
2. Create an API token scoped to that workspace with only the permissions the client needs. Tokens begin with `orca_`.
3. Configure the client with an `Authorization` header:

```json
{
  "mcpServers": {
    "orcha": {
      "url": "https://app.tryorcha.com/mcp",
      "type": "http",
      "headers": {
        "Authorization": "Bearer orca_your_token"
      }
    }
  }
}
```

## Example prompts

- "Search my workspace for the current authentication requirements and summarize them with the source file names."
- "Show me the five files most related to our deployment runbook."
- "Check which Orcha memories apply to this task and use the best match as the source of truth."
- "Create a Markdown file named `launch-checklist.md` from these release requirements."
- "Get the `engineering-onboarding` memory and identify any conflicting guidance."

## Claude Code plugin

Claude Code can install the connection and the usage guides together:

```
/plugin marketplace add westonhancock/orcha-mcp
/plugin install orcha@orcha
```

The plugin carries its own server configuration, so it replaces the
`claude mcp add` step above rather than following it. It also installs the four
usage guides as skills, so Claude loads the relevant one when a task calls for
it. See [plugins/orcha](./plugins/orcha) for details.

This is a convenience for one client. Every other client gets the same guidance
from the server through `get_usage_guide`.

## Registry listing

The server is published in the [Official MCP Registry](https://registry.modelcontextprotocol.io) as `io.github.westonhancock/orcha`. The manifest in [`server.json`](./server.json) mirrors the registry entry. A machine-readable server card is served at `https://app.tryorcha.com/.well-known/mcp/server-card.json`.

## Links

- Product: [tryorcha.com/product](https://tryorcha.com/product)
- Documentation: [tryorcha.com/docs/reference/mcp](https://tryorcha.com/docs/reference/mcp)
- Privacy: [tryorcha.com/privacy](https://tryorcha.com/privacy)
- Terms: [tryorcha.com/terms](https://tryorcha.com/terms)
- Support: support@tryorcha.com
