---
name: aiakiv-contents
description: >-
  Where to start, and what not to do, in AiAkiv Contents: the app where a
  creator writes a work (parts, scenes, settings such as characters and places,
  facts and world lines, memos, drafts) through
  run_aiakiv_app_action(app="contents"). Use when the user talks about their
  Contents work, a scene, a part, a character or place setting, a fact or world
  line, a memo or a draft; says things like "let's continue my work in
  Contents", "leave this as a memo", "put this in as a draft" ("Contents 에서
  내 작품 이어서 쓰자", "메모로 남겨 줘", "초안으로 넣어 줘"); wants to search
  the work's memos or style memory, including through links from the work
  team; wants to attach another team's memory (their own brainstorming team,
  someone else's team, a public org) to the work or detach it; wants to set
  the value basis of the work or the narrative stance of a part; says "ak save"
  while connected to a Contents work; or is about to call
  run_aiakiv_app_action with app="contents".
---

# AiAkiv Contents

The rules of Contents live in the app, not in this skill.
`run_aiakiv_app_action(app="contents", action="describe", data={"topic": ...})`
is the live contract, and it changes whenever the app is deployed. This skill
only says where to start and which mistakes to avoid. Whenever this file and
`describe` disagree, follow `describe`. Take action arguments from `describe`,
never from memory.

## Start here

1. Read the overview:
   `run_aiakiv_app_action(app="contents", action="describe", data={"topic": "overview"})`.
2. Read one more topic that fits the task, with the same call and another
   `topic` (writing a scene: `tree`; facts and world lines: `facts`). The
   topics today are `overview`, `tree`, `slots`, `settings`, `facts`, `time`,
   `links`, `assets`, `lang`, `memory`, `review`, `names`, `screens`,
   `errors`. The list and what each topic says come from `describe`; if they
   differ from this line, `describe` wins.
3. Pick the work with `get_work`. Called without `doc_id` it lists the user's
   works with `recent_work_id`. If the user named a work by its title and you
   do not know its `doc_id`, find it in that list, then call `get_work` with
   that `doc_id`.

## Finding memos and style

There are two ways to read the memo team and the style team.

1. **App actions**, from any connection: `search_scratch` and `get_scratch`
   for memos, `search_work_memory` and `get_work_memory` for confirmed content
   (`team` `work`) and the creator's style (`team` `technique`). They search
   and read the full text.
2. **Links**, only when this connection is bound to the work team (see
   "Is this connection the work team?" below). The work team reads the memo
   team and the style team through links. Pass the `link_id` to
   `search_partner_memory`, `find_partner_memory_connections` or
   `query_partner_memory_graph`. Use links to follow connections between
   memories, to look at them as a graph, or to scan many memos broadly. Follow
   the `aiakiv-links` skill for how. Links only read.

