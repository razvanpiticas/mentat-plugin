# The build check's numbers, the rows to add, and the escalation

Load this when presenting the build check, choosing the rows to add after a failed one, or writing to
the inbox. Which of these a routine's run may do is in the skill body, not here.

## A gate's table

Drawn from one entry of `get_gate_status.gates`, in the tool's order, one row per block it checks:

> **{code} {name}** — {question}

| Block | Confidence | Bar | Clears | Short by | Holds entries |
| --- | --- | --- | --- | --- | --- |
| {blockCode} {blockName} | {confidence, two decimals} | {threshold} | yes / no | {shortfall, two decimals; "—" when it clears} | yes / no |

Below it, one line per entry of the gate's `fitPairShortfalls`, in the tool's order — the gate that
reads the pairs (the Value gate) fails on each, whatever its blocks are worth:

> Not passed: {referenceCode} has no value proposition of ours at problem–solution fit

Then: "Would pass: yes/no. Weakest block: {weakestBlockCode}. Operation to run: {position}
{slug}, because {the reason in words}." The reason in words: `WritesTheBlock` — "it writes
{block}, which holds no entry"; `DesignsTheTests` — "{block}'s hypotheses wait for a test";
`RunAgain` — "it wrote {block} last". A gate that would pass has no weakest block and no operation
to run; a gate failed on the pairs alone has no weakest block, and its row is chosen as
[the row on a gate failed on the pairs alone](#the-row-on-a-gate-failed-on-the-pairs-alone) says.

## The build line

"Build {held | waits for your unlock | unlocked}. {gatesThatWouldPass} of 6 gates would pass;
{blocksAtTheirBar} of {blockCount} blocks at their bar; {hypothesesWaitingForATest} hypotheses wait
for a test. {When due: every row below the build is done, so the check can be recorded now. | Else:
{rowsLeftBeforeTheCheck} rows of the plan are left before the check.} Failed checks in a row:
{consecutiveFailures}. Last check: {passed | failed} on {latestCheck.checkedAtUtc, as a date}."

## The rows to add after a failed check

1. **The operations to run again.** Each failed gate's `operationToRun`, by its `slug`; a gate failed
   on the pairs alone has none, and its row is chosen as the next section says. One row per distinct
   operation, in the order of the positions they name.
2. **The mapping row after them.** One row of `hypotheses-mapping-and-experiments-design`, last:
   whatever the rows run again change, its hypotheses are mapped again and the experiments designed
   from them. When an operation to run is the mapping operation itself (`DesignsTheTests`), it is
   that one row, last, and no second one is added.
3. **The ids.** `list_operation_definitions(projectId)` answers each operation's `id` by its `slug`:
   that is the `operationDefinitionId` `append_roadmap_operation` takes.
4. **The tier.** Each row takes the tier of the row it runs again — `get_roadmap` with that row's
   position as `fromPosition`, the first row answered. The mapping row takes the tier of the last row
   before it, or, when it is the only one, the tier of the mapping row the gate named.
5. **Append, then place.** `append_roadmap_operation(projectId, roadmapVersion,
   operationDefinitionId, tier, runId)` for each, in order, the mapping row last: each answers the
   plan with the new row at its end and the next `roadmapVersion`. A row appended at the end stands
   after the build's rows, where nothing runs while the build is held; so `place_roadmap_item(projectId,
   roadmapVersion, itemId, position, tier, runId)` puts each one right after the last row of `T0` to
   `T6`, the first first, each answer carrying the version the next call sends.
6. **Report** the rows by position and slug, and the project's tier, read with `get_project`: it
   stands at the tier of the first of them.

## The row on a gate failed on the pairs alone

A segment to win — an accepted beachhead, or a segment that resells to one — is won by our value
proposition aimed at its end user profile, or at one of its context profiles, at problem–solution
fit. `concept-and-use-case` creates our value proposition and aims it at the profile;
`value-proposition` writes its value map and reaches the fit.

1. The segment is the first line of the gate's `fitPairShortfalls`.
2. `get_customer_segment` with its `segmentId`: the ids of its end user profile and context profiles.
   `get_block` VP lists our value propositions among its cards; `get_value_proposition` on each, with
   the section `FitCheck`, answers the profile it aims at in its header and, in its fit check,
   `nextKindOfFit`. Only one whose `isOursToDesign` is true counts:
   our business as it is today is mapped, never designed, and never reaches a fit.
3. None of ours aims at one of those profiles: the operation is `concept-and-use-case`. One does and
   its `nextKindOfFit` is still `ProblemSolution`: the operation is `value-proposition`.
4. That operation is run again as the section above says, the mapping row after it. No row of the
   plan holds it: say "no row of the plan writes {CS code}'s value proposition; the planner is the
   place to add one."

Every block of that gate cleared its bar, so the business model held there, and what is missing is
the offer on that segment.

## A pivot

A person may ask for one at any time. From `list_pivot_definitions`: for each, the name, what changes
and the rows it puts back on the plan; recommend the one whose `revisits` include the operation to run
again, or, when several do, the one with the fewest revisits, and say when none fits. The tier to go
back to: the tier of the earliest row the pivot revisits on this plan, never below `T0`. Applied only
on the person's instruction.

## Escalation: the build check failed

Its row is in `mentat-inbox`'s `reference/asks.md` ("The build check failed"). It stands against the
project with no subject row. Through `mentat-inbox`, `raise_escalation`, `runId` set; then
`observe_run` with the `inboxItemId` it hands back, and `end_run` with the digest.

Title: `The build check failed: {failed gate names}` — the weakest block of the first after its name
when there is one, such as `The build check failed: Go-to-market (Channels 0.40 against 0.70)`.

Body:
> {each failed gate's table, with its "Not passed" lines} Recorded: build check {id}, {consecutive
> failures} failed in a row. Each failed gate, its weakest block and the operation to run: {one line
> each}. Rows I would add to the plan, right after the last row below the build: {each operation to
> run again by slug, with its tier}, then a row of hypotheses-mapping-and-experiments-design, so the
> experiments are designed again from what those rows change. Reply "add the rows" and the next firing
> adds them, add them yourself on the roadmap screen, or say what to do instead. Acknowledging without
> a word adds them.

## Reading the answer, next firing

The brief's "answered since the last run" carries the escalation and the person's note.
Acknowledged with no note, or "add the rows" → step 6 of the loop, unattended, adding exactly the rows
the body named, and the digest names the escalation they answer. A note that says "apply {pivot name} to {tier}" → `get_roadmap` for the version,
`list_pivot_definitions` for the id by name, `apply_pivot` with that id, that tier, notes "Applied on
the person's answer to escalation {id}", `runId`. Any other note is done as it says; a note that only
declines adds nothing, and the digest says what was answered. A pivot is never applied on a bare
acknowledgement: its words are only ever the person's.
