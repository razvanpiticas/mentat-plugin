# Hypotheses: writing, scoring, shortlisting, deciding

A hypothesis is a sentence about the business model that an experiment can settle. It lives on one
block, may be about one entry of that block, and carries a polarity and a business concern. The
reference code is `H-<block code>-<n>`, such as `H-CS-002`.

## Write it well

`create_hypothesis` takes `weBelieve`, `polarity`, `businessConcern` and optionally `aboutEntryId`.

- **One sentence, "We believe that …"**, sent as `weBelieve`.
- **Polarity** — `Positive` or `Negative`.
- **Business concern** — `Desirability`, `Feasibility` or `Viability`. It is a hard filter on which
  method cards can test the hypothesis.

What each of these means, and how a hypothesis is written, is the procedure `P.V1` the
`mentat-hypothesis` skill fetches; this file says only what the call takes.

Example: on CS, about the pain `CS-04`, Desirability, Positive: "We believe that freelance designers
who bill by the hour lose at least two billable hours a week to tracking their time by hand."

## Score it and pass the quality checks

`update_hypothesis` applies three groups, each sent whole:

| Group | Arguments | What BusinessIntelligence enforces |
| --- | --- | --- |
| Wording | `weBelieve`, `polarity`, `businessConcern` | Refused once the hypothesis is scored: rewrite before scoring. |
| Quality | `isTestable`, `isPrecise`, `isDiscrete` | All three true is what allows scoring. |
| Scores | `importance`, `evidence` | Refused while a quality check fails. |

Both run −5 to 5, and the direction of each is fixed by the domain, not chosen by the sender:
`importance` is −5 when the business model barely depends on the sentence and 5 when it rests on
it; `evidence` is **−5 when there is no evidence at all, or what there is points against the
hypothesis**, and **5 when there is strong, recent evidence for it**. A hypothesis nobody has
found anything out about is scored −5 on evidence, never 5. `P.V1` says how to judge the two;
this file says only which way the numbers run. Send the groups in one call when you have them
all; the tool applies them in the order above.

## Shortlist a method

`list_experiment_definitions` with the hypothesis's `businessConcern` answers the cards that can
test it, each with its evidence strength, cost, setup time and run time on a 1 to 5 scale;
`get_experiment_definition` shows one whole, `bestFor` among it.
`recommend_experiment_definition` puts a card on the hypothesis's shortlist;
`withdraw_experiment_definition` takes it off. A card whose concerns do not include the
hypothesis's is refused.

## Statuses and the calls that move them

| Status | Reached by |
| --- | --- |
| `Drafted` | `create_hypothesis`; `unpark_hypothesis` |
| `Prioritized` | scoring through `update_hypothesis` |
| `Testing` | `start_experiment` on a run designed against it |
| `Validated` / `Invalidated` | completed experiments and the confidence they roll up |
| `Parked` | `park_hypothesis` (refused while a run is in flight) |
| `Retired` | `retire_hypothesis` with a reason (refused for a hypothesis under test) |

Confidence is computed from the hypothesis's completed, non-inconclusive experiments: a weighted
average of evidence strength by data-point count, plus a small bonus per extra experiment, on a 1 to
5 scale. It is never written directly.

## The decision is the founder's

`decide_hypothesis` records `Persevere`, `Pivot`, `Kill` or `ContinueTesting` with the reasoning, and
is refused until a run has started. It is the person's call across the experiments that ran. Lay out
the learning cards and the confidence, ask, then record what they said — in their words for the
reasoning.
