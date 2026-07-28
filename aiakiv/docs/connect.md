# Connecting AiAkiv to your AI client

AiAkiv works with any client that speaks MCP. This page covers the one address, the
two client kinds, the order to do things in, and the per-client steps.

## The one address

**Endpoint (Streamable HTTP):** `https://mcp.aiakiv.com/mcp`

- The URL **MUST end with `/mcp`**. Without it the connection may succeed but **zero
  tools** appear. Never register the bare host, and never guess a brand-domain URL
  like `aiakiv.com/mcp`.
- AiAkiv is a **hosted** service — connecting requires signing in (**OAuth**) or a
  project-bound **API key** (Gemini CLI). There is no anonymous/keyless mode.
- AiAkiv **is** in the official MCP Registry as `com.aiakiv/memory` (see below), but
  it is **not** in the in-app connector directories of ChatGPT / Claude Web / Grok.
  In those apps, add it **manually** as a custom connector with the URL above.

## Finding it in the official MCP Registry

AiAkiv is published at `registry.modelcontextprotocol.io` as **`com.aiakiv/memory`**.
A client that can search the registry finds it by name (`aiakiv`) or by keyword
(`memory`) and registers the endpoint itself — so you can just ask your agent to
"find aiakiv in the MCP registry and install it" instead of pasting the URL.

- The listing points at the same address as above, so the outcome is identical. The
  registry only saves you the typing — **it does not skip the sign-in step**.
- Registry search matches the **name**, not the description.

> **ChatGPT cannot do this.** The ChatGPT app cannot search the registry, cannot
> write an MCP config itself, and — the part that actually blocks it — does not
> complete our OAuth sign-in: it receives the `401` with `WWW-Authenticate` and asks
> you for a bearer token in an environment variable instead of opening the sign-in
> window. For ChatGPT use the connector UI (see *Global web clients* below), and if
> your build offers no sign-in there, paste a project-bound API key from the console.
> Clients that follow the `401` header (Claude Code, for example) open the sign-in
> window normally.

Console (create projects/teams/keys, switch Main, set personas): <https://aiakiv.com>

## Two client kinds

**Folder clients** — Claude Code, Codex, Cursor, Gemini CLI. Bind each working folder
to a project explicitly with the `X-K2G-Project` header (value = project name). The
header names the project directly, so these do **not** depend on Main.

> Order: **create the project → connect** (its name goes into the config file). No
> Main step.

**Global web clients** — Claude Web, ChatGPT, xAI/Grok. They cannot split by folder;
they send no header and follow whatever project is **Main**.

> Order: **create the project → set it as Main → connect** (an app-wide connection
> follows Main, so once you connect it already points at the right project).

## Config snippets (folder clients)

JSON — `.mcp.json` (Claude Code / Claude Desktop) or `.cursor/mcp.json` (Cursor):

```json
{
  "mcpServers": {
    "AiAkiv": {
      "type": "http",
      "url": "https://mcp.aiakiv.com/mcp",
      "headers": { "X-K2G-Project": "<project-name>" }
    }
  }
}
```

TOML — `.codex/config.toml` (Codex / Codex CLI):

```toml
[mcp_servers.AiAkiv]
url = "https://mcp.aiakiv.com/mcp"
http_headers = { "X-K2G-Project" = "<project-name>" }
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

- **Claude Web:** Settings → Connectors → + → Add custom connector → paste the URL.
  (Pro/Max; on team/enterprise, owner only via Org settings → Connectors.)
- **ChatGPT Web/App:** Settings → Apps → Advanced → enable Developer mode → create an
  app (connector) → paste the URL. (Web, Plus and up.)
- **xAI/Grok:** grok.com/connectors → New Connector → Custom → paste the URL.

**Persona pairing (ChatGPT / Claude Web).** A persona returned inside a tool result
usually does NOT change a web client's behavior on its own. To make it stick, paste
one line into the client's own custom instructions (ChatGPT: Settings →
Personalization → Custom Instructions; Claude Web: the Project's custom instructions):

> "At the start of each chat, call AiAkiv's mweft_active_target, read the `persona`
> field, and fully adopt it (tone / role / language) for the rest of the
> conversation. It never overrides the save rules — saving still needs the explicit
> command."

## Cautions

1. The `/mcp` suffix is mandatory (or zero tools appear).
2. **ChatGPT + Codex together:** do NOT connect AiAkiv on the ChatGPT side — the
   ChatGPT connection wins and Codex's per-folder config is ignored.
3. **OAuth clients (Claude Code, Cursor):** changing the project (the header) requires
   re-authenticating. Keep a separate config per folder to avoid repeated logins.
4. **Gemini CLI:** its OAuth session does not persist — you must use the API-key
   method.
5. **ChatGPT app:** it does not complete OAuth. Given a `401`, it asks for a bearer
   token instead of signing in, so registry/URL registration stops there. Use a
   project-bound API key from the console for that client.

## Verify

Once connected, the tools appear with the `mweft_` prefix (`mweft_remember`,
`mweft_search`, `mweft_active_target`). Call `mweft_active_target` to confirm which
team / project / domain your saves go to. No tools showing? The URL is almost
certainly missing the `/mcp` suffix.

See [concepts.md](./concepts.md) for the Team → Project → Domain → persona model.
