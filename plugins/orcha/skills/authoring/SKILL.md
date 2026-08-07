---
name: authoring
description: "Write to an Orcha workspace safely: create and update files and folders, and add or change structured database records. Use when an Orcha MCP session needs to save something back rather than only read."
---

# Writing to Orcha

Most Orcha sessions are read-only. When you do write, the failure modes are
overwriting content you never read and creating things where nobody will find
them.

## What is writable

| Writable | Read-only |
|---|---|
| Orcha-native files (`create_file`, `update_file`, `delete_file`) | Documents synced from a connected source |
| Folders (`create_folder`) | Anything under `/sources` in `browse_context` |
| Structured database records | Bundle membership |

There is no tool that edits a synced document, and that is deliberate. The
upstream system owns it. If a synced document is wrong, tell the maintainers
with `submit_feedback` rather than trying to correct it in Orcha.

## update_file replaces the whole body

`update_file` takes `id` and `content`, and `content` becomes the entire file.
There is no patch or append. So:

1. `get_file` with the id, with no line range, to get the current full body
2. Apply your change to that text
3. `update_file` with the complete new body

Skipping step 1 silently destroys everything you did not include. If the file
is large enough that reading it whole is uncomfortable, that is a signal to
reconsider whether you should be rewriting it at all.

## create_file

```json
{
  "name": "incident-2026-08-06.md",
  "content": "...",
  "fileType": "markdown",
  "folderId": "<optional>"
}
```

`fileType` is one of `markdown`, `json`, or `yaml`. Omitting `folderId` puts
the file at the workspace root, which is usually not what anyone wants. Call
`list_folders` or `get_workspace_overview` first and place it somewhere with a
`useWhen` that matches what you are writing.

Match the naming convention of the files already in that folder rather than
inventing one.

## Database records

`values` maps **property IDs** to typed values, and property IDs are UUIDs, not
names. Call `get_database` first to get the schema, then build `values` from
it. Guessing property names produces a record with no populated fields.

- `create_database_record(databaseId, values)`
- `update_database_record(databaseId, recordId, values)` merges the values you pass and leaves the rest alone
- `delete_database_record(databaseId, recordId)` moves the record to trash

Unlike `update_file`, a record update is a merge, so you only need to send the
properties you are changing.

## Deletes are soft

`delete_file` and `delete_database_record` move things to trash rather than
destroying them, which makes them recoverable. That is a reason to be calm
about a mistake, not a reason to be careless: the user still has to go find it
and restore it.

## Permission and scope errors

Writes need the `write` permission and deletes need `delete`. A token can also
be scoped to specific folders, in which case creating outside that scope fails
with "Cannot create file outside token scope". Both are configuration facts
about the token, not something to work around. Report what the token cannot do
and let the user widen it if they want to.
