# The tools, in full

Every tool publishes its own input schema. Read it there rather than guessing at field names — this
file carries what the schema cannot say: which permission each tool demands, whether it changes
anything, and which take and answer a canvas version. How to use them well is the `mentat-canvas`
skill's job.

One hundred and sixty-eight tools: forty-four reads and one hundred and twenty-four writes.

## Reads — `mcp:tools.read`

| Tool | Reaches | Answers |
| --- | --- | --- |
| `server_info` | Nothing. Answers from the process | Deployment name, route, the tool list |
| `list_portfolios` | BusinessIntelligence | One page of the caller's portfolios, `pageNumber` for the next |
| `list_projects` | BusinessIntelligence | One page of a portfolio's projects |
| `get_project` | BusinessIntelligence | The project and its overview, with the canvas version. Its description is the idea as last stated, or the founder's own words before the idea is stated |
| `get_project_state` | BusinessIntelligence | Where a project stands: the canvas in summary with every block's counts, the plan position and whether it has a thesis, the decisions, your open jobs, what a person answered while you were away, how many inbox rows still wait for the person you work for, and what changed since your last run ended |
| `get_canvas` | BusinessIntelligence | The canvas document and its version: every block whole, every segment as a whole entry with its card, parts and refusals — large. What is written inside a segment or a value proposition comes through `get_customer_segment` and `get_value_proposition` |
| `get_block` | BusinessIntelligence | One block by code, its kinds with each kind's schema, everything on it, the version. On Customer Segments, its trees under `customerSegments`: an outline row per live segment (id, code, title, status, order, and only what holds a value: parent, state, score, reasons, the true flags), the retired count, every beachhead in full but its end user profile with its adjacent markets in short, and the shortlisted segments of a tree with no beachhead in short. `includeSchemas`, `includeRetired` and `allInFull` add the kinds' schemas, the retired segments and every segment as a whole entry. On Value Proposition, our value propositions and the notes |
| `get_customer_segment` | BusinessIntelligence | One customer segment whole: its ratings and score, the three conditions of a market, its comparison, its customers today, its TAM with the total and the market share, its beachhead part with the adjacent markets and the segments it turns away, and its end user profile — who they are, the jobs, pains and gains in rank order, the purchasing criteria in order, the persona with its picture's address and its picture prompt, the context profiles with their items — with the canvas version |
| `get_value_proposition` | BusinessIntelligence | One value proposition whole: the profile it aims at, its products and services, pain relievers and gain creators in rank order with what each is connected to, the value propositions of what they use today aimed at the same profile, and the fit check, with the canvas version |
| `get_hypothesis` | BusinessIntelligence | One hypothesis with its runs |
| `get_experiment` | BusinessIntelligence | One run with metric and criterion ids, evidence |
| `list_experiment_definitions` | BusinessIntelligence | The method cards, optionally for one concern |
| `get_experiment_definition` | BusinessIntelligence | One card's full method text |
| `get_roadmap` | BusinessIntelligence | A project's plan: thesis, rows in order with the blocks each operation writes to, the gate line, the roadmap version |
| `get_ideation` | BusinessIntelligence | The project's idea: the founder's starting point and route, the candidates weighed with their signals and ratings, the statement in force, the proposal waiting for a person, every revision with its decision, and the idea's `version`, which every idea write sends as `ideationVersion`; each write answers the next `version` |
| `get_operation_package` | BusinessIntelligence | One row's method text: its steps in order with what each asks the founder, its targets — blocks and kinds, or the project's idea with the idea itself — the operations it goes back to, in order, and on an `Experiment` operation its hand-over to hypotheses and the steps every `Experiment` operation shares; its prerequisites and any run to resume |
| `get_gate_status` | BusinessIntelligence | What the method gate would answer now, recorded nowhere |
| `list_gate_evaluations` | BusinessIntelligence | The gate evaluations a project has recorded |
| `get_run` | BusinessIntelligence | One run of any kind with its events, and — on an attempt at a row of the plan — the verdict, its steps with each one finished and the founder's answer to each that asked, and what it produced; on an `Experiment` operation's run, the hand-over to hypotheses as the canvas stands now |
| `list_runs` | BusinessIntelligence | One page of a project's runs of every kind, newest first, each with the summary it left. Takes the portfolio and the project |
| `list_operation_definitions` | BusinessIntelligence | The operation definitions, shipped or as one project has them |
| `get_operation_definition` | BusinessIntelligence | One operation definition in full: what it asks the founder and at which steps, the operations it goes back to, and on an `Experiment` operation its hand-over and the steps it shares |
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
and `record_project_insight`, which take none; the ten changes to a plan, which take `roadmapVersion`
instead and answer the whole plan; the seven writes to the idea, which take `ideationVersion` instead
and answer the whole idea; and the six writes to a run, the organisation's writes, the board's
writes, the two writes to a method card, `raise_escalation`, `send_message`, `record_decision` and `evaluate_gate`, which take none of the three. Deletes answer
only the version.

**`canvasVersion`, `roadmapVersion` and `ideationVersion` are three different numbers.** A project's
canvas, its plan and its idea are separate records that move independently, so a change to one
carries its own version and a conflict names which of them moved.

