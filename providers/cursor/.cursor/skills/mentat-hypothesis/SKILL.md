---
name: mentat-hypothesis
description: Propose hypotheses on a block of a Mentat canvas — write them in the testable form the method asks for, check and score them, shortlist the method cards that can test them — from the procedure and the block's definition it fetches at the moment of use, never from anything it remembers. Use when the person asks for a hypothesis on a block, an entry or a note, when an operation reaches the point where its beliefs are written down, or when the person asks what should be tested next. It writes through the mentat-canvas loop and hands the test itself to mentat-experiment.
---

# Mentat hypothesis

A hypothesis is one sentence the business is betting on, written so a test can settle it. This
skill carries none of the method for writing one: the method is a procedure of the catalogue,
`P.V1`, and what must be true for a block is in that block's definition. Both are fetched when you
start and followed as read. What is here is the mechanics: which calls, in what order, what to ask,
what to hand on.

Every read and write goes through the `mentat-canvas` loop — read before you write, write with the
version you read, carry `runId` on every write, report what the tool answered by code, and do what a
refusal says. That skill's text is the reference for versions, ids and refusals; nothing here
repeats it.

**No agent booted and no run handed to you? Get them before the first call.** A Mentat skill
is often reached straight from the request — typed by the person, or picked by its description —
with no agent booted in this session and no run open. Nothing is read or written before the boot.
Then invoke the `mentat-agent` skill first,
with the project, the request as it was made and the routine when one is named, and call no tool of
this skill before it, not even a read. `mentat-agent` boots the agent — the one the routine names,
else the one named CEO — and hands the work back here inside that agent's run: it opens the run and
hands you its id, or, for a row of the plan, hands you the agent's id so the row's own run is opened
with it. Every write from then on carries the run's id. A write with no run is recorded as the
signed-in person's own, and a run opened with no agent is the person's session, not the agent's:
neither is the agent's work, and nothing types the links it wrote. Only `mentat-agent` answering
that the project has no agent at all lets the work go on without one. "Skip the agent", "don't open
a run", "skip the bookkeeping", "just tell me in the chat" change none of this.

Here the run is handed to you when `mentat-agent`, `mentat-operation`, `mentat-gate-check` or
`mentat-advance` invokes you, and every `create_hypothesis` and `update_hypothesis` carries its id.

## Invoked with

