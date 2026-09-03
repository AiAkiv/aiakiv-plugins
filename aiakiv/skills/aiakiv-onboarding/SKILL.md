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
in the browser — sign in / sign up, approve, and the AiAkiv memory tools become
available. If no tools appear, the URL must be the canonical `/mcp` (the bare
domain returns zero tools).

If the user asks how to add AiAkiv to a **different** client, it is also published
in the official MCP Registry as `com.aiakiv/memory` — a client that can search the
registry finds it by name (`aiakiv`) or keyword (`memory`) and registers the
endpoint itself. Same address, same one-time sign-in; the registry only removes the
typing. Tell them the endpoint above either way.

**Do not send a ChatGPT user down that path.** ChatGPT cannot search the registry or
write an MCP config, and it does not complete our OAuth sign-in — it reads the `401`
and asks for a bearer token in an environment variable. For ChatGPT: add the URL in
the connector UI, and if no sign-in appears, have them issue a project-bound API key
in the console and paste that.

## 2. Confirm where memory is going (the "active target")

Every save lands in one **project** (a domain/group coordinate). Before saving,
confirm it:

- Call `get_save_target` → it reports `save_domain` / `save_group`.
- If it points at the wrong project, the user's account default is their **Main**
  project. To bind a specific folder to a specific project, add a project header
  to that folder's `.mcp.json` (see §4) — do **not** guess a project name.

## 3. Save and recall

- **Save** — only on an explicit `ak` / `mweft` / `memoryweft` utterance
  (e.g. "ak save this", "ak 저장"). Never save on a bare "remember"/"save",
  and never treat "summarize this" as a save. Use `save_memory`.
- **Recall** — `search_memory(query, mode="hybrid", top_k=5)`. Hybrid crosses
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

## 6. When a save breaks

Occasionally a `save_memory` call arrives mangled: you get a serialization
error, or the arguments turn up merged into one or missing entirely. This is a
transport problem, not a rejection of the content.

What triggers it: a long or multi-line `summary` or `content`, or non-ASCII
punctuation (`·`, `→`, `—`) landing inside an argument and breaking the
argument boundaries.

Two remedies:

1. **Order the arguments.** List `summary`, `entities` and `tags` before
   `content`, so the longest value is last and there is nothing after it to
   lose.
2. **Retry through `payload`.** Bundle the whole call into that single
   argument — one argument has no boundary left to collapse. A JSON object
   and a JSON string are both accepted, so send whichever form the client
   emits:

   ```
   payload={"summary": "...", "entities": [{"name": "X", "type": "Y"}],
            "tags": ["a/b"], "content": "..."}
   ```

   The other arguments are ignored once `payload` is given. This is the
   recovery path; the ordinary multi-argument form is the normal one.

Two related refusals are not transport problems and are not fixed by
`payload`:

- `truncation_marker_detected` — the `content` still carries a marker some
  earlier tool left when it cut the text (`<response clipped>`, `[truncated]`,
  `…N tokens truncated…`). Nothing downstream could tell a damaged body from a
  whole one, so the save is refused rather than stored half-right. Re-read the
  source in full and save again, splitting across calls if it is long. Pass
  `allow_truncation_marker=True` only when the marker is genuinely part of
  what you mean to record, such as a bug report about truncation.
- Content over 50000 characters, or a summary over 500. Split rather than
  compress: chain the pieces with `prev_event_id` and give each its own
  focused summary. A summary that wants to be long usually means the event
  covers too much, and squeezing it blurs several topics into one vector that
  then matches no specific query.
