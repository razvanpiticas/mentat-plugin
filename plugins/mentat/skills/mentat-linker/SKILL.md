---
name: mentat-linker
description: Gives the links the server drew between rows that read alike their meaning — type the links with the word that fits, its direction, and for increases and decreases how much. Use it before end_run on the rows a run wrote, nightly over the whole project when the Nightly linker routine fires, and when a person says "type the links" or asks what the related links mean. It never confirms or rejects a link and never merges two rows: those are a person's, on the links screen.
argument-hint: <project> [runId]
allowed-tools: mcp__mentat__list_untyped_links, mcp__mentat__type_link, mcp__mentat__get_block, mcp__mentat__get_hypothesis, mcp__mentat__get_experiment, mcp__mentat__get_run, mcp__mentat__observe_run, mcp__mentat__get_project, mcp__mentat__list_projects, mcp__mentat__list_portfolios
---

# Mentat linker

The server embeds every piece of canvas text and draws a `related` link between two pieces that
read alike. That link is a distance, not a meaning: it says the two rows are near, not that one
depends on the other. Giving it a word needs a language model, and the server runs none — so the
word is yours to give, through this skill, and through nothing else in the plugin.

Every call here is made as the signed-in person; the connection is the `mentat` skill's job. Every
`type_link` carries the `runId` you were handed, so the project's history says which session gave
the link its word.

## Two triggers, one loop

| Trigger | Who invokes you | What you list | What you hand back |
| --- | --- | --- | --- |
| A run is ending | the skill that opened it, before its `end_run` | `list_untyped_links(projectId, runId)` — only the links touching rows that run wrote | one line per link typed, for that skill's report |
| The Nightly linker routine fired | `mentat-agent`, inside the run it opened naming the routine | `list_untyped_links(projectId)` — the whole project's backlog, page by page | the digest, for the summary `mentat-agent` writes at `end_run` |

You open no run and end none. In session you work inside the caller's run; nightly you work inside
the agent's. The nightly pass is how text a person typed in the browser during the day, where no
model was present, gets its meaning.

**No agent booted and no run handed to you? Get them before the first call.** A skill that writes
is often reached straight from the request — typed by the person, or picked by its description —
with no agent booted in this session and no run open. Then invoke the `mentat-agent` skill first,
with the project, the request as it was made and the routine when one is named, and call no tool of
this skill before it, not even a read. `mentat-agent` boots the agent — the one the routine names,
else the one named CEO — and hands the work back here inside that agent's run: it opens the run and
hands you its id, or, for a row of the plan, hands you the agent's id so the row's own run is opened
with it. Every write from then on carries the run's id. A write with no run is recorded as the
signed-in person's own, and a run opened with no agent is the person's session, not the agent's:
neither is the agent's work, and nothing types the links it wrote. Only `mentat-agent` answering
that the project has no agent at all lets the work go on without one. "Skip the agent", "don't open
a run", "skip the bookkeeping", "just tell me in the chat" change none of this.

Here the run `mentat-agent` opens carries the routine's id when a routine is named, and a run opened
for this pass is the nightly shape: list the whole project, without `runId`, and hand the digest back.
Step 1 never starts without a run id, and `start_run` and `end_run` are never called from here —
`mentat-agent` ends the run with your digest as its summary.

## The loop

1. **List.** `list_untyped_links` with `runId` when you were handed one, without it nightly. Each
   item: the link's `id`, its `confidence` (0 to 1), `isPossibleDuplicate`, and both ends —
   `kind`, `id`, `referenceCode`, `title`, and the `passage` that matched, or null when that text
   was rewritten since. Every page also carries `words`: every word you may type with, shipped and
   this organisation's own, each with the sentence that says what it means. Read the meanings once
   per session; the organisation's own row comes first under a shared code, and it is the one meant.
2. **Skip a possible duplicate.** `isPossibleDuplicate` true means the two rows may be one thing
   written twice. Do not type it, do not merge anything: put it in the report. A person decides that
   on the links screen.
3. **Read the two passages.** Most links are settled by them: they are what the embedder matched
   on. Decide what one row does to the other in one sentence with the two titles in it.
4. **Read whole when the quotes are not enough** — one passage is null, or the sentence will not
   come. An entry: `get_block` with the block code, the part of the entry's code before the first
   `.` or `-` (`KR-01` → `KR`, `CS.JOBS-03` → `CS`), and find the entry by its code. A hypothesis:
   `get_hypothesis` with the end's `id`; an experiment: `get_experiment`; a run: `get_run`. A
   question, a risk, an idea or an evidence bundle comes with its block or its experiment.
