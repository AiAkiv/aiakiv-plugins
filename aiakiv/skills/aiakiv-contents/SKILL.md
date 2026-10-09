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
  the value basis of the work or the narrative stance of a part; wants beats
  extracted from a scene's prose, or a scene written or rendered in another
  language with the work's form settings; wants a confirmed body used as the
  model (an anchor) for the work's style, or asks what a style or beats mark
  in a response means; wants to set how a scene is told (its pace,
  interiority or what it withholds) or who the narrator is, or asks what a
  narration review such as a POV violation means; wants scenes like the one
  being planned or written found in the creator's style memory (cases);
  wants the work process kept or read (which decisions were made and why,
  what was tried and dropped), says "keep this as process" ("과정으로 남겨
  줘"); wants the whole work read at once to make material for another
  medium (a game world file, a video script), or asks what an item,
  creature or faction setting is for, or wants a scene's shot list written
  or the work's visual style set; says "ak save" while connected to a
  Contents work; or is about to call run_aiakiv_app_action with
  app="contents".
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
   `topic` (writing a scene: `tree`; facts and world lines: `facts`; beats:
   `beats`; another language or style: `lang`; the narrator or how a scene
   is told: `settings` and `slots`; cases of similar scenes: `memory`; the
   whole work for a game or another medium: `export`). The topics today are
   `overview`, `tree`, `slots`, `settings`, `facts`, `time`, `links`,
   `assets`, `export`, `lang`, `beats`, `memory`, `review`, `names`,
   `screens`, `errors`. The list and what each topic says come from
   `describe`; if they differ from this line, `describe` wins.
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
  ending in `:technique`), is not the work team. A project ending in
  `:process` is the memo team's process place (see "The process log"), not
  the work team either: do not use the link tools there; read memos and
  style with the app actions.
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

## Beats

A body node (kind `body`) holds the prose in `description`. Beats (pack
`beat_script`) are a language-neutral script of that prose, which the AI
extracts into the body's `slots.beats`, a list of beat objects. The app
stores, counts and publishes beats and notices when the prose has moved on;
it does not judge them. Read the `beats` topic of `describe` before the
first extraction; the fields, `think_modes`, `rhythms` and `relation_axes`
come from there (`novel.beat_fields`, `novel.relation_axes`).

- Extract in the draft, right after writing or fixing the prose and before
  the creator confirms.
- Call `get_node` on the scene, not on the body node. Its `body` (and each
  entry of `bodies[]`) carries `node_id`, `prose_hash`, `beats_count` and
  `beats_stale`.
- Then call `update_node` on that body `node_id` with `fields.slots` holding
  `beats` and `beats_from: {"prose_hash": ...}`, echoing the `prose_hash`
  you read exactly; never compute a hash. `fields.slots` overwrites the whole
  object, so send the body's other slots (such as `source`) back with it.
- Write `act` and `say.content` plainly, in the work's default language.
- `say.relation` is a closed vocabulary, tokens joined by spaces, at most one
  per axis, any order, possibly empty: power `up|level|down` (speaker
  relative to listener), distance `close|neutral|far`, formality
  `formal|plain|casual`, for example `"down far formal"`. An unknown token,
  or two on one axis, is `invalid_data`. Write the relationship, not a speech
  style; `get_node`'s `language.relation_map` says how a language renders it.
- Check the round trip: unfold the beats into a flat paraphrase and compare
  with the prose. If an event is missing or invented, or something withheld
  leaks, leave `add_review` with `kind: "beats_drift"` on the body node.
- `beats_missing`, `beats_stale` and `beats_count` appear as suggestions in
  the confirm preview and never block confirming. If confirmed prose is
  edited later, the app opens a `beats_stale` review item; extracting again
  in that same new version closes it.
- Do not rewrite the prose from the beats; the prose is canonical. Do not
  open a new version of a confirmed body only to add beats; extract in the
  next version. Do not translate from another language's body alone; the
  source is the author-language prose plus its beats.

## Writing in another language and form settings

Style lives on form settings: `kind: "setting"` with `facet: "form"`. Only a
form setting takes `lang` (`payload.lang` in `add_node`, `fields.lang` in
`update_node`). Empty means every language; a value limits the setting to
bodies in that language. The `lang` topic of `describe` has the rules.