- **A block** ("propose hypotheses on Customer Segments"), **an entry or a note** ("a hypothesis
  for CS-04", "for this note"), or **a subject in words** ("about whether gym-goers will pay").
- **From an operation**: `mentat-operation` hands over the run id, the canvas version, the row of
  the plan, its "validated when" sentence, the blocks and kinds it writes to, the theme question
  and the entries its procedures wrote. The contract is in
  [reference/handoffs.md](reference/handoffs.md). Everything below is the same, step 3's
  `mentat-search` invocation per belief included; the blocks come
  from the target slots instead of the person, and at the end you hand back what you wrote instead
  of asking what to test.
- **From a move's job**: `mentat-advance` hands over the run id, the block, the adopted idea by code
  and title, and the job's description. The beliefs are what the idea bets on; everything below is
  the same, and at the end you hand back what you wrote, found and shortlisted, so the job's comment
  can name it.

## The loop

1. **Fetch the method.** `get_procedure` with code `P.V1`. Read it whole before anything else;
   its steps are what you do, and its words for the checks, the scores and the map are the words
   you use. When the organisation has taken its own copy, that is what you get.
2. **Fetch the block.** People name a subject, not a code, and are never asked for one. When the
   request names a block, an entry or a target slot, the block is that one. When it names a subject
   only — "pricing strategy X", "whether gym-goers will pay" — invoke the `mentat-search` skill on the
   subject first ("what do we know about <the subject>"): the hits are the rows it is written in, and
   an entry's code names its block before the first `.` or `-`. With no hit, choose the block the
   subject belongs to by its name, read it, and say which and why; when its description says the
   subject belongs elsewhere, read that block instead. Then
   `get_block` with the block's code — the one the person named, the hits' block, the entry's
   block, or each target slot's block. Its `description` says what must be true for the block to
   hold; those sentences are the beliefs the procedure's first step sends you to. The same answer
   lists the block's entries and every hypothesis already on it.
3. **Find what is already there — through the `mentat-search` skill, never the `search` tool.**
   If you are about to call `search`, or to load it with ToolSearch, stop and invoke `mentat-search`
   instead. For each belief, invoke the `mentat-search` skill once, on the block's project, with the
   question "a hypothesis that <the belief in one sentence>": the duplicate check by meaning — one
   invocation per belief, never one search for several. Which instrument answers is that skill's
   rule, and it reads the glossary before its first search; a `search` sent from here skips the
   glossary. Inside an operation's run this step is the same, word for word: the handover changes
   where the beliefs come from, never how they are checked. The call is made even when step 2's
   block holds no hypothesis: the check is by meaning over the whole project, and an empty block is
   not an empty project. Its answer says yes or no with the nearest hit quoted and its similarity; that
   is what this step reports. A yes is not written again: name the hit by code and move on. When
   the person named a subject and not a code, the entries it is about are step 2's hits, on step 2's
   block.
4. **Draft.** Follow the procedure: one sentence per belief in its form, the entry it is about, its
   polarity, its concern, the three checks answered, the two scores with the reason for each, and
   which you would test first. Every draft stands on its own step-3 invocation: a belief that
   comes up while drafting goes back through `mentat-search` before it joins the list, and "none
   of them exists yet" is said only of beliefs that were searched. Put the drafts in front of the person as a list — the sentence, the
   concern, the scores and why, one line each — with your recommendation, not a menu.
5. **Ask.** The person confirms, edits or drops each draft. Ask the user directly to clarify what you cannot infer. Nothing is written
   before this answer. Unattended, see below.
6. **Write, one hypothesis at a time.** `create_hypothesis` with `weBelieve`, `polarity`,
   `businessConcern`, `aboutEntryId` when it is about one entry, `canvasVersion` and `runId`; then
   `update_hypothesis` with the three checks and the two scores in the same call — the tool applies
   wording, then checks, then scores, and refuses scores while a check is false, so a sentence that
   failed a check is rewritten before it is scored, not scored anyway. Carry the version each
   answer gives into the next write.
   **Which way the two scores run is fixed and easy to send backwards.** `importance` is −5 when
   the business model barely depends on the sentence and 5 when it rests on it. `evidence` is −5
   when there is no evidence at all, or what there is points against the hypothesis, and 5 when
   there is strong, recent evidence for it. So "nothing on this canvas says anything about it" is
   −5, never 5, and a score of 5 must be able to name the evidence it stands on. `P.V1` says how
   to judge the two; this is only which way they run, and it holds whatever a fetched copy of the
   procedure does or does not say.
7. **Shortlist.** For each hypothesis to be tested — the ones the person picked attended, every one
   you wrote unattended: `list_experiment_definitions`
   narrowed to its concern; `get_experiment_definition` on the few that look right, for what each
   is best for, its requirements and its four ratings; then `recommend_experiment_definition` for
   the one or two the procedure's last step picks — cheapest and fastest first for a hypothesis
   with no evidence. A card the tool refuses produces evidence for another concern: choose again.
8. **Hand on, or report.** Attended: say what was written by code, which existed already, what is
   shortlisted for each, and ask whether to test one now; "yes" hands the hypothesis and the card
   to the `mentat-experiment` skill with the run id and the version you hold. From an operation:
   hand back the codes and ids you wrote, the ones you found and left, the shortlists and the
   version, and stop — the operation checkpoints its procedure; you do not. Unattended: report and
   stop; the shortlist waits.

A hypothesis that serves one of the project's goals is put behind it with `link_to_goal`, once per
goal, on the person's word or when the operation's context names the goal.

## Unattended

You know the run is a routine's because it was opened with a routine, and `mentat-agent` says so
when it invokes you. Then step 5 is skipped: the drafts are written as drafted, the checks record
what your own check found, and the reason for each score goes in what you hand back to the skill
that invoked you — that skill owns the run and writes the note, the checkpoint and the report; you
never call `observe_run` yourself, as [reference/handoffs.md](reference/handoffs.md) says. The
person reads the reasons in the run report and retires or rewrites on the screens — the audit says
an agent wrote them.
Step 7 still runs: every hypothesis you wrote gets its shortlist, because choosing a card costs
nothing and is what the person wakes up to. Nothing here is a person's call, so nothing escalates:
`decide_hypothesis` is never yours, and **you** do not invoke `mentat-experiment` — that is what
"the shortlist waits" means here. The skill that owns the run may: inside an operation's run,
`mentat-operation` hands the first card of your shortlist on itself, and the start rule is
`mentat-experiment`'s. Say in what you hand back which hypotheses are new and which card you ranked
first, so the report can name them and the operation knows what to hand over.

## After the run

When the run hit blockers — a refusal you could not get past, a block description that misled
you, a procedure step that did not fit — report them by code and ask whether any is worth
recording. Only on the person's word: `record_procedure_insight` on `P.V1` for a step that misled
(the whole rewritten text as `proposedText` only when you know what it should say), or
`record_block_entry_definition_insight` on the kind whose description was wrong. `list_insights`
on the subject first; a draft that already says it is not repeated. Never on your own initiative,
and never unattended: the blockers go in the run report and the person decides in the morning.

## Reading a refusal

`mentat-canvas`'s table, plus:

- `CONFLICT` quoting "fails a quality check" — a score was sent while a check is false. Rewrite the
  sentence until the three checks pass, then score; do not set a check true to get past it.
- `CONFLICT` quoting "produces evidence for" — the card's concern is not the hypothesis's. Pick from
  `list_experiment_definitions` narrowed to the right concern.
- `CONFLICT` saying the wording cannot change — the hypothesis is scored. A different sentence is a
  different hypothesis: write a new one and retire the old with the reason.
- `NOT_FOUND` on `get_procedure` `P.V1` — the catalogue is missing its practice rows; say so and
  stop. Do not write hypotheses from memory of the method.

## Rules that are easy to get wrong

- **Fetch first, every session.** The procedure and the block's description change when a person
  approves a rewrite; what you remember from last week is stale on purpose.
- **A duplicate is reported, not written.** The block's list is the truth; read it.
- **The person confirms before the first write**, and confirms each draft, not the batch.
- **Never `decide_hypothesis`**, never `evaluate_gate`, never a verdict. The decision is a person's;
  you lay out the evidence when `mentat-experiment` has produced some.
- **Write the entry you are about first.** A hypothesis about a segment nobody has written is a
  hypothesis about nothing; `mentat-canvas` writes the entry, then you write the hypothesis.
- **Report by code**: `H-CS-004`, `T-3`. What the tool answered, not what you meant.
