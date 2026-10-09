---
name: mentat-canvas
description: Reads and writes a Mentat project's business model canvas through the Mentat MCP tools — entries on blocks, hypotheses, experiments, evidence, questions, risks, ideas, contradictions and project insights — and the mechanics of a project's roadmap and runs: the plan's rows, the run tools, the method gates and pivots. The low-level skill every other Mentat skill reads and writes the canvas through: it knows the write loop, the versions, the run id and the refusals, and nothing of the method. Use it for any read or write by code on a canvas or a plan, whoever asks for it.
---

# Mentat canvas

The canvas is a project's business model: twelve blocks, each holding entries of the kinds it
accepts, plus the hypotheses made about it, the experiments that test those hypotheses, the evidence
they produce, and the questions, risks and ideas pinned alongside. Every tool call is made as the signed-in
person inside their own organisation — the connection and sign-in are the `mentat` skill's job; this
skill is about what to write and how.

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

Here the run is handed to you by `mentat-agent` or by the skill that invoked you inside it, and every
write carries its id.

## The loop every write follows

1. **Find the project.** `list_portfolios`, then `list_projects` with the portfolio's id, or
   `get_project` when you hold the project id. `get_project` also answers the current canvas version.
2. **Read before you write.** `get_block` with the block's code answers the block's id, the kinds of
   entry it accepts (each with the `kindDefinitionId` and `storage` a write needs), everything already
   written on it, and the canvas version. Reading first is how you avoid writing a duplicate of an
   entry that already exists and how you learn which ids to address.
3. **Write with the version you read.** Every write takes `canvasVersion` and answers with the new
   one. Carry the answered version into the next write. Two writes in a row against the same version
   means the second is refused as stale — that is the canvas protecting itself from a decision made on
   old state, not a fault.
4. **Report what the tool answered**, by reference code (`CS-04`, `H-CS-002`, `T-3`, `E-7`), never
   what you intended. A refusal changed nothing unless its message says otherwise.

**`runId` on every write.** Every write tool on this server but `start_run` takes `runId`: the run
the call is being made from inside. When you are working inside a run, send the id `start_run` answered
on every write until that run ends, so the project's history records which session wrote each row. A
write without it is recorded as the person's own, not the agent's, so it is never left out. A write refused with a `CONFLICT` naming `runId` means the
run was ended, paused or opened by somebody else — nothing was written, every other call carrying it
will be refused the same way, and the fix is a new `start_run`, never a retry.

## Addressing

Reads take **codes**. A block's code comes from `get_canvas`, which lists every block with its code,
or from a refusal of `get_block`, which lists the codes that exist; never from memory, because a
project may hold custom blocks with codes of their own. Writes take **ids** a read answered:
`blockId`, `kindDefinitionId`, `entryId`, `hypothesisId`, `experimentId`, `evidenceId`, `metricId`,
`criterionId`. Never guess an id.

## Which kind, which shape

Every kind `get_block` lists carries its `schema`: the columns a write of that kind takes, each with
its type, whether it is required, what it points at and the words it accepts with what each word
means. Read the schema of the kind you are about to write; it is the only source of its columns and
words.

- A kind whose schema has no columns is free-form: `add_entry` takes `title` and `body` (markdown)
  only.
- A kind whose storage is `Custom` takes `attributes`: a JSON object keyed by the schema's column
  names.
- Every other kind takes `fields`: a JSON object whose `kind` property is the kind's `storage` name,
  exactly as `get_block` spells it, plus one property per schema column whose `writtenThrough` is
  `Fields`. A column whose `writtenThrough` is `OwnAction` or `ReadOnly` is never sent; its
  description says where it is written. A customer segment's `isBeachhead` is one: mark the market
  to win first with `choose_beachhead` (it takes the mark off whichever segment held it) and withdraw
  it with `clear_beachhead`.
- A `Pointer` column takes the id of an entry of the storage its `pointer.targetStorages` names, on
  the same block when `pointer.scope` is `SameBlock` and anywhere on the canvas when it is
  `SameCanvas`. Write the entry pointed at first; the document answers its reference code beside the
  id, under the member `pointer.referenceCodeMember` names.
