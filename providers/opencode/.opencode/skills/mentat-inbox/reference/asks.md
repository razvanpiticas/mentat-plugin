# The asks a run raises under the unattended rule

Each is one escalation, raised by the skill that is at the point, with `runId`, followed by
`pause_run` in that skill. The body's last sentence is always the proposed answer.

| Ask | Raised by | Subject | Body, in this order | Proposed answer |
| --- | --- | --- | --- | --- |
| Checkpoint verdict | `mentat-operation`, at `end_run` of a Checkpoint operation | the run (`Runs`) | the operation by slug; what the procedures wrote, by code; what `validatedWhen` says; how the written rows meet it or do not | `Validated` or `Invalidated`, one sentence why |
| A start that spends, or needs the person's hands | `mentat-experiment`, before `start_experiment` | the experiment (`Experiments`) | the hypothesis by code; the card; the test card as designed; the cost and what it buys; what the person has to do themselves | "start it", or "do not, because …" |
| Hypothesis decision | `mentat-experiment`, after `complete_experiment` | the hypothesis (`Hypotheses`) | the learning card; the evidence strength; the judged criteria | one of Persevere, Pivot, Kill, ContinueTesting |
| Gate would pass with rows pending | `mentat-gate-check`, before `evaluate_gate`, when `get_gate_status` says the gate would pass and the tier still has rows not run or skipped | the project (no subject) | what the gate would say, block by block against its bar; the rows of the tier still pending, by slug; what evaluating now forgoes | "evaluate now", or "wait until <rows> are done" |
| Gate failed, pivot proposed | `mentat-gate-check`, after `evaluate_gate` failed | the project (no subject) | what the gate said, block by block against its bar, and the tier it was evaluated at; the pivot definition that fits, by code, and what it puts back on the plan | "apply <pivot> to <tier>", or "no pivot; work <block> first". **The exception to the bare-acknowledgement rule:** a pivot is never applied on an acknowledgement with no note — `apply_pivot` is called only when the note says "apply <pivot> to <tier>", or the person applies it on the roadmap screen and the next firing finds it applied |
| A decision of the business | any skill | the row it concerns, when there is one | what was decided, why, what it cuts across (`list_decisions` first) | the decision as one sentence |
| A missing key, tool or access | any skill | the row it concerns, when there is one | what the procedure needed; what was tried; what the run can do without it | "skip this procedure", or "wait for the key" |
| A prerequisite refusal | `mentat-operation`, on `start_run` | the roadmap row (`RoadmapItems`) | the operations that have not run, by slug; what forcing past them would mean | "run <them> first", or "force past" |
| The plan is missing or finished | `mentat-advance` | the project (no subject) | no thesis, or no next row; what the planner needs the person for | "run the planner with me" |

Never an escalation: a canvas write (an entry, a hypothesis, a question, a test card) — done and
reported; the run report — `end_run`'s summary; a job's approval — `create_work_item` raises it; a
blocked job — `Blocked` raises it; a learning with proposed text — recording it raises its review.