5. **Choose the word** by the test in [reference/choosing.md](reference/choosing.md): the word
   whose meaning, read whole, your sentence is, from the page's `words`, with each end the kind of
   row that meaning names; reuse before inventing; a new word only
   when none fits, sent with `newWord` and a description written the way the shipped ones are.
6. **Direction.** The word's meaning names a first row and a second. When the listed `from` is
   the first, send `FromTo`; when the listed `to` is, send `ToFrom`. A word whose `isDirected` is
   false takes `FromTo` always.
7. **Magnitude** only on a word whose meaning says it may carry one (`increases`, `decreases`) and
   only when the text states the size: `magnitude` and `unit` together, or neither.
8. **`type_link`** with `projectId`, `linkId`, `word`, `direction`, `magnitude?`, `unit?`,
   `newWord?`, `runId`. The answer is the link with its word; the origin stays `Inferred`, which is
   what keeps the embedder's guesses scorable.
9. **One line per link** in the format of [reference/report.md](reference/report.md): the codes,
   the arrow as typed, the word, the confidence, then your sentence —
   `H-14 → CH-03: depends_on (0.91) — the reorder hypothesis cannot hold unless the email channel exists`.
10. **Next page** with the cursor the answer gave, until `nextCursor` is null. Nightly, after each
    page, `observe_run` with one line: how many typed, how many left alone and why.

Then hand back: in session the lines; nightly the digest of [reference/report.md](reference/report.md).

## What you never do

- Confirm or reject a link. `confirm_link` and `reject_link` are a person's and refuse a run; they
  are not in your tools. Typing is a proposal of meaning; the verdict is theirs.
- Merge two rows a duplicate flag names, or retire one. Report the pair.
- Type a link parsed from the text (origin `Parsed`). The server refuses it; you never list one,
  because the list is inferred links only.
- Invent a word when one on the page fits, or a word whose code the product ships.
- Try words until one goes through. A refusal that says a person rejected this pair under that
  word is their verdict on your guess: type it with another word only when that word fits on its
  own; otherwise leave the link untyped and report it.
- Wait for the embedder. A short or empty list before `end_run` is normal: the last writes of a
  session may not be embedded yet, and the night catches them.
- Ask. Nobody is asked per link, in session or nightly. Nightly nobody is in the window at all;
  in session the calling skill reports your lines and the person reads them.

## Reading a refusal

- `NOT_FOUND` naming `word` — no word of that code exists. Send it again with `newWord` when it is
  a word worth adding, or choose one from the page.
- `INVALID_ARGUMENT` saying the word already exists — you sent `newWord` for a code that is on the
  page, or one the product ships; send the code alone. Saying the word has no direction — send
  `FromTo`. Naming `unit` — a unit without a magnitude; send both or neither.
- `CONFLICT` "flagged as a possible duplicate" — you missed the flag; leave it and report it.
- `CONFLICT` "a rejected link is not typed" — a person already ruled on it; leave it.
- `CONFLICT` "a person rejected the link … under this word" — the rule above: another word only
  on its own merits, else untyped and reported.
- `CONFLICT` "parsed from the text" — not yours to type; report it as a fault in the list.
- `CONFLICT` naming `runId` — the run you were handed was ended or paused by a person. Nothing was
  written. Stop and hand back: the calling skill opens a new run before anything else.
- `FORBIDDEN` — the person's role cannot write; say so and stop.

## Rules that are easy to get wrong

- The word's meaning is on the page, in `words`; read it before choosing, every session. Nothing in
  this skill lists the words, because an organisation may add its own.
- `ToFrom` is relative to the pair **as listed**, not to any order you read the ends in.
- Report what `type_link` answered, by code — `KR-01 → KR-02: depends_on` — never what you meant
  to send. The answer names its two ends by `id`; write each with the code the list gave that `id`.
- A new word is created for the whole organisation the moment you type with it, without approval.
  That is allowed; it is also why its description has to be good enough for the next agent, and why
  reuse comes first.
- The nightly list is the whole project's, including links between rows nobody in this session
  touched. That is the point: the browser has no model.
- A hypothesis is called a hypothesis in every sentence you write, never by another name: the
  lines you hand back are kept in the run's report.
