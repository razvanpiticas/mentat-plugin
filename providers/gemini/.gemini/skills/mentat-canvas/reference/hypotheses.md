# Hypotheses: writing, scoring, shortlisting, deciding

A hypothesis is a claim about the business model that an experiment can settle. It lives on one
block, may be about one entry of that block, and carries a polarity and a business concern. The
reference code is `H-<block code>-<n>`, such as `H-CS-002`.

## Write it well

`create_hypothesis` takes `weBelieve`, `polarity`, `businessConcern` and optionally `aboutEntryId`.

- **One sentence, "We believe that …".** Testable (an experiment can show it false), precise (who,
  what, how much), discrete (one claim, not two joined by "and").
- **Polarity** — `Positive` when the experiment tries to prove it, `Negative` when it tries to
  disprove it.
- **Business concern** — `Desirability` (do they want it), `Feasibility` (can we do it),
  `Viability` (does the money work). It is a hard filter on which method cards can test the claim,
  so choose it by what the claim is really about.

Example: on CS, about the pain `CS-04`, Desirability, Positive: "We believe that freelance designers
who bill by the hour lose at least two billable hours a week to tracking their time by hand."

## Score it and pass the quality checks

`update_hypothesis` applies three groups, each sent whole:

| Group | Arguments | What BusinessIntelligence enforces |
| --- | --- | --- |
| Wording | `weBelieve`, `polarity`, `businessConcern` | Refused once the claim is scored: rewrite before scoring. |
| Quality | `isTestable`, `isPrecise`, `isDiscrete` | All three true is what allows scoring. |
| Scores | `importance`, `evidence` | Refused while a quality check fails. Both −5 to 5. |

Importance answers "if this is wrong, how bad?": 5 is fatal to the business, −5 is harmless.
Evidence answers "how much do we already know?": 5 means no evidence, test first; −5 means plenty.
The top-right of the map — important and unproven — is what gets tested first. Send the groups in
one call when you have them all; the tool applies them in the order above.

## Shortlist a method

`list_experiment_definitions` with the claim's `businessConcern` answers the cards that can test it,
each with its evidence strength, cost, setup time and run time on a 1 to 5 scale. Rank by fit to the
claim's wording (`get_experiment_definition` shows `bestFor`), evidence strength against importance
(a fatal claim deserves strong evidence), and cost against what the person can spend.
`recommend_experiment_definition` puts a card on the claim's shortlist; `withdraw_experiment_definition`
takes it off. A card whose concerns do not include the claim's is refused.

## Statuses and the calls that move them

| Status | Reached by |
| --- | --- |
| `Drafted` | `create_hypothesis`; `unpark_hypothesis` |
| `Prioritized` | scoring through `update_hypothesis` |
| `Testing` | `start_experiment` on a run designed against it |
| `Validated` / `Invalidated` | completed experiments and the confidence they roll up |
| `Parked` | `park_hypothesis` (refused while a run is in flight) |
| `Retired` | `retire_hypothesis` with a reason (refused for a claim under test) |

Confidence is computed from the claim's completed, non-inconclusive experiments: a weighted average
of evidence strength by data-point count, plus a small bonus per extra experiment, on a 1 to 5
scale. It is never written directly.

## The decision is the founder's

`decide_hypothesis` records `Persevere`, `Pivot`, `Kill` or `ContinueTesting` with the reasoning, and
is refused until a run has started. It is the person's call across the experiments that ran. Lay out
the learning cards and the confidence, ask, then record what they said — in their words for the
reasoning.
