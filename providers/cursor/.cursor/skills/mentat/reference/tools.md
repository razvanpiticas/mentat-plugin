# The tools, in full

Every tool publishes its own input schema. Read it there rather than guessing at field names — this
file carries what the schema cannot say: which permission each tool demands, whether it changes
anything, and which take and answer a canvas version. How to use them well is the `mentat-canvas`
skill's job.

One hundred and thirty-eight tools: forty-one reads and ninety-seven writes.

## Reads — `mcp:tools.read`

| Tool | Reaches | Answers |
| --- | --- | --- |
| `server_info` | Nothing. Answers from the process | Deployment name, route, the tool list |
| `list_portfolios` | BusinessIntelligence | One page of the caller's portfolios, `pageNumber` for the next |
| `list_projects` | BusinessIntelligence | One page of a portfolio's projects |
| `get_project` | BusinessIntelligence | The project and its overview, with the canvas version |
| `get_project_state` | BusinessIntelligence | Where a project stands: the canvas in summary with every block's counts, the plan position and whether it has a thesis, the decisions, your open jobs, what a person answered while you were away, how many inbox rows still wait for the person you work for, and what changed since your last run ended |
| `get_canvas` | BusinessIntelligence | The whole canvas document and its version |
| `get_block` | BusinessIntelligence | One block by code, its kinds with each kind's schema, everything on it, the version |
| `get_hypothesis` | BusinessIntelligence | One hypothesis with its runs |
| `get_experiment` | BusinessIntelligence | One run with metric and criterion ids, evidence |
| `list_experiment_definitions` | BusinessIntelligence | The method cards, optionally for one concern |
| `get_experiment_definition` | BusinessIntelligence | One card's full method text |
| `get_roadmap` | BusinessIntelligence | A project's plan: thesis, rows in order with the blocks each operation writes to, the gate line, the roadmap version |
| `get_operation_package` | BusinessIntelligence | One row's method text, targets, prerequisites and any run to resume |
| `get_gate_status` | BusinessIntelligence | What the method gate would answer now, recorded nowhere |
| `list_gate_evaluations` | BusinessIntelligence | The gate evaluations a project has recorded |
| `get_run` | BusinessIntelligence | One run of any kind with its events, and — on an attempt at a row of the plan — the verdict, the procedures finished and what it produced |
| `list_runs` | BusinessIntelligence | One page of a project's runs of every kind, newest first, each with the summary it left. Takes the portfolio and the project |
| `list_operation_definitions` | BusinessIntelligence | The operation definitions, shipped or as one project has them |
| `get_operation_definition` | BusinessIntelligence | One operation definition in full |
| `get_procedure` | BusinessIntelligence | One procedure of the method catalogue by its code, with the wording this organisation reads |
| `list_pivot_definitions` | BusinessIntelligence | The kinds of pivot a project may adopt |
| `boot_agent` | BusinessIntelligence | One agent's whole brief: the company writing, the mission and its objectives, the project's state, the team, the person, the agent's own instructions |
| `list_agents` | BusinessIntelligence | The agents a project may boot: its own and the portfolio's shared ones |
| `get_agent` | BusinessIntelligence | One agent with its documents, its routines and the canvases it works |
| `list_routines` | BusinessIntelligence | A project's scheduled work, for the harness that fires it |
| `get_routine` | BusinessIntelligence | One routine whole, its instructions included — through `mentat-routines` |
| `list_charter_documents` | BusinessIntelligence | Company writing under exactly one projectId or portfolioId, with requested bodies by id |
| `list_agent_documents` | BusinessIntelligence | An employee's instructions and custom document titles |
| `get_agent_document` | BusinessIntelligence | One employee document with its full text, write mode and version |
| `list_goals` | BusinessIntelligence | A project's goal tree with the latest reading of each metric |
| `list_insights` | BusinessIntelligence | The learnings recorded about one subject, of any of the six kinds, or every learning since a moment; narrowed by status either way, at most 50 an answer, and it says how many more matched |
| `get_insight` | BusinessIntelligence | One learning, of any of the six kinds |
| `list_activity` | BusinessIntelligence | A project's change feed, newest first, cursor-paged |
| `get_history` | BusinessIntelligence | Everything that ever happened to one row, newest first, cursor-paged |
| `list_work_items` | BusinessIntelligence | A project's board: top-level jobs, each carrying the chunks under it |
| `get_work_item` | BusinessIntelligence | One job with its chunks, its thread and the spells it spent in each column |
| `list_decisions` | BusinessIntelligence | The decisions a project is working to, newest first |
| `list_untyped_links` | BusinessIntelligence | The links the embedding job drew and nobody has given a word to, both ends named and quoted, with every word you may type with |
| `search` | BusinessIntelligence | The rows of a project that are about something, found by embedding the question once; each row with the passage that matched quoted and a similarity from 0 to 1. Spends one embedding against the organisation's allowance |
| `get_neighbours` | BusinessIntelligence | Everything one hop from one row, by its code: the structural edges and every accepted link, each with its word, its direction and its confidence |
| `run_analytical_query` | BusinessIntelligence | The rows one read-only query you write answers over the project's property graph, with the query echoed |

