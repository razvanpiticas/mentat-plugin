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

## Writes — `mcp:tools.write`

Every write below but `create_project` and `record_project_insight` takes `canvasVersion` and answers
the version it produced. Deletes answer only the version.

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

`delete_entry`, `delete_hypothesis`, `delete_experiment` and `delete_evidence` are marked destructive;
a harness may ask before running them.

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
