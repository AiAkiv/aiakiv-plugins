---
name: aiakiv-graph-query
description: >-
  Write graph queries (Cypher subset) against AiAkiv/MWeft memory with
  mweft_graph_query (and mweft_link_graph_query across a link). Use when the
  user asks HOW memories are connected — shared entities, bridges, threads,
  timelines, multi-hop paths, "similar but structurally related", entity
  co-occurrence — or when a search hint block contains a `graph_query`
  example. Not for plain recall (use mweft_search).
---

# AiAkiv graph query

`mweft_graph_query` runs a **read-only Cypher subset** over the user's memory
graph. It answers *structure* questions that flat search cannot: which events
share entities, what bridges two topics, what happened next in a thread, what
is semantically far but structurally connected.

**When to reach for it** — the question is about connections, paths, shared
participants, sequences, or "one step beyond these search results". A search
response may hand you a ready-made query in `hint.graph_query.example` with
`params` — run it as-is, then adapt.

**When NOT to** — plain recall ("what did we decide about X") is
`mweft_search`. If `mweft_graph_query` is not in the tool list, the server has
it disabled; say so instead of inventing it.

## Grammar in one screen

```
START a = events(text: $q, k: 5)        ← anchor set is ALWAYS events
START a = events(entity: "name or id")
START a = events(ids: [$id1, $id2])
MATCH (a)-[s:SHARES {min: 2}]-(b)       ← one or more MATCH clauses
WHERE b.id <> a.id                       ← optional
RETURN b, s.count, s.via ORDER BY s.weight DESC LIMIT 20
```

- No `WITH`, no `CREATE`/`SET` (read-only). Conditions that would need `WITH`
  go into relation params like `{min: 3}`.
- `$name` params are passed via the `params` argument — always parameterize
  user text; never inline it.
- Aggregates (`count(DISTINCT …)`) live in `RETURN`. `cos(a, b)` gives vector
  cosine between two event variables.
- Variable-length: `-[n:NEXT*1..3]->` (hop count comes back as `n.hops`).

## Relations (the whole vocabulary)

| Relation | Between | Meaning / params |
|---|---|---|
| `SHARES {min, min_w}` | event–event | share ≥min entities; carries `s.count`, `s.weight` (rarity-weighted), `s.via` (the shared entities = bridges) |
| `SIMILAR {k, min}` | event–event | vector nearest neighbours; `f.cos` |
| `FAR {max}` | event–event | vector distance filter (cos < max) — pair with SHARES for "connected but semantically far" |
| `NEXT` | event→event | conversation/thread order (directional, supports `*1..n`) |
| `PARTICIPATED_IN` | event–entity | membership; walk event→entity→event for co-participation |
| `MEMBER_OF` | event–tag | category/tag membership |
| `CONNECTED` | entity–entity | stored entity co-occurrence edge |
| `SAME_AS` | entity–entity | stored alias edge (exact identity, not fuzzy match) |

## Recipes

- **Structurally close but semantically far** (the highest-value walk from
  search results — surfaces non-obvious connections):
  `START a = events(ids: $ids) MATCH (a)-[s:SHARES {min: 2}]-(b)-[f:FAR {max: 0.5}]-(a) RETURN b, s.count, s.via, f.cos ORDER BY s.weight DESC LIMIT 10`
- **What bridges these results** — same query; read `s.via` (shared entities),
  then `mweft_entity_lookup` the interesting ones.
- **Thread / what happened next**:
  `START a = events(ids: $ids) MATCH (a)-[n:NEXT*1..3]->(b) RETURN b.id, b.summary, n.hops ORDER BY n.hops LIMIT 30`
- **Entity co-participation ranking**:
  `START a = events(text: $q, k: 5) MATCH (a)-[:PARTICIPATED_IN]-(e)-[:PARTICIPATED_IN]-(b) WHERE b.id <> a.id RETURN e.name, count(DISTINCT b) AS n ORDER BY n DESC LIMIT 15`

Rows are projected small (Event → `{id, summary, timestamp, order_index}`);
open full content with `mweft_get_event_content`.

## Reading the response, handling refusals

- `partial` / `truncated` report budget caps — **never silent**. If truncated,
  narrow instead of retrying the same query: fewer anchors (`k`), tighter
  `SHARES {min}` / `FAR {max}`, smaller `LIMIT`.
- A rejection returns `{error, blocked_by: syntax|grammar|params, hint?,
  allowed_*}` — read `hint` and the `allowed_*` lists, fix the query once;
  do not loop blind retries.
- Budget errors ("statement timeout") mean the walk was too wide, not that
  the tool is broken.

## Across a link (partner org)

`mweft_link_graph_query(link_id, query, params)` — same language over your org
plus a linked partner org. `link_id` comes from `mweft_list_links`. `START`
resolves in YOUR org only; the walk crosses sides through entities the link
has **aliased** (they count as one entity for `PARTICIPATED_IN`/`SHARES`),
and `SIMILAR`/`FAR` cross by vector. `NEXT`/`MEMBER_OF`/`CONNECTED` stay
within a side. Returned events carry `side` (`remote` = partner); open remote
content with `mweft_link_get_event_content`. Remote budgets are tighter —
prefer small `k` and `LIMIT` first. Not every server exposes this tool; if
absent, only home queries are available.
