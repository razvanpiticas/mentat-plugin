# The next-action rule

The one copy. The front door says the outcome; `mentat-advance` does it. Read from the top; the first rule that applies is the action. Fields are the brief's `project` section (or `get_project_state`), `get_roadmap` and `get_gate_status`.

| # | Condition | Read from | Action | Owner | Unattended, what lands in the inbox |
| --- | --- | --- | --- | --- | --- |
| 1 | The project is not `Active` | `status` | Stop. `AwaitingUnlock`: "the gate to <next tier> passed and waits for your unlock on the project screen"; `Killed` or `Archived`: "nothing runs on this project" | — | nothing; the digest says it |
| 2 | A `Paused` run exists — a `Paused` operation row in the plan, or `list_runs(status: Paused)` answers one — whose ask is answered (`resolvedSinceLastRun` has the line the run's events name) | `roadmap.next.status`, `list_runs`, `get_run` events, `resolvedSinceLastRun` | Resume it from the answer: a verdict goes on `end_run`; a start that was answered "start it" is `start_experiment`, through `mentat-experiment` | `mentat-operation` for an attempt at a row (and through it `mentat-experiment` for a start ask); `mentat-experiment` for an experiment run of its own | what the owner raises next, if anything |
| 2a | A `Paused` run exists whose ask is **not** answered | the same, no line | Stop: one `Waiting:` line. *(Flip for O-2: skip to the next outstanding row after it.)* | — | nothing new |
| 3 | The last evaluation of the gate failed and its pivot ask was answered "apply <pivot> to <tier>" | `get_gate_status.latestEvaluation.passed` false; `resolvedSinceLastRun` line with that note | Apply the pivot with exactly the values proposed | `mentat-gate-check` | the plan's new tail in the digest |
| 4 | `hasThesis` is false | `roadmap.hasThesis` | The plan is missing | `mentat-planner` — one escalation, nothing written; the caller does not pause | "The plan is missing or finished — run the planner with me" |
| 5 | The gate is due and has not been evaluated since the tier's work last moved — `gatesAwaitingPerson` is not empty, or `get_gate_status.roadmap.isDue`, and there is no `latestEvaluation` from this tier or it is older than the last `endedAtUtc` among the tier's rows | `roadmap.gatesAwaitingPerson`, `isDue`, `latestEvaluation.fromTier` and `evaluatedAtUtc`, the rows' `operation.latestRun.endedAtUtc` | Evaluate it | `mentat-gate-check` | on a pass under a manual policy: nothing, the project reads `AwaitingUnlock`; on a fail: "The <gate> gate failed: …" with the proposed pivot |
| 6 | The gate would pass and rows of the tier are pending, and no ask about it is waiting or answered "wait" | `wouldPass` true, `isDue` false, `pendingOperationsInTier` > 0; the last digest's `Waiting:` and `Decided:` lines | Ask whether to evaluate now; answered "evaluate now" → evaluate | `mentat-gate-check` | "The <gate> gate would pass with <n> row(s) still pending" |
| 7 | The last evaluation from this tier failed and no row of the tier finished since it | `latestEvaluation.passed` false, `nextOperationSlug` set, `evaluatedAtUtc` later than every row's `endedAtUtc` | Its pivot ask unanswered: stop, one `Waiting:` line. Answered "no pivot", or acknowledged with no note: run the row `nextOperationSlug` names again | `mentat-operation` on the row whose slug is `nextOperationSlug` | what the operation raises |
| 8 | An outstanding row exists (the first, by position, of either kind — the definition below) and it is an operation | `get_roadmap` rows | Run it | `mentat-operation` | a blocker, a Checkpoint verdict, or — on an Experiment-level row — a start that spends or needs the person's hands, with the card and the cost, when it hits one |
| 9 | The first outstanding row is a move | `get_roadmap` rows, `move.ideaStatus`, `move.workItemId`, the job's status | Write its job, or work it, or stop on it — the "A move" section of the skill | `mentat-advance` itself, `mentat-hypothesis` and `mentat-canvas` for the idea's rows | the job's approval request when one was written |
| 10 | No row is outstanding and no rule above applied — every row is finished or skipped: the plan is finished | `get_roadmap` rows | The plan is finished; the person hears it once. Its ask not yet raised: the planner raises it. Raised and unanswered (a `Waiting:` line for it): nothing new, the line repeated "still waiting". Then hand back, and the Heartbeat's step 5 (the goal loop, `mentat-agent`) proposes one job if a goal has nothing serving it | `mentat-planner` — one escalation, nothing written; the caller does not pause | "The plan is missing or finished — run the planner with me", once; the goal loop's proposal, when one is made |

Rules 4 and 10 are one ask: no thesis and no row left are both a plan the person has to make again with the planner, and the Heartbeat's own text says the same. A finished plan is never silence — it is the one escalation, raised once. Rule 6 outranks 7, 8 and 9 on purpose: evaluating early is a trade the person makes. Rule 5 outranks the rows because a due gate means the tier's rows are done. Rule 5 asks for a fresh evaluation only when the tier's work moved after the last one: a gate that failed stays due until its row is run again, and evaluating it again unchanged would count one more failure towards the kill criteria for nothing.

## The outstanding row, of either kind

`get_roadmap` rows in `position` order, `isSkipped` rows excluded:

- an **operation** row is outstanding when `operation.status` is `NotStarted`, `Paused`, `Failed` or `Cancelled` — the server's own definition, the one `roadmap.next` uses;
- a **move** row is outstanding when `move.ideaStatus` is `Adopted` and either `move.workItemId` is null or the job it names (`get_work_item`) is not `InReview`, `Done` or `Cancelled`.

The first outstanding row is the next row. When no move stands before the next outstanding operation, it is `roadmap.next`.

## The memory between firings

`lastRun.summary` is the previous firing's digest (the agent's last ended run; the firing's run is ended last). Its `Waiting:` lines, each with an inbox row id, are every ask that firing left open. An id with a line in `resolvedSinceLastRun` is answered; one without is still open and is never raised again — the digest repeats it. Its `Decided:` lines are answers still in force that the brief no longer carries — "wait" on an early gate — and are repeated while they hold. This is the only memory the rule has; there is no list of the inbox. When `lastRun.summary` carries no `Waiting:` block — the last firing broke off and the service ended its run as abandoned — the paused runs' own events are the memory, and an ask raised by no paused run is not raised again in this firing: the digest says the memory was lost.

## Reading an answer

`mentat-inbox`'s rule: a `resolutionNote` is the answer; `Acknowledged` with no note means the proposed answer; no line means unanswered. The one exception is the pivot: applied only on a note saying "apply <pivot> to <tier>", never on a bare acknowledgement.

## The short form the front door says

Six lines, in this order, the first that applies: (1) not active → the unlock or nothing; (2) a paused run answered → resume it; unanswered → wait; (3) no thesis → the planner; (4) the gate due, or would pass with rows pending → the gate check; (5) an outstanding row → run it, or write the move's job; (6) no row left → the plan is finished: the planner, and the heartbeat's goal loop proposes one job when a goal has nothing serving it.
