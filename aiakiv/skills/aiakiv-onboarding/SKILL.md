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

The plugin registers one remote MCP server, `AiAkiv`. On first use Claude Code
opens an **OAuth** login in the browser: sign in or sign up, approve, and the
AiAkiv memory tools become available.

The service address is `https://mcp.aiakiv.com/mcp`, and it must end with `/mcp`.
If a connection the user set up by hand shows no tools, check that its URL ends
with `/mcp` (the bare domain returns zero tools).

If the user asks how to add AiAkiv to a **different** client, it is also published
in the official MCP Registry as `com.aiakiv/memory` — a client that can search the
registry finds it by name (`aiakiv`) or keyword (`memory`) and registers the
endpoint itself. Same address, same one-time sign-in; the registry only removes the
typing. Tell them the endpoint above either way.

**Do not send a ChatGPT user down that path.** ChatGPT cannot search the registry or
write an MCP config. In ChatGPT, AiAkiv is installed from the plugin directory inside
the app: search for AiAkiv Memory, install it and sign in. The steps are at
https://www.aiakiv.com/docs/chatgpt.

## 2. Confirm where memory is going (the "active target")

Every save lands in one **project** (a domain/group coordinate). Before saving,
confirm it:

- Call `get_save_target` → it reports `save_domain` / `save_group`.
- If it points at the wrong project, the user's account default is their **Main**
  project. To bind a specific folder to a specific project, put the project name
  in the URL of that folder's `.mcp.json` (see §4) — do **not** guess a project name.

## 3. Save and recall

- **Save** — only on an explicit `ak` / `mweft` / `memoryweft` utterance
  (e.g. "ak save this", "ak 저장"). Never save on a bare "remember"/"save",
  and never treat "summarize this" as a save. Use `save_memory`.
- **Recall** — `search_memory(query, top_k=5)`. Ask with a full sentence rather
  than keywords; the ranking is semantic, so a guessed keyword is worse than the
  real question. Read each hit's `reason` tags and the `hint` connection map,
  not just the flat `hits`.
- To browse one person's contributions, start the query with `@<handle>`.

## 4. Bind a folder to a specific project (optional)

Default (no `project` in the URL) → memory goes to the account's Main project. To
pin a folder to another project, give that folder its own `.mcp.json` whose URL
names the project:

```json
{ "mcpServers": { "AiAkiv": {
  "type": "http",
  "url": "https://mcp.aiakiv.com/mcp?project=<project-name>"
} } }
```

URL-encode the name if it has spaces or non-ASCII characters (`My Project` becomes
`My%20Project`). The console's project tab gives a ready-to-copy version of this
config with the name already filled in and encoded.

Name the project in the URL, not only in an `X-K2G-Project` header. Some clients
do not send a custom header on every request, so a folder that relies on the
header alone can end up following Main. If the user has an older header-only
config, move the project name into the URL.

Changing the project in the URL may bring up the OAuth login again (a login
prompt, not a failure). For friction-free per-folder switching, the console can
issue a static project-bound key instead — see the AiAkiv console → project tab.

If the user turns Main off in the console, every connection that follows Main (no
`project` in the URL and no project-bound key) refuses all tools until Main is
turned back on; folders pinned to a project keep working.

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

## 7. Which connection has app features

**App features** means asking the AI to do app work through
`run_aiakiv_app_action`, such as console management (teams, invitations,
projects, Main, cards) or working in Contents.

- Saving, searching, reading full text, reading linked teams, graph queries
  and creating cards work on every AiAkiv connection.
- A connection installed with the AiAkiv plugin for Claude Code or Cursor,
  like the one this plugin registers, has app features. So do folder
  connections (a `project` in the URL, §4) and API key connections.
- A connection made in another app with the address alone and a sign-in,
  such as the account connector in Claude on the web or in the mobile apps,
  does not. There the tool is missing from the tool list, and a call by name
  is refused with "This tool is not available on this connection." When a
  conversation has both kinds, call the tool on the connection that has it.
- Everything the console app does, the user can also do by hand in the web
  console (the dashboard), https://app.aiakiv.com. Having the AI do it from
  a connection without app features takes a new MCP connection; the
  `aiakiv-console` skill says what to tell the user.
