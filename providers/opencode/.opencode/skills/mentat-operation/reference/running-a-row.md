# Running a row, step by step

Load this when starting a run, when the level is Experiment, or when the run is unattended. The
rules a run has to follow unattended are in the skill body, not here; this file carries the detail
of what each part of the package is for, the exact escalation bodies, and the shape of the report.

## What the package carries, and where each part is used

| Package member | Used for |
| --- | --- |
| `definition.description`, `themeQuestion` | Read first; the framing every step is done under. The description opens with "Read first:" and the reads it names, each with its tool |
| `definition.summary.validationLevel` | Which ending: verdict, hand-over, or neither |
| `definition.validatedWhen`, `ifNot` | The Checkpoint judgement; the pass sentence the hand-over repeats; which operations an ending goes back to |
| `definition.procedures[]` (`procedureId`, `code`, `name`, `instructions`, `executionOrder`, `source`, `founderInputNeeded`, `founderInputPurpose`) | The operation's own steps, in order; `procedureId` is what `checkpoint_run` names; a step's founder input says whether it asks the founder and what |
| `definition.founderInput` | The operation's founder input in short: the strongest a step asks (`Always`, then `WhenInstructionsSay`, then `None`) and which steps |
| `revisits` | The operations this one may send the project back to, in order: the only ones `revisitOperationIds` may name |
| `handOver` | On an `Experiment` operation: the entry kinds it hands to hypotheses, how many of each from the top, in test orders, and its instruction. The checkpoint of the last own step answers it resolved: the entries themselves. No lines: the checkpoint answers no test order, and Hypothesise works from the "validated when" sentence and the entries the run wrote |
| `sharedSteps` | On an `Experiment` operation: Hypothesise, Design the test, Learn from the evidence, Decide — each with the `procedureId` its checkpoint names and the code `get_procedure` reads it by |
| `resolvedTargetSlots[]` (`blockCode`, `entryKindCode`, `entryKindStorage`, or the idea) | Where the steps write; an idea slot is written with the idea tools |
| `ideation` | The project's idea, when a slot targets it: the starting point, the candidates, the statement in force |
| `prerequisites[]` (`slug`, `kind`, `state`) | What `start_run` will check; a `Mandatory` one not `Met` is the refusal coming |
| `latestRun`, `resumableRun` | Resume, or start fresh |
| `gate.isDue`, `gate.pendingSlugs` | Whether this row finishes the tier |

## The step loop

For each of the operation's own steps, in `executionOrder`:

1. Read `instructions` whole.
2. Read what the step reads: the target blocks (`get_block`), the segments and value propositions it
   names (`get_customer_segment`, `get_value_proposition`), the idea (`get_ideation`).
3. A step whose `founderInputNeeded` is `Always`, or `WhenInstructionsSay` when its instructions say
   so: ask before anything else — in the chat in a session, through the inbox unattended (the body
   below), and stop there until the answer comes.
4. Draft what the step produces, as the entries and parts it asks for, on the kinds the target slots
   name, with the fields the kind's schema lists (`get_block` carries it; on Customer Segments only
   with `includeSchemas: true`). A step that produces a
   hypothesis drafts it in the "We believe that" form; a step that opens a question drafts the
   question.
5. In a session: show the draft, ask, write. Unattended: write.
6. Write through `mentat-canvas`, each call with `runId`; a nested kind with its owner's id as
   `ownerId`; keep the ids answered.
7. `checkpoint_run` with the step's `procedureId`, the ids, one line of note, the founder's words as
   `founderAnswer` when the step asked them in the chat, and `revisitOperationIds` when the step
   sends the project back.

A step whose text says to interview, to search the web, to compute from data the canvas does not hold:
do what can be done from what is in hand and say in the checkpoint note what was not; unattended,
that is a blocker (below) only when nothing at all can be produced.

## The hand-over to `mentat-hypothesis` (the Hypothesise step)

The contract is `mentat-hypothesis`'s own (`reference/handoffs.md` there); hand over, in this order
and nothing else: the project id; this run's id and the canvas version you last held; the row (id,
slug, position); the Hypothesise step's code, from `sharedSteps`; the hand-over exactly as the last
own step's checkpoint answered it — "Hypothesise now", the "validated when" sentence, the operation's
own instruction, and every test order with its status and, when it is open, its entries by id and
code. A hand-over with no test order — the operation names no hand-over lines — adds two things: the
target slots (`resolvedTargetSlots`, block code and entry kind code) and the entries the run's own
steps wrote, the `writtenEntryIds` the checkpoint answered, by id and code; `mentat-hypothesis` then
writes the hypotheses the "validated when" sentence depends on, about those entries. It hands back
the new hypotheses' codes and ids, the ones left alone, a shortlist per hypothesis, and the canvas
version. Checkpoint the Hypothesise step (its `procedureId` from `sharedSteps`) with the new ids as
`producedHypothesisIds`.

A test order waiting for an earlier one is named in the summary as waiting, never hypothesised now.
Once it opens, the row is worked again for it in a new run that writes no own step: the open order
is read from `get_run` on the earlier run of the row, because a new run answers the hand-over only
after its last own step is checkpointed; then `mentat-hypothesis` with that hand-over, the Hypothesise
step checkpointed by its `sharedSteps` `procedureId`, and `end_run` with no verdict — the skill
body's "A later test order, once it opens".

## Design the test — who hands to `mentat-experiment`

