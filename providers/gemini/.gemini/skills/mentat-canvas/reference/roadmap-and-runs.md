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
| `T1` | Hypothesis | The hypotheses are written down |
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
| `Invalidated` | The last run completed with an `Invalidated` verdict; the operations it goes back to stand right after it |
| `Completed` | The last run completed on an operation whose validation level is `Experiment` or `None`, so the run carries no verdict |
| `Failed` | The last run could not be carried out |
| `Cancelled` | A person stopped the last run and chose not to resume it |
| `Skipped` | The project decided the row does not apply; the reason is on the row |

A skipped row cannot be run. Put it back with `include_roadmap_item` first.

## Validation level decides whether a verdict is allowed

Each operation definition carries one, and `get_operation_package` answers it before the run starts.
**Exactly one of the three levels takes a verdict**, and sending one where it does not belong is
refused as firmly as omitting one where it does:

| Validation level | What `end_run` takes on a completed run |
| --- | --- |
| `Checkpoint` | **A verdict is required**: `Validated` or `Invalidated`, judged against what the definition's "validated when" says |
| `Experiment` | **Send no verdict.** The hypothesis is tested by a real experiment, and the verdict lives on that experiment (`complete_experiment`), not on the run |
| `None` | **Send no verdict.** The operation produces work rather than a conclusion |

Of the forty-eight operations the method ships, thirty-two are `Checkpoint`, seven are `Experiment`
and nine are `None` — so most rows take a verdict and a good few do not. Read the level from the
package rather than assuming: a verdict on an `Experiment` or a `None` operation comes back
"A `<level>` operation carries no verdict on its run. Pass null.", the run stays `Running`, and a
missing verdict on a `Checkpoint` operation comes back "A checkpoint operation ends with a verdict".

`Invalidated` is a real conclusion: the thing the operation set out to establish did not hold.
BusinessIntelligence sends the project back: the operations the ending names, or the first on the
operation's list when it names none, go on the plan right after the row; report them. A run that could
not be carried out — no interviews could be booked, the tool was down — concluded nothing and is
`end_run` with `outcome: "Failed"` and the reason instead, at any level.

## Going back

`get_operation_package` answers `revisits`: the earlier operations this one may send the project back
to, in order. `revisitOperationIds` names the ones to run again, in the order to run them, on
`checkpoint_run` (mid-way, when a step says the work before it no longer holds) or on `end_run`. The
server inserts them right after the run's row, each with the row's tier and marked "Revisit, from
<slug>"; every later row moves down. A revisit off the list is refused — "customer-profile goes back
only to beachhead-selection, end-user-profile or beachhead-tam; persona is not one of them." — and so
is any revisit on a `Validated` ending: "A validated run sends the project back nowhere." A run of a
revisit row is opened with `revisitOfRunId`, the run that sent the project back.

## The founder's answers

Each step the package lists carries `founderInputNeeded` — `None`, `Always`, or `WhenInstructionsSay`,
when its instructions say when — and `founderInputPurpose`, the question in the founder's words. The
answer goes on the step's checkpoint as `founderAnswer`. An `Always` step checkpointed without one is
refused with its question: "Step choose-how-the-persona-is-made asks the founder: … Send their answer as founderAnswer." A
step that asks nothing refuses an answer. A run a routine fired asks in the inbox instead:
`raise_escalation` with the step's `procedureId` pauses the run, the person's answer becomes the step's
answer on the run, `get_run` shows it under the step, and the run that resumes carries it, so the step
is checkpointed without `founderAnswer`.

## The hand-over of an `Experiment` operation

The checkpoint of an `Experiment` operation's last own step answers the hand-over: "Hypothesise now",
the "validated when" sentence, the operation's instruction, and its test orders — each `Open` with the
entries to write hypotheses about, or waiting for an earlier order to hold. An order opens once every
entry of the orders before it has a validated hypothesis; `get_run` on the run answers the hand-over as
the canvas stands. An operation that names no hand-over lines answers "Hypothesise now" and the
"validated when" sentence with no test order: its hypotheses are the ones that sentence depends on,
about the entries the run wrote. The four steps every `Experiment` operation shares — Hypothesise, Design the test,
Learn from the evidence, Decide — are not among the operation's own steps: the package lists them in
`sharedSteps`, each with the `procedureId` its checkpoint names, and a run of any other level refuses
them.

## Prerequisites

An operation may wait on others, `Mandatory` or `Recommended`. `get_roadmap` and
`get_operation_package` answer each prerequisite with a state: whether it has run on this plan.
`start_run` refuses when a mandatory one has not. The refusal names them.

