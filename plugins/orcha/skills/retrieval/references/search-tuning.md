# Tuning search_context

## When the plain query is enough

Short factual lookups where the wording in the document probably matches the
wording in the question:

```json
{ "query": "deployment target for production" }
```

## When to add typed sub-queries

Add `searches` when the first attempt returned thin or off-target results, or
when the question uses vocabulary the documents probably do not.

The three types feed different retrieval arms. `lex` goes to full-text, `vec`
and `hyde` go to the vector arm. Mixing them is the point; three `lex` entries
buy you very little over one.

```json
{
  "query": "why do MCP sessions drop after a deploy",
  "searches": [
    { "type": "lex", "text": "streamable-http session sessionIdGenerator stateless" },
    { "type": "vec", "text": "MCP connections failing or resetting following a new deployment" },
    { "type": "hyde", "text": "The MCP transport runs in stateless mode, so each POST constructs a fresh server and transport. Sessions are not retained across requests, and a redeploy therefore ends any in-flight connection. Clients reconnect on the next call." }
  ]
}
```

Write the `hyde` passage as if it were already the answer, in the register the
document would use. It is a retrieval probe, not a guess you will present to
anyone, and it never appears in your output.

## Narrowing

- `sourceIds` restricts to up to 10 connected sources. Get the IDs from `get_workspace_overview`.
- `minScore` drops weak matches. Useful when a broad query returns noise.
- `limit` defaults to 10 and caps at 50.
- `tokenBudget` bounds roughly how much content comes back. Set it when you are assembling context for a long task.

## Reading the response

```
results[]   evidence. Each has chunkId, documentTitle, fileId, content, score,
            location {startLine, endLine, headingPath}, context, updatedAt, citation
coverage    routing notes. What was fetched live, what exists but was skipped,
            budget or availability limits. Not evidence.
mode        the freshness mode actually used
```

`context` tells you which folder or connected source a hit came from, along
with that container's `useWhen` guidance. It is the fastest way to judge
whether a hit is relevant before you spend a fetch on it.

## Following up

```json
{ "id": "<fileId from the result>", "fromLine": 120, "maxLines": 80 }
```

`get_file` returns a `[lines 120-200 of 640]` header so you know where you
landed. When you already know several files you need, `get_files` takes up to
20 IDs and a `maxBytes` cap that skips oversized files rather than blowing your
context.

## Escalating freshness

Use `mode: "fresh"` when the question is explicitly about recent state and
`coverage` from a `balanced` run said something was cataloged but not fetched.
Fresh mode consults live providers and spends the connection's daily provider
budget, so it is a deliberate choice rather than a default.
