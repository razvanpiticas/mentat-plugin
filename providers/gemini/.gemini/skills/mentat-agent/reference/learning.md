# Recording what a session learned

Load this before your first insight of a session.

## One mechanism, five subjects

Everything the product learns is an **insight**: a title, a body, a status saying whether it holds, a
review state saying whether a person allowed it, and — when you know what the text should say instead
— a **proposed replacement**. The five subjects and their tools:

| Subject | Record it with | What a proposal replaces |
| --- | --- | --- |
| A charter document | `record_charter_insight` | the document's whole body |
| One operation of the method | `record_operation_insight` | one of its procedures' instructions |
| An entry kind | `record_block_entry_definition_insight` | the kind's description |
| A method card | `record_experiment_definition_insight` | the card's execution instructions |
| The project itself | `record_project_insight` | nothing — a project has no text |

`list_insights` and `get_insight` read any of them; both take the kind.

## Propose the whole text, not the edit

`proposedText` is what the document should say **from now on**, in full. It is applied by replacing
the body, so a fragment applied is a document with a hole in it. Write the whole thing, with your
change in place, or send no proposal at all and let the body describe what is wrong.

An insight with no proposal is still worth recording. "The export times out above five thousand rows"
is useful even before anybody knows what the procedure should say instead.

## Before you record: read

`list_insights` on the subject first, every time.

- **Somebody already said it.** Say nothing new; `supersede_insight` the older draft with yours only
  when yours genuinely replaces it — a sharper statement, a better proposal. Two drafts saying the
  same thing in different words is the noise this check exists to prevent.
- **A `Confirmed` insight you can show no longer holds** is `contradict_insight`, with the evidence in
  the body. Contradicting is not deleting: the row stays, and the chain of what was believed and why
  stays readable.
- **Nothing like it exists.** Record yours.

## What you may not do

`Approve` and `Reject` are a person's, on the screens. Your insight is recorded as a draft and the
skills that read insights into their prompts read only the confirmed and approved ones — so an
un-ruled insight of yours changes nothing until a person allows it. That is the design.

## Revising a charter document

Only when the document's write mode is `Living`, and then `revise_charter_document` carries the new
body, the version you read, **and** the insight it applies, in one call. The insight is recorded,
auto-applied and the document revised together.

On a `HumanApprovalOnly` document — which is every seeded document until somebody changes it — the
call is refused. Record the insight with its proposed text and stop; a person applies it on the
charter screen. `AppendOnly` allows a revision only if the new body starts with the current one.

Sending a version that is not the current one is refused with `CONFLICT`. Re-read with
`list_charter_documents`, rewrite the proposal against what the document says **now** — somebody's
edit is in it — and resend.

## A session that learned nothing

Say so. "Nothing surprised me" is a finding; silence is indistinguishable from forgetting to look.
