# The asks a run raises under the unattended rule

Each is one escalation, raised by the skill that is at the point, with `runId`, followed by
pause_run in that skill — except a step's question to the founder, which carries the step's
`procedureId`, and a verdict proposal, which carries `proposedVerdict`: each of those two pauses the
run itself. A goal to review and a failed build check pause nothing. The body's last sentence is always
the proposed answer.

| Ask | Raised by | Subject | Body, in this order | Proposed answer |
| --- | --- | --- | --- | --- |
| A step that asks the founder | `mentat-operation`, at a step whose founder input applies, before the step is done | none: the step, by its `procedureId` | the step's question whole; where the run stands, by code; the ways to answer the step's instructions give | the way the instructions recommend for this project, in one sentence; the person writes their own answer |
| Checkpoint verdict | `mentat-operation`, after the last step of a Checkpoint operation, sent with `proposedVerdict`; the raise pauses the run and shows the row's item in review | the run (`Runs`) | the operation by slug; what the procedures wrote, by code; what `validatedWhen` says; how the written rows meet it or do not; on the mapping operation, each experiment waiting for the person to start it on its page, with its cost | `Validated` or `Invalidated`, one sentence why |
| A goal to review | `mentat-operation`, after a checkpoint answered the goal under goals to review, or after the mapping operation's `design-the-experiments` handed back an experiment that serves no goal; the run goes on, unpaused | the goal (`Goals`); for an experiment, the goal the objective would go under | the entry the step rewrote, before and after, or the experiment and the hypotheses it tests; what the goal's target rests on and its latest reading; the change you would propose | keep the goal as it is, change a figure from what it was to what it becomes, or add an objective — one sentence why |
| A start that spends, or needs the person's hands | `mentat-experiment`, before start_experiment; never for the mapping operation's experiments, which its verdict proposal names | the experiment (`Experiments`) | the hypothesis by code; the card; the test card as designed; the cost and what it buys; what the person has to do themselves | "start it", or "do not, because …" |
| Hypothesis decision | `mentat-experiment`, after complete_experiment | the hypothesis (`Hypotheses`) | the learning card; the evidence strength; the judged criteria | one of Persevere, Pivot, Kill, ContinueTesting |
| The build check failed | `mentat-gate-check`, after the build check it recorded failed; the run is not paused, and whoever owns it ends it — the gate check its own run, `mentat-agent` the Heartbeat firing's | the project (no subject) | each failed gate, block by block against its bar, with its weakest block — or the segments to win with no value proposition of ours at problem–solution fit — and the operation to run; the rows to add: each operation to run again, then a row of `hypotheses-mapping-and-experiments-design` | "add the rows". A note that names a pivot is read as one: **the exception to the bare-acknowledgement rule** — a pivot is never applied on an acknowledgement with no note; apply_pivot is called only when the note says "apply <pivot> to <tier>", or the person applies it on the roadmap screen |
| A decision of the business | any skill | the row it concerns, when there is one | what was decided, why, what it cuts across (`list_decisions` first) | the decision as one sentence |
| A missing key, tool or access | any skill | the row it concerns, when there is one | what the procedure needed; what was tried; what the run can do without it | "skip this procedure", or "wait for the key" |
| A prerequisite refusal | `mentat-operation`, on start_run | the roadmap row (`RoadmapItems`) | the operations that have not run, by slug; what forcing past them would mean | "run <them> first", or "force past" |
| A build row while the build is held | `mentat-operation`, on start_run of a row in `T7` or `T8` | the roadmap row (`RoadmapItems`) | the refusal, naming the gates that have not passed the build check; what forcing the start would mean | run what the build check names, or "force the start" |
| The plan is missing or finished | `mentat-advance` | the project (no subject) | no thesis, or no next row; what the planner needs the person for | "run the planner with me" |

Never an escalation: a canvas write (an entry, a hypothesis, a question, a test card) — done and
reported; the run report — the summary at end_run; a task's approval — `create_work_item` raises it; a
blocked task — `Blocked` raises it; a learning with proposed text — recording it raises its review; a
change to a goal proposed with propose_goal_change — the decision it writes is what the person accepts.
