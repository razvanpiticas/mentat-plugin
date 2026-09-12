# The tools, in full

Every tool publishes its own input schema. Read it there rather than guessing at field names — this
file carries what the schema cannot say: which permission each tool demands, whether it changes
anything, and which take and answer a canvas version. How to use them well is the `mentat-canvas`
skill's job.

## Reads — `mcp:tools.read`

| Tool | Reaches | Answers |
| --- | --- | --- |
| `server_info` | Nothing. Answers from the process | Deployment name, route, the tool list |
| `list_portfolios` | BusinessIntelligence | The caller's portfolios, first page |
| `list_projects` | BusinessIntelligence | One page of a portfolio's projects |
| `get_project` | BusinessIntelligence | The project and its overview, with the canvas version |
| `get_canvas` | BusinessIntelligence | The whole canvas document and its version |
| `get_block` | BusinessIntelligence | One block by code, its kinds, everything on it, the version |
| `get_hypothesis` | BusinessIntelligence | One claim with its runs |
| `get_experiment` | BusinessIntelligence | One run with metric and criterion ids, evidence |
| `list_experiment_definitions` | BusinessIntelligence | The method cards, optionally for one concern |
| `get_experiment_definition` | BusinessIntelligence | One card's full method text |
| `get_roadmap` | BusinessIntelligence | A project's plan: thesis, rows in order, the gate line, the roadmap version |
| `get_operation_package` | BusinessIntelligence | One row's method text, targets, prerequisites and any run to resume |
| `get_gate_status` | BusinessIntelligence | What the method gate would answer now, recorded nowhere |
| `list_gate_evaluations` | BusinessIntelligence | The gate evaluations a project has recorded |
| `get_operation_run` | BusinessIntelligence | One run with its events |
| `list_operation_runs` | BusinessIntelligence | One page of a project's runs |
| `list_operation_definitions` | BusinessIntelligence | The operation definitions, shipped or as one project has them |
| `get_operation_definition` | BusinessIntelligence | One operation definition in full |
| `list_pivot_definitions` | BusinessIntelligence | The kinds of pivot a project may adopt |

## Writes — `mcp:tools.write`

Every write below takes `canvasVersion` and answers the version it produced, except: `create_project`
and `record_project_insight`, which take none; the ten changes to a plan, which take `roadmapVersion`
instead and answer the whole plan; and the seven writes to an operation run and `evaluate_gate`, which
take neither. Deletes answer only the version.

**`canvasVersion` and `roadmapVersion` are different numbers.** A project's plan and its canvas are
separate records that move independently, so a change to one carries its own version and a conflict
names which of the two moved.

| Tool | Changes |
| --- | --- |
| `create_project` | Creates a project and opens its canvas |
| `record_project_insight` | Adds a learning about the project |
| `add_entry`, `update_entry`, `retire_entry`, `delete_entry` | Entries on a block |
| `create_hypothesis`, `update_hypothesis`, `recommend_experiment_definition`, `withdraw_experiment_definition`, `park_hypothesis`, `unpark_hypothesis`, `retire_hypothesis`, `decide_hypothesis`, `delete_hypothesis` | Claims |
| `design_experiment`, `add_metric`, `add_criterion`, `start_experiment`, `record_observation`, `judge_criterion`, `complete_experiment`, `abort_experiment`, `record_spend`, `delete_experiment` | Runs |
| `record_evidence`, `update_evidence`, `add_data_point`, `delete_evidence` | Evidence bundles |
| `add_question`, `update_question`, `resolve_question` | Questions |
| `add_risk`, `update_risk`, `resolve_risk` | Risks |
| `add_idea`, `update_idea`, `resolve_idea` | Ideas |
| `record_contradiction`, `rule_contradiction` | Contradictions |
| `set_roadmap_thesis`, `place_roadmap_item`, `skip_roadmap_item`, `include_roadmap_item`, `annotate_roadmap_item`, `append_roadmap_operation`, `add_roadmap_move`, `remove_roadmap_move`, `apply_pivot` | A project's plan — these take `roadmapVersion` |
| `start_operation_run`, `checkpoint_operation_run`, `observe_operation_run`, `pause_operation_run`, `complete_operation_run`, `fail_operation_run`, `cancel_operation_run` | Operation runs — no version |
| `evaluate_gate` | Records what a method gate answered — no version |

`delete_entry`, `delete_hypothesis`, `delete_experiment`, `delete_evidence`, `remove_roadmap_move`,
`cancel_operation_run` and `apply_pivot` are marked destructive; a harness may ask before running them.

`evaluate_gate` and `apply_pivot` move a project's tier, so BusinessIntelligence asks for
`projects.manage` on them rather than `canvas.write`. A caller who may edit a canvas and not move a
project is refused there, correctly.

## Where the two permissions come from

`mcp:tools.read` and `mcp:tools.write` are this server's own permissions, granted on Mentat's roles
screen per tenant. They gate whether a tool may be *called*.

## The second enforcement point

Every tool but `server_info` reaches BusinessIntelligence, which enforces **its own** permissions on
the forwarded token when the call arrives: `business-intelligence:portfolios.read`,
`projects.read`/`projects.manage`, `canvas.read`/`canvas.write`, `definitions.read`. So a caller needs
both, and being refused by BusinessIntelligence rather than by this server is the ordinary, correct
outcome for somebody who may use this server and may not use that data. A refusal that names
BusinessIntelligence is not a bug in this server.

## Composed writes

Three tools make several BusinessIntelligence calls in one: `design_experiment` (the experiment, then
each metric, then each criterion), `record_evidence` (the bundle, then its rating, axes and readings)
and `complete_experiment` (the learning card, then the completion). A refusal part-way says what was
already written; the row exists and is finished with the single-step tools.

Two more compose on the other side of the wire rather than here. `start_operation_run` runs the
prerequisite check before it starts anything: a refusal naming operations that have not run means
nothing was started, and `forcePastPrerequisites: true` starts it anyway with that fact recorded on the
run — a person's decision, never the model's. `apply_pivot` moves the project's tier and appends the
operations the pivot revisits in one transaction; either both happen or neither does.
