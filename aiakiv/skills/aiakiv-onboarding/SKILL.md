---
name: aiakiv-onboarding
description: >-
  First-run helper for the AiAkiv (MemoryWeft / MWeft) memory server. Use when the
  user first connects AiAkiv, asks how to use AiAkiv, how to save or recall memory
  in AiAkiv, which AiAkiv project their memory is going to, or how to point a folder
  at a specific AiAkiv project. Explains connect → authenticate → pick project → save/recall.
---

# AiAkiv onboarding

AiAkiv is a long-term memory server (MemoryWeft / MWeft): it stores important
conversation content as a **knowledge graph** and retrieves it later. This skill
gets a new user connected and productive.

## 1. Connect & authenticate

The plugin registers one remote MCP server, `AiAkiv`, at
`https://mcp.aiakiv.com/mcp`. On first use Claude Code opens an **OAuth** login
in the browser — sign in / sign up, approve, and the `mweft_*` tools become
available. If no tools appear, the URL must be the canonical `/mcp` (the bare
domain returns zero tools).

## 2. Confirm where memory is going (the "active target")

Every save lands in one **project** (a domain/group coordinate). Before saving,
confirm it:

- Call `mweft_active_target` → it reports `save_domain` / `save_group`.
- If it points at the wrong project, the user's account default is their **Main**
  project. To bind a specific folder to a specific project, add a project header
  to that folder's `.mcp.json` (see §4) — do **not** guess a project name.

## 3. Save and recall

- **Save** — only on an explicit `ak` / `mweft` / `memoryweft` utterance
  (e.g. "ak save this", "ak 저장"). Never save on a bare "remember"/"save",
  and never treat "summarize this" as a save. Use `mweft_remember`.
- **Recall** — `mweft_search(query, mode="hybrid", top_k=5)`. Hybrid crosses
  naming variants; prefer it over guessing. Read each hit's `reason` tags and the
  `hint` connection map, not just the flat `hits`.
- To browse one person's contributions, start the query with `@<handle>`.

## 4. Bind a folder to a specific project (optional)

Default (no header) → memory goes to the account's Main project. To pin a folder
to another project, give that folder its own `.mcp.json` with a project header:

```json
{ "mcpServers": { "AiAkiv": {
  "type": "http",
  "url": "https://mcp.aiakiv.com/mcp",
  "headers": { "X-K2G-Project": "<project-name>" }
} } }
```

Changing the header value re-triggers OAuth (a login prompt, not a failure). For
friction-free per-folder switching, the console can issue a static project-bound
key instead — see the AiAkiv console → project tab.

## 5. Trust & privacy

AiAkiv is a **hosted** service: saved content leaves the machine and is stored on
AiAkiv servers. Only save what the user intends to persist. The user controls
their data through the AiAkiv console (view, export, purge).
