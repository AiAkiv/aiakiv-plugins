---
name: aiakiv-graph-query
description: >-
  Write graph queries (Cypher subset) against AiAkiv/MWeft memory with
  query_memory_graph (and query_partner_memory_graph across a link). Use when the
  user asks with the request phrase "ak graph" / "ak 그래프", or otherwise
  explicitly asks for a graph query over how memories are connected — shared
  entities, bridges, threads, multi-hop paths, co-occurrence. Lower priority: a
  search hint block carrying a `graph_query` example. A question without the
  phrase is plain recall (use search_memory).
---

# AiAkiv graph query

`query_memory_graph` runs a **read-only Cypher subset** over the user's memory
graph. It answers *structure* questions that flat search cannot: which events
share entities, what bridges two topics, what happened next in a thread, what
is semantically far but structurally connected.

**When to reach for it** — first, the request phrase `ak graph` (`ak 그래프`),
usually followed by a question in the user's own words: the user chose the graph
query, and the work is translating it faithfully (next section). Also an
explicit request in other words for a structural walk. Lower priority:
`hint.graph_query.example` in a search response can be run as-is, then adapted.

**When not to** — a question without the phrase, even about connections, starts
with `search_memory`; the graph query is not picked on its own initiative. If
the tool is not in the list, the server has it disabled; say so.

## When the user asks with `ak graph`

1. **Translate.** "Among memories about <name>, the ones about <topic>" is
   `events(entity: $name, text: $q, k: 20)`; "in the <tag> tag, …" uses `tag:`;
   "A and B both appear" and "names containing …" are recipes below. Names
   match exactly (`find_memories_by_entity` shows the stored spelling).
   A request for "all", "every" or "the full list" is not a closeness ranking:
   it takes the listing path — `tag:` or `entity:` alone with `k: 200`, an
   explicit `LIMIT`, and a `CONTAINS` or name filter when the request names
   one. If `start.hub` is still `true`, the answer says there are more than 200
   and not all of them were seen.
2. **Run it**, then **show the query that ran and what it means in one line**,
   next to the answer, in the user's language (for the first example below:
   the query, then "the 20 BP-7 memories closest in meaning to q").
3. **Put the completeness markers into the user's words** ("Reading the
   response"): "could not confirm" and "none" are different answers, and
   "the closest 20 of 224" is not "all 224".
4. **On a rejection**, read `hint` and `allowed_*` and fix the query once. On
   "scoped match is off", fall back to `text:` or `entity:` alone and tell the
   user the new syntax is off on this server.
5. **The phrase alone** gets three example requests back, like these, adapted to
   names from the user's memory:

```
ak 그래프 BP-7 에 연결된 기억 중에서 요금제 가격을 정한 논의
  START a = events(entity: "BP-7", text: $q, k: 20) RETURN a, a.score
  params {"q": "요금제 가격을 정한 논의"}

ak 그래프 ops 태그 안의 기억에서 백업 복구를 연습한 기록
  START a = events(tag: "ops", text: $q, k: 20) RETURN a, a.score
  params {"q": "백업 복구를 연습한 기록"}

ak 그래프 Redis 와 같은 기억에 나오는 엔티티 중 이름에 cache 가 들어간 것 전부
  START a = events(entity: "Redis", k: 200) MATCH (a)-[:PARTICIPATED_IN]-(n)
  WHERE n.name CONTAINS "cache" RETURN DISTINCT n.name LIMIT 200
```

The third uses `k: 200` because the default `k` 5 walks only the five newest
Redis memories; `start.hub: true` there means Redis has more than 200.

## Grammar in one screen

```
START a = events(text: $q, k: 5)                  ← always events
START a = events(entity: $name)                   newest of the entity's events
START a = events(ids: [$id1, $id2])
START a = events(tag: $t, k: 20)                  newest in the tag
START a = events(entity: $name, text: $q, k: 20)  entity's events closest to $q
START a = events(tag: $t, text: $q, k: 20)        tag's events closest to $q
MATCH (a)-[s:SHARES {min: 2}]-(b)                 ← zero or more
WHERE b.id <> a.id                                ← optional
RETURN b, s.count, s.via ORDER BY s.weight DESC LIMIT 20
```

- **Three row caps that are not budget cuts** — none of them sets `partial`
  or `truncated`:
  - `k` defaults to 5, up to 20 for `text:` and the combined starts and 200 for
    `entity:` / `tag:` alone, clamped with a note. The combined starts and
    `tag:` alone set `start.has_more: true` when the scope holds more
    candidates than came back, with a `notes` line "start: top-k of N
    candidates; … more exist in scope (not a budget cut)". A `+` after a
    number means it is a lower bound. When the scope was not fully covered
    (`filled: false`, a names cap, or the start cut by a statement timeout or
    the deadline) the "could not confirm" line replaces this one; `has_more`
    itself is still set.
  - `start.hub: true` — `entity:` or `tag:` alone found more events than the
    `k` newest it returned. For `entity:` alone this is the only signal.
  - Without `LIMIT`, rows stop at 20 (`LIMIT` goes up to 200). With the switch
    on, `notes` says "rows cut to the default LIMIT 20; add LIMIT to return
    more"; with it off, the cut is silent.
