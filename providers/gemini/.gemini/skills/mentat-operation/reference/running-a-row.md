# Running a row, step by step

Load this when starting a run, when the level is Experiment, or when the run is unattended. The
rules a run has to follow unattended are in the skill body, not here; this file carries the detail
of what each part of the package is for, the exact escalation bodies, and the shape of the report.

## What the package carries, and where each part is used

| Package member | Used for |
| --- | --- |
| `definition.description`, `themeQuestion` | Read first; the framing every step is done under |
| `definition.validationLevel` | Which ending: verdict, hand-off, or neither |
| `definition.validatedWhen`, `ifNot`, `ifNotRevisitSlug` | The Checkpoint judgement; the context sentence handed to the hypothesis skill; what an Invalidated verdict appends |
| `definition.procedures[]` (`procedureId`, `code`, `name`, `instructions`, `executionOrder`, `source`) | The steps, in order; `procedureId` is what `checkpoint_run` names |
| `resolvedTargetSlots[]` (`blockCode`, `entryKindCode`, `entryKindStorage`) | Where the steps write; the block codes handed to the hypothesis skill |
| `prerequisites[]` (`slug`, `kind`, `state`) | What `start_run` will check; a `Mandatory` one not `Met` is the refusal coming |
| `latestRun`, `resumableRun` | Resume, or start fresh |
| `gate.isDue`, `gate.pendingSlugs` | Whether this row finishes the tier |

## The step loop

For each step, in `executionOrder`:

1. Read `instructions` whole.
2. Read what the step reads: the target blocks (`get_block`), and any block the text names.
3. Draft what the step produces, as the entries it asks for, on the kinds the target slots name, with
   the fields the kind's schema lists (`get_block` carries it). A step that produces a hypothesis
   drafts it in the "We believe that" form; a step that opens a question drafts the question.
4. In a session: show the draft, ask, write. Unattended: write.
5. Write through `mentat-canvas`, each call with `runId`; keep the ids answered.
6. `checkpoint_run` with the step's `procedureId`, the ids, and one line of note.

A step whose text says to interview, to search the web, to compute from data the canvas does not hold:
do what can be done from what is in hand and say in the checkpoint note what was not; unattended,
that is a blocker (below) only when nothing at all can be produced.

## The handoff to `mentat-hypothesis` (step `P.V1`)

The contract is `mentat-hypothesis`'s own (`reference/handoffs.md` there); hand over, in this order
and nothing else: the project id; this run's id and the canvas version you last held; the row (id,
slug, position); `validatedWhen`; the resolved target slots, block code and kind code each; the theme
question; the reference codes of the entries this run wrote. It hands back the new hypotheses' codes
and ids, the ones left alone, a shortlist per hypothesis, and the canvas version. Checkpoint `P.V1`
with the new ids as `producedHypothesisIds`.

## Step `P.V2` — who hands to `mentat-experiment`

**In a session**, `mentat-hypothesis` does, on the person's word, inside this run. When it hands
back, `get_hypothesis` on each new hypothesis: an experiment designed in this run (its `runId` is
this run's) means `P.V2` is done — checkpoint it with the note "Designed {experiment codes} on
{hypothesis codes}"; none means checkpoint nothing for `P.V2` and say in the summary that the design
waits.

**Unattended**, you do, and the card is the shortlist's — the skill body's rule: a hypothesis on the
row's target blocks that stood shortlisted before this run first, else the one `mentat-hypothesis`
ranked first; its card is the first of its `recommendedDefinitionIds`, in order, the next on the same
list when one does not fit, and a card chosen now only when the list is empty or none fits. Hand
over, as `mentat-hypothesis` would: that hypothesis (id, code), that card (id, slug), this run's id
and the canvas version, the "validated when" sentence. It hands back the experiment id and one of two
states. *Started* — the card was `Digital`, cost rating 1, no spend named: checkpoint `P.V2` with
"Designed {experiment code} on {hypothesis code}; started", then end the run. *Paused* — it raised
"A start that spends, or needs the person's hands" through `mentat-inbox` and paused the run: write
nothing more; the report is the run's events, and the skill's own "Resuming" section says how the
next firing finishes `P.V2`.

## The Checkpoint judgement

Read the entries the run wrote (`get_block` on the target blocks, the codes from the checkpoints).
Against the "validated when" sentence, write two or three sentences naming codes: what meets it, what
does not. Propose the verdict. In a session the person confirms; unattended, the escalation below.

## The three escalations, unattended

Their rows — who raises, the subject, the body's order, the proposed answer — are in
`mentat-inbox`'s `reference/asks.md`: "A missing key, tool or access", "Checkpoint verdict", "A
prerequisite refusal". The bodies this skill writes, through `mentat-inbox`, `raise_escalation`,
`runId` set:

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
> codes}. {On Invalidated: If not: {ifNot}. The plan would gain {revisit slug} at its end.} Reply
> "validated" or "invalidated"; the next firing ends the run with your verdict.

Then `observe_run` with the `inboxItemId` and `pause_run`. Next firing: the brief's "answered since
the last run" carries the answer — the note, or the proposed verdict when acknowledged without one;
`start_run(operationId, resumedFromRunId)`, then `end_run(Completed, verdict, summary)`.

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

One block, in this order: the row (position, slug, new status); written (codes, grouped by block);
hypotheses (codes, with the concern and the scores when `P.V1` ran); experiments designed (codes);
the verdict and, on Invalidated, the revisit row appended (slug, position); the gate line when it
became due; the blockers, each in one line, then the question whether any is worth an insight.

The tier names, the row statuses, the verdict rule per level and two worked runs are in
`mentat-canvas`'s `reference/roadmap-and-runs.md`; they are not repeated here.