## Writes — `mcp:tools.write`

Every write below takes `canvasVersion` and answers the version it produced, except: `create_project`
and `record_project_insight`, which take none; the nine changes to a plan, which take `roadmapVersion`
instead and answer the whole plan; and the six writes to a run, the organisation's writes, the board's
writes, the two writes to a method card, `raise_escalation`, `send_message`, `record_decision` and `evaluate_gate`, which take neither. Deletes answer
only the version.

**`canvasVersion` and `roadmapVersion` are different numbers.** A project's plan and its canvas are
separate records that move independently, so a change to one carries its own version and a conflict
names which of the two moved.

| Tool | Changes |
| --- | --- |
| `create_project` | Creates a project and opens its canvas |
| `record_project_insight` | Adds a learning about the project |
| `add_entry`, `update_entry`, `retire_entry`, `delete_entry` | Entries on a block |
| `choose_beachhead`, `clear_beachhead` | The one customer segment marked as the market to win first; choosing one takes the mark off the segment that held it |
| `create_hypothesis`, `update_hypothesis`, `recommend_experiment_definition`, `withdraw_experiment_definition`, `park_hypothesis`, `unpark_hypothesis`, `retire_hypothesis`, `decide_hypothesis`, `delete_hypothesis` | Hypotheses |
| `design_experiment`, `add_metric`, `add_criterion`, `start_experiment`, `record_observation`, `judge_criterion`, `complete_experiment`, `abort_experiment`, `record_spend`, `delete_experiment` | Runs |
| `record_evidence`, `update_evidence`, `add_data_point`, `delete_evidence` | Evidence bundles |
| `add_question`, `update_question`, `resolve_question` | Questions |
| `add_risk`, `update_risk`, `resolve_risk` | Risks |
| `add_idea`, `update_idea`, `resolve_idea` | Ideas |
| `record_contradiction`, `rule_contradiction` | Contradictions |
| `set_roadmap_thesis`, `place_roadmap_item`, `skip_roadmap_item`, `include_roadmap_item`, `annotate_roadmap_item`, `append_roadmap_operation`, `add_roadmap_move`, `remove_roadmap_move`, `apply_pivot` | A project's plan — these take `roadmapVersion` |
| `set_roadmap_move_work_item` | Points a move at the job that carries it out |
| `start_run`, `checkpoint_run`, `observe_run`, `pause_run`, `end_run`, `cancel_run` | Runs of a project — no version |
| `evaluate_gate` | Records what a method gate answered — no version |
| `create_agent` | Adds an agent to a project, or to the portfolio when no project is named |
| `update_agent` | Renames and retitles an agent, and shares a project's agent with the whole portfolio |
| `set_agent_status` | Pauses, resumes or archives an agent |
| `record_charter_insight` | Records a learning about one charter document, with the full text it proposes instead |
| `revise_charter_document` | Replaces a charter document's body — for an agent, only on a Living document and only with the insight it applies |
| `write_charter_document` | Creates any company charter kind under a project or portfolio |
| `set_charter_document_flags` | Changes company document settings; person-only |
| `write_agent_document` | Creates employee instructions or a custom employee document |
| `revise_agent_document` | Rewrites an employee document against expectedVersion, under its write-mode rules |
| `set_agent_document_flags` | Changes employee document settings; person-only |
| `create_goal` | Sets the project's mission, or adds an objective under a goal |
| `update_goal` | Changes a goal's title, description, metric, deadline or owner |
| `set_goal_status` | Activates, achieves, misses or abandons a goal |
| `record_goal_measurement` | Appends a measurement: the value, the date it was true and the source it came from |
| `create_routine` | Adds a piece of scheduled work to a project, switched off — through `mentat-routines` |
| `update_routine` | Rewrites a routine's name, cron line, time zone, instructions and agent together — through `mentat-routines` |
| `enable_routine` | Switches a routine on, so the harness is to schedule it — through `mentat-routines` |
| `disable_routine` | Switches a routine off, keeping everything it says — through `mentat-routines` |
| `record_operation_insight` | Records a learning about one operation: what to watch for and what goes wrong |
| `record_block_entry_definition_insight` | Records a learning about one entry kind |
| `record_experiment_definition_insight` | Records a learning about one method card |
| `customise_experiment_definition` | Takes the organisation's own copy of a method card that ships with Mentat, which every project then reads in its place — on the person's word; takes no run |
| `update_experiment_definition` | Changes a method card's name, texts (the execution instructions among them) or ratings, keeping the rest — the organisation's own card or its copy of a shipped one; on the person's word; takes no run |
| `record_procedure_insight` | Records a learning about one procedure of the catalogue, with the whole new instructions it proposes |
| `supersede_insight` | Marks an older insight replaced by a newer one that says it better |
| `contradict_insight` | Marks a confirmed insight as no longer holding |
| `link_to_goal` | Puts a hypothesis or an experiment behind a goal it serves |
| `unlink_from_goal` | Takes one out from behind a goal |
| `type_link` | Gives one inferred link its meaning: the word, its direction and its size, creating the word when none fits |
| `confirm_link` | Records a person's judgement that a link holds |
| `reject_link` | Records a person's judgement that a link does not hold, and stops the guess being proposed again |
| `create_work_item` | Writes one piece of the project's work down, under a parent job or on its own |
| `update_work_item` | Rewrites a job's line, brief, three ratings, due date and assignee — the whole text, not a patch |
| `transition_work_item` | Moves a job to another column of the board |
| `comment_on_work_item` | Appends one remark to a job's thread |
| `raise_escalation` | Puts a question a run cannot answer on its own in front of a person — through `mentat-inbox` |
| `send_message` | Tells a person something they should know and need not answer — through `mentat-inbox` |
| `record_decision` | Writes a decision of the business into the project's ledger |