- The optional style slots (pack `prose_voice`) are in the `settings` topic
  under `novel.slots_by_facet.form`: `register`, `dialogue_convention`,
  `description_density` (`sparse`, `even`, `dense`), `sense_bias`,
  `image_system` (key to meaning), `banned_patterns` (regular expressions;
  one that does not compile is `invalid_data`) and `ending_profile`.
- Put style on a form setting, never on a scene or a body: at the work root
  for the whole work, or under a part for that part only.
- Before writing a body in a language, or translating into it, call
  `get_node` on the scene with `data.lang` set to that language (left out,
  it is the body's language for a body node, else the work's default).
- Read `form` from that response: the merged form settings, with `slots`
  holding the values, `sources` naming which setting gave each value, and
  `texts` holding each setting's description, far to near. The nearer
  ancestor wins; `banned_patterns` are joined rather than overwritten.
- Read `language`: the language pack brief, with `relation_map`,
  `rendering_rules`, `typography`, `tells` and `asks`. It is `null` when the
  app has no pack for that language.
- Read `continuity.withhold`: what earlier scenes' beats still withhold
  (the latest five scenes with beats, up to 40 items, `truncated` when cut),
  so the new prose does not give it away.
- Two suggestions may appear, and neither blocks: `banned_pattern_hit` when
  the prose matches a banned pattern (the merged list plus the language
  pack's defaults), and, for Korean bodies, `relation_mismatch` when dialogue
  endings and the beats' formality tokens disagree in distribution.

## Anchors and the marks in a response

A form setting has an optional slot `anchors` (pack `prose_voice`): a list of
`{node_id, version, take, weight}`. `node_id` and `version` name a confirmed
body and its version. `take` lists what to borrow from it: `rhythm`
(sentence length and rhythm), `endings` (sentence endings), `dialogue`
(dialogue manner), `description` (description density), `imagery` (figures
and images); empty means everything. `weight` is `strict` or `loose` (empty
is `loose`; `strict` makes the checks narrower). Both vocabularies come from
the `settings` topic of `describe` (`anchor_takes`, `anchor_weights`).

- Write `anchors` only for a body the creator named as the model. Never pick
  one yourself, not even because it reads well or scored well.
- Write it with `update_node` on the form setting. `fields.slots` overwrites
  the whole object, so read the setting with `get_node` first and send its
  other slots back. Like every AI edit it is a draft the creator confirms. A
  wrong shape, or the same `node_id` and `version` twice, is `invalid_data`.
  A nearer form setting's anchors add to a farther one's.
- Before writing a body, call `get_node` on the scene with `data.lang` (as
  in the previous section) and read three more keys under `form`.
- `form.anchors`: each anchor with `state` and, when the app has it, `text`.
  `current` means that body is confirmed now at that version and `text` is
  its prose; `archived` means the version has moved on and `text` is that
  version's confirmed prose; `other_lang` and `missing` carry no text. Learn
  only what `take` names from the text; never lift its sentences. Long texts
  are cut (`truncated`), farther settings' anchors first.
- `form.profile`: statistics of all confirmed bodies in that language except
  this scene's. It is `null` with fewer than two bodies or too few sentences.
- `form.baseline`: what the style checks compare against, with `axes` (per
  axis `anchors`, `profile` or `null`), `stats` and `thresholds`. Read
  `thresholds` before writing; it says what the app will measure. When
  `baseline` is `null` no style statistics run; tell the creator the checks
  start once they pick one anchor (or on their own once enough confirmed
  content in that language has built up).
- `add_node` with `kind: "body"`, and `update_node` that sends a body's
  `description` or `slots`, answer with `marks`: a list of
  `{kind, level, text}` for that body only. They are the style checks
  (`banned_pattern_hit`, `uniform_rhythm`, `ending_monotony`,
  `figure_density`, `adverb_density`) and the beats checks. The `lang` topic
  of `describe` has each check's condition.
- Marks are suggestions: they never block confirming and leave no review
  item. When one points at something worth fixing, fix the prose and send it
  again. Do not loop on a mark you judge wrong, and do not tell the creator
  the app rejected anything.

## Narration: pace, interiority, withhold and the narrator

Pack `narration` adds how each scene is told and who tells the work; every
slot is optional. Its tokens are `slots_by_resolution.treatment` in the
`slots` topic of `describe`; the narrator's are `narrator_persons`,
`narrator_tenses`, `narrator_intrusions` and `narrator_reliabilities` in
`settings`.

- Three slots go on the scene (resolution `treatment`). `pace` is telling
  time against story time (`scene`, `summary`, `stretch`, `ellipsis`,
  `pause`); it is not `tempo`, which is sentence speed. `interiority` is how a
  mind is shown. `withhold` is text: what the reader must not yet know in
  this scene, apart from the beats' sentence-level `withhold`.
- `narrator` goes on a form setting: `{person, tense, access, attitude,
  intrusion, reliability}`, every key optional, `attitude` free text.
  `access` lists the character settings whose minds the narrator may enter:
  `[]` is nobody (external narration), no key is not decided, and a node that
  is missing, abandoned or not a character setting is `invalid_data`.
- The nearer form setting's `narrator` replaces the farther one whole (unlike
  `banned_patterns` and `anchors`, which join), so to change only the person
  for one chapter, write the whole object on that chapter's form setting. It
  is language neutral and usually sits on a form setting without `lang`. As
  with `anchors`, read the setting first and send its other slots back.
