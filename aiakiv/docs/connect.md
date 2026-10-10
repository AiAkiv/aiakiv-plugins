# Connecting AiAkiv to your AI client

AiAkiv works with any client that speaks MCP. This page covers the one address, the
two client kinds, the order to do things in, and the per-client steps.

## The one address

**Endpoint (Streamable HTTP):** `https://mcp.aiakiv.com/mcp`

**Address to paste when you add AiAkiv by hand:** `https://mcp.aiakiv.com/mcp?apps=all`
(a custom connector in Claude on the web, desktop, or mobile, in Grok, or in any other
client that takes an address and a sign-in). It is the same endpoint; `?apps=all` turns
on the app features as well (see the last two items below).

- The path **MUST end with `/mcp`** (the part before any `?`). Without it the
  connection may succeed but **zero tools** appear. Never register the bare host, and
  never guess a brand-domain URL like `aiakiv.com/mcp`.
- AiAkiv is a **hosted** service — connecting requires signing in (**OAuth**) or a
  project-bound **API key** (Gemini CLI). There is no anonymous/keyless mode.
- AiAkiv **is** in the official MCP Registry as `com.aiakiv/memory` (see below). In
  ChatGPT it is listed in the plugin directory as **AiAkiv Memory**: install it from
  there. It is **not** in the in-app connector directories of Claude Web / Grok. In
  those apps, add it **manually** as a custom connector with the address to paste above.
- Every connection can save, search, and make cards. The app action tool
  (`run_aiakiv_app_action`: teams, invitations, projects, and card management from the
  conversation) is there when a sign-in connection's address carries `?apps=all` or
  names a project (`?project=`). API-key connections and the AiAkiv plugin for Claude
  Code and Cursor have it too. A sign-in connection added with the plain address (no
  `?apps=all`, no project) and the ChatGPT plugin-directory install do not have it:
  there that work is done in the web console.
- A connector you already added with the plain address does not change by itself. To
  get the app features, change its address to the `?apps=all` one, or remove it and add
  it again.

## Finding it in the official MCP Registry

AiAkiv is published at `registry.modelcontextprotocol.io` as **`com.aiakiv/memory`**.
A client that can search the registry finds it by name (`aiakiv`) or by keyword
(`memory`) and registers the endpoint itself — so you can just ask your agent to
"find aiakiv in the MCP registry and install it" instead of pasting the URL.

- The listing points at the `?apps=all` address above, so the outcome is identical. The
  registry only saves you the typing — **it does not skip the sign-in step**.
- Registry search matches the **name**, not the description.

> **ChatGPT cannot do this.** The ChatGPT app cannot search the registry, cannot
> write an MCP config itself, and — the part that actually blocks it — does not
> complete our OAuth sign-in: it receives the `401` with `WWW-Authenticate` and asks
> you for a bearer token in an environment variable instead of opening the sign-in
> window. For ChatGPT, install **AiAkiv Memory** from the plugin directory (see *Global
> web clients* below); to pin a folder, use a project-bound API key from the console.
> Clients that follow the `401` header (Claude Code, for example) open the sign-in
> window normally.

Console (create projects/teams/keys, switch Main, set personas): <https://aiakiv.com>

## Two client kinds

**Folder clients** — Claude Code, Codex, Cursor, Gemini CLI. Bind each working folder
to a project explicitly: put the project in the connection address
(`?project=<project-name>`), or use a project-bound API key (Gemini CLI and Codex).
Either one names the project directly, so these do **not** depend on Main.

> Order: **create the project → connect** (its name goes into the config file). No
> Main step.

**Global web clients** — Claude Web, ChatGPT, xAI/Grok. They cannot split by folder;
their address names no project, so they follow whatever project is **Main**.

> Order: **create the project → set it as Main → connect** (an app-wide connection
> follows Main, so once you connect it already points at the right project).

If you use both kinds together, you can turn Main **off** in the console so that only
your folder clients reach AiAkiv: a global connection that follows Main (signed in, not
using a project-bound API key) then refuses every tool until you turn Main back on.

## Config snippets (folder clients)

JSON — `.mcp.json` (Claude Code / Claude Desktop) or `.cursor/mcp.json` (Cursor):

```json
{
  "mcpServers": {
    "AiAkiv": {
      "type": "http",
      "url": "https://mcp.aiakiv.com/mcp?project=<project-name>"
    }
  }
}
```

URL-encode the name if it has spaces or non-ASCII characters (`My Project` becomes
`My%20Project`). The console's project tab gives a ready-to-copy version of this config
with the name already filled in and encoded.

