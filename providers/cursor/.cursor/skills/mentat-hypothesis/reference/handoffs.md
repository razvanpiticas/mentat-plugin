# Hand-offs: what comes in from an operation, what goes back, what goes on to a test

## From `mentat-operation` (in)

The operation invokes this skill when its package's validation level is `Experiment`. It passes,
in words, in this order:

| What | Where it came from |
| --- | --- |
| project id, `runId`, `canvasVersion` | the run the operation is working inside; the version its last write answered |
| the row of the plan: id, slug, position | `get_operation_package` |
| `validatedWhen` | the package's definition: what the operation has to make true |
| the target slots: block code and entry kind code, each | the package's resolved target slots |
| the theme question | the package |
| the entries the operation's procedures wrote in this run, by code | the operation's own checkpoints |

Read `P.V1` and every target block yourself. The beliefs to write are the ones "validated when"
depends on and the ones the target blocks' descriptions name; the entries the operation just
wrote are the ones the hypotheses are most often about.

## To `mentat-operation` (back)

| What | Why |
| --- | --- |
| the hypotheses you wrote: code and id, each | the operation sends the ids as `producedHypothesisIds` on its checkpoint |
| the ones you found already there and left alone: code, each | so the report does not count them as written |
| the reason behind each score, per hypothesis | unattended nobody confirmed them, and this is where those reasons live: the run's owner puts them in the note and the report, because you never write a run note yourself |
| the shortlist per hypothesis: card slugs | the operation's report, and the next step when the person wants a test |
| the canvas version you last held | the operation's next write |

You never call `checkpoint_run`, `observe_run`, `pause_run` or `end_run` from here: the run is the
operation's, and a checkpoint belongs to one of its procedures.

## To `mentat-experiment` (on)

Only on the person's word, attended. Pass the hypothesis (id, code), the card the person picked
(id, slug), `runId` and `canvasVersion`, and — when the hypothesis came from an operation — that
operation's `validatedWhen`, so the test's criteria are written against it. Unattended, **you** pass
nothing on: the shortlist is what you hand back. The skill that owns the run hands it over instead —
`mentat-operation` invokes `mentat-experiment` itself on the hypothesis your shortlist ranks first,
with the same payload — and `mentat-experiment`'s own rule decides whether the card it designs
starts or waits for the person.