- Before writing a body, read `narration` from `get_node` on the scene:
  `pace`, `interiority` and `withhold` (`null` when not set) and `narrator`,
  the merged value with `source` and `access` as `{node_id, title, state}`.
  A `missing` state means that setting is gone; tell the creator instead of
  guessing. The key is absent when the pack is off.
- Read `continuity.scene_withhold` (earlier scenes' `withhold`, the latest
  five, oldest first) with `continuity.withhold`, so the new prose gives
  nothing away. The `lang` topic describes both.
- Four review kinds, left by the AI with `add_review` on the body node; the
  app measures none of them (the `review` topic has them):
  - `pov_violation`: the prose tells the inside of a character outside
    `narrator.access`. Not `pov_drift`, which is whose eyes see the scene.
  - `withhold_leak`: the prose reveals a scene's or a continuity `withhold`.
  - `narrator_tone_drift`: the voice departs from `attitude` or `intrusion`.
  - `honorific_mismatch`: the relationship changed (beats `say.relation`,
    settings) but the speech level did not, or the other way round.
- These are the creator's story decisions. Fill them when the creator states
  them, or to match a scene you just drafted and say so; they are drafts the
  creator confirms. When prose leaks a `withhold`, fix the prose, not the slot.

## Cases from the style memory

`get_node` on a scene, or on a scene's body node, returns `memory`:
`{case_query, medium, lang, tool, how}`. The app built `case_query` from the
scene's slots, title and `done_when`, with the same function that wrote the
summary of every case in the style team, so the two share one vocabulary.
The `memory` topic of `describe` has the recipe ("사례 찾기", finding cases).

- Search when making a scene plan (after reading the arc with `get_node`,
  before `add_node` with `kind: "plan"`) and again before writing a body.
- Pass `case_query` verbatim as `query` to `search_work_memory(doc_id,
  query=..., team="technique")`, then read the top three hits with
  `get_work_memory(doc_id, event_id, team="technique")`. `how` says the same
  in one sentence; if it differs from this line, `how` wins.
- A case is an analysis block (where the scene enters, what it withholds,
  the beat order, the slot values) followed by the text of a scene from one
  of the creator's own works. Take its structure; never copy its sentences.
- `lang` is not in the query: cases are shared across languages, so a scene
  written in English still finds Korean cases.
- No `memory` key above a scene or on a setting, a `null` `case_query`, zero
  hits, or `memory_unavailable` (`reason` `no_binding`, `no_session` or
  `read_failed`) all mean: write on without cases, and do not ask the creator
  to change the connection.
- On a connection bound to the work team, the same `case_query` works as the
  `query` of `search_partner_memory` with `links.style.link_id`.

## The process log

Inside the memo team, a work can have a process project,
`contents:<name>:process`. The connection folder the creator downloads from
the "AI connection" screen is tied to it. It keeps the work process: which
decisions were made and why, what was tried and dropped, where the direction
changed. The creator chooses whether to keep each one.

