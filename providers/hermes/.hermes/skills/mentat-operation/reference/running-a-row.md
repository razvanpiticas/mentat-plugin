# Running a row, step by step

Load this when starting a run, when the level is Experiment, when the row is the mapping operation,
when the run is unattended, or when `start_run` says the agent already holds a run. The rules a run
has to follow unattended are in the skill body, not here; this file carries the detail of what each
part of the package is for, who writes what in the mapping operation, the exact escalation bodies,
what to do when the agent already holds a run, and the shape of the report.

## What the package carries, and where each part is used

| Package member | Used for |
| --- | --- |
| `definition.description`, `themeQuestion` | Read first; the framing every step is done under. The description opens with "Read first:" and the reads it names, each with its tool |
| `definition.descriptionPart`, `descriptionParts` | Which part of the description `description` holds, of how many. Above 1: read every other part, in order, with `get_operation_package` and `descriptionPart`, before the run starts; the parts joined are the description exactly, and the rest of the package is the same in each |
| `definition.summary.validationLevel` | Which ending: verdict, hand-over, or neither |
| `definition.validatedWhen`, `ifNot` | The Checkpoint judgement; the pass sentence the hand-over repeats; which operations an ending goes back to |
| `definition.procedures[]` (`stepId`, `procedureId`, `code`, `name`, `executionOrder`, `founderInputNeeded`, `founderInputPurpose`, `founderQuestion`) | The operation's own steps in short, in order; `procedureId` is what `checkpoint_run` names; a step's founder input says whether it asks the founder, why (`founderInputPurpose`, for you) and what, in the founder's own words (`founderQuestion`, what you ask). A step's instructions, book, version and explainer are not here: `get_procedure` reads them by its code when the step starts |
| `definition.explainer` (`text`, `webUrl`, `media[]`), `outputWebUrl` | The first message of a run in a session: what the operation is, the page where its video plays (`webUrl`), and where what the run writes is read (`outputWebUrl`). `media[]` says what each video or picture is: its `kind`, `alternativeText`, `caption`, and on a video its `durationInSeconds`. In short: a video's transcript is not here, the founder watches the video on `webUrl` and `get_operation_definition` answers it. Null when none is written |
| `definition.founderInput` (`need`, `stepCodes`) | The operation's founder input in short: the strongest a step asks (`Always`, then `WhenInstructionsSay`, then `None`) and the codes of the steps that ask |
| `revisits` | The operations this one may send the project back to, in order: the only ones `revisitOperationIds` may name |
| `handOver` | On an `Experiment` operation: the entry kinds it hands to hypotheses, in test orders — by rank, how many of each from the top (`EntryKind`), or the entries the run chose (`ChosenEntries`), named on the `chosenHypotheses` of the step that chooses them — and its instruction. The checkpoint of the last own step answers it resolved: the entries themselves, each one the run chose with its `signedOffSentence`. No lines: the checkpoint answers no test order, and Hypothesise works from the "validated when" sentence and the entries the run wrote |
| `sharedSteps` | On an `Experiment` operation: Hypothesise, the one step every such operation shares, in short with the `procedureId` its checkpoint names and the code `get_procedure` reads it by |
| `resolvedTargetSlots[]` (`blockCode`, `entryKindCode`, `entryKindStorage`, or the idea) | Where the steps write; an idea slot is written with the idea tools |
| `ideation` | The project's idea, when a slot targets it: the starting point, the candidates, the statement in force |
| `prerequisites[]` (`slug`, `kind`, `state`) | What `start_run` will check; a `Mandatory` one not `Met` is the refusal coming |
| `latestRun`, `resumableRun` | Resume, or start fresh. Each in short: its id, status and when it started; `get_run` reads the paused run whole before it is resumed |
| `buildCheck` (`standing`, `isDue`, `rowsLeftBeforeTheCheck`) | Where the build stands, and how many rows below the build are left before the build check; a build row (`T7`, `T8`) does not start while the build is held |

