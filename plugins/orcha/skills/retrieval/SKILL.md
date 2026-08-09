---
name: retrieval
description: "Pick the right Orcha tool for a question and get evidence back on the first try. Use when working with an Orcha workspace over MCP and you need to find something: content in files or synced sources, a value in a structured database, the shape of the workspace, or a curated memory."
---

# Finding things in Orcha

Orcha holds four kinds of context, and each one has a tool that reaches it well
and several that reach it badly. Pick by the shape of the question, not by
whichever tool you called last.

| The question is | Use | Not |
|---|---|---|
| About content: "what does X say", "how do we handle Y" | `search_context` | `list_files` then reading everything |
| Structural: what exists, where it lives, an exact string | `browse_context` | `search_context` with a filename as the query |
| About typed fields: statuses, owners, dates, counts | `list_databases` → `get_database` → `query_database` | `search_context` |
| Named a memory, or the task matches a memory's `useWhen` | `list_memories` → `get_memory` | loading files one at a time |
| Aimed at one provider: "check the Linear issues" | `query_source` | `search_context` across everything |
| "What is in this workspace?" | `get_workspace_overview` | guessing |

When you are new to a workspace, call `get_workspace_overview` once. It returns
folders with their `useWhen` guidance, memories, connected sources with their
`sourceId` and `lastSyncedAt`, and index counts. Most bad retrieval sessions
start with skipping it and inventing folder names.

## The trap that catches everyone

Documents synced from a connected source are **not files**. `list_files`,
`get_file`, `get_files`, and `search_files` only ever see Orcha-native files. A
GitHub repository's contents will not appear in `list_files` no matter how many
times you call it.

Reach synced content with `search_context` (searches everything indexed),
`list_source_documents` (browse one source), or `fetch_source_document` (one
document by `externalId`).

## Using search_context well

The query supports `"quoted phrases"` and `-excluded` terms.

For a hard question, add 2 to 4 typed sub-queries in `searches`. Your original
query always runs at double weight, so sub-queries broaden recall without
taking over the ask:

- `lex`: exact keywords likely to appear verbatim (identifiers, error strings, names)
- `vec`: a natural-language restatement of the question
- `hyde`: a 30 to 80 word passage written as if it were the document that answers the question

`mode` controls how much upstream freshness costs you. `fast` never contacts
providers. `balanced` (the default) may fetch a few cataloged-but-unindexed
documents live when the catalog decisively matches. `fresh` actively consults
live providers. Reach for `fresh` only when recency is the point, such as "what
changed this week".

The response separates `results` from `coverage`. Results are evidence.
Coverage is routing notes: what was fetched live, what exists but was not
fetched, where a budget ran out. **Never quote coverage as content.**

Do not fetch whole files off the back of a hit. Each result carries `fileId`
and `location.startLine`, so follow up with `get_file` using `fromLine` and
`maxLines`, or batch up to 20 known files with `get_files` and a `maxBytes`
cap.

See `references/search-tuning.md` for worked query examples.

## Using browse_context well

`browse_context` runs a read-only shell over a virtual tree: `/files`,
`/memories`, `/databases` (one markdown file per record), and `/sources`.
Available commands are `ls`, `tree`, `find`, `cat`, `grep`, `rg`, `head`,
`tail`, `wc`, and `stat`. Pipe with `|`, chain with `;` or `&&`.

Start with `cat /README.md` or `tree -L 2 /files`. Use `stat <path>` when you
need a node's citation, deep link, and freshness.

This is the right tool for "does anything mention `ORCHA_TOKEN`" and the wrong
tool for "how does authentication work". Keep semantic questions on
`search_context`.

See `references/browse-context-recipes.md` for scripts that work.

## Querying a structured database

`query_database` filters and sorts by **property ID**, not property name, and
those IDs are UUIDs. Call `get_database` first to map names to IDs, or every
filter you write will fail. One property filter per query, with operators
`contains`, `equals`, `not_equals`, `is_empty`, and `is_not_empty`.

## Memories

`get_memory` takes a **slug**, not an ID, and returns the complete current
contents of every member: files, folders expanded to their current files,
retained connected-source documents, and database records. That is a
deliberate, sometimes large load, and its `coverage` array reports anything
excluded, truncated, or unavailable. Use it when the user names a memory or
the task matches the `useWhen` text from `list_memories`. For an open-ended
question, search instead.

## A sensible default sequence

1. `get_workspace_overview` if you do not already know the workspace
2. `search_context` with a real question and 2 to 4 typed sub-queries
3. Follow the best hits with `get_file` using the returned line anchors
4. `browse_context` if you need to confirm what exists or grep an exact string
5. Cite what you used (see the `citations` skill)