- A `Choice` or `Rating` column takes one of its `words`, spelled exactly. A misspelt word is refused
  naming the column and the words that would have worked, so it costs a round trip, not a wrong row.

When the kind you want does not exist on the block, write to the block's free-form Notes kind and say
in the body what it is; do not force content into a kind it does not fit. Worked examples and the
statuses are in [reference/entries.md](reference/entries.md).

## Status says how true an entry is, so choose it honestly

`Confirmed` means evidence is in hand. `Reported` means someone said so. `Inferred` means you
reasoned it from other entries. `Hypothesis` means it is a guess to be tested. `Unknown` means the
slot is acknowledged and empty. An entry written from your own reasoning is `Inferred`, not
`Confirmed`; a gate later reads these statuses, so an inflated one misleads the person who
relies on the canvas.

## Hypotheses, experiments, evidence

A hypothesis is one testable sentence, "We believe that …", on a block, optionally about one entry,
with a polarity and the business concern it belongs to (Desirability, Feasibility or Viability). Score it
(importance and evidence, −5 to 5) and mark its quality checks with `update_hypothesis`; then
`list_experiment_definitions` narrowed by its concern shows the method cards that can test it, and
`recommend_experiment_definition` puts one on its shortlist. `design_experiment` plans a run in one
call: test card, metrics and criteria. Start it, record observations and judge criteria as the run
goes, `record_evidence` for each bundle gathered, then `complete_experiment` with the learning card
and a verdict that agrees with the judged criteria. The details, the order BusinessIntelligence
enforces, and worked examples are in [reference/hypotheses.md](reference/hypotheses.md) and
[reference/experiments-and-evidence.md](reference/experiments-and-evidence.md).

Two calls record a person's decision and never yours: `decide_hypothesis` (Persevere, Pivot, Kill,
ContinueTesting) and `rule_contradiction`. When a decision is needed, present the evidence and ask.
STOP and call the question tool to clarify.

## Questions, risks, ideas, contradictions

When you cannot write an entry with confidence because something is unknown, `add_question` on the
block rather than guessing. When you notice something that could go wrong, `add_risk` with likelihood
and severity. When you have a thought that is not yet an entry, `add_idea` with effort and impact.
When two entries cannot both be true, `record_contradiction`; a Critical one that nobody has ruled
on blocks the canvas gates, which is the point. Verbs and states are in
[reference/ledgers.md](reference/ledgers.md).

## The roadmap and operation runs

A project's **roadmap** is the plan it runs: forty-six operations from the method, in order, banded by
tier, plus the moves it has decided to make. Start with `get_roadmap` — it answers the thesis, every
row with its id, tier, status and prerequisites, the gate line above the plan, and the `roadmapVersion`
every change to the plan carries. **That version is not the canvas version.** Changing the plan and
writing on the canvas are two different aggregates with two different numbers; send each where it
belongs.

Changing the plan: `set_roadmap_thesis`, `place_roadmap_item` (one row to one position and one tier, in
one call — every row after it is renumbered), `skip_roadmap_item` with a reason and
`include_roadmap_item` to undo it, `annotate_roadmap_item`, `append_roadmap_operation`,
`add_roadmap_move` for an idea a person has adopted, `remove_roadmap_move`.
Each answers the whole plan with the version it produced; carry that into the next change.

**Running an operation** is `mentat-operation`'s loop; these are the calls it is made of:

1. `get_operation_package` with the row's id. It answers the method text as this project has it, the
   procedures in order with their ids, the places on the canvas the operation writes to, what it waits
   on, and any run left paused.
2. `start_run` with `operationId` set to the row. A refusal naming operations that have not run is the
   prerequisite check: report which ones and ask whether to run them first or to force past them.
   STOP and call the question tool to clarify. Never set `forcePastPrerequisites` on your own judgement. Keep the run id it
   answered: every write for the rest of the run carries it as `runId`.
3. Do the procedures. Write what they produce with the canvas tools above — `add_entry`,
   `create_hypothesis`, `add_question` — each carrying `runId`, and keep the ids those writes answered.