## The step loop

For each of the operation's own steps, in `executionOrder`:

1. `get_procedure` with the step's code and the `runId`, and read `procedure.instructions` whole,
   before the step's first read or write — the package lists the step without its text. When
   `instructionsParts` is above 1, read the other parts in order with `part` (and the `runId`); the
   parts joined are the instructions exactly. The `runId` is what writes each read into the run's
   log, before the step's checkpoint. When it answers a `procedure.explainer`, give the founder its
   `text` and `webUrl` as the step starts, in a session.
2. Read what the step reads: the target blocks (`get_block`), the segments, value propositions and
   products it names (`get_customer_segment`, `get_value_proposition` with the sections the step
   names, `get_product`, `get_entry`), the idea (`get_ideation`).
3. A step whose `founderInputNeeded` is `Always`, or `WhenInstructionsSay` when its instructions say
   so: ask before anything else — in the chat in a session, through the inbox unattended (the body
   below), and stop there until the answer comes.
4. Draft what the step produces, as the entries and parts it asks for, on the kinds the target slots
   name, with the fields the kind's schema lists (on every block only in a read of the kinds:
   `get_block` with `includeSchemas: true` in short, `kindCode` whole, read before the write). A step that produces a
   hypothesis drafts it in the "We believe that" form; a step that opens a question drafts the
   question.
5. In a session: show the draft, ask, write. Unattended: write.
6. Write through `mentat-canvas`, each call with `runId`; a nested kind with its owner's id as
   `ownerId`; keep the ids answered.
7. `checkpoint_run` with the step's `procedureId`, the ids, one line of note, the `stepOutcome` —
   one line of at most 160 characters saying what came of the step, its finding and numbers, never
   the action — the founder's words as `founderAnswer` when the step asked them in the chat, and
   `revisitOperationIds` when the step sends the project back. On the last step, the In short
   (below).

## The In short

The operation's item opens on its "In short": up to four boxes saying what the operation found, in
its own terms. The run writes them as `inShortReadings` on its last checkpoint or on `end_run` —
one to four, each a `label` (at most 40 characters, "Researched in full"), a `value` (at most 60,
"5 competitors") and an optional one-line `note` (at most 160, "2 more wait, each with why"),
each label once. Send `isWarning: true` on a box that says something still waits that should not
("Founder checks" / "None run yet"): the item draws its value in the warning ink. Draw them from what the run wrote and its numbers: the count of what it found, the
strongest one by name, what still waits; never the codes, never a sentence of the canvas. The
latest set sent replaces the item's set whole, so a later run of the row sends the set again when
what it found moved; leaving it out keeps the set. Without any, the item shows general readings.
Example, competitor-research: "Researched in full" / "5 competitors" / "2 more wait, each with
why"; "Strongest threat" / "Notion" / "High threat: same buyer, lower price".

A step whose text says to interview, to search the web, to compute from data the canvas does not hold:
do what can be done from what is in hand and say in the checkpoint note what was not; unattended,
that is a blocker (below) only when nothing at all can be produced.

Desk research: the tools in [research-tools.md](research-tools.md) may be at your disposal. Use them,
or better tools and techniques you know; they are options, never a required procedure. What they find
goes into the entries the step writes, each with its status and every quote or figure as a citation
on the statement it backs (the entry, or the fact inside a part), never into a note.

## The hand-over to `mentat-hypothesis` (the Hypothesise step)

The contract is `mentat-hypothesis`'s own (`reference/handoffs.md` there); hand over, in this order
and nothing else: the project id; this run's id and the canvas version you last held; the row (id,
slug, position); the Hypothesise step's code, from `sharedSteps`; the hand-over exactly as the last
own step's checkpoint answered it — "Hypothesise now", the "validated when" sentence, the operation's
own instruction, and every test order with its status and, when it is open, its entries by id and
code, each with the `signedOffSentence` it carries when the run chose it. A hand-over with no test order — the operation names no hand-over lines — adds two things: the
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