- Recognise it: `get_save_target` reports a `project` ending in `:process`,
  or one equal to `process.project` at the top level of the `get_work`
  response. There `session_in_work` is `false`, and that is normal; it is
  `true` only on the work team itself. `get_work`'s `session_hint` says the
  same.
- At the start of a conversation on that connection, read the past process
  once with `search_memory`. It returns memos and process entries together.
- After a big decision or a change of direction, ask the creator once
  whether to keep it as process. Do not ask every turn.
- When the creator agrees, or says "ak save", call `save_memory` right away.
  Follow the `aiakiv-save-and-recall` skill for the summary and entities.
- `update_memory` is allowed there on your own process entries.
- Do not put the work's content there. Scenes, settings and bodies go through
  `add_node` and `update_node` as drafts, and the app publishes them when the
  creator confirms.
- Do not put memos there. Ideas still go to memos with `add_scratch`.
- On Main or a `:canonical` connection, do not ask the creator to switch or
  rebind the connection only to log process. If they ask how to keep
  process, point to the connection folder on the "AI connection" screen.
- When `get_work` has no `process` key, the work has no process place yet.
  Opening the "AI connection" screen once creates it; tell the creator that
  if they want one. Do not try to create it yourself.

## The game world: places, creatures, items and the whole work

Contents keeps the facts of the world and the creator's intent, whatever
medium the work becomes. It does not keep engine numbers and does not write
engine files. The AI reads the whole work and writes the engine file from
it, the way it writes prose from a scene plan. The response shape and the
recipe are in the `export` topic of `describe`.

- Three setting facets sit beside `character` and `world`: `item` (an
  object), `creature` and `faction`. They have no slots; the description
  says what the thing looks like and what it is in the story, never its
  numbers. Rooms and regions are places (`facet: "world"`), and a place
  inside a place is a `part_of` link, not a child in the tree.
- Four directed link relations, free text in `note`: `part_of` (place in
  place), `connects_to` (place to place, one way: a two-way passage is two
  links), `located_in` (a character's usual place; a scene's own place is
  its `place_node_id` slot) and `member_of` (character to faction). No
  compass direction is stored; "the north gate" in `note` is only text.
  `add_link` refuses other pairings; see the `links` topic.
- Order of work for a game:
  1. Read the `export` topic, then the game tool's file format in its own
     repository.
  2. `export_work` with `confirmed_only: true`; add `include_bodies: true`
     only when the prose itself is needed, since a big work is large.
  3. Map. A place becomes a room (title to shown name, description to
     description, you choose the key). A `connects_to` link becomes one
     exit on its `from` room; the direction comes from `note` or the game
     worker, and a way back exists only if the reverse link does. A
     creature becomes a monster; its usual place is in its description or
     a fact, since `located_in` takes characters only. An item becomes an
     entry in the game's own item list. `part_of`, factions, characters,
     scenes, facts and world lines feed descriptions, quests and dialogue,
     not file entries.
  4. Settle every number (hit points, attack, drops, caps, levels) with the
     game worker, and write the file into the engine repository.
