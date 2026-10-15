# AiAkiv skills as ZIP files

One ZIP per skill, ready to upload to the Skills screen of Claude or ChatGPT.
Each ZIP holds one folder named after the skill with its `SKILL.md` inside,
which is the shape the "Upload a skill" screen expects.

Download the one you need and upload it:

- Claude: Customize > Skills > + > Create skill > Upload a skill. Skills
  uploaded to your account also show up in Claude Code when it is signed in
  with the same account.
- ChatGPT: Skills > Create > Upload from your computer.

| File | What the skill does |
|---|---|
| `aiakiv-onboarding.zip` | Connect, sign in, pick a project, first save and recall |
| `aiakiv-save-and-recall.zip` | Write a memory that stays findable, and recall it |
| `aiakiv-graph-query.zip` | Ask questions across the memory graph |
| `aiakiv-cards.zip` | Turn a memory topic into a public share card |
| `aiakiv-console.zip` | Manage teams, invitations, links and settings from the chat |
| `aiakiv-links.zip` | Read a linked partner's memory |
| `aiakiv-contents.zip` | Write a work in AiAkiv Contents |

A skill alone does nothing: the AiAkiv MCP connection must be there as well.
See `../aiakiv/docs/connect.md`.

These files are built from `../aiakiv/skills/` and change only when a skill
changes. They are the same text as the skills the plugin installs.
