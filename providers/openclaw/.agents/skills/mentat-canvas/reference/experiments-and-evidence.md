# Experiments and evidence

An experiment is one test of a method card against one or more hypotheses. Its reference code is
`T-<n>`; a bundle of evidence is `E-<n>`. A **run** here is always the session it was written from
(`runId`), never the experiment. The order below is the order BusinessIntelligence enforces; a call
out of order is refused with the rule it broke.

## 1. Design — `design_experiment`

One call plans the whole test card, and writes the experiment's work item on the board:

| Argument | What it is |
| --- | --- |
| `hypothesisIds`, `experimentDefinitionId` | Every hypothesis the experiment tests, one or more, and the card from their shortlist. Each hypothesis must be on the Assumptions Map — scored, `Prioritized` (or already `Testing`) — and the card must give evidence for every one's concern. Several share an experiment only when the same people, in the same sitting, answer each. |
| `testName`, `testDescription` | What will be done, to verify the hypotheses. |
| `assignedToUserId`, `deadline` (yyyy-MM-dd), `plannedDuration` (ISO 8601, `P3DT4H`) | Optional plan details. |
| `plannedDurationInWords` | How long it runs as the founder says it, a few words on one line of at most 80 characters: `"one week of evenings"`. The board's card reads it after the planned count ("Designed · 10 developers, one week of evenings") and the Plan part reads it too; without it both read `plannedDuration`. Send it whenever the founder said how long in words. |
| `whoDoesIt`, `whatIsMeasured` | The plan's two sentences, markdown: "You hold the conversations. Ian writes each one down the same day." and "How many name getting paying users, unprompted." The experiment's Plan part shows them under Who and We measure; without them it names the assignee and the metrics. |
| `plannedEvidenceCount`, `evidenceUnitSingular`, `evidenceUnitPlural` | How many bundles of evidence the test gathers, 1 or more, with what one is and what several are, at most 40 characters each: `10`, `"conversation"`, `"conversations"`. The three together or none. One bundle is one `record_evidence` call, so the count is of bundles. The board's card reads "5 of 10 conversations"; the Evidence part draws the bundles not gathered yet. `get_experiment` answers them in `testCard`, and the experiment's item the count beside `evidenceCount`. An open task under the experiment's item (create_work_item with `parentWorkItemId` its item) books the bundles not gathered yet: the item answers its code in `bookedByTaskReferenceCodes`, oldest first, which reads "Five conversations not held yet. They are booked by W-01."; no open task, no sentence. |
| `metrics` | What we measure: `[{ "name": "Interviewees naming the pain unprompted", "unit": "out of 10" }]`. |
| `criteria` | When we are right: `[{ "description": "At least 8 of 10 name it unprompted", "metricName": "Interviewees naming the pain unprompted", "hypothesisId": "…", "saysYesWords": "named it", "saysNoWords": "did not" }]`. `saysYesWords` and `saysNoWords` are what a bundle reads on the criterion when it supports the hypothesis and when it contradicts it, a few words of at most 40 characters, both or neither; the evidence rows and the "So far" line print them instead of "says yes" / "says no". One per hypothesis at least, each naming the hypothesis it judges; on an experiment that tests one hypothesis `hypothesisId` may be left out and the criterion judges that one. `metricName` must match one of the metrics; leave it out for a qualitative criterion. |

The criteria come from the procedure `design-the-experiments` and, inside an operation, from its
"validated when" sentence; the `mentat-experiment` skill writes them. An experiment does not start
while one of its hypotheses has no criterion. If a metric or criterion is refused after the
experiment was created, the experiment exists with what came before it: read it with
`get_experiment` and finish with `add_metric` and `add_criterion` (`hypothesisId`, `saysYesWords` and
`saysNoWords` the same way).

**The plan is rewritten whole while the experiment is `Designed`** — `rewrite_test_card` with
`experimentId` and every argument of the plan above, from `testName` to `evidenceUnitPlural`. It replaces
the plan as sent: read `get_experiment` first and send again every part of its `testCard` that stays,
since a part left out is cleared. It is how an experiment designed without `plannedDurationInWords`, `whoDoesIt`,
`whatIsMeasured` or a planned count gets them. Metrics and criteria are untouched; `add_metric` and
`add_criterion` add to them. Refused once the experiment has started.