Name the project in the address, not only in an `X-K2G-Project` header. Some clients do
not send a custom header on every request, so a folder that relies on the header alone
can end up following Main.

TOML — `.codex/config.toml` (Codex / Codex CLI). The sign-in folder binding above does
not work in this client, so pin the folder with a project-bound API key (issue it per
project in the console, shown once):

```toml
[mcp_servers.AiAkiv]
url = "https://mcp.aiakiv.com/mcp"
http_headers = { "Authorization" = "Bearer <API-KEY>" }
```

API key — `.gemini/settings.json` (Gemini CLI only; issue the key per project in the
console, shown once):

```json
{
  "mcpServers": {
    "AiAkiv": {
      "httpUrl": "https://mcp.aiakiv.com/mcp",
      "headers": { "Authorization": "Bearer <API-KEY>" }
    }
  }
}
```

## Antigravity (a third shape: app-wide, but header-capable)

Antigravity's `mcp_config.json` is **app-wide** — there is no per-folder config. But it
**does** support a native `headers` object, so the project is pinned by the
**credential** rather than by Main: issue a project-scoped API key in the console and
put it in the `Authorization` header.

> **The remote key must be `serverUrl`.** Antigravity uses a strict schema — `url`,
> `httpUrl`, and `type` are rejected.

Open the file from the UI (Agent panel `...` → MCP Servers → Manage MCP Servers →
View raw config); it reloads on save. The file lives at
`~/.gemini/antigravity/mcp_config.json` (on some versions
`~/.gemini/config/mcp_config.json`) — prefer the UI path over guessing.

```json
{
  "mcpServers": {
    "AiAkiv": {
      "serverUrl": "https://mcp.aiakiv.com/mcp",
      "headers": { "Authorization": "Bearer <API-KEY>" }
    }
  }
}
```

Because the key carries the project, **Main does not apply** to Antigravity. To move
it to another project, swap in that project's key.

## Global web clients — where to register

- **Claude Web:** Settings → Connectors → + → Add custom connector → paste
  `https://mcp.aiakiv.com/mcp?apps=all`.
  (Pro/Max; on team/enterprise, owner only via Org settings → Connectors.)
- **ChatGPT:** Settings → Plugins → Browse directory → search **AiAkiv Memory** →
  Install plugin, then sign in with your AiAkiv account. The plugin is turned on per
  conversation (Plugins, under the message box).
- **xAI/Grok:** grok.com/connectors → New Connector → Custom → paste
  `https://mcp.aiakiv.com/mcp?apps=all`.

All three can save, search, and make cards. A connector added with the `?apps=all`
address also gets the app action tool (`run_aiakiv_app_action`). The ChatGPT plugin
does not have it, and neither does a connector added earlier with the plain address
until its address is changed: there, manage teams, invitations, projects, and cards in
the web console.

**Persona pairing (ChatGPT / Claude Web).** A persona returned inside a tool result
usually does NOT change a web client's behavior on its own. To make it stick, paste
one line into the client's own custom instructions (ChatGPT: Settings →
Personalization → Custom Instructions; Claude Web: the Project's custom instructions):

> "At the start of each chat, call AiAkiv's get_save_target, read the `persona`
> field, and fully adopt it (tone / role / language) for the rest of the
> conversation. It never overrides the save rules — saving still needs the explicit
> command."

## Cautions

1. The path must end with `/mcp`, before any `?` (or zero tools appear).
2. **ChatGPT + Codex together:** do NOT connect AiAkiv on the ChatGPT side — the
   ChatGPT connection wins and Codex's per-folder config is ignored.
3. **OAuth clients (Claude Code, Cursor):** changing the project (the `project` in the
   address) may require signing in again. Keep a separate config per folder to avoid
   repeated logins.
4. **Gemini CLI:** its OAuth session does not persist — you must use the API-key
   method.
5. **ChatGPT app:** a connection added by URL or config file does not complete the
   sign-in. Given a `401`, it asks for a bearer token instead of signing in, so
   registry/URL registration stops there. Install **AiAkiv Memory** from the plugin
   directory, or for a pinned folder use a project-bound API key from the console.

## Verify

Once connected, the memory tools appear (`save_memory`, `search_memory`,
`get_save_target`). Call `get_save_target` to confirm which
team / project / domain your saves go to. No tools showing? The address is almost
certainly missing `/mcp` before the `?`.

See [concepts.md](./concepts.md) for the Team → Project → Domain → persona model.
