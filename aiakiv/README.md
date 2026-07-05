# AiAkiv Memory — Claude Code plugin

Long-term memory for AI agents. AiAkiv (MemoryWeft / MWeft) stores your important
conversation content as a **knowledge graph** and recalls it across sessions,
folders, and clients. This plugin bundles the hosted AiAkiv MCP server plus a
short onboarding skill.

## What you get

- **`AiAkiv` MCP server** (remote, hosted at `https://mcp.aiakiv.com/mcp`) —
  tools to save (`mweft_remember`), search (`mweft_search`), explore the graph
  (`mweft_neighbors`, `mweft_relations`, `mweft_community_explore`), and confirm
  where memory is going (`mweft_active_target`).
- **`aiakiv-onboarding` skill** — walks you through connect → authenticate →
  pick project → save/recall.

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

## Quick start

- **Save**: say `ak save this` (or `ak 저장`). Saves are explicit — a bare
  "remember" will not trigger a save.
- **Recall**: ask a question; Claude calls `mweft_search`. Or ask
  "what do I have on X".
- **Check target**: "which project is my memory going to?" → `mweft_active_target`.

See the `aiakiv-onboarding` skill for binding a folder to a specific project.

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
