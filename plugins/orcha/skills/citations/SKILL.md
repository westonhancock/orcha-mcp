---
name: citations
description: "Cite Orcha context correctly and judge whether it is current. Use when answering from an Orcha workspace and you need to attribute a claim, link back to a document, or decide whether a synced source is too stale to trust."
---

# Citing and dating Orcha context

Orcha's value is that an answer can be traced back to the exact document and
version it came from. That only holds if you carry the provenance through.

## Every result already carries its citation

`search_context` results include:

- `citation`: the provenance record, pointing at the exact upstream version
- `documentTitle`: what to name in prose
- `location`: `startLine`, `endLine`, and `headingPath` when the chunk came from structured text
- `context`: the owning folder or connected source, plus its `useWhen` guidance
- `updatedAt`: when the underlying document last changed
- `fileId`: for Orcha-native files, the handle for a follow-up `get_file`

`browse_context` gives you the same thing for any node through `stat <path>`,
which returns the citation, a deep link, and freshness.

Never construct a citation by hand from a file name. Use the one you were
given.

## What to put in an answer

Name the document, and give the line anchor when you have one:

> Deploys are gated on CI passing for the same commit
> (`RELEASE_PROCESS.md`, lines 40-58).

When several chunks support one claim, cite the one you actually read rather
than all of them. When nothing supports it, say that instead of reaching for
the nearest document.

## coverage is not evidence

`search_context` and `query_source` both return a `coverage` block. It explains
routing: what was fetched live, what exists but was not fetched, where a budget
ran out. It is diagnostic text about the search, not content from the
workspace.

Quoting coverage as though it were a source is the single most misleading thing
you can do with this API. If coverage says a document exists but was not
fetched, either fetch it with `fetch_source_document` or tell the user it was
not read.

## Judging freshness

Three signals, in increasing specificity:

1. `get_workspace_overview` gives each connected source a `lastSyncedAt` and `lastSyncStatus`. A source whose last sync failed is a snapshot of unknown age.
2. `list_source_documents` gives each document an `indexStatus` and `lastSyncedAt`.
3. A result's own `updatedAt` is the tightest signal for that specific chunk.

When the question is about current state and the newest thing you found is old,
say so plainly: "the most recent note on this is from March, so this may have
changed." That sentence is worth more than a confident wrong answer.

When recency is the whole point, run `search_context` with `mode: "fresh"` so
live providers are consulted, and note in your answer that you did.

## Content you cannot cite

An unindexed document does not appear in search results at all. It is not
missing from the workspace, it is missing from the index. `get_workspace_overview`
reports `pending` and `failed` counts, and the `troubleshooting` skill covers
what to do about them. Do not report "the workspace has nothing on X" when what
you mean is "the index has nothing on X yet".
