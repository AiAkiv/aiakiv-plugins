---
name: aiakiv-cards
description: >-
  Create a public AiAkiv card from the user's memory. Use when the user asks to
  make a card, share a memory as a link/page, publish a summary card, or asks
  what AiAkiv cards are or how to fix/delete one. A card is a one-page public
  site (card.aiakiv.com) written by YOU from memory search results and baked by
  the card service — live on the open web the moment it is created.
---

# AiAkiv cards

A **card** is a one-page public summary of the user's AiAkiv memory: title,
3–5 bullets, a one-line conclusion, optional long body — plus two share images
with the short URL stamped in. Anyone with the address can view it, no login.
You create it through the shared app gateway; the server never calls a model —
**you write the content**.

## The one fact that governs everything

**Creating IS publishing.** The card is live on the open web the instant the
create call returns. Unlisted (nobody finds it without the address) but NOT
private. There is no draft state and no undo except deletion in the console.
So: filter before you create, and relay the response's `notice` to the user
first, verbatim.

## Procedure

1. **Search memory first** (`search_memory`). The card summarizes what is
   actually stored — do not invent content. Set `source_count` to the number
   of memories you actually used.
2. **Filter for sensitivity** — personal names, emails, internal addresses,
   decisions not yet public. If the topic is at all sensitive, show the user a
   draft and get a go-ahead before creating.
3. **Get the live format**: `run_aiakiv_app_action(app="card", action="describe")`.
   Only `describe` and `create` exist — this tool cannot delete, edit, or
   change search visibility.
4. **Write to fit the image, not just the limits.** `describe` returns
   `bullets.renders` with the measured safe line width (~30 Hangul chars) and
   max lines per bullet. Bullets share one fixed block of vertical space:
   5 bullets ≈ one safe line each; 3 bullets can run 2–3 lines each. Staying
   under `max_chars_each` (120) is NOT enough. Aim to fit on the first try —
   re-creating issues a NEW key and leaves the old card published and counted
   against quota until the user deletes it in the console.
5. **Create**: `run_aiakiv_app_action(app="card", action="create", data={...})`.
   Fields: `title` (≤80), `bullets` (3–5), `conclusion` (≤200),
   `source_count`, `tags` (≤3, ≤20 chars each — these are the search terms on
   card.aiakiv.com), `body` (optional, ≤30k chars, restricted markdown:
   paragraphs, `##`, lists, `>`, code, bold/italic, http(s) links, `---`;
   anything else renders as plain text). Title/bullets/conclusion/tags are
   single-line — newlines collapse. If a long body gets mangled in transport,
   pass `data` as one JSON **string**.

## Handling the response

- **`notice` comes first.** Relay it to the user before the link, before any
  summary, uncompressed. It states the card is already public and how to take
  it down.
- **`url`** — show verbatim as a clickable link. `og_url` / `square_url` are
  the share images (Instagram / boards that don't unfurl links).
- **`fit`** (present only when something was cut) — report it as-is:
  `bullets_truncated` / `bullets_dropped` are original bullet numbers, `hint`
  says how much to shorten. The PAGE always shows full text; only the images
  crop. Let the user decide whether to shorten and re-create — and if they do,
  remind them the old card stays up until deleted.
- **Errors** (`{error, hint, field}`): `invalid_card` → fix the named field
  and retry; `payload_too_large` → shrink the body; `quota_exceeded` (the
  response carries `window`, `limit`, `used` — rate caps are 10 per hour /
  30 per day; the holding cap is far higher) → tell the user; anything else →
  relay and stop.
  On any error the card was NOT created — never say it was.

## What you must not do

- Create a card the user didn't ask for ("summarize this" is not a card request).
- Fill a card with content that isn't in memory.
- Claim you deleted a card or toggled search visibility — you can't. Manage-
  ment (list, copy link, images, delete, search-visibility toggle) lives in
  the AiAkiv console → Data → Cards (app.aiakiv.com).
- Offer to set the user's public nickname or attach filing tags — you can't.
  Those are console-only too (see below). `data.tags` is a different thing:
  card tags are baked into the image at create time and cannot be changed.
- Quietly re-create after a `fit` warning — that leaves two public cards.
- Paste the card's URL anywhere on the user's behalf; sharing is their call.

## Useful context for the user

- KakaoTalk / Slack / Discord / Notion / blogs unfurl the bare URL into a card.
  Instagram and some board sites don't — upload the square/wide image instead;
  the URL is stamped inside it.
- Search visibility is OFF by default. Turning it on (console) lists the card
  on card.aiakiv.com search by title/tags; turning it off later un-lists it
  but the card stays public to anyone with the address.
- card.aiakiv.com search narrows three ways — text, author nickname, filing
  tag — and the address bar tracks whatever is narrowed, so any view can be
  shared as a link. **Nickname and filing tags are set in the console, by the
  user, not by you.** A nickname is optional; without one nothing identifying
  the user is published. Their email is never published in any case. Filing
  tags show on the card page and group cards in search; they never change the
  card image.
- Deleting removes the page immediately, but third-party preview caches can
  linger. Deleting the account does NOT delete cards — clean up in the console
  first.
