# The roadmap, its rows and the runs of its operations

What the tool schemas cannot say: the exact words each vocabulary accepts, the rules
BusinessIntelligence enforces about the order of a run, and two worked examples. The skill body has the
loop; this file has the detail you need before the first run of a session.

## Tiers

Every row of a plan sits in a tier band, and the project itself stands on one. The nine names are
exact — send the member name, not the label:

| Name | What it is called | What the project is doing there |
| --- | --- | --- |
| `T0` | Ideation | An idea, nothing tested |
| `T1` | Hypothesis | The claims are written down |
| `T2` | Discovery | Talking to people: segmentation, beachhead, persona, jobs, pains, gains, market size |
| `T3` | Design | Shaping the offer: value proposition, competitive position |
| `T4` | Go to market | Life cycle, decision-making unit, channels, revenue, unit economics; the first real spend |
| `T5` | Pre-launch | Growth engine, relationships, resources, partnerships, environment, follow-on markets |
| `T6` | Validation | Minimum viable product, market test, pivot or persevere |
| `T7` | Build | Canvas validated, specification generated, the product developed |
| `T8` | Live and scaling | The business runs, and distribution with it |

A gate stands at each boundary. `get_roadmap` answers the project's tier and the gate line; the bands
below it are where the rows sit, which is not the same thing — a plan may hold rows for tiers the
project has not reached.

## Row statuses

`get_roadmap` answers a status on every operation row.

| Status | What it means |
| --- | --- |
| `NotStarted` | No run of this row has ever begun |
| `Running` | A run is open now |
| `Paused` | A run was paused and can be resumed |
| `Validated` | The last run completed with a `Validated` verdict |
| `Invalidated` | The last run completed with an `Invalidated` verdict; the revisit it names has been appended to the plan |
| `Completed` | The last run completed on an operation whose validation level is `Experiment` or `None`, so the run carries no verdict |
| `Failed` | The last run could not be carried out |
| `Cancelled` | A person stopped the last run and chose not to resume it |
| `Skipped` | The project decided the row does not apply; the reason is on the row |

A skipped row cannot be run. Put it back with `include_roadmap_item` first.

## Validation level decides whether a verdict is allowed

Each operation definition carries one, and `get_operation_package` answers it before the run starts.
**Exactly one of the three levels takes a verdict**, and sending one where it does not belong is
refused as firmly as omitting one where it does:

| Validation level | What `complete_operation_run` takes |
| --- | --- |
| `Checkpoint` | **A verdict is required**: `Validated` or `Invalidated`, judged against what the definition's "validated when" says |
| `Experiment` | **Send no verdict.** The claim is tested by a real experiment, and the verdict lives on that experiment (`complete_experiment`), not on the run |
| `None` | **Send no verdict.** The operation produces work rather than a conclusion |

Of the forty-six operations the method ships, twenty-eight are `Checkpoint`, seven are `Experiment`
and eleven are `None` — so most rows take a verdict and a good few do not. Read the level from the
package rather than assuming: a verdict on an `Experiment` or a `None` operation comes back
"A `<level>` operation carries no verdict on its run. Pass null.", the run stays `Running`, and a
missing verdict on a `Checkpoint` operation comes back "A checkpoint operation ends with a verdict".

`Invalidated` is a real conclusion: the thing the operation set out to establish did not hold.
BusinessIntelligence appends the revisit that operation names to the end of the plan, so report the new
row. A run that could not be carried out — no interviews could be booked, the tool was down — concluded
nothing and is `fail_operation_run` instead, at any level.

## Prerequisites

An operation may wait on others, `Mandatory` or `Recommended`. `get_roadmap` and
`get_operation_package` answer each prerequisite with a state: whether it has run on this plan.
`start_operation_run` refuses when a mandatory one has not. The refusal names them.

That refusal is not an error to work around. Report which operations it names and ask whether to run
them first or to start anyway; `forcePastPrerequisites: true` is recorded on the run for whoever reads
it later, which is why it is never yours to set.

## The checkpoint payload

`checkpoint_operation_run` and `observe_operation_run` take an optional `payload`: any JSON object. The
service stores it as sent and never reads it. Use it for what has no column — the interview's question
list, the search terms that worked, a count. Keep it small and keep it the same shape across a run, so
the person reading the run's events can follow it.

`writtenEntryIds` and `producedHypothesisIds` are not free-form: send the ids `add_entry` and
`create_hypothesis` answered, so the run points at the rows it actually produced. Send empty lists
rather than omitting them when a checkpoint produced none.

## A worked run, on an operation that takes no verdict

The plan's row `customer-jobs-pains-gains` is `NotStarted` at `T2`, and the project stands at `T2`.

1. `get_operation_package(projectId, operationId)` → the definition with **validation level
   `Experiment`**, three procedures (`P1.6` customer jobs, `P1.7` customer pains, `P1.8` customer
   gains), three target slots on `CS` (`CS.JOBS`, `CS.PAINS`, `CS.GAINS`), one mandatory prerequisite
   `persona` in state `Met`, no resumable run.
2. `start_operation_run(projectId, operationId, forcePastPrerequisites: false)` → a run, status
   `Running`.
3. Do `P1.6`: `add_entry` on `CS` with the `CustomerJob` kind for each job, carrying the segment's id.
   `checkpoint_operation_run(runId, completedProcedureId: <the id of P1.6>, writtenEntryIds: [those
   ids], producedHypothesisIds: [], note: "Four jobs from the five transcripts.")`.
4. Do `P1.7`, the pains, from five interviews. `observe_operation_run(runId, payload: { "interviews":
   5 }, note: "Two people said the opposite of the third.")`, then `add_entry` with the `CustomerPain`
   kind and `checkpoint_operation_run(runId, completedProcedureId: <the id of P1.7>, …)`.
5. Do `P1.8`, the gains, the same way with the `CustomerGain` kind.
6. `complete_operation_run(runId, summary: "Five interviews with gym-goers. …")` — **no `verdict`**.
   The level is `Experiment`, so the run only records what was written; whether the top three pains
   hold is settled by an experiment, and sending a verdict here is refused.
7. Report the row's new status — `Completed` — and the reference codes the entries answered, then
   offer the claim the level asks for: `create_hypothesis` on the strongest pain and an experiment to
   test it.

On a `Checkpoint` row the same loop ends differently: `persona` is one, so its last call is
`complete_operation_run(runId, verdict: "Validated", summary: "…")`, judged against its "validated
when" sentence, and `"Invalidated"` there appends its revisit to the plan.

## A worked resume

A run was paused mid-way yesterday.

1. `get_roadmap(projectId)` → the row's status is `Paused` and its `latestRun` names the run.
2. `get_operation_run(projectId, runId)` → the procedures already checkpointed and what they wrote.
3. `get_operation_package(projectId, operationId)` → the same package, with `resumableRun` naming that
   run.
4. `start_operation_run(projectId, operationId, forcePastPrerequisites: false, resumedFromRunId:
   runId)` → a new run that continues the old one.
5. Carry on from the first procedure the old run did **not** checkpoint. Do not repeat the ones it did;
   repeating a write is how a canvas ends up with duplicate entries.

## Moves

A move is an adopted idea given a place in the plan. Only an idea a person has adopted
(`resolve_idea` with `Adopted`) may become one, and adopting is their decision rather than yours. A
move has no run and no verdict: it is work to do. `remove_roadmap_move` takes the row off the plan and
leaves the idea on the canvas.
