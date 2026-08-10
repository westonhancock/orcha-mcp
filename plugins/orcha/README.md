# Orcha plugin for Claude Code

Connects Claude Code to an Orcha workspace and installs the usage guides that
tell an agent how to search it well.

## Install

```
/plugin marketplace add westonhancock/orcha-mcp
/plugin install orcha@orcha
```

The plugin brings its own MCP server configuration, so there is no separate
`claude mcp add` step. On first use Claude Code walks you through OAuth, where
you pick the workspace and choose which of `read`, `write`, and `delete` the
connection gets. Permissions are capped at your workspace role at consent time
and re-checked on every request.

## What you get

The MCP server, with 28 tools covering ranked search across every indexed
document, file and folder operations, curated memories, structured
databases, direct access to connected read-only sources, and a
filesystem-shaped browser over the whole workspace.

Four skills that Claude loads when they are relevant:

| Skill | Covers |
|---|---|
| `/orcha:retrieval` | Which tool answers which kind of question, and how to tune `search_context` |
| `/orcha:citations` | Attributing claims to documents and judging whether a source is stale |
| `/orcha:authoring` | Writing files, folders, and database records back safely |
| `/orcha:troubleshooting` | Empty results, scope errors, unindexed content, and sending feedback |

Claude uses these on its own when the task calls for one. You can also invoke
one directly by name.

## Why the skills matter

Orcha exposes several retrieval paths that look interchangeable and are not.
Semantic search, exact-string grep over a virtual filesystem, typed database
queries, and curated memories each answer a different shape of question. An
agent guessing between them wastes calls and returns worse answers. The skills
encode the routing rules, the response formats, and the failure modes, and they
cost nothing until Claude actually loads one.

## Other clients

The same guidance reaches non-Claude-Code clients through the server itself.
Every MCP client can call the `get_usage_guide` tool, which returns these exact
files. Clients that implement the draft `io.modelcontextprotocol/skills`
extension can also discover them through `skills/list` and read them as
`skill://orcha/<name>/<path>` resources.

## Source

These skills are generated from the Orcha application repository and synced
here. File issues at https://github.com/westonhancock/orcha-mcp/issues.