An `Experiment` operation's run designs no experiment after Hypothesise: the summary names the
hypotheses it wrote, and the mapping operation designs their tests later, from every hypothesis on
one map.

## The mapping operation

`hypotheses-mapping-and-experiments-design` is a `Checkpoint` operation whose steps hand most of their
writes to other skills. Its text, read with `get_procedure` step by step, is the method; this is who
writes what, in its run, with its `runId`:

- **`gather-the-hypotheses`** — the beliefs nobody wrote go to `mentat-hypothesis` ("from the mapping
  operation" in its own text), each after one `mentat-search` for a hypothesis that already says it.
  It writes the new ones, scores them with a reason each and hands back the ones it found already
  written: the checkpoint's note names those by code, and `producedHypothesisIds` the new ones.
- **`map-the-hypotheses`** — every open hypothesis scored with `update_hypothesis`: both scores, each
  with its reason in one line (`importanceReason`, `evidenceReason`), the checks first when they are
  not set. A hypothesis an experiment already tested keeps what the experiment showed: read its
  confidence (`get_hypothesis`), never score over it.
- **`choose-what-to-test`** asks the founder always (the skill's "Asking the founder"): the
  ranked hypotheses in plain sentences, each with why the business depends on it and what is known,
  then the ones proposed to wait, each with why, and your recommendation. In a session, ask and
  checkpoint with their answer; in a routine's run, the step's question goes to the inbox and the run
  pauses until they answer.
- **`design-the-experiments`** — invoke `mentat-experiment` with the hypotheses the founder's answer
  chose, the answer itself, this run's id and version and the "validated when" sentence. It designs
  only for those, and none for a hypothesis an experiment already tests — on a second row of the
  operation, what the first row designed stays as it is. It links each experiment to the goal it
  serves and hands back the ones that serve none, each with the objective it would serve. For each
  of those, say so by the experiment's code and propose the objective, as the skill's "Goals to
  review" says: in a session, ask first, then `propose_goal_change` with `kind: "NewObjective"` under
  the goal it would add to; unattended, no `propose_goal_change`: one "A goal to review" escalation
  per experiment, subject that goal (the body below), its `inboxItemId` recorded with `observe_run`,
  the run not paused, and the summary naming it. When every hypothesis the business depends on
  already holds evidence, no experiment is designed: the checkpoint's note says what the evidence is.
- **`plan-the-sequence`** — nothing is run or built in this operation. An experiment that spends
  money or needs the founder's hands stays `Designed`, its item in To do, for the founder to start on
  its page; no start ask of its own is raised and no run is paused for it. It waits for the founder
  in the inbox inside the verdict proposal, which names it (the body below). A free digital one —
  run mode `Digital`, cost rating 1, no spend on its card — the order says starts now is started
  through `mentat-experiment`. The summary lists the experiments in order, each with the hypotheses
  it tests, the goal it serves, its cost and who starts it; then the hypotheses that wait.

The verdict is the founder's, judged against the "validated when" sentence: in a session they confirm
it; unattended it is a verdict proposal. Nothing important left without evidence is `Validated` too.

## The mapping operation's hand-overs

`hypotheses-mapping-and-experiments-design` hands two of its steps to other skills, inside its run:

- **To `mentat-hypothesis`, at `gather-the-hypotheses`**: the project id, this run's id and the canvas
  version, the row (id, slug, position), and the beliefs the step found nobody wrote, each with its
  block and the entry it is about. It hands back the hypotheses written, by code and id, the ones it
  found already written, the reasons of the scores, the shortlists and the version. Checkpoint the
  step with the new ids as `producedHypothesisIds` and a note naming, by code, every hypothesis that
  already said a belief.
- **To `mentat-experiment`, at `design-the-experiments`**: the hypotheses the founder chose at
  `choose-what-to-test`, by code and id, the founder's answer, this run's id and the canvas version,
  and the "validated when" sentence. It reads `design-the-experiments` itself, designs the
  experiments — one may test several hypotheses, each with its own criterion — links each to the goal
  it serves and hands back each experiment's code and id, the hypotheses it tests, the goal it serves
  or that it serves none, and its cost; it starts none. Checkpoint the step with a note naming the
  experiments and the hypotheses each tests.

## The Checkpoint judgement

Read what the run wrote (`get_block` on the target blocks, `get_customer_segment`,
`get_value_proposition` by the sections it wrote and `get_product` on what it wrote inside, the codes
from the checkpoints). Against the
"validated when" sentence, write two or three sentences naming codes: what meets it, what does not.
Propose the verdict, and on `Invalidated` the operations it sends the project back to — the first of
`revisits` unless what failed says another. In a session the person confirms; unattended, the
escalation below.

## The escalations, unattended

Their rows — who raises, the subject, the body's order, the proposed answer — are in
`mentat-inbox`'s `reference/asks.md`: "A step that asks the founder", "A missing key, tool or access",
"Checkpoint verdict", "A goal to review", "A prerequisite refusal", "A build row while the build is
held". The bodies this skill writes, through `mentat-inbox`, `raise_escalation`, `runId` set:

**A step that asks the founder** — `procedureId` the step's, no subject. Title: the step's
`founderQuestion` (its `founderInputPurpose` when that is null), cut at the end of its first
sentence when it runs past 200 characters.
`answerHint`: one line of at most 160 characters saying what a good answer holds, from what the
step's instructions need to go on ("Which checks, on which competitors, and what each costs."), never
the question again. Body:
> {the step's founderQuestion, whole, else its founderInputPurpose} Asked at {step code} {step name} of {slug}, run {run id}.
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

**Verdict** — `proposedVerdict` the verdict you propose, subject the run (`Runs`). Title: `Verdict on
{slug}: proposed {Validated|Invalidated}`. Body:
> The operation: {slug}. Written this run: {entry codes with their titles, hypotheses by code}. The
> pass sentence: "{validatedWhen}". How the rows meet it or do not: {two or three sentences, naming
> codes}. {On Invalidated: If not: {ifNot}. The project would go back to {revisit slugs, in order},
> right after this row.} {On the mapping operation: Waiting for you to start them, each on its page:
> {experiment code, the hypotheses it tests, its cost and what you do yourself, one line each}.}
> Reply "validated" or "invalidated"; the next firing ends the run with your verdict.

No `observe_run` and no `pause_run`: the raise paused the run, and the row's item reads in review.
Hand back the `inboxItemId`. Next firing: the brief's "answered since the last run" carries the
answer — the note, or the proposed verdict when acknowledged without one;
`start_run(operationId, resumedFromRunId)`, then `end_run(runId, outcome: "Completed", verdict, summary)`, with the
revisits the body named on `Invalidated`.

**A goal to review** — subject the goal (`Goals`), one per goal the checkpoint answered. Title:
`{goal code} {goal title} is to review: {entry code} changed`. Body:
> {slug}, run {run id}, at {step code}, rewrote {entry code} {entry title}: {what it held before} →
> {what it holds now}. {goal code} rests on it; its target is {target} {unit} by {deadline}, its latest
> reading {value}. The change I would propose: {keep it, with why | change {figure} from {was} to
> {becomes}, with why | add the objective {title}, counted in {unit}, with why}. Reply with your
> answer; "stands as it is" on the goals page clears the mark.

An experiment of the mapping operation that serves no goal is the same ask, subject the goal the
objective would go under (the mission when no objective fits). Title: `{goal code} {goal title}:
{experiment code} serves no goal`. Body:
> {slug}, run {run id}, at design-the-experiments, designed {experiment code} for {hypothesis codes};
> no goal covers what it would show. The change I would propose: add the objective {title}, counted
> in {unit}, {target} by {deadline}, under {goal code}, with why. Reply with your answer.

Then `observe_run` with the `inboxItemId`, and go on with the next step: this ask does not pause the
run.

**Prerequisites** — subject the plan row (`RoadmapItems`). Title: `Run {slug}: {n} prerequisite(s)
have not run`. Body:
> {the refusal sentence, verbatim} Two ways forward: run {prerequisite slugs} first — they are rows
> {positions} of the plan — or force past them, which the run records for whoever reads it later.
> Reply "run {slugs} first" or "force past", or run them yourself; I pick this row up on the next
> firing.

No run was opened; nothing to pause and nothing to observe. Stop. Next firing: "force past" →
`start_run` with `forcePastPrerequisites: true`, which is the person's word; anything else →
`mentat-advance`'s next row.

**A build row while the build is held** — subject the plan row (`RoadmapItems`). Title: `Run {slug}:
the build is held`. Body:
> {the refusal sentence, verbatim} Two ways forward: run what the build check names and check the
> gates again, or force the start, which the run records as started while the build was held. Reply
> "force the start" or say what to run first.

No run was opened. Next firing: "force the start" → `start_run` with `forcePastTheBuildCheck: true`,
the person's word; anything else → `mentat-advance`'s next action.

## The agent already holds a run

`start_run` answering `CONFLICT` with "That name is already used in this organisation. Names are
unique here; choose another.": nothing was named, and `start_run` takes no name. It is the
one-running-run-per-agent refusal wearing the wrong sentence — the agent you booted as already holds
a run. `list_runs(portfolioId, projectId, status: "Running")` — both identifiers — and `get_run` it:
an attempt at this row is the run the agent opened for this work, so carry on inside it; a run on the
project itself with nothing written in it is closed with `end_run` and a one-line summary, and then
the attempt is opened. Never open a third and never abandon the row over it.

When it names none, the run is on a sibling project, because the agent is the portfolio's and not
this project's, and `list_runs` answers one project at a time: there is no call that finds it, and it
would not be yours to work in or to end if there were. Do not hunt for it project by project.
Unattended, that is one escalation through `mentat-inbox` naming the refusal verbatim, the row left
unworked and the proposal — end the agent's other run so the next firing takes the row — then stop;
in a session, say the same to the person.

## Going back

The package's `revisits` are the earlier operations this one may send the project back to, in order.
`revisitOperationIds` on `checkpoint_run` or `end_run` names the ones to run again, in the order to
run them, each one of that list; the server puts them on the plan right after this row, each marked
"Revisit, from <this slug>", ahead of every row that was next.

- **Mid-way**, on the checkpoint of the step whose instructions send the project back, and then go on
  with the steps: what comes after is still this run's work.
- **At the end**, on `end_run`: an `Invalidated` verdict that names none goes back to the first on
  the list; an `Experiment` run names the ones its "if not" sentence gives for what the run found; a
  `Validated` run names none — "A validated run sends the project back nowhere."
- One the list does not hold is refused: "customer-profile goes back only to beachhead-selection,
  end-user-profile or beachhead-tam; persona is not one of them." Send one the list holds, or none.

## The report

One block, in this order: the row (position, slug, new status) and its item (`W-` code, column);
the founder's answers, by step; written (codes, grouped by block and, inside a segment or a value
proposition, by owner); hypotheses (codes, with the concern and the scores with their reasons when
they were scored) and the test orders still waiting; experiments designed (codes, the hypotheses each
tests, the goal each serves); the goals sent to review, by code, and what was proposed for each; the
verdict; the rows the project goes back to, in order, with their positions; the build check line when
it became due; the blockers, each in one line, then the question whether any is worth an insight.

The tier names, the row statuses, the verdict rule per level and two worked runs are in
`mentat-canvas`'s `reference/roadmap-and-runs.md`; they are not repeated here.
