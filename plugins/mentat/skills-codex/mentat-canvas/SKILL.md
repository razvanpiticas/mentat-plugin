---
name: mentat-canvas
description: Reads and writes a Mentat project's business model canvas through the Mentat MCP tools — entries on blocks, hypotheses (claims), experiments, evidence, questions, risks, ideas, contradictions and project insights. Use this whenever a task touches a project's canvas in any way, even if the user never says the word canvas — filling Customer Segments, writing a value proposition, proposing or scoring a hypothesis, designing or completing an experiment, recording interview evidence, logging a question or risk, or reading what a project already holds. It also drives a project's roadmap: reading the plan, ordering and skipping its rows, running an operation end to end, and the method gates and pivots above it. Every method skill writes through this one; the operation skill (mentat-operation, coming next) runs a roadmap operation end to end with the tools below.
---

# Mentat canvas

The canvas is a project's business model: twelve blocks, each holding entries of the kinds it
accepts, plus the claims made about it, the experiments that test those claims, the evidence they
produce, and the questions, risks and ideas pinned alongside. Every tool call is made as the signed-in
person inside their own organisation — the connection and sign-in are the `mentat` skill's job; this
skill is about what to write and how.

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

## Addressing

Reads take **codes**: block codes are `CS` Customer Segments, `VP` Value Proposition, `CH` Channels,
`CR` Customer Relationships, `RS` Revenue Streams, `KR` Key Resources, `KA` Key Activities, `KP` Key
Partners, `CO` Cost Structure, `CL` Competitive Landscape, `MM` Market Mechanics, `ID` Innovation and
Dislocation; a tenant may have custom blocks with codes of their own. Writes take **ids** a read
answered: `blockId`, `kindDefinitionId`, `entryId`, `hypothesisId`, `experimentId`, `evidenceId`,
`metricId`, `criterionId`. Never guess an id.

## Which kind, which shape

A block's kinds come in three storages, and the storage decides what `add_entry` takes:

| Storage | What to send | Example kinds |
| --- | --- | --- |
| `FreeForm` | `title`, `body` (markdown) | every block's Notes kind; `VP.DREAM`, `VP.ENH`, `VP.NAME` |
| a typed storage (`CustomerSegment`, `CustomerJob`, `CustomerPain`, `CustomerGain`, `ValueProposition`, `ProductService`, `PainReliever`, `GainCreator`, `Channel`, `CustomerRelationship`, `RevenueStream`, `KeyResource`, `KeyActivity`, `KeyPartner`, `CostItem`) | `title`, `body`, and `fields`: a JSON object whose `kind` is that storage name plus the columns for it | `CS`, `CS.JOBS`, `CS.PAINS`, `VP.PS`, `CH`, `RS`, `KR`, `KA`, `KP`, `CO` |
| `Custom` | `title`, `body`, and `attributes`: the fields the kind declares | `CL.POS`, `CL.ENV`, `CL.FOL` |

The columns of every typed storage, with the exact words their enum-valued columns accept, are in
[reference/entries.md](reference/entries.md). Read it before your first typed write in a session; a
misspelt column value is refused naming the column, so it costs a round trip, not a wrong row.

Jobs, pains and gains belong to a segment: write the `CustomerSegment` entry first and put its id in
`segmentId`. Products, pain relievers and gain creators belong to a value proposition the same way.
When the kind you want does not exist on the block, write to the block's Notes kind and say in the
body what it is; do not force content into a kind it does not fit.

## Status is a claim about truth, so choose it honestly

`Confirmed` means evidence is in hand. `Reported` means someone said so. `Inferred` means you
reasoned it from other entries. `Hypothesis` means it is a guess to be tested. `Unknown` means the
slot is acknowledged and empty. An entry written from your own reasoning is `Inferred`, not
`Confirmed`; a gate later reads these statuses, so an inflated one misleads the person who
relies on the canvas.

## Claims, experiments, evidence

A claim is one testable sentence, "We believe that …", on a block, optionally about one entry, with a
polarity and the business concern it belongs to (Desirability, Feasibility or Viability). Score it
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
STOP and use Codex's structured user-input tool when available; if it is unavailable, ask directly in chat to clarify.

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

**Running an operation** is the loop this skill exists for:

1. `get_operation_package` with the row's id. It answers the method text as this project has it, the
   procedures in order with their ids, the places on the canvas the operation writes to, what it waits
   on, and any run left paused.
2. `start_operation_run`. A refusal naming operations that have not run is the prerequisite check:
   report which ones and ask whether to run them first or to force past them.
   STOP and use Codex's structured user-input tool when available; if it is unavailable, ask directly in chat to clarify. Never set `forcePastPrerequisites` on your own judgement.
3. Do the procedures. Write what they produce with the canvas tools above — `add_entry`,
   `create_hypothesis`, `add_question` — and keep the ids those writes answered.
4. `checkpoint_operation_run` after **each** procedure, naming the procedure and those ids. A run
   interrupted between checkpoints resumes from the last one; an uncheckpointed procedure is one
   somebody repeats. `observe_operation_run` records what belongs to no procedure.
5. `complete_operation_run` with a summary written for whoever reads this project later, and a verdict
   **only when the package says the validation level is `Checkpoint`** — an `Experiment` operation is
   judged by its experiment and a `None` operation by nothing, and a verdict on either is refused.
   `Invalidated` appends the revisit the operation names to the end of the plan; say so when you report
   it. A run that could not be carried out at all is `fail_operation_run`, not an invalidated verdict.

Pause with `pause_operation_run` and continue by starting the next run with `resumedFromRunId` set to
the paused one. `get_operation_run` and `list_operation_runs` read what has been done.

**Gates and pivots are a person's call, never yours.** `get_gate_status` answers what the gate would
say now — the question, every block with the confidence it holds against the bar it must clear, and
whether it would pass — and records nothing. Present those numbers, then `evaluate_gate` only once the
person has decided; a passing evaluation moves the project up a tier. When a gate fails, read
`list_pivot_definitions`, show what each kind of pivot would change and which operations it puts back
on the plan, and call `apply_pivot` only on their instruction. STOP and use Codex's structured user-input tool when available; if it is unavailable, ask directly in chat to clarify.

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
- `CONFLICT` quoting a rule of the business model — the call is not allowed in the canvas's current
  state (a run with no criterion cannot start, a scored claim cannot be reworded). Change what you
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
  have existed. Deletes cascade to claims and contradictions about the entry.
- One `update_*` call per changed thing; each takes the version the previous answered.
- Changes to the plan take `roadmapVersion`; writes on the canvas take `canvasVersion`; writes to an
  operation run take neither. Sending the wrong one is refused, not silently accepted.
- Do not cache what a read answered across turns: another person in the same organisation may have
  written since.
