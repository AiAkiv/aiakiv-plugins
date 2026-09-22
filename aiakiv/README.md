# AiAkiv Memory — Claude Code plugin

Long-term memory for AI agents. AiAkiv (MemoryWeft / MWeft) stores your important
conversation content as a **knowledge graph** and recalls it across sessions,
folders, and clients. This plugin bundles the hosted AiAkiv MCP server plus skills for
onboarding, share cards, graph queries, and partner links.

## What you get

- **`AiAkiv` MCP server** (remote, hosted at `https://mcp.aiakiv.com/mcp`) —
  tools to save (`save_memory`), search (`search_memory`), explore the graph
  (`find_related_memories`, `find_memory_connections`, `list_memory_timeline`), and confirm
  where memory is going (`get_save_target`).
- **`aiakiv-onboarding` skill** — walks you through connect → authenticate →
  pick project → save/recall.
- **`aiakiv-cards` skill** — turns a memory topic into a public share card
  (`card.aiakiv.com`): the create procedure, image-fit guidance, and the
  publish/delete rules the AI must relay.
- **`aiakiv-graph-query` skill** — how to write graph queries (Cypher subset)
  with `query_memory_graph`: the grammar, the relation vocabulary, ready-made
  recipes (shared-entity bridges, threads, "similar but structurally
  connected"), and how to react to budgets and rejections.
- **`aiakiv-links` skill** — reading a linked partner org's memory (the
  `*_partner_*` tools): which tool fits which question, keeping partner content
  labelled as theirs, and what each link error reason means.
- **`aiakiv-console` skill** — managing the account from the conversation
  through `run_aiakiv_app_action(app="console")`: teams and invitations, org
  links, audit logs, save-target history, project presets, and the cards the
  user created. Irreversible operations stay in the web console.
- **`aiakiv-save-and-recall` skill** — writing a memory that stays findable
  (summary and entity spelling), what to do when a save is rejected, and how to
  read the related-memory signal that search responses carry.

## Install

From the AiAkiv marketplace:

```
/plugin marketplace add aiakiv/aiakiv-plugins
/plugin install aiakiv@aiakiv
```

Or load locally for development. Launch Claude Code from the **K2G repo root**, then:

```
/plugin marketplace add ./packaging/plugins
/plugin install aiakiv@aiakiv
```

The local marketplace registers under the name `aiakiv` (from `marketplace.json`),
so the install command is identical to the production one above — only the
`marketplace add` source differs (`./packaging/plugins` vs `aiakiv/aiakiv-plugins`).

On first use, Claude Code opens an **OAuth** login in your browser. Sign in /
sign up and approve — the `mweft_*` tools then appear.

### Other clients — ask your agent to install it

Outside Claude Code, AiAkiv is a remote MCP server at `https://mcp.aiakiv.com/mcp`.
It is published in the official MCP Registry as **`com.aiakiv/memory`**, so a client
that can search the registry will find it by name (`aiakiv`) or keyword (`memory`)
and register the endpoint for you — no URL to paste. Either way you still approve the
sign-in once. See [docs/connect.md](docs/connect.md) for per-client steps.

> **ChatGPT is the exception.** It cannot search the registry or write an MCP config,
> and it does not complete the OAuth sign-in — it asks for a bearer token instead. In
> ChatGPT, add the URL through the connector UI and, if no sign-in is offered, use a
> project-bound API key from the console.

## Quick start

- **Save**: say `ak save this` (or `ak 저장`). Saves are explicit — a bare
  "remember" will not trigger a save.
- **Recall**: ask a question; Claude calls `search_memory`. Or ask
  "what do I have on X".
- **Check target**: "which project is my memory going to?" → `get_save_target`.

See the `aiakiv-onboarding` skill for binding a folder to a specific project.

## Learn more

- [docs/concepts.md](./docs/concepts.md) — the **Team → Project → Domain** model,
  Main, and per-project **personas** (roles the AI adopts).
- [docs/connect.md](./docs/connect.md) — connecting **every** client (Claude Web,
  ChatGPT, Cursor, Codex, Gemini CLI, xAI/Grok…), folder-vs-global binding, the setup
  order, and cautions.

## Trust & data

AiAkiv is a **hosted** service. Content you save leaves your machine and is stored
on AiAkiv servers. You control your data (view, export, purge) through the AiAkiv
console at <https://aiakiv.com>. Anthropic does not control the MCP server, files,
or software included in this plugin — trust is between you and AiAkiv.

## License

The plugin bundle in this repository (manifests, configuration, the onboarding
skill, and documentation) is released under the **MIT License** — see
[LICENSE](./LICENSE).

MIT covers only these connector files. The hosted **AiAkiv** service reached
through the MCP endpoint (`https://mcp.aiakiv.com/mcp`) is operated by
**OnMinimum** and governed separately by the AiAkiv **Terms of Service**
(<https://aiakiv.com/terms>) and **Privacy Policy** (<https://aiakiv.com/privacy>).

## Links

- Website: <https://aiakiv.com>
- MCP endpoint: `https://mcp.aiakiv.com/mcp`