- A name resolves to at most 20 entities or tags;
  `start.entities_truncated` / `tags_truncated` says more matched.
- Behind a server switch: `tag:`, the combined starts, `a.score`, `CONTAINS`,
  no `MATCH`, arrow filling. Off, they are rejected with "… (scoped match is
  off)" and a hint.
- **No `MATCH`** when the start is the answer:
  `START a = events(entity: $name, text: $q, k: 20) RETURN a, a.score ORDER BY a.score DESC, a.id`.
- **`a.score`** — the start event's cosine to the start text, not clipped; null
  without start text and on hop-reached events, nulls last. `RETURN a` omits it.
- **`WHERE n.name CONTAINS $part`** — `Entity.name` / `Tag.name` only, ignoring
  case and full-width forms; an all-ASCII piece needs 2 characters, one Hangul
  or Han character is enough. It filters rows the walk already returned.
- **Directed** `NEXT`, `MEMBER_OF`, `RESOLVED_BY` take an arrow. A missing one
  is filled in (with a `notes` line) only when the end labels allow one way, as
  `(a)-[:MEMBER_OF]-(t)` from an event; event-to-event `NEXT` and `RESOLVED_BY`
  still need it. A wrong arrow is rejected, not flipped.
- No `WITH`, no `CREATE`/`SET` (read-only); conditions go in relation params
  like `{min: 3}`. The user's sentence goes through `params` (`$q`), not inline.
- Aggregates live in `RETURN`; `cos(a, b)` is the cosine of two event variables.
  Variable length on `NEXT` and `SHARES` only, 1..3: `-[n:NEXT*1..3]->`
  (`n.hops`).

## Relations (the whole vocabulary)

| Relation | Between | Meaning / params |
|---|---|---|
| `SHARES {min, min_w}` | event–event | share ≥min entities; carries `s.count`, `s.weight` (rarity-weighted), `s.via` (the shared entities = bridges) |
| `SIMILAR {k, min}` | event–event | vector nearest neighbours; `f.cos` |
| `FAR {max}` | event–event | vector distance filter (cos < max) — pair with SHARES for "connected but semantically far" |
| `NEXT {source}` | event→event | conversation/thread order (directional, supports `*1..3`); `n.source` |
| `RESOLVED_BY {relation}` | event→event | plan or expectation → the event that realized it (directional); `relation`: `realization`; `r.asserted_at`; revoked links excluded |
| `PARTICIPATED_IN` | entity–event | membership; walk event→entity→event for co-participation |
| `MEMBER_OF {kind}` | event→tag | tag membership (directional); `kind` is `contains` or `refers` |
| `CONNECTED` | entity–entity | stored entity co-occurrence edge; `c.event_count` |
| `SAME_AS` | entity–entity | stored alias edge (exact identity, not fuzzy match); `x.confidence` |

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
- **A and B both appear** (no switch needed; default `k` 5 sees only A's five
  newest events):
  `START a = events(entity: $a, k: 200) MATCH (a)-[:PARTICIPATED_IN]-(n {name: $b}) RETURN DISTINCT a`

Rows are projected small (Event → `{id, summary, timestamp, order_index}`);
open full content with `get_memory_content`.

## Reading the response, handling refusals

- `start.scoped_mode`: `exact` means every candidate was measured and only the
  top `k` returned; `filtered_ann` means a large scope was searched through the
  vector index. `start.candidates`: `{count, exact}` (with `exact: false` the
  count is a lower bound); `start.filled: false`: fewer than `k` found within
  budget, not "the scope ran out". `filled: false`, a names cap, or a start
  cut by a statement timeout or the deadline adds a start entry to `truncated`
  and sets `partial`.
- **0 rows plus any of those means "could not confirm"; 0 rows with nothing cut
  means "none".** `notes` says so too, along with clamps and inferred arrows.
- **More candidates than rows.** `start.has_more: true` — or, where the server
  does not send it, `candidates.count` above the rows the start returned —
  means the scope holds more than came back. `exact` with count 224 and `k` 20
  is the closest 20 of 224; saying "compared all 224, nothing missing" is
  wrong. `partial: false` means no budget cut, not "everything was seen".
- Budget caps (`partial` / `truncated`) are **never silent**; `k`, `hub` and
  the default `LIMIT` are separate signals (Grammar above). If truncated,
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
prefer small `k` and `LIMIT` first. The new syntax works here too; `CONTAINS`
sees the home spelling of aliased names. Not every server exposes this tool; if
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
