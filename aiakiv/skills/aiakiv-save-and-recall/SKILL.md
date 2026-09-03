---
name: aiakiv-save-and-recall
description: How to write a memory that will be found again, and how to read what AiAkiv memory returns. Covers summary and entity spelling rules, what to do when a save is rejected (summary or content too long, a truncation marker, arguments arriving merged or empty), and how to interpret the related-memory signal that search responses carry. Use when saving to AiAkiv memory, when a save call fails or its arguments collapse, or when deciding whether a search result needs a follow-up expansion call.
---

# Saving to and recalling from AiAkiv memory

The tool descriptions state the contract. This skill covers the parts that
decide whether a saved memory is findable a month later, and what to do when
a call is rejected.

## 1. The summary is the retrieval surface

Searches rank against the summary, not the content. Write it with the words a
future question would use.

- **Keep exact identifiers verbatim.** Code symbols, document numbers, config
  keys. Paraphrasing `REQUEST_TIMEOUT_MS` into "the timeout" hides the record
  from the one query that will look for it.
- **Name the actor in the first sentence** when you record someone's judgment,
  design, or claim. Conclusion-style summaries ("decided that…") drop who said
  it, and "what did X propose?" then misses the record.
- Write a complete sentence, not a keyword pile.

## 2. Entity spelling decides what connects

Recall weaves records together on shared entity nodes, so spelling drift
silently loses connections.

- **One canonical spelling per entity** — the full name *or* the acronym, not
  both. Pick one and use it in every save.
- **Keep proper nouns and acronyms as they are written**: `PostgreSQL`, `RLS`.
  Do not lowercase or expand them.
- **Strip transient tokens** — ids, hashes, dates. They never repeat, so they
  add a node that connects to nothing.
- A suggested entity returned by the server is advisory. A literal name match
  does not establish that it denotes the same thing in this record.

## 3. When a save is rejected

### The summary is over 500 characters

**Split the event; do not compress the summary.** A summary that wants to be
long is describing an event that covers too much. Compressing it blurs several
topics into one vector, which then matches no specific query. Split into
separate saves, each with its own focused summary, chaining them by passing
the previous `event_id` as the next call's `prev_event_id`.

Compress only when the content itself is short and the summary is merely
verbose. Do not fragment one small record.

### The content is over 50000 characters

Split the full text across calls the same way, chaining with `prev_event_id`,
giving each piece its own summary. Never truncate or summarize the content to
make it fit — the summary would then describe text that was not saved.

### `truncation_marker_detected`

The content carries a marker that some earlier tool left when it cut the text
(`…2183 tokens truncated…`, `<response clipped>`, `[truncated]`). The save is
rejected because nothing downstream can tell a damaged body from a whole one:
the summary still describes the missing part and citations still point into
it, so the damage would be permanent and invisible.

Re-read the source in full and save again, splitting if it is long. Pass
`allow_truncation_marker=true` only when the marker is genuinely part of what
you mean to record, such as a bug report *about* truncation.

### The arguments arrive merged, empty, or the call errors on serialization

Two causes, both about argument boundaries:

- A long or multi-line `summary` or `content`. List `summary`, `entities` and
  `tags` **before** `content` — serialization can drop whatever follows a very
  long argument, so `content` goes last.
- Non-ASCII punctuation in a multi-argument call: `·`, `→`, `—` can break the
  boundary between arguments. Use ASCII punctuation in multi-argument saves.

**Recovery: retry with `payload`.** One argument has no boundary to collapse.
Send the whole call as a JSON string or an object:

```json
{"summary": "...", "entities": [{"name": "X", "type": "Y"}], "tags": ["a/b"], "content": "..."}
```

Other arguments are then ignored. The multi-argument form stays the default;
this is the recovery path.

## 4. Reading the related-memory signal

A search response may carry `hint.related_memory_expansion`. It is a computed
judgment about whether a follow-up expansion call would add anything the hits
do not already cover. Its fields:

| Field | Meaning |
|---|---|
| `tool` | Which tool performs the expansion |
| `decision` | `must_be_done`, `should_be_done`, `can_skip`, `must_not_be_done` |
| `candidate_count` | How many candidates sit outside the returned hits |
| `candidate_dimension` | The relation the candidates share with the hits |
| `candidate_ids` | The candidate memory ids |
| `reason_codes` | Which gates the candidates passed |
| `reason_facts` | The measured values behind those codes |

`should_be_done` means candidates outside the hits passed the relevance and
novelty gates, so expanding is likely to add something before you conclude.
`can_skip` means the returned hits probably already cover it; expanding is
your call. The block is omitted entirely when there is nothing to say.

Results that feel sufficient are not evidence that they are. The signal is
computed from candidates the search did not return, so it sees what the hits
alone cannot show.
