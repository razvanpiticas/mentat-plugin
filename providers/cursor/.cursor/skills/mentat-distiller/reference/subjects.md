# The six subjects

Load this when reading a subject's text or recording a distillate. One row per kind
`list_insights` knows.

| Subject | `kind` | Read its current text with | Record with | A proposal replaces | The person rules on |
| --- | --- | --- | --- | --- | --- |
| A procedure of the method | `Procedure` | `get_procedure(code)` — the organisation's copy first, the shipped row behind it; note `version` | `record_procedure_insight(procedureId, title, body, proposedText?, runId)` | the whole instructions | the procedure's Guidance tab under Settings → Procedures |
| An entry kind | `BlockEntryDefinition` (needs `blockDefinitionId`) | `get_block(blockCode)` — the kind's description, its schema, `definitionId`, `version` | `record_block_entry_definition_insight(blockDefinitionId, kindDefinitionId, title, body, goodExample?, antiPattern?, structuralHeuristic?, proposedText?, runId)` | the whole description | the block's Guidance tab under Settings → Blocks |
| A method card | `ExperimentDefinition` | `get_experiment_definition(id)` — the organisation's copy when it customised a shipped card, the card otherwise; the execution instructions, `version` | `record_experiment_definition_insight(definitionId, title, body, sampleSizeHeuristic?, budget?, confound?, proposedText?, runId)` | the whole execution instructions; on a card the product ships, Apply first takes the organisation's own copy | the card's Guidance tab under Settings → Experiment definitions |
| A charter document | `Charter` | `list_charter_documents(portfolioId, projectId?)` — body and version | `record_charter_insight(documentId, title, body, proposedText?, runId)` | the whole body | the Charter screen's insights panel |
| One operation | `Operation` | `get_operation_definition(id)` — description, validated-when, if-not | `record_operation_insight(operationDefinitionId, title, body, pitfall?, appliesWhenNiche?, runId)` — no proposal; the text to change is a procedure's, on the procedure | nothing: a distilled statement, a pitfall or a niche | the operation's Guidance tab under Settings → Operations |
| The project | `Project` | the brief `mentat-agent` read at boot | `record_project_insight(projectId, title, body, category, runId)` — category `OpenThread`, `Idiosyncrasy`, `Contradiction` or `Decision`; no proposal | nothing: a distilled statement | the project's Insights screen |

`supersede_insight` and `contradict_insight` take the same `kind`, `subjectId` and
`blockDefinitionId` as the list that answered the learning, and `runId`.

A proposal is written against the version the subject's read answered. If the person edits the
text before ruling, Apply refuses the proposal as stale and says so; the next pass writes a new one
against the new text. Do not pre-empt it.
