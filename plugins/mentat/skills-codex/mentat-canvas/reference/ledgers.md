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
| `resolve_question` | `resolution` Answered (+ `answer`), Dropped, Reopened | The answer is recorded as the caller's. |

Statuses: Open, Answered, Dropped.

## Risks

Record what could go wrong, scored so it takes its place in the risk index. Exposure is likelihood
times severity, 1 to 25, and is what the index sorts on.

| Call | Arguments | Effect |
| --- | --- | --- |
| `add_risk` | `blockId`, `statement`, `likelihood`, `severity` (VeryLow … VeryHigh) | Open. |
| `update_risk` | `statement`; `likelihood` + `severity` together; `triggerSignal`; `response` | Restates, rescores, describes what would be seen first, plans what will be done. |
| `resolve_risk` | `resolution` Mitigated, Accepted, Closed, Reopened | Mitigated needs a response plan. |

Statuses: Open, Mitigated, Accepted, Closed.

## Ideas

A thought that is not yet an entry, ranked by effort against impact.

| Call | Arguments | Effect |
| --- | --- | --- |
| `add_idea` | `blockId`, `title`, `body`, `effort`, `impact` (VeryLow … VeryHigh) | Proposed. |
| `update_idea` | `title` + `body` together; `effort` + `impact` together | Rewrites, rescores. |
| `resolve_idea` | `resolution` Exploring, Adopted, Rejected, Reconsidered | Reconsidered reopens a rejected one. |

Statuses: Proposed, Exploring, Adopted, Rejected. An adopted idea usually becomes an entry; write
the entry and say in its body which idea it came from.

## Contradictions

Two entries of one canvas that cannot both be true: a segment that "cannot pay" and a revenue stream
that charges it a premium. Record what you notice; the pair is stored unordered.

| Call | Arguments | Effect |
| --- | --- | --- |
| `record_contradiction` | `entryAId`, `entryBId`, `summary`, `severity` (Low, Medium, High, Critical) | Open. A Critical open contradiction blocks the canvas gates, deliberately. |
| `rule_contradiction` | `severity`; `ruling` AcceptedA, AcceptedB, Merged, Deferred, Dismissed + `notes` | Severity first, then the ruling. Every ruling but Deferred is final and never touches the two entries. |

The ruling is a person's; record the one they made. A changed mind is a new contradiction, not a
rewritten ruling.
