# Hand-offs: what comes in from an operation, what goes back, what goes on to a test

## From `mentat-operation` (in)

The operation invokes this skill when its package's validation level is `Experiment` and the
checkpoint of its last own step answered the hand-over. It passes, in words, in this order — the last
two rows only when the hand-over carries no test order:

| What | Where it came from |
| --- | --- |
| project id, `runId`, `canvasVersion` | the run the operation is working inside; the version its last write answered |
| the row of the plan: id, slug, position | the operation's package (get_operation_package) |
| the Hypothesise step's code | the package's `sharedSteps`: the step named Hypothesise |
| the hand-over, exactly as it was answered: "Hypothesise now", the "validated when" sentence, the operation's own instruction | the checkpoint of the operation's last own step |
| every test order: its number, its status — `Open`, or waiting for an earlier order — and, when open, its entries by id and code | the same checkpoint |
| only when the hand-over carries no test order: the target slots, each a block code and an entry kind code | the package's `resolvedTargetSlots` |
| only when the hand-over carries no test order: the entries the run's own steps wrote, by id and code | the run's `writtenEntryIds`, as the same checkpoint answered them, with the codes the writes answered |

Read the Hypothesise step (`get_procedure` with that code) yourself. What you write about depends on
whether the hand-over carries a test order.

**One or more test orders.** Read the block of every entry handed over. Write one hypothesis about
each entry of every **open** test order — `aboutEntryId` the entry's id, on the block its code names —
and, where an order hands over a value proposition itself, one about that value proposition. The
operation's instruction says what they are about ("Write hypotheses for the top job. Once it holds,
write them for the top 3 pains and the top 3 gains."); the "validated when" sentence is what they must
make true. A waiting order is not written: its entries are picked from the canvas as it stands when
it opens, and the operation reads them then.

**No test order.** An operation that names no hand-over lines answers "Hypothesise now" and the
"validated when" sentence, with no test order and no instruction of its own. Read every target slot's
block and the block of every entry the run wrote. The beliefs to write are the ones the "validated
when" sentence depends on and the ones the target blocks' descriptions name; the entries the run wrote
are the ones they are most often about. A hypothesis about one of them carries its id as
`aboutEntryId`; one about none of them carries no `aboutEntryId`. A run whose steps wrote no entry
still writes the beliefs the sentence depends on.

## To `mentat-operation` (back)

| What | Why |
| --- | --- |
| the hypotheses you wrote: code and id, each, with the entry it is about, when it is about one | the operation sends the ids as `producedHypothesisIds` on the Hypothesise checkpoint |
| the ones you found already there and left alone: code, each | so the report does not count them as written |
| the reason behind each score, per hypothesis | unattended nobody confirmed them, and this is where those reasons live: the run's owner puts them in the note and the report, because you never write a run note yourself |
| the shortlist per hypothesis: card slugs | the operation's report, and the next step when the person wants a test |
| the canvas version you last held | the operation's next write |

You never call checkpoint_run, observe_run, pause_run or end_run from here: the run is the
operation's, and a checkpoint belongs to one of its steps.

## To `mentat-experiment` (on)

Only on the person's word, attended. Pass the hypothesis (id, code), the card the person picked
(id, slug), `runId` and `canvasVersion`, and — when the hypothesis came from an operation — that
operation's `validatedWhen`, so the test's criteria are written against it. Unattended, **you** pass
nothing on: the shortlist is what you hand back. The skill that owns the run hands it over instead —
`mentat-operation` invokes `mentat-experiment` itself on the hypothesis your shortlist ranks first,
with the same payload — and `mentat-experiment`'s own rule decides whether the card it designs
starts or waits for the person.