4. `checkpoint_run` after **each** procedure, naming the procedure and those ids. A run interrupted
   between checkpoints resumes from the last one; an uncheckpointed procedure is one somebody repeats.
   `observe_run` records what belongs to no procedure.
5. `end_run` with `outcome: "Completed"` and a summary written for whoever reads this project later,
   and a verdict **only when the package says the validation level is `Checkpoint`** — an `Experiment`
   operation is judged by its experiment and a `None` operation by nothing, and a verdict on either is
   refused. `Invalidated` appends the revisit the operation names to the end of the plan; say so when
   you report it. A run that could not be carried out at all is `end_run` with `outcome: "Failed"` and
   the reason, not an invalidated verdict.

Pause with `pause_run` and continue by starting the next run with `resumedFromRunId` set to the paused
one. `get_run` reads one run whole and `list_runs` — which takes the portfolio beside the project —
lists a venture's runs of every kind with the summary each left. `cancel_run` abandons a run nobody intends to
finish and is a person's call, not yours.

A run is opened on other subjects too — an experiment, a job off the board, or the project itself —
by sending `experimentId`, `workItemId` or none of the three instead of `operationId`. Only an attempt
at a row of the plan checkpoints; every other shape of run records its progress with `observe_run`.

**Gates and pivots have a skill of their own.** `get_gate_status` answers what the gate would say now
— the question, every checked block with the confidence it holds against the bar it must clear, and
whether it would pass — and records nothing. `evaluate_gate` records an evaluation for life and moves
the project as the outcome and its boundary policy say; `apply_pivot` sends it back a tier and appends
the rows the pivot revisits. The `mentat-gate-check` skill is what calls them: in a session on the
person's word, from a routine on the rule that skill carries. From here, present the numbers and hand
over; never evaluate or pivot on your own judgement.

The exact tier names, row statuses, the verdict rule per validation level, the checkpoint payload
convention and two worked runs are in [reference/roadmap-and-runs.md](reference/roadmap-and-runs.md).

## Reading a refusal

A refused call names its code, what happened, the arguments at fault, and what to do next. Do what
the recovery says rather than resending.

- `INVALID_ARGUMENT` — fix the named arguments. Nothing was read or changed.
- `NOT_FOUND` — an id or code named nothing; read again rather than guessing another.
- `CONFLICT` naming a canvas version — the canvas moved: re-read what you are changing with
  `get_block`, then resend with the version the refusal names.
- `CONFLICT` naming a roadmap version — somebody else changed the plan: re-read it with `get_roadmap`,
  check that the row you meant is still where you thought, then resend with the version the refusal
  names. Row positions may have moved.
- `CONFLICT` naming `runId` — the run you are working inside cannot be worked any more. Nothing was
  written. Open a new one with `start_run` and send the write again; do not resend it unchanged.
- `CONFLICT` quoting a rule of the business model — the call is not allowed in the canvas's current
  state (a run with no criterion cannot start, a scored hypothesis cannot be reworded). Change what you
  asked for.
- `SERVER_MISCONFIGURED` or `UPSTREAM_UNAVAILABLE` — not about your call; follow the `mentat` skill's
  troubleshooting.

Composed writes (`design_experiment`, `record_evidence`) may be refused part-way: the row created
before the refused part exists. The message says so; read it back with `get_experiment` and finish
with `add_metric`, `add_criterion`, `update_evidence` or `add_data_point` rather than designing again.

## Rules that are easy to get wrong

- Never invent a tenant, a portfolio or a project. Lists answer what exists.
- `get_canvas` is the whole model and large; reach for `get_block` unless the task spans blocks.
- Retire what was once believed (`retire_entry`, `retire_hypothesis`); delete only what should never
  have existed. Deletes cascade to hypotheses and contradictions about the entry.
- One `update_*` call per changed thing; each takes the version the previous answered.
- Changes to the plan take `roadmapVersion`; writes on the canvas take `canvasVersion`; writes to a
  run take neither. Sending the wrong one is refused, not silently accepted.
- `runId` is not a version and is not one of those two: it says which run a write was made from
  inside, and it is sent on every write of a run including the canvas ones.
- Do not cache what a read answered across turns: another person in the same organisation may have
  written since.
