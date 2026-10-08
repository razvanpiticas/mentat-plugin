# Recording an insight, when the person asks

Load this when the person has named something worth recording after a run's blockers were reported.

## One mechanism, six subjects

Everything the product learns is an **insight**: a title, a body, a status saying whether it holds, a
review state saying whether a person allowed it, and — when you know what the text should say instead
— a **proposed replacement**. The subjects and their tools:

| Subject | Record it with | What a proposal replaces |
| --- | --- | --- |
| A charter document | `record_charter_insight` | the document's whole body |
| A procedure of the method | `record_procedure_insight` | the procedure's whole instructions |
| One operation | `record_operation_insight` | nothing — a pitfall or a niche it does not fit; an operation is not a text |
| An entry kind | `record_block_entry_definition_insight` | the kind's description |
| A method card | `record_experiment_definition_insight` | the card's execution instructions |
| The project itself | `record_project_insight` | nothing — a project has no text |

`list_insights` and `get_insight` read any of them; both take the kind. Read them only when the person
asks: they are not part of any definition and no skill loads them on its own.

## Propose the whole text, not the edit

`proposedText` is what the text should say **from now on**, in full. It is applied by replacing the
body, so a fragment applied is a document with a hole in it. Write the whole thing, with your change in
place, or send no proposal and let the body describe what is wrong. An insight with no proposal is
still worth recording.

## Before you record: read

`list_insights` on the subject first, every time. Somebody already said it: say nothing new, or
`supersede_insight` the older draft with yours only when yours genuinely replaces it. A `Confirmed`
insight you can show no longer holds: `contradict_insight`, with the evidence in the body. Nothing like
it exists: record yours.

## What you may not do

`Approve` and `Reject` are a person's, on the screens. An insight carrying a proposal raises its own
review row in the inbox when you record it inside a run; you send no message about it.
`revise_charter_document` is allowed only on a `Living` document and only with the insight it applies,
in one call; a `HumanApprovalOnly` document — every seeded one — refuses it, and the answer is the
insight with its proposed text, nothing more. A version that is not the current one is refused with
`CONFLICT`: re-read with `list_charter_documents` and rewrite against what the document says now.

Employee instructions and custom documents follow the same approval rules. Read with
`get_agent_document`; record a proposal with `record_charter_insight`; apply permitted Living
changes using `revise_agent_document` with that `insightId` and the version just read.
Company reads and revisions accept exactly one of `projectId` or `portfolioId`.