Which `link_id`: if the `get_work` response carries `links`, use
`links.memo.link_id` and `links.style.link_id`. `links.sources`, when
present, lists other memory attached to this work (see "Attaching other
memory" below); each entry's `label` says what it is, and its `link_id` goes
to the same partner tools. If `links` is missing, call
`list_partner_links` and find the memo team's link by matching its
`partner_org_id` with the `scratch_org_id` in a `search_scratch` response. If
you cannot tell which link is which, use the app actions.

Is this connection the work team? Check before the first link tool call,
not after. Either sign is enough; without one, do not use the link tools for
Contents at all.

- `get_save_target` reports `binding` `folder` or `api-key` and a `project`
  whose name ends in `:canonical` (the work's own team, for example
  `contents:myproject:canonical`). A `binding` of `main`, or a project with
  any other name (another app's project, the user's own project, a style team
  ending in `:technique`), is not the work team.
- `get_work(doc_id=...)` reports `session_in_work: true`. The app decides
  this from the session itself, so it is the final word when the two
  disagree.

Most conversations reach Contents through `run_aiakiv_app_action` from a
connection that follows Main or sits on some other project. There the work
team's links are not this connection's links: `list_partner_links` shows
other things, and `search_partner_memory` with a `link_id` from `get_work`
fails or reads the wrong team. Use the app actions there.

Which way: on a connection bound to the work team, try links first for memos
and style. Otherwise use the app actions. If a link tool returns an error or
there is no link, go back to the app actions. The app actions work whichever
project the connection follows, so do not ask the creator to change the
connection. Leaving a memo is always `add_scratch`; links never write.

## Attaching other memory to the work

The work team can read any other team's memory through a link: the creator's
own brainstorming team, someone else's team, or a public org. Two steps, both
from the conversation.

1. Get a code. For a team the user owns, call
   `run_aiakiv_app_action(app="console", action="create_link_invite", ...)`
   on that team with the user's own email as the invitee (the `aiakiv-console`
   skill has the arguments; the user's `link:invite` permission must be on).
   For someone else's team, the user pastes the code they received. A public
   org needs no code, only its org id.
2. Attach it to the work:
   `run_aiakiv_app_action(app="contents", action="add_work_source",
   data={"doc_id": ..., "code": ..., "label": "brainstorm"})`, or
   `public_org_id` instead of `code`. Take the arguments from `describe`.

Direction matters: the team that issued the code is the one being read, and
the work team is the reader. Do not redeem a code issued by the work team on
the other side; that links the other way round. A code is consumed when it is
redeemed, so never retry a successful call with the same code. A link covers
the whole team, so everything in that team's memory becomes readable from the
work; if the user wants a narrower scope, suggest a separate team for it.

Attached memory is read only through links, so it needs a connection bound to
the work team (the check above). On a connection that follows Main
there is no app action that reads it; tell the user to connect through the
working folder from the advanced settings on the "AI connection" screen.
`remove_work_source` detaches; the memo team and style team links that the
app created with the work cannot be detached.

## Value basis and narrative stance

`slots.value_basis` is one key with two meanings. On the One line (the
work's top sentence) it is the **Value basis**: what the Value line on the
Flow board measures, and from whose side (pack `dramaturgy_core`). On a Part
(resolution `synopsis`) it is the **Narrative stance**: from what standpoint
that part is told and judged (pack `stance_core`, for every kind of work).
The `slots` topic of `describe` has the wording.

- Read the value basis, and the part's narrative stance, before you pick a
  scene's `turn`.
- `fields.slots` in `update_node` overwrites the whole `slots` object, so
  read it with `get_node` first and send it back whole.
- The creator can write both on the "Value basis and narrative stance"
  screen (`basis` in the `screens` topic), so the AI is not the only way in.
  Keep the two apart: do not put a part's narrative stance into the One
  line's `value_basis`, or the work's value basis into a Part's.
- The Flow board draws the Value line itself, one line per zoom joined from
  the `turn` values; a part's narrative stance does not break it. Fill in
  `turn` and the basis; do not draw or compute the line.

## Common mistakes

- **Everything the AI creates or edits is a draft.** The creator confirms it
  on the Contents screens, and the app publishes the confirmed version to
  memory at that moment. Do not tell the creator it is saved before then.
- **Do not save the work's content with `save_memory`.** On a work team and
  the memo team, `save_memory` and `hide_memory` are refused by default (while
  the account permission `app-memory:write` is off, which only the user can
  change). Do not use `update_memory` to edit memory the app published either:
  the app's publish records and review marks would stop matching it.
- **Facts and world lines are not in the Studio tree.** Do not send the
  creator to Studio to find or confirm one. The creator reads, edits, deletes
  and confirms facts on the fact list and fact view screens, which the
  `facts` topic of `describe` names. When asked to confirm a fact, point to
  the fact view; the AI cannot confirm. A world line is edited (title,
  description, calendar), deleted and confirmed on the world line view,
  which the `facts` topic also names; send the creator there, not to Studio.
- **Changing a world line's calendar opens review items.** Changing a world
  line's `slots.calendar` through `update_node` opens a `calendar_changed`
  item on that line's confirmed facts that have a time, and the response
  reports how many in `review_items_opened`. Do not change it as a side
  effect of another edit; tell the creator first what it will open.
- **There are no episodes.** A scene's place in story time comes from its
  own `time` or from the facts it `tells`. A scene with neither shows "Not
  placed in story time" among its Gaps on the Flow board. Do not look for,
  ask about or set episodes; an `episode_id` left on an old node is a
  leftover and places nothing.
- **Ideas go to memos** with `add_scratch`. The creator sees them as memos
  ("메모" on the Korean screens), so use that word with the creator.
- **Style notes are the one place for `save_memory`.** Building up style
  descriptions on the style team is done with `save_memory`, from a folder
  connected to the style team, following the `aiakiv-save-and-recall` skill.
- **"ak save" on a work connection.** If the user says "ak save" and this
  connection follows a Contents work team, say that the save will be refused.
  Then ask whether to leave it as a memo (`add_scratch`) or to save it through
  another connection such as Main. Do not pick another destination on a guess.
- **On an error**, read the `errors` topic of `describe` before trying again.