An experiment's hypotheses are fixed once it starts. While it is `Designed`, a person adds or takes
one off on the experiment's page; no tool here does, and an experiment always tests at least one.

## 2. Run — `start_experiment`, `record_observation`, `judge_criterion`, `record_spend`

`start_experiment` moves the experiment to Running and every hypothesis it tests to Testing. As it
goes:

- `record_observation` puts a reading on a metric: a number (`observedValue`), a word
  (`observedText`), or both. Metric ids come from the designed experiment.
- `judge_criterion` records whether one criterion was cleared. Every criterion must be judged before
  completion.
- `record_spend` records money (amount and currency together) and hours.

## 3. Evidence — `record_evidence`

One bundle per unit the plan counts — one per conversation of "10 conversations" — or, on a plan that
counts none, one per kind of signal gathered, recorded in one call:

| Argument | Notes |
| --- | --- |
| `title` | The short title the bundle is listed under, a name on one line of at most 80 characters: "Conversation 1 · video call". Never what it found: that is the summary. Write one for every bundle; left out, the experiment's page names the bundle by its code and kind. |
| `experimentId`, `kind`, `collectedAtUtc`, `summary` | Kinds: Quote, Behavior, ConversionRate, Order, Purchase, BidReceived, SearchVolume, RelatedQuery, PageSession, DropOff, AttentionSignal, FeatureRequest, Workaround, NearMissFeedback, SupportTicket, Other. |
| `hypotheses` | Required. Each hypothesis of the experiment the bundle bears on, once, as `{ hypothesisId, bearing }`: `bearing` is `Supports`, `Contradicts` or `SaysNothingEitherWay`, fixed once recorded. An empty list means every hypothesis the experiment tests, each `SaysNothingEitherWay`. A hypothesis named twice, or one the experiment does not test, is refused. A hypothesis's confidence counts only the bundles that bear on it, and the experiment's page counts the bearings as each hypothesis's "So far": `Supports` a yes, `Contradicts` a no, the third neither. |
| `strength` | 1 to 5. Leave it out to inherit the card's rating; override when what was gathered is worth more or less. |
| `polarity`, `modality`, `setting`, `investmentSize` | The book's four axes: Opinion/Fact, Say/Do, Lab/RealWorld, Small/Large. Leave an axis out when it does not apply. |
| `dataPoints` | The readings behind the bundle, each with `source` and a `textValue`, `numericValue` or `extra`. The confidence rollup weights the bundle by how many there are, so record every interviewee, every conversion. |

A bundle answers `hypotheses` the same way, each with its bearing; a bundle recorded before bearings were
written reads `SaysNothingEitherWay` on each. `update_evidence` does not change a bearing.

`update_evidence` retitles (`title` alone is how a bundle recorded without one gets its name), re-rates, re-characterises or re-summarises; `add_data_point` appends a reading;
`delete_evidence` is for a bundle recorded by mistake, not for a weak one.

## 4. Complete — `complete_experiment`

One call writes the learning card and ends the experiment:

- Learning card: `insightName`, `learnedOn` (yyyy-MM-dd), `personResponsibleUserId`, `observation`
  ("We observed …"), `learningsAndInsights` ("From that we learned …"), optional
  `decisionsAndActions` ("Therefore we will …").
- `verdicts`: one per hypothesis, `[{ "hypothesisId": "…", "verdict": "Validated" }]`, each judged
  by that hypothesis's own criteria alone: `Validated` needs every one of them met; `Invalidated`
  needs at least one missed; `Inconclusive` needs `notes` saying why that part cannot be read. A
  verdict a hypothesis's criteria contradict is refused, naming the hypothesis and the criterion.

Each hypothesis's confidence recomputes from the completed experiment. The answer lists the goals the
experiment serves as goals to review: what it showed may change them. `abort_experiment` stops an
experiment without a verdict and its evidence is no longer read; `delete_experiment` removes one that
never started, with its work item.

## After the verdicts

The decision on each hypothesis is the person's; see hypotheses.md.