The organisation's writes take no version either: an agent, a routine, a goal and a measurement belong
to the organisation or to the project rather than to the canvas or the plan. A charter document is the
exception and carries a version of its own, sent as `expectedVersion` on `revise_charter_document` or `revise_agent_document` and
answered by every read of it, so a proposal written against text somebody has since edited is refused
rather than applied silently over their words.

The four routine writes carry no `runId` either, and BusinessIntelligence refuses one that arrives
inside a run: a schedule is set by somebody who has read it. This server also never fires a routine —
the scheduler on the person's machine does — so `enable_routine` and `disable_routine` are half of the
act each way, and the entry on the machine is the other half.

The board, the inbox and the decision ledger carry no version either: a job, an escalation and a
decision hang off the project rather than off the canvas, and the last write to a job stands.

`delete_entry`, `delete_hypothesis`, `delete_experiment`, `delete_evidence`, `remove_roadmap_move`,
`unlink_from_goal`, `cancel_run` and `apply_pivot` are marked destructive; a harness may ask before
running them.

`evaluate_gate` and `apply_pivot` move a project's tier, so BusinessIntelligence asks for
`projects.manage` on them rather than `canvas.write`. A caller who may edit a canvas and not move a
project is refused there, correctly. The same split applies to the newer surfaces:
`business-intelligence:work.read` and `work.write` gate the board and the escalation, `activity.read`
gates the change feed and one row's history, `projects.read` and `projects.manage` gate reading and
writing the decision ledger, and `work.approve` — approving a proposal, resolving an inbox row — is a
person's alone and is on no tool here. `get_project_state` needs `projects.read`, `work.read` and
`activity.read` together, so it is the read most likely to be refused for a caller who holds only some
of them. The five run writes need `agents.run`; `cancel_run` needs `projects.manage`, because
abandoning somebody else's session is not something working as an agent carries.

## `runId` — the run a write was made from inside

**Every write tool but `start_run` takes `runId`**, and it means the same thing on all of them: the
run the call is being made from inside. Send the id `start_run` answered on every write for the rest of
that run, and the project's history records who did what under which session. A person writing for
themselves leaves it out.

On the five other run tools — `checkpoint_run`, `observe_run`, `pause_run`, `end_run`, `cancel_run` —
it is required rather than optional, because those tools are about that run. The run addressed and the
run worked inside are one run, which is why there is no second argument name for it.

`get_project_state` is the one **read** that takes `runId`, and it should be sent: without it the open
work and the delta answered are the signed-in person's rather than the agent's.

A write sent with a `runId` naming a run that is not running, or a run somebody else opened, is
refused with `run-not-usable`. That is not retried: open a new run with `start_run` and send the write
again with the new id, or send it without a `runId` at all if the work is the person's.

## Where the two permissions come from

`mcp:tools.read` and `mcp:tools.write` are this server's own permissions, granted on Mentat's roles
screen per tenant. They gate whether a tool may be *called*.

## The second enforcement point

Every tool but `server_info` reaches BusinessIntelligence, which enforces **its own** permissions on
the forwarded token when the call arrives: `business-intelligence:portfolios.read`,
`projects.read`/`projects.manage`, `canvas.read`/`canvas.write`, `definitions.read`,
`work.read`/`work.write`/`work.approve`, `activity.read`. So a caller needs both, and being refused by
BusinessIntelligence rather than by this server is the ordinary, correct outcome for somebody who may
use this server and may not use that data. A refusal that names BusinessIntelligence is not a bug in
this server.

## Composed writes

Three tools make several BusinessIntelligence calls in one: `design_experiment` (the experiment, then
each metric, then each criterion), `record_evidence` (the bundle, then its rating, axes and readings)
and `complete_experiment` (the learning card, then the completion). A refusal part-way says what was
already written; the row exists and is finished with the single-step tools.

One read composes the same way: `get_work_item` asks for the job and for the spells it spent in each
column, because those are derived from the change log rather than stored on the job.

Two more compose on the other side of the wire rather than here. `start_run` on a row of the plan runs
the prerequisite check before it starts anything: a refusal naming operations that have not run means
nothing was started, and `forcePastPrerequisites: true` starts it anyway with that fact recorded on the
run — a person's decision, never the model's. `apply_pivot` moves the project's tier and appends the
operations the pivot revisits in one transaction; either both happen or neither does.
