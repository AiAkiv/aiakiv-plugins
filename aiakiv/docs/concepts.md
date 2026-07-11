# AiAkiv concepts — teams, projects, domains, personas

AiAkiv (MemoryWeft / MWeft) is a long-term memory that AI agents and teammates
share over MCP. Memory is addressed as **Team → Project → Domain**. Understanding
these three (plus **Main** and **persona**) is all you need to organize it.

## Team — who shares the memory

A *team* is the set of people who can read and write a body of memory.

- Every account already has a **personal team** (a team of one) — nothing to create
  for solo use.
- **Shared teams** are for working with others and are **invite-only**: a team
  owner/admin invites an email in the console, and the invitee accepts under "My
  Invitations". You cannot self-join a team.
- Roles: **admin** / **member** / **viewer** (viewer is read-only; member and up can
  read and write).

A separate team is the **firmest boundary** between bodies of memory — a hard wall
between organizations. You can own several teams, including extra solo "teams of
one", and use them for different purposes.

## Project — a named save-target

A *project* is a named bucket inside a team that your saves land in.

- Each project has its own **domain / group** coordinate, an optional **expiry**, and
  an optional **persona** (below).
- **Project names are English only** — plain letters/numbers, no accented or
  non-Latin characters (the name travels in an HTTP header).
- Every new account already has a project named **`default`** (domain `inbox`) that is
  its **Main** — so a brand-new user can save with zero setup.
- You create additional projects in the console.

## Main — where a save lands with no project hint

**Main** is the project a save goes to *right now* when the client sends no explicit
project. **Global** web clients (which cannot bind per folder) always follow Main;
you switch Main in the console. **Folder** clients name their project directly and do
not depend on Main (see [connect.md](./connect.md)).

## Domain — the isolation boundary

A *domain* keeps unrelated memory from mixing. **Memories in different domains never
mix.** Two levers keep memory apart:

1. A separate **team** — the firmest boundary (a hard wall between orgs).
2. Within one team, give projects a different **domain** — a lighter separation for
   splitting topics inside the same team.

Same team **and** same domain = memory is **pooled** (shared), even across several
projects. That is exactly what enables persona-only projects (next).

## Persona — a role the AI adopts

A *persona* is a standing role instruction the project owner writes (e.g. "You are a
careful senior reviewer; reply in English").

- Any AI that connects to that project reads it — via `mweft_active_target` or the
  `active_target` echo on tool responses — and **adopts it** (tone / role / language)
  for the conversation.
- It is **optional**: unset means no persona, nothing changes.
- It is set by the **human in the console** only; an AI can never change it, and it
  **never overrides the save rules** (the explicit save command is still required).

**Persona-only projects** — a useful pattern: several projects in the **same team and
same domain** that differ *only* in persona. They all read and write the ONE shared
memory (same domain = pooled), but each makes the AI act in a different role
(`reviewer` / `brainstormer` / `writing-coach`). Use this for "one body of memory,
several working modes". Give projects a different *domain* only when you want the
memory itself kept apart.

## Managing all of this

Projects, teams, invites, keys, Main-switching, and personas are all human actions in
the console at <https://aiakiv.com>. An AI (including one connected to AiAkiv) cannot
perform them — it can only read the current target and adopt the persona.
