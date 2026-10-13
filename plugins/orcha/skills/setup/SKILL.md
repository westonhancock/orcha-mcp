---
name: setup
description: "Set up the Orcha plugin after installation: confirm the workspace the connection points at, check which permissions it was granted, and run a first search so the user sees cited results. Use right after installing Orcha, or when a user asks to connect, reconnect, or switch their Orcha workspace."
---

# Setting up Orcha

Orcha connects over OAuth. The user signed in during installation and chose a
workspace and a permission set (`read`, `write`, `delete`) on Orcha's consent
screen. This skill confirms that choice worked and shows the user what the
plugin can now do. It needs no secrets and asks for none.

## Steps

1. Call `get_workspace_overview`. It returns the workspace name, file and
   folder counts, index status, available memories, and connected sources.
   - If the call fails with an authentication error, the connection was not
     completed. Ask the user to reconnect the plugin; Orcha opens its sign-in
     and consent screen again.
   - If it reports `Token must be scoped to a workspace`, the consent step was
     skipped. Ask the user to reconnect and pick a workspace.
2. Tell the user which workspace is connected and summarize what is in it in
   two or three sentences: how many files, which memories exist and when each
   applies (its `useWhen`), and which connected sources are synced.
3. Check permissions by reading the overview only. Do not test write access by
   creating anything. If the user wants to write from ChatGPT or Codex, tell
   them write permission is chosen on Orcha's consent screen at connection
   time and capped at their workspace role; they can reconnect to change it.
4. Offer one first search based on what the overview shows, for example a
   folder name or a connected source, and run `search` with it. Present the
   top results with their links so the user sees how citations work.
5. If the overview shows `pending` or `failed` documents, say so: those are
   not searchable yet. Point the user to the source's Manage view in Orcha for
   sync status.

## Switching workspaces

A connection serves one workspace. To use another, the user reconnects the
plugin and picks the other workspace on the consent screen. Revoking the old
grant happens on Orcha's MCP settings page.

## What not to do

- Do not ask the user for an API token. The plugin uses OAuth only.
- Do not create files, memories, or records during setup.
- Do not describe content you have not retrieved. Everything you say about the
  workspace should come from the overview or a search result.
