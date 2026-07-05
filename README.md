# AiAkiv plugins

Official plugin marketplace for **AiAkiv** — long-term memory for AI agents
(MemoryWeft). AiAkiv stores your important conversation content as a knowledge
graph and recalls it across sessions, folders, and AI clients.

## Install

### Claude Code

```
/plugin marketplace add aiakiv/aiakiv-plugins
/plugin install aiakiv@aiakiv
```

On first use, Claude Code opens an OAuth login in your browser. Sign in / sign up
and approve — the `mweft_*` memory tools then appear.

### Cursor

One-click: use the **Add to Cursor** button in the [AiAkiv console](https://aiakiv.com),
or bundle the plugin from the [`aiakiv/`](./aiakiv) directory.

## What's inside

- [`aiakiv/`](./aiakiv) — the AiAkiv memory plugin: remote MCP server
  (`https://mcp.aiakiv.com/mcp`) + an onboarding skill. Carries manifests for
  Claude Code (`.claude-plugin/`) and Cursor (`.cursor-plugin/`).

## Trust & data

AiAkiv is a hosted service operated by **OnMinimum**. Content you save leaves your
machine and is stored on AiAkiv servers. Terms: <https://aiakiv.com/terms> ·
Privacy: <https://aiakiv.com/privacy>.

The plugin files in this repository are released under the **MIT License**
(see [`aiakiv/LICENSE`](./aiakiv/LICENSE)); the hosted service is governed
separately by the terms above.
