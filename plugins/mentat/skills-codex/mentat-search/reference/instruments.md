# The instruments, with examples

Load this when the rule in the skill leaves a doubt about which instrument a question wants, or
when writing the answer.

## Exact — the code or the block is known

| Asked | Do |
| --- | --- |
| "What is on Key Resources?" | `get_block("KR")` |
| "Read KR-01" | `get_block("KR")`, the entry `KR-01` in it |
| "Read CS.PAINS-04" | `get_neighbours("CS.PAINS-04")` → `in_profile` to its profile, `profile_of` to the segment with its id → `get_customer_segment(segmentId)`, the pain `CS.PAINS-04` in its end user profile |
| "Read VP.PR-02" | `get_neighbours("VP.PR-02")` → `in_value_map` to the value proposition with its id → `get_value_proposition(valuePropositionId)`, the reliever `VP.PR-02` in its value map |
| "What does H-12 say?" | `get_neighbours("H-12")` → the row's id → `get_hypothesis(projectId, hypothesisId)` |
| "What did run … find?" | `get_run(projectId, runId)` |
| "Everything about segments and the value proposition" | `get_block("CS")`, `get_block("VP")` — two blocks, not the canvas; then `get_customer_segment` and `get_value_proposition` on the segments and value propositions whose parts the question needs, because the blocks list them without what is written inside |
| "Everything we have" | `get_canvas` — last resort; it lists every block's cards and the segments in short, never what is written inside a segment or a value proposition, which comes through `get_customer_segment` and `get_value_proposition` |

## By meaning — the subject is known, the place is not

| Asked | `search` question | Limit |
| --- | --- | --- |
| "What do we know about pricing?" | "pricing: how prices are set, pressure on prices, price comparisons with competitors" | 5 |
| "Is there already a hypothesis that gift buyers reorder?" | "a hypothesis that gift buyers reorder" | 5 |
| "Anything on the tax warehouse?" | "the tax warehouse: reopening it, excise status, bonded storage" | 5 |
| "Find every mention of Finestore" | "Finestore" | 20 |

One question, one sentence, the words the glossary uses. A duplicate check answers yes or no with
the hit quoted; a similarity above 0.85 with the same subject and the same polarity is a
duplicate, below 0.5 is not, in between the quoted passage decides and the answer says so.

## By connection — a row is known

| Asked | Do |
| --- | --- |
| "What hangs on KR-45?" | `get_neighbours("KR-45")` |
| "Which hypotheses are about CS-02?" | `get_neighbours("CS-02")`, the `about` edges, direction `In` |
| "What did experiment T-4 produce?" | `get_neighbours("T-4")`: `tests` out to its hypothesis, `from_experiment` in from its evidence |
| "Which open questions block a hypothesis a high risk depends on?" | the two-hop shape in shapes.md |
| "Which rows did last night's run touch?" | `get_neighbours` on the run's code is not possible (a run has no code); `get_run(projectId, runId)` instead |

## Both halves

"What is tied to whatever we know about pricing": `search("pricing …", 5)`, then
`get_neighbours` on each of the five, answered grouped by row.

## The answer's shape

```
<Instrument> — <the call as sent>:
  <code> <title> (<kind>, <status>) <number> — "<passage>"
  …
<for a walk: the query as the server echoed it, then one row per line>
<for neighbours: the origin row, then one line per neighbour: code, kind, status — edge, direction, and for a link its word, origin, confidence>
nothing found — <instrument> answered no rows
```

When search by meaning was off: the first line says so, and the rest is what the other instruments
found.
