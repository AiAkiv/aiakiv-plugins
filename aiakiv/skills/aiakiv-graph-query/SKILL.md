---
name: aiakiv-graph-query
description: >-
  Write graph queries (Cypher subset) against AiAkiv/MWeft memory with
  query_memory_graph (and query_partner_memory_graph across a link). Use when the
  user asks HOW memories are connected — shared entities, bridges, threads,
  timelines, multi-hop paths, "similar but structurally related", entity
  co-occurrence — or when a search hint block contains a `graph_query`
  example. Not for plain recall (use search_memory).
---

# AiAkiv graph query

`query_memory_graph` runs a **read-only Cypher subset** over the user's memory
graph. It answers *structure* questions that flat search cannot: which events
share entities, what bridges two topics, what happened next in a thread, what
is semantically far but structurally connected.

**When to reach for it** — the question is about connections, paths, shared
participants, sequences, or "one step beyond these search results". A search
response may hand you a ready-made query in `hint.graph_query.example` with
`params` — run it as-is, then adapt.

**When NOT to** — plain recall ("what did we decide about X") is
`search_memory`. If `query_memory_graph` is not in the tool list, the server has
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
  then `find_memories_by_entity` the interesting ones.
- **Thread / what happened next**:
  `START a = events(ids: $ids) MATCH (a)-[n:NEXT*1..3]->(b) RETURN b.id, b.summary, n.hops ORDER BY n.hops LIMIT 30`
- **Entity co-participation ranking**:
  `START a = events(text: $q, k: 5) MATCH (a)-[:PARTICIPATED_IN]-(e)-[:PARTICIPATED_IN]-(b) WHERE b.id <> a.id RETURN e.name, count(DISTINCT b) AS n ORDER BY n DESC LIMIT 15`

Rows are projected small (Event → `{id, summary, timestamp, order_index}`);
open full content with `get_memory_content`.

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

`query_partner_memory_graph(link_id, query, params)` — same language over your org
plus a linked partner org. `link_id` comes from `list_partner_links`. `START`
resolves in YOUR org only; the walk crosses sides through entities the link
has **aliased** (they count as one entity for `PARTICIPATED_IN`/`SHARES`),
and `SIMILAR`/`FAR` cross by vector. `NEXT`/`MEMBER_OF`/`CONNECTED` stay
within a side. Returned events carry `side` (`remote` = partner); open remote
content with `get_partner_memory_content`. Remote budgets are tighter —
prefer small `k` and `LIMIT` first. Not every server exposes this tool; if
absent, only home queries are available.

## Probe: relation modes and expansion rounds

`find_memory_connections` walks the same entity-event graph without a query language. It
takes a `relation` instead of a `MATCH` clause, and its candidates are
unverified pointers — the same standing as anything the recipes above return.

Entry is not one thing. A sentence-only call rides the vector ranking, so it
recovers when you have no ids yet; an `anchor_entity_id` call walks from the
entity directly and does not depend on the ranking at all. A useful loop is to
read a bridge entity out of one response and re-call with it as the anchor.
That chaining is yours to steer — it is not something the API does for you.

The relations:

- `similar` (default) — time-stratified neighbours of what the sentence found.
- `before` / `after` — one side of a point in time. Exactly one of
  `reference_time` (ISO-8601) or `anchor_event_id` is required; there is no
  implicit "now".
- `evolution` — stratifies the candidate pool's own timeline into first
  sighting, transition points (the largest content shift between adjacent
  sightings), and latest sighting. It answers "how did this develop". It works
  on the pool your sentence built and returns a handful of stratified picks,
  whereas `list_memory_timeline` returns one entity's full chronology.
- `contrast` — narrows by cosine and hands back candidates. The engine does
  not judge opposition: semantic tension lives as small displacements inside
  high cosine, not as opposite vectors, so geometry cannot see it. Read the
  candidates and judge contradiction yourself;
  `grounding_basis.contrast_protocol` restates this per call.
- `explore` — question-free wandering. Requires `anchor_entity_id`, forbids
  `sentence`. There is no cosine axis: candidates are ordered by bridge lift
  (specificity) and recency, with the earliest sighting stratified in, and
  `grounding` comes back as `"exploration"` because the existence gate does
  not apply.

`expansion_rounds=2` re-expands from the round-1 picks through bridges that
round 1 did not use, reaching things two structural steps away. `top_n` caps
candidates *per round*, so a 2-round call can return up to twice `top_n` in
total, and each candidate carries its `round` so you can tell how far out it
sits. Round 2 costs a second walk — reach for it on relational and
`evolution`-shaped questions rather than on direct lookups.

Across a link, `find_partner_memory_connections` takes the same inputs plus `link_id`. Entry
happens in your org only; the partner side is reached through bridges alone,
which is why it never reports `absent` — check
`grounding_basis.remote_entry_not_searched` instead.