That refusal is not an error to work around. Report which operations it names and ask whether to run
them first or to start anyway; `forcePastPrerequisites: true` is recorded on the run for whoever reads
it later, which is why it is never yours to set.

## The checkpoint payload

`checkpoint_run` and `observe_run` take an optional `payload`: any JSON object. The
service stores it as sent and never reads it. Use it for what has no column — the interview's question
list, the search terms that worked, a count. Keep it small and keep it the same shape across a run, so
the person reading the run's events can follow it.

`writtenEntryIds` and `producedHypothesisIds` are not free-form: send the ids `add_entry` and
`create_hypothesis` answered, so the run points at the rows it actually produced. Send empty lists
rather than omitting them when a checkpoint produced none.

## A worked run, on an operation that takes no verdict

The plan's row `customer-profile` is `NotStarted` at `T2`, and the project stands at `T2`.

1. `get_operation_package(projectId, operationId)` → the definition with **validation level
   `Experiment`**, five own steps (`map-jobs`, `map-pains`, `map-gains`, `check-one-kind-of-people`,
   `rank-jobs-pains-and-gains`), target slots on `CS` (`CS.JOBS`, `CS.PAINS`, `CS.GAINS`, `CS.PROFILE`, `CS`),
   its revisits `beachhead-selection`, `end-user-profile`, `beachhead-tam`, its hand-over and the four
   shared steps, one mandatory prerequisite `beachhead-tam` in state `Met`, no resumable run.
2. The description's "Read first:" reads: `get_customer_segment` on the beachhead `CS-08` — its end
   user profile `CS.PROFILE-02` — and on the shortlisted segment it was split from; `get_ideation`.
3. `start_run(projectId, operationId, forcePastPrerequisites: false)` → a run, status `Running`.
4. Do `map-jobs`: `add_entry` with the `CS.JOBS` kind for each job, `ownerId` the profile's
   id. `checkpoint_run(runId, completedProcedureId: <the id of map-jobs>, writtenEntryIds:
   [those ids], producedHypothesisIds: [], note: "Four jobs from the five transcripts.")`.
5. Do `map-pains` from five interviews. `observe_run(runId, payload: {
   "interviews": 5 }, note: "Two people said the opposite of the third.")`, then `add_entry` with the
   `CS.PAINS` kind inside the profile and `checkpoint_run(runId, completedProcedureId: <the id of
   map-pains>, …)`.
6. Do `map-gains` the same way; `check-one-kind-of-people` and `rank-jobs-pains-and-gains` correct the
   profile and rank the items with `rerank_entries`. The checkpoint of `rank-jobs-pains-and-gains` answers the hand-over: "Hypothesise now", order 1 open with
   the top job `CS.JOBS-03`, order 2 waiting for order 1.
7. `mentat-hypothesis` writes the hypothesis about `CS.JOBS-03`; `checkpoint_run` on the Hypothesise
   step, its `procedureId` from `sharedSteps`, with the hypothesis's id in `producedHypothesisIds`.
8. `end_run(runId, outcome: "Completed", summary: "Five interviews with gym-goers. …")` — **no
   `verdict`**. The level is `Experiment`, so the run only records what was written; whether the top
   job holds is settled by an experiment, and sending a verdict here is refused.
9. Report the row's new status — `Completed` — the reference codes the entries answered, the
   hypothesis, and the order still waiting.

On a `Checkpoint` row the same loop ends differently: `persona` is one, so its last call is
`end_run(runId, outcome: "Completed", verdict: "Validated", summary: "…")`, judged against its "validated
when" sentence, and `"Invalidated"` there sends the project back to `customer-profile`, the first on its
list.

## A worked resume

A run was paused mid-way yesterday.

1. `get_roadmap(projectId)` → the row's status is `Paused` and its `latestRun` names the run.
2. `get_run(projectId, runId)` → the run, and in its attempt the steps already checkpointed, the
   founder's answers and what they wrote.
3. `get_operation_package(projectId, operationId)` → the same package, with `resumableRun` naming that
   run.
4. `start_run(projectId, operationId, forcePastPrerequisites: false, resumedFromRunId:
   runId)` → a new run that continues the old one.
5. Carry on from the first procedure the old run did **not** checkpoint. Do not repeat the ones it did;
   repeating a write is how a canvas ends up with duplicate entries.

## Moves

A move is an adopted idea given a place in the plan. Only an idea a person has adopted
(`resolve_idea` with `Adopted`) may become one, and adopting is their decision rather than yours. A
move has no run and no verdict: it is work to do. `remove_roadmap_move` takes the row off the plan and
leaves the idea on the canvas.
