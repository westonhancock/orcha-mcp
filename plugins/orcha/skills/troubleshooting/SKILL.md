---
name: troubleshooting
description: "Diagnose why an Orcha tool returned nothing, refused a call, or gave stale results, and report context problems back to the workspace maintainers. Use when an Orcha MCP call fails, comes back empty, or the workspace seems to be missing something it should have."
---

# When Orcha does not answer

An empty result has several very different causes, and telling the user "the
workspace has nothing on that" is wrong for most of them.

## Read the error before retrying

| What you saw | What it means |
|---|---|
| `Missing required permission: read` / `write` / `delete` | The token lacks that permission. Retrying will not help |
| `Token must be scoped to a workspace` | The token has no workspace. Nothing will work until that is fixed |
| `File is outside token scope` / `Database not found or outside token scope` | It may well exist. This token is folder-scoped and cannot see it |
| `Folder-scoped tokens cannot query connected sources` | All connected-source tools are closed to this token |
| `Bundle not found` | Check you passed the **slug**, not the ID or the display name |
| `(no output)` from `browse_context` | The script ran and matched nothing. The path may be wrong; try `ls` one level up |

Scope errors are the ones most often misread. "Not found or outside token
scope" is deliberately ambiguous so the API does not leak what exists outside
your scope. Report it as a scope limit, not as absence.

## Empty search results

Work through these in order:

1. **Is it indexed?** `get_workspace_overview` reports `pending` and `failed` counts. A document that is pending is invisible to `search_context` even though it is in the workspace.
2. **Is it a synced document you searched for with a file tool?** `list_files` and `search_files` never see connected-source documents. Use `search_context`.
3. **Is the vocabulary wrong?** Add typed sub-queries: a `lex` entry with the exact identifiers, a `hyde` entry written as the answer. See the `retrieval` skill.
4. **Does it exist at all?** `browse_context` with `rg -il` over `/files` and `/sources` answers this directly. If ripgrep finds nothing, the content is genuinely not there.
5. **Is it upstream but not retained?** Check `coverage`. If it names documents that were cataloged but not fetched, run again with `mode: "fresh"` or pull the specific one with `fetch_source_document`.

## Partial results

`browse_context` output ending in `[scan budget exhausted: ...]` means the scan
stopped early. Narrow the path or pattern and rerun. Never present a
budget-truncated list as complete.

`search_context` with a `tokenBudget` set will also trim. If your answer
depends on having seen everything, raise the budget or narrow the query.

## Stale results

Check `lastSyncedAt` and `lastSyncStatus` per source in
`get_workspace_overview`. A source whose last sync failed is a snapshot of
unknown age, and nothing in it should be presented as current state. Say what
you found and when it was last synced.

## Tell the maintainers

`submit_feedback` sends a message to the workspace team. Nothing is stored in
the workspace, so it does not pollute the context you are searching.

```json
{
  "message": "Searched for the on-call rotation four ways and found only a 2025 draft. The runbooks folder has nothing current.",
  "category": "missing_context",
  "context": { "queries": ["on-call rotation", "who is paged"], "folder": "/files/engineering/runbooks" }
}
```

Categories are `missing_context`, `stale_content`, `wrong_result`,
`tool_problem`, and `other`.

Use it when the gap is worth someone fixing: context that should exist and does
not, a document that is confidently out of date, a search that returns the
wrong thing for an obvious query, or a tool that misbehaves. Be specific about
what you looked for, what you expected, and what you got instead. A vague
report is not actionable.

Do not use it as a scratchpad, and do not send one on every empty search. A
question the workspace was never meant to answer is not a defect.
