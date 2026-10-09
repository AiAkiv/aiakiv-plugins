---
name: aiakiv-cards
description: >-
  Create a public AiAkiv card from the user's memory with create_aiakiv_card.
  Use when the user asks to make a card, share a memory as a link or page,
  publish a summary card, or asks what AiAkiv cards are or how to fix or delete
  one. A card is a one-page public site (card.aiakiv.com) written by YOU from
  memory search results and baked by the card service. It is live on the open
  web the moment it is created.
---

# AiAkiv cards

<!-- shared-create-section: begin. This block is identical in every edition of the aiakiv-cards skill. -->
A **card** is a one-page public summary of the user's AiAkiv memory: a title,
3 to 5 bullets, a one-line conclusion and an optional longer body, plus two
share images (a square one and a wide 1200x630 one) with the short address
stamped in. Anyone with the address can view it, no login. You create it with
`create_aiakiv_card`. The server never calls a model: **you write the
content**.

## The one fact that governs everything

**Creating IS publishing.** The card page and both images are live on the open
web the instant `create_aiakiv_card` returns. The card is unlisted (nobody
finds it without the address) but NOT private. There is no draft state, no
editing and no undo except deleting the card (see "Managing cards" below). So
filter before you create, and relay the response's `notice` to the user first,
verbatim.

## Procedure

1. **Search memory first** (`search_memory`, then `get_memory_content` when a
   summary is not enough). The card summarizes what is actually stored; do not
   invent content. Set `source_count` to the number of memories you actually
   used.
2. **Filter for sensitivity**: personal names, emails, internal addresses,
   decisions not yet public.
3. **Get consent to publish.** A request to make a card is consent to publish
   that card. If the content touches anything sensitive, or you had to choose
   what to leave out, show the user the draft and wait for an explicit yes
   before creating.
4. **Write to fit the images, not just the limits** (see "Fitting the images").
   Aim to fit on the first try: creating again makes a NEW card at a new
   address and leaves the first one published, and counted against the user's
   card quota, until the user deletes it.
5. **Create** with `create_aiakiv_card`, passing the fields below directly.
   There is no format lookup step before it.

## Fields and limits

The card service checks these after it trims whitespace and drops empty items.

- `title`: one line, up to 80 characters. Required.
- `bullets`: 3 to 5 key points, one line each, up to 120 characters each. Required.
- `conclusion`: one line, up to 200 characters. Required.
- `source_count`: how many memories the card was written from, an integer of 0 or more. Defaults to 0.
- `tags`: up to 3 search terms for card.aiakiv.com, up to 20 characters each. They are drawn into the images and **cannot be changed later**. Defaults to none.
- `body`: optional longer text shown on the page but not in the images, up to 30,000 characters of restricted markdown: paragraphs, `##` headings, `-` and `1.` lists, `>` quotes, inline and fenced code, bold, italic, http(s) links and `---`. Anything else shows as plain text.

Title, bullets, conclusion and tags are single lines: line breaks in them
collapse to spaces.

## Fitting the images

Bullets share one fixed block of vertical space, so their count and length
trade against each other. A line holds about 31 Korean characters (Latin
letters and spaces fit more) and each bullet gets up to 3 lines, but when the
total does not fit, every bullet is cut to fewer lines and shortened with an
ellipsis. Staying under 120 characters is not enough.

What fits both images: a title of up to about 20 Korean characters, a
conclusion of up to about 30 Korean characters, and 3 or 4 bullets of up to
about 40 Korean characters each. With 5 bullets the last one is left out of
the wide image. The page always shows the full text; only the images crop.

## Handling the response

- **`notice` comes first.** Show it to the user before the link and before any
  summary, uncompressed. It states what just became public and how to take it
  down.
- **`url`** is the card page: show it verbatim as a clickable link. `og_url`
  (wide) and `square_url` (square) are the share images.
- **`fit`** is present only when an image cut something. Report it as is:
  `bullets_truncated` and `bullets_dropped` are original bullet numbers,
  `title_truncated` or `conclusion_truncated` may also appear, and `hint` says
  how much to shorten. Let the user decide whether to shorten and create again,
  and if they do, remind them the first card stays up until they delete it.
- **Errors** come as `{error, message or hint, field}`. On any error no card was
  made; never say it was.
  - `invalid_card`: fix the named `field` and try again.
  - `payload_too_large` or `request_too_large`: shorten the body.
  - `quota_exceeded`: the response carries `window`, `limit` and `used`. Tell
    the user which limit was reached; do not retry.
  - Anything else: relay it and stop.

## What you must not do

- Create a card the user did not ask for ("summarize this" is not a card request).
- Fill a card with content that is not in memory.
- Quietly create again after a `fit` warning; that leaves two public cards.
- Say a card was made when the response was an error.
- Paste the card's address anywhere on the user's behalf; sharing is their call.

## Sharing, for the user

- KakaoTalk, Slack, Discord, Notion and blogs unfurl the bare address into a
  card. Instagram and some board sites do not: upload the square or wide image
  instead; the address is stamped inside it.
- Deleting removes the page immediately, but other sites' preview caches can
  linger. Deleting the account does NOT delete cards, so delete them first.
<!-- shared-create-section: end -->

## Managing cards

`create_aiakiv_card` only creates. Editing a card's text is not possible
anywhere; create a new card instead (the old one stays up until deleted).

Listing, deleting, search visibility and filing tags for cards the user
already made go through `run_aiakiv_app_action(app="console")`: `list_cards`,
`set_card_searchable`, `set_card_tags` and `delete_card`. Each needs the
matching console AI permission the user turned on; see the aiakiv-console
skill. The user can do the same things by hand in the AiAkiv web console
(app.aiakiv.com, Data, Cards).

- Claim you deleted a card or changed its search visibility only when the
  console call actually returned `ok: true`. A `delete_card` error means the
  card is still public.
- Search visibility is OFF by default. Turning it on lists the card in
  card.aiakiv.com search by title and tags; turning it off later un-lists it,
  but the card stays public to anyone with the address.
- Filing tags (`set_card_tags`) are not the `tags` field. Filing tags show on
  the card page and group cards in search; they never change the image.
- The public nickname and the tag palette are set by the user in the web
  console, not by you. A nickname is optional; without one nothing
  identifying the user is published. Their email is never published.
- card.aiakiv.com search narrows three ways (text, author nickname, filing
  tag), and the address bar tracks whatever is narrowed, so any view can be
  shared as a link.
