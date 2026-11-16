# Questions, risks, ideas and contradictions

Three ledgers hang off every block, and contradictions span the canvas. Each row has a reference
code (`Q-…`, `R-…`, `I-…`), a status, and the same version rule as every other write.

## Questions

Ask when something unknown stops you writing an entry with confidence — who the buyer is, what the
current tool costs, whether a channel is reachable. A question on the block is worth more than an
`Inferred` entry that guesses.

| Call | Arguments | Effect |
| --- | --- | --- |
| `add_question` | `blockId`, `text`, `ownerUserId` or `assignToMe` | Open, optionally handed to someone. |
| `update_question` | `text`; one of `ownerUserId`, `assignToMe`, `unassign` | Rewords, hands over, or takes back. |
| `add_what_was_learned` | `questionId`, `fact` (text, status, citations) | Appends one thing learned to its "What was learned" list, dated the day it is written. Open questions only. |
| `resolve_question` | `resolution` Answered (+ `answer`), Dropped, Reopened | The answer is recorded as the caller's. |

Statuses: Open, Answered, Dropped.

What was learned is not the answer. A step that learns something towards an open question — what
customers value most, what the business does best — appends it with `add_what_was_learned`, and the
list grows run by run; `resolve_question` with `Answered` closes the question when the list settles
it. A question that is no longer open takes nothing more: "Cannot add what was learned to question
<code>: it is Answered. Reopen it first."

## Risks

Record what could go wrong, scored so it takes its place in the risk index. Exposure is likelihood
times severity, 1 to 25, and is what the index sorts on.

| Call | Arguments | Effect |
| --- | --- | --- |
| `add_risk` | `blockId`, `statement`, `likelihood`, `severity` (VeryLow … VeryHigh); `title`, `valuePropositionId`, `arisesAtUseCaseFactor` on a risk about a value proposition | Open. |
| `update_risk` | `title`; `statement`; `valuePropositionId`; `arisesAtUseCaseFactor`; `likelihood` + `severity` together; `triggerSignal`; `response` | Retitles, restates, says what it is about and where it arises, rescores, describes what would be seen first, plans what will be done. What is left out is kept. |
| `resolve_risk` | `resolution` Mitigated, Accepted, Closed, Reopened | Mitigated needs a response plan. |

Statuses: Open, Mitigated, Accepted, Closed.

**A risk about a value proposition.** A risk is about its block unless it names one value proposition
of ours: `valuePropositionId`, a live one on the Value Proposition block — ours to design or our
business as it is today, never what they use today — with the Value Proposition block's id as
`blockId`. It then needs a `title`, a name of a few words, at most 80 characters, the sentence in
`statement`: inside the value proposition it is read by its title. It is never a card of the block:
`get_block` lists it among the block's risks with `isCard` false, and it goes with the value
proposition when that is deleted.

| What it is | Sent | Read |
| --- | --- | --- |
| a risk about the block | `blockId` alone; a `title` if it has one | a card of the block |
| a barrier to adoption that is not the customer's own pain | `title`, `valuePropositionId` (ours to design) and `arisesAtUseCaseFactor`, one of the ten factors of `write_use_case` | on that factor of the use case, `get_value_proposition` section `UseCase`, with its exposure and status |
| an objection of the persona nothing answers | `title` and `valuePropositionId`, no factor | inside the value proposition; `write_promise` names it as an objection's `writtenAsRiskId` |

A barrier that is the customer's own pain is a pain of the profile, marked in its fields, not a risk.

| Refused | Because |
| --- | --- |
| "A risk that arises at a factor of the use case is a barrier to adoption of one value proposition. Name the value proposition it is about." | a factor with no value proposition |
| "A risk about <code> is read inside it by its title." | a value proposition with no title |
| a title over 80 characters | a title is a name of a few words |
| the value proposition named | it is retired, on another block or of what they use today; a factor on our business as it is today, which has no use case |

`update_risk` sends what you leave out as the risk holds it, so a title, a value proposition or a factor
already on a risk cannot be taken off through it; a person does that on the risk's own form. Giving a
risk of the Value Proposition block its title, `valuePropositionId` and `arisesAtUseCaseFactor` makes
it a barrier read on that factor, no longer a card of the block.

## Ideas

A thought that is not yet an entry, ranked by effort against impact.

| Call | Arguments | Effect |
| --- | --- | --- |
| `add_idea` | `blockId`, `title`, `body`, `effort`, `impact` (VeryLow … VeryHigh), `picturePrompt` when it has a sketch | Proposed. |
| `update_idea` | `title` + `body` + `picturePrompt` together; `effort` + `impact` together | Rewrites, rescores. |
| `resolve_idea` | `resolution` Exploring, Adopted, Rejected, Reconsidered | Reconsidered reopens a rejected one. |

Statuses: Proposed, Exploring, Adopted, Rejected. An adopted idea usually becomes an entry; write
the entry and say in its body which idea it came from.

An idea can carry a sketch: `picturePrompt` is the prompt it is drawn from, and the sketch itself goes
up through `request_picture_upload` with the slot `IdeaSketch` and the idea's id as `ownerId`, after the
idea is written. A rewrite sends the prompt again: one left out or changed drops the sketch drawn from
the old prompt. An adopted or rejected idea takes no sketch.

## Contradictions

Two entries of one canvas that cannot both be true: a segment that "cannot pay" and a revenue stream
that charges it a premium. Record what you notice; the pair is stored unordered.

| Call | Arguments | Effect |
| --- | --- | --- |
| `record_contradiction` | `entryAId`, `entryBId`, `summary`, `severity` (Low, Medium, High, Critical) | Open. A Critical open contradiction blocks the canvas gates, deliberately. |
| `rule_contradiction` | `severity`; `ruling` AcceptedA, AcceptedB, Merged, Deferred, Dismissed + `notes` | Severity first, then the ruling. Every ruling but Deferred is final and never touches the two entries. |

The ruling is a person's; record the one they made. A changed mind is a new contradiction, not a
rewritten ruling.