- Register the written file as an asset setting: `facet: "asset"`, the five
  slots `asset_type`, `asset_state`, `file_url`, `spec`, `canon_revision`
  (the `export_work` response's `doc.revision`), and `depicts` links to what
  it shows (none for a file of the whole work). Ask the creator to confirm
  the entry: `list_assets` signals (`stale`, `review_open` with
  `depicted_changed`) start only after that first confirmation.
- When the game needs a place or creature the story lacks, tell the
  creator, who adds it on the Contents screens or asks you to add a draft;
  only then does it enter the file. Nothing flows back from the file.
- The creator gets the same JSON as a file from "Export" in the top menu.

## Video and images: shot lists and the visual style

The canon for video and images is a scene's shot description, the looks in
the setting descriptions and the work's visual style. Prompts, seeds,
numbers, takes and the tool's file format are not canon.

- A shot list is a body: `add_node` with `kind: "body"` and
  `medium: "shots"` (`prose` is the default). A scene holds a `prose` and a
  `shots` body per language side by side. `medium` cannot change after
  creation, and a `source` must have the same medium.
- Read it with `get_node` on the scene and `data.medium: "shots"`: `body` is
  that medium's body, and `bodies` lists all with `lang` and `medium`.
- Write plain text, a numbered list, one shot per paragraph: what is seen,
  who, where, the action, the line spoken, the sound. Shot size and angle in
  words are fine. The shape is advised, not checked.
- Style checks, anchors, beats and cases are for prose: `marks` stays empty
  and `beats` on a shot list is refused. That is intended.
- The look to keep across video and images is `visual_style` on a form
  setting (pack `visual`): mood, colour, light, what to avoid. A nearer form
  setting replaces it whole; read it in `get_node`'s `form`. The looks of
  characters and places stay in their setting descriptions.
- Register only the adopted file, as an asset with `depicts` to the scene,
  the shot number in `spec` and `canon_revision`, as above.
- The creator writes and confirms shot lists on the same Studio screens as
  prose.

## Common mistakes

- **Everything the AI creates or edits is a draft.** The creator confirms it
  on the Contents screens, and the app publishes the confirmed version to
  memory at that moment. Do not tell the creator it is saved before then.
- **Do not save the work's content with `save_memory`.** On the work team
  and the memo team's other projects, `save_memory` and `hide_memory` are
  refused by default (while the account permission `app-memory:write` is
  off, which only the user can change). The process project is the
  exception, and only for the work process; see "The process log". Do not
  use `update_memory` to edit memory the app published either: the app's
  publish records and review marks would stop matching it.
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
- **Style belongs to form settings, not to scenes.** Register, banned
  patterns and the other style slots go on a form setting; see "Writing in
  another language and form settings".
- **Beats never replace prose.** Do not rewrite prose from beats; see
  "Beats".
- **Anchors are the creator's choice.** Do not write `anchors` for a body the
  creator did not name, and do not copy an anchor's text; see "Anchors and
  the marks in a response".
- **Marks are suggestions, not errors.** A `marks` entry in a body response,
  or a style suggestion in the confirm preview, blocks nothing; fix what is
  worth fixing and move on.
- **The narrator lives on a form setting, not on a scene.** `narrator` goes
  on a form setting (`facet: "form"`), and a nearer one replaces the whole
  object; `pace`, `interiority` and `withhold` go on the scene. See
  "Narration: pace, interiority, withhold and the narrator".
- **Withheld things stay withheld.** A scene's `withhold`,
  `continuity.scene_withhold` and `continuity.withhold` are what the reader
  must not learn yet; never write them into the prose. When prose leaks one,
  fix the prose and leave a `withhold_leak` review rather than deleting the
  slot.
- **Do not write your own case query.** Use `memory.case_query` unchanged.
  The app built it with the function that wrote the cases' summaries, so a
  reworded or translated query misses them.
- **No cases is not a blocker.** A missing `memory` key, a `null` query,
  `memory_unavailable` or zero hits all mean "write without cases". Do not
  stop, do not ask the creator to bind or switch anything, and do not invent
  a case.
- **Engine numbers are not canon.** Hit points, attack, drop weights, mint
  caps, compass directions and the file format belong to the game tool and
  its repository. Do not write them into a setting's description or slots,
  and do not ask the creator to decide them in Contents.
- **The app does not write the world file.** `export_work` gives the whole
  work as one neutral JSON; the AI assembles the engine file from it and
  registers the file as an asset. Do not look for an export action that
  returns `world.toml`, and do not send the JSON to the creator as if it
  were the game file.
- **Prompts, seeds and takes are not canon.** The shot list and the visual
  style are canon; the prompt text, seed, duration, aspect ratio, tool ids
  and the tries belong to the tool. Register only the adopted file as an
  asset.
- **Ideas go to memos** with `add_scratch`. The creator sees them as memos
  ("메모" on the Korean screens), so use that word with the creator.
- **`save_memory` has two places: style notes and the process log.** Build
  up style descriptions on the style team with `save_memory`, from a folder
  connected to the style team, following the `aiakiv-save-and-recall` skill.
  Keep the work process in the process project; see "The process log".
  Nowhere else in a Contents work.
- **"ak save" on a work connection.** If the user says "ak save" and this
  connection's project ends in `:process`, save right away with
  `save_memory`. If it ends in `:canonical`, or is the memo team's own
  project, say that the save will be refused there. Then offer a memo
  (`add_scratch`), or the connection folder from the "AI connection"
  screen, which is tied to the process project. Do not pick another
  destination on a guess.
- **On an error**, read the `errors` topic of `describe` before trying again.
