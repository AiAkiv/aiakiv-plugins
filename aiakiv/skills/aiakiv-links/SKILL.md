---
name: aiakiv-links
description: >-
  Read a LINKED partner org's memory through the mweft_link_* tools. Use when
  the user asks what a partner/linked org/team knows, wants to compare their
  memory with a partner's, mentions a link or partner org, or when a probe
  candidate carries side=remote. Covers choosing between link_search /
  link_probe / link_graph_query, reading results, and what each error reason
  means.
---

# AiAkiv links (partner-org memory)

A **link** is a read capability two orgs agreed to — not a project you switch
into. You pass `link_id` per call; your own scope is untouched. Everything is
read-only: you can never write into a partner org.

**Always start with `mweft_list_links`.** It is local (works even when the
partner is down) and tells you which links are `usable` before you spend a
remote call. If the tools are absent from the tool list, the server has links
disabled — say so.

## Which tool for which question

| Question shape | Tool |
|---|---|
| "What does the partner know about X?" — search THEIR memory directly | `mweft_link_search(sentence, link_id)` — entry happens on their side; results are their events only |
| "From what WE know, what connects to their side?" — walk from your context across shared ground | `mweft_link_probe` — entry in YOUR org, expansion crosses through **aliased entities** (entities the link declared "same thing on both sides") |
| Explicit relations across both orgs (shared entities, vector distance, ranked bridges) | `mweft_link_graph_query(link_id, query, params)` — same language as `mweft_graph_query`; see the `aiakiv-graph-query` skill |
| Read one partner event's body | `mweft_link_get_event_content(link_id, event_id)` — paged; `event_id` comes from the other tools' results |

## Reading results

- Partner events carry `side: "remote"` (or a `link_id`). **Keep them
  labelled as the partner's** when you reason or summarize — do not present
  partner knowledge as the user's own, and do not re-save partner content
  into the user's memory unless the user explicitly asks.
- `mweft_link_probe` never claims something is *absent* on the partner side —
  entry is home-only, so the partner is only reached through bridges. Absence
  of remote candidates means "no bridge found", not "they don't know".
- Aliased bridge entities carry `df_home` / `df_remote` (how common the
  entity is on each side). Asymmetry is a signal: "1 here, 109 there" means a
  passing mention for the user is a central topic for the partner — often the
  most interesting finding in the response.
- `found: false` from `link_get_event_content` means the link does not
  surface that event — deliberately the same answer whether it is absent or
  out of scope. Don't retry; don't speculate which.

## When a call returns `status: "error"`

The `reason` names what to do — an error here **never** means the user's own
memory failed:

- `no_links`, `link_not_found`, `link_not_active`, `link_expired`,
  `org_not_party`, `principal_not_active` — this link cannot be read (or this
  direction is closed). Show `mweft_list_links` output; fixing it is an
  owner/console action, not a retry.
- `we_are_provider` — the user's org is the *providing* side; there is
  nothing to read in this direction. Normal state, not a failure.
- `contract_version_too_old` — both owners must re-consent;
  `mweft_list_links` → `reconsent` shows who is missing.
- `budget_exceeded` — the remote row/time budget ran out mid-walk. Narrow the
  ask (smaller `top_n`/`k`/`LIMIT`, tighter relation params) instead of
  repeating it.
- `busy`, `remote_failure`, `internal` — transient infrastructure; try later.
  Repeated `busy` means the partner side is protecting itself (bulkhead) —
  back off rather than hammering.

Results can also be `partial` (budget hit mid-walk): what came back is valid,
just incomplete — say so instead of treating it as the full picture.
