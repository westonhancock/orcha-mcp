# Orcha MCP server

Orcha gives AI agents a shared, permission-scoped context library: native files and folders, structured databases, curated context bundles, and read-only documents synced from connected sources, all searchable with citations back to the exact source version.

This repository is the public connection guide and registry listing for Orcha's hosted MCP server. The service runs at:

```
https://app.tryorcha.com/mcp
```

Transport is Streamable HTTP. The application source code is private; nothing needs to be installed or built from this repository to use the server.

## What the server provides

24 tools spanning:

- Ranked search over all indexed workspace content (`search_context`) with fast, balanced, and fresh routing modes, plus a citation on every result
- File and folder reading, creation, update, organization, and deletion, scoped to what the connection is allowed to touch
- Curated context bundles with usage guidance (`list_bundles`, `get_bundle`)
- Structured databases with typed queries (`list_databases`, `query_database`, record CRUD)
- Direct access to connected read-only sources (`query_source`, `fetch_source_document`, `list_source_documents`)
- Filesystem-shaped browsing (`browse_context`): ls, tree, find, cat, grep, and stat over a virtual tree of the workspace

Every connection is isolated to the organization, workspace, permissions, and optional folder scope it was granted.

## Connect with OAuth (recommended)

Add the server URL to any OAuth-capable MCP client. The client discovers Orcha's authorization server automatically and walks you through consent; the grant is scoped to the organization and workspace you approve.

Claude Code:

```bash
claude mcp add --transport http orcha "https://app.tryorcha.com/mcp"
```

claude.ai and Claude Desktop: Settings, Connectors, Add custom connector, then enter `https://app.tryorcha.com/mcp`.

Cursor and other OAuth-capable clients: add a remote MCP server with the same URL and follow the sign-in prompt.

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
- "Check which Orcha context bundles apply to this task and use the best match as the source of truth."
- "Create a Markdown file named `launch-checklist.md` from these release requirements."
- "Get the `engineering-onboarding` context bundle and identify any conflicting guidance."

## Registry listing

The server is published in the [Official MCP Registry](https://registry.modelcontextprotocol.io) as `io.github.westonhancock/orcha`. The manifest in [`server.json`](./server.json) mirrors the registry entry. A machine-readable server card is served at `https://app.tryorcha.com/.well-known/mcp/server-card.json`.

## Links

- Product: [tryorcha.com/product](https://tryorcha.com/product)
- Documentation: [tryorcha.com/docs/reference/mcp](https://tryorcha.com/docs/reference/mcp)
- Privacy: [tryorcha.com/privacy](https://tryorcha.com/privacy)
- Terms: [tryorcha.com/terms](https://tryorcha.com/terms)
- Support: support@tryorcha.com
