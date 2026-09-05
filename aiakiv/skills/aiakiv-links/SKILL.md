---
name: aiakiv-links
description: >-
  Read a LINKED partner org's memory through the partner memory tools. Use when
  the user asks what a partner/linked org/team knows, wants to compare their
  memory with a partner's, mentions a link or partner org, or when a probe
  candidate carries side=remote. Covers choosing between search_partner_memory,
  find_partner_memory_connections and query_partner_memory_graph, reading results, and what each error reason
  means.
---

# AiAkiv links (partner-org memory)

A **link** is a read capability two orgs agreed to — not a project you switch
into. You pass `link_id` per call; your own scope is untouched. Everything is
read-only: you can never write into a partner org.

**Always start with `list_partner_links`.** It is local (works even when the
partner is down) and tells you which links are `usable` before you spend a
remote call. If the tools are absent from the tool list, the server has links
disabled — say so.

Links run **one way**, and the list holds the ones whose reading direction
points at the user's org — blocked ones included, with `usable` saying which
answer right now. Links their org **provides** (their team issued the invite, so
the link opens their memory to the partner) are left out entirely — nothing to
read, nothing to fix; `provided_link_count` says how many exist, and they are
managed in the console.

**Call link tools one at a time.** Your org may have only one link call in
flight at once. Two link tools issued in the same batch — the habit that serves
you well everywhere else — means the second returns `error: busy` without
running, even on a completely idle server. Await each call before starting the
next; `list_partner_links` is local and does not count.

## Which tool for which question

| Question shape | Tool |
|---|---|
| "What does the partner know about X?" — search THEIR memory directly | `search_partner_memory(sentence, link_id)` — entry happens on their side; results are their events only |
| "From what WE know, what connects to their side?" — walk from your context across shared ground | `find_partner_memory_connections` — entry in YOUR org, expansion crosses through **aliased entities** (entities the link declared "same thing on both sides") |
| Explicit relations across both orgs (shared entities, vector distance, ranked bridges) | `query_partner_memory_graph(link_id, query, params)` — same language as `query_memory_graph`; see the `aiakiv-graph-query` skill |
| Read one partner event's body | `get_partner_memory_content(link_id, event_id)` — paged; `event_id` comes from the other tools' results |

## Reading results

- Partner events carry `side: "remote"` (or a `link_id`). **Keep them
  labelled as the partner's** when you reason or summarize — do not present
  partner knowledge as the user's own, and do not re-save partner content
  into the user's memory unless the user explicitly asks.
- `find_partner_memory_connections` never claims something is *absent* on the partner side —
  entry is home-only, so the partner is only reached through bridges. Absence
  of remote candidates means "no bridge found", not "they don't know".
- Aliased bridge entities carry `df_home` / `df_remote` (how common the
  entity is on each side). Asymmetry is a signal: "1 here, 109 there" means a
  passing mention for the user is a central topic for the partner — often the
  most interesting finding in the response.
- `found: false` from `get_partner_memory_content` means the link does not
  surface that event — deliberately the same answer whether it is absent or
  out of scope. Don't retry; don't speculate which.

## When a call returns `status: "error"`

The `reason` names what to do — an error here **never** means the user's own
memory failed:

- `no_links`, `link_not_found`, `link_not_active`, `link_expired`,
  `org_not_party`, `principal_not_active` — this link cannot be read (or this
  direction is closed). Show `list_partner_links` output; fixing it is an
  owner/console action, not a retry.
  - One case has nothing to fix: a `link_id` that `list_partner_links` does not
    list at all (the user may have copied it from the console's "links we
    provide" group) is one their org **provides**, and its refusal is
    `principal_not_active`. Say that it opens their memory outward rather than
    sending them to fix a direction.
- `contract_version_too_old` — both owners must re-consent;
  `list_partner_links` → `reconsent` shows who is missing.
- `budget_exceeded` — the remote row/time budget ran out mid-walk. Narrow the
  ask (smaller `top_n`/`k`/`LIMIT`, tighter relation params) instead of
  repeating it.
- `busy` — a concurrency limit on **your** server (never the partner's). The
  `detail` says which of two: the per-org limit, which is almost always
  self-inflicted (two link calls issued together) — re-issue the refused one
  once the other finishes, it cost nothing and did not run; or the server-wide
  limit, shared with other tenants, where a short wait is the right response.
  Either way it is not a partner problem.
- `remote_failure`, `internal` — infrastructure failed; try later.

Results can also be `partial` (budget hit mid-walk): what came back is valid,
just incomplete — say so instead of treating it as the full picture.