| Tool | Changes |
| --- | --- |
| `create_project` | Creates a project and opens its canvas; its description is the founder's own words |
| `record_project_insight` | Adds a learning about the project |
| `add_entry`, `update_entry`, `retire_entry`, `delete_entry` | Entries. A kind written inside another entry takes its owner's id as `ownerId` when it is added: a child segment its parent, an end user profile its segment, a context profile its end user profile, a job, pain or gain its profile, a product, reliever or creator its value proposition. A confirmed or reported customer or value entry carries its `source`. `update_entry` on our value proposition also writes what it forgoes and why |
| `rerank_entries` | Puts every entry of one kind inside one owner in a new order, top first: the order is the rank |
| `set_customer_segment_state` | Shortlists a segment, drops it with why, or makes it a candidate again |
| `split_customer_segment` | Adds child segments under a segment, with why it is split |
| `rate_customer_segment` | Writes a segment's seven ratings, each with its reason, in one call |
| `test_market_conditions` | Writes the three conditions of a market, each with whether it holds and its evidence |
| `write_comparison` | Writes a shortlisted segment's whole comparison, the founder's own rows included |
| `write_customers_today` | Writes the customers an existing business has in a segment, which marks it served |
| `write_total_addressable_market` | Writes a beachhead's TAM: the end users counted bottom up and top down, the annual revenue per end user and how it was worked out, the growth rate |
| `choose_beachhead` | Proposes a leaf segment as a beachhead with its part — why this one, what it gives up, when to switch, the offers to try first, whether it is the hardest side of a platform, the adjacent markets in order, the beachhead it replaces, and why not on every other shortlisted segment — with its decision in the ledger: an agent's is proposed and a person accepts it on the decisions tab, a person's is accepted as written |
| `write_turned_away_segments` | Writes the whole list of segments a beachhead turns away, each with how to tell them and what to refer them to |
| `describe_end_user` | Writes who the end user is: the end user profile's fifteen characteristics and any added ones, in one call |
| `write_purchasing_criteria` | Writes an end user profile's whole list of purchasing criteria, in order, each pointing at one of its jobs, pains or gains |
| `write_persona` | Writes or replaces the persona of an end user profile: how it was made, its fact sheet, its words for the top pain and its picture prompt |
| `upload_persona_picture`, `remove_persona_picture` | Puts a picture on the persona — a PNG, JPEG, WebP or SVG file of at most 25 MB, its bytes sent in base64 — or takes it off |
| `erase_persona` | Erases a real end user's persona when they ask: its facts and its picture, and their values in the history |
| `move_end_user_profile` | Moves an end user profile, with its context profiles and persona, onto a leaf segment |
| `aim_value_proposition` | Aims a value proposition at a profile |
| `connect_value_map_entry`, `disconnect_value_map_entry` | Connects a product, reliever or creator to the job, pain or gain it addresses, or takes the connection off |
| `add_what_they_use_today` | Adds the value proposition of what they use today, aimed at the same profile as ours, on a Competitive Position entry that exists or a new one |
| `reach_fit` | Records the next kind of fit our value proposition reached; problem–solution fit takes the reason to switch from what they use today |
| `write_starting_point`, `add_ideation_candidate`, `update_ideation_candidate`, `drop_ideation_candidate`, `add_ideation_signal`, `update_ideation_signal`, `propose_ideation_revision` | The project's idea — these take `ideationVersion`: the founder's starting point and the route, the ideas weighed and the signals read for them, and a statement of the idea naming its riskiest hypothesis; an agent's statement is proposed for a person to accept, a person's is accepted as written |
| `create_hypothesis`, `update_hypothesis`, `recommend_experiment_definition`, `withdraw_experiment_definition`, `park_hypothesis`, `unpark_hypothesis`, `retire_hypothesis`, `decide_hypothesis`, `delete_hypothesis` | Hypotheses |
| `design_experiment`, `add_metric`, `add_criterion`, `start_experiment`, `record_observation`, `judge_criterion`, `complete_experiment`, `abort_experiment`, `record_spend`, `delete_experiment` | Runs |
| `record_evidence`, `update_evidence`, `add_data_point`, `delete_evidence` | Evidence bundles |
| `add_question`, `update_question`, `resolve_question` | Questions |
| `add_risk`, `update_risk`, `resolve_risk` | Risks |
| `add_idea`, `update_idea`, `resolve_idea` | Ideas |
| `record_contradiction`, `rule_contradiction` | Contradictions |
| `set_roadmap_thesis`, `place_roadmap_item`, `skip_roadmap_item`, `include_roadmap_item`, `annotate_roadmap_item`, `append_roadmap_operation`, `add_roadmap_move`, `remove_roadmap_move`, `apply_pivot` | A project's plan — these take `roadmapVersion` |
| `set_roadmap_move_work_item` | Points a move of the plan at the job that carries it out — it takes `roadmapVersion` too |
| `start_run`, `checkpoint_run`, `observe_run`, `pause_run`, `end_run`, `cancel_run` | Runs of a project — no version. `checkpoint_run` carries the founder's answer to a step that asked them; it and `end_run` carry the operations the project goes back to, in order |
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
| `raise_escalation` | Puts a question a run cannot answer on its own in front of a person — through `mentat-inbox`. Sent with the `procedureId` of a step that asks the founder, it is that step's question, and the run pauses until the person answers it |
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
`unlink_from_goal`, `cancel_run`, `apply_pivot`, `remove_persona_picture`, `erase_persona` and
`disconnect_value_map_entry` are marked destructive; a harness may ask before running them.

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