**In a session**, `mentat-hypothesis` does, on the person's word, inside this run. When it hands
back, `get_hypothesis` on each new hypothesis: an experiment designed in this run (its `runId` is
this run's) means Design the test is done — checkpoint it with the note "Designed {experiment codes}
on {hypothesis codes}"; none means checkpoint nothing for it and say in the summary that the design
waits.

**Unattended**, you do, and the card is the shortlist's — the skill body's rule: a hypothesis
Hypothesise handed back that stood shortlisted before this run first, else the one
`mentat-hypothesis` ranked first; its card is the first of its `recommendedDefinitionIds`, in order,
the next on the same list when one does not fit, and a card chosen now only when the list is empty
or none fits. Hand over, as `mentat-hypothesis` would: that hypothesis (id, code), that card (id,
slug), this run's id and the canvas version, the "validated when" sentence. It hands back the
experiment id and one of two states. *Started* — the card was `Digital`, cost rating 1, no spend
named: checkpoint Design the test with "Designed {experiment code} on {hypothesis code}; started",
then end the run. *Paused* — it raised "A start that spends, or needs the person's hands" through
`mentat-inbox` and paused the run: write nothing more; the report is the run's events, and the
skill's own "Resuming" section says how the next firing finishes the step.

## The Checkpoint judgement

Read what the run wrote (`get_block` on the target blocks, `get_customer_segment` and
`get_value_proposition` on what it wrote inside, the codes from the checkpoints). Against the
"validated when" sentence, write two or three sentences naming codes: what meets it, what does not.
Propose the verdict, and on `Invalidated` the operations it sends the project back to — the first of
`revisits` unless what failed says another. In a session the person confirms; unattended, the
escalation below.

## The four escalations, unattended

Their rows — who raises, the subject, the body's order, the proposed answer — are in
`mentat-inbox`'s `reference/asks.md`: "A step that asks the founder", "A missing key, tool or access",
"Checkpoint verdict", "A prerequisite refusal". The bodies this skill writes, through `mentat-inbox`,
`raise_escalation`, `runId` set:

**A step that asks the founder** — `procedureId` the step's, no subject. Title: the step's
`founderInputPurpose`, cut at the end of its first sentence when it runs past 200 characters. Body:
> {the step's founderInputPurpose, whole} Asked at {step code} {step name} of {slug}, run {run id}.
> Written so far: {entry codes, or "nothing yet"}. The ways to answer: {the options the step's
> instructions give, one line each}. Write your answer here; the next firing goes on from it. I
> would {the option the instructions recommend for this project, and why, in one sentence}.

No `observe_run` and no `pause_run`: the raise paused the run. Next firing: `get_run` shows the
step's founder's answer → resume, do the step from it, checkpoint it without `founderAnswer`; no
answer → the run stays paused.

**Blocked** — subject the run (`Runs`). Title: `Run {slug} is blocked at {step code} {step name}:
{what is missing}`. Body:
> Doing: {step code} {step name} of {slug}, run {run id}. Done so far: {steps checkpointed, by code};
> written: {entry codes}. Tried: {what}. Missing: {what, one line}. What the run can do without it:
> {one line}. Reply "skip this procedure" to go on without it, or "wait for the key" and fix it; the
> run is paused and resumes from the last checkpoint.

Then `observe_run` with the `inboxItemId` and `pause_run`. Next firing: "skip this procedure" →
resume and checkpoint the step with an empty list and the note "skipped on the person's answer";
"wait for the key" or nothing → the run stays paused.

**Verdict** — subject the run (`Runs`). Title: `Verdict on {slug}: proposed {Validated|Invalidated}`.
Body:
> The operation: {slug}. Written this run: {entry codes with their titles, hypotheses by code}. The
> pass sentence: "{validatedWhen}". How the rows meet it or do not: {two or three sentences, naming
> codes}. {On Invalidated: If not: {ifNot}. The project would go back to {revisit slugs, in order},
> right after this row.} Reply "validated" or "invalidated"; the next firing ends the run with your
> verdict.

Then `observe_run` with the `inboxItemId` and `pause_run`. Next firing: the brief's "answered since
the last run" carries the answer — the note, or the proposed verdict when acknowledged without one;
`start_run(operationId, resumedFromRunId)`, then `end_run(runId, outcome: "Completed", verdict, summary)`, with the
revisits the body named on `Invalidated`.

**Prerequisites** — subject the plan row (`RoadmapItems`). Title: `Run {slug}: {n} prerequisite(s)
have not run`. Body:
> {the refusal sentence, verbatim} Two ways forward: run {prerequisite slugs} first — they are rows
> {positions} of the plan — or force past them, which the run records for whoever reads it later.
> Reply "run {slugs} first" or "force past", or run them yourself; I pick this row up on the next
> firing.

No run was opened; nothing to pause and nothing to observe. Stop. Next firing: "force past" →
`start_run` with `forcePastPrerequisites: true`, which is the person's word; anything else →
`mentat-advance`'s next row.

## The report

One block, in this order: the row (position, slug, new status); the founder's answers, by step;
written (codes, grouped by block and, inside a segment or a value proposition, by owner);
hypotheses (codes, with the concern and the scores when Hypothesise ran) and the test orders still
waiting; experiments designed (codes); the verdict; the rows the project goes back to, in order, with
their positions; the gate line when it became due; the blockers, each in one line, then the question
whether any is worth an insight.

The tier names, the row statuses, the verdict rule per level and two worked runs are in
`mentat-canvas`'s `reference/roadmap-and-runs.md`; they are not repeated here.
