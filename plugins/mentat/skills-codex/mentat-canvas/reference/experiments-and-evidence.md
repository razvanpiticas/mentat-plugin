# Experiments and evidence

An experiment is one run of a method card against one claim. Its reference code is `T-<n>`; a bundle
of evidence is `E-<n>`. The order below is the order BusinessIntelligence enforces; a call out of
order is refused with the rule it broke.

## 1. Design — `design_experiment`

One call plans the whole test card:

| Argument | What it is |
| --- | --- |
| `hypothesisId`, `experimentDefinitionId` | The claim and the card from its shortlist. The claim must be scored (Prioritized). |
| `testName`, `testDescription` | What will be done, to verify the claim. |
| `assignedToUserId`, `deadline` (yyyy-MM-dd), `plannedDuration` (ISO 8601, `P3DT4H`) | Optional plan details. |
| `metrics` | What we measure: `[{ "name": "Interviewees naming the pain unprompted", "unit": "out of 10" }]`. |
| `criteria` | When we are right: `[{ "description": "At least 8 of 10 name it unprompted", "metricName": "Interviewees naming the pain unprompted" }]`. `metricName` must match one of the metrics; leave it out for a qualitative criterion. |

Derive the criteria from what would validate the claim — the step's "validated when" sentence when
you are running a method step. A run cannot start with no criterion. If a metric or criterion is
refused after the experiment was created, the experiment exists with what came before it: read it
with `get_experiment` and finish with `add_metric` and `add_criterion`.

## 2. Run — `start_experiment`, `record_observation`, `judge_criterion`, `record_spend`

`start_experiment` moves the run to Running and the claim to Testing. As the run goes:

- `record_observation` puts a reading on a metric: a number (`observedValue`), a word
  (`observedText`), or both. Metric ids come from the designed experiment.
- `judge_criterion` records whether one criterion was cleared. Every criterion must be judged before
  completion.
- `record_spend` records money (amount and currency together) and hours.

Much of a run is done by people — interviews, a landing page, a concierge delivery. Do the parts you
can, and say plainly which parts the person has to do and what to bring back.

## 3. Evidence — `record_evidence`

One bundle per kind of signal gathered, recorded in one call:

| Argument | Notes |
| --- | --- |
| `experimentId`, `kind`, `collectedAtUtc`, `summary` | Kinds: Quote, Behavior, ConversionRate, Order, Purchase, BidReceived, SearchVolume, RelatedQuery, PageSession, DropOff, AttentionSignal, FeatureRequest, Workaround, NearMissFeedback, SupportTicket, Other. |
| `strength` | 1 to 5. Leave it out to inherit the card's rating; override when what was gathered is worth more or less. |
| `polarity`, `modality`, `setting`, `investmentSize` | The book's four axes: Opinion/Fact, Say/Do, Lab/RealWorld, Small/Large. Leave an axis out when it does not apply. |
| `dataPoints` | The readings behind the bundle, each with `source` and a `textValue`, `numericValue` or `extra`. The confidence rollup weights the bundle by how many there are, so record every interviewee, every conversion. |

Strongest evidence is what people **did**, in the **real world**, at **large** investment, as
**fact**; a quote from an interview is Say, Opinion, Lab, Small. Rate honestly. `update_evidence`
re-rates, re-characterises or re-summarises; `add_data_point` appends a reading; `delete_evidence`
is for a bundle recorded by mistake, not for a weak one.

## 4. Complete — `complete_experiment`

One call writes the learning card and ends the run:

- Learning card: `insightName`, `learnedOn` (yyyy-MM-dd), `personResponsibleUserId`, `observation`
  ("We observed …"), `learningsAndInsights` ("From that we learned …"), optional
  `decisionsAndActions` ("Therefore we will …").
- `verdict`: `Validated` needs every criterion met; `Invalidated` needs at least one missed;
  `Inconclusive` needs `notes` saying why the run cannot be read.

The claim's confidence recomputes from the completed run. `abort_experiment` stops a run without a
verdict and its evidence is no longer read; `delete_experiment` removes a designed run that never
started.

## After the verdict

Validated moves the claim's confidence up; Invalidated moves it down and is a pivot signal worth
saying out loud; Inconclusive means redesign — two retries, then the claim is deferred. The decision
on the claim (`decide_hypothesis`) is the founder's; see hypotheses.md.
