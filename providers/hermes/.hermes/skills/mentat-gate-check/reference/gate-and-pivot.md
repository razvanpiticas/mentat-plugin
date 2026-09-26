# The gate's numbers, the next row, and the two escalations

Load this when presenting a gate, choosing the next row on a fail, or writing to the inbox. Which of
these a routine's run may do is in the skill body, not here.

## The scores table

Drawn from `get_gate_status.currentScores`, one row per checked block, in the tool's order:

| Block | Confidence | Bar | Clears | Short by |
| --- | --- | --- | --- | --- |
| {blockCode} {blockName} | {confidence, two decimals} | {threshold} | yes / no | {shortfall, two decimals; "—" when it clears} |

Below it: "Would pass: yes/no. Rows of {tier} pending: {n} ({slugs}). Consecutive failures here: {n}.
Last evaluation: {passed/failed} on {date}, next row {slug}." A gate with no checked block (the last
gate) has no table: "This gate checks no block; a person passes it by saying so."

## The next row on a fail

1. The weakest block is the score with the largest `shortfall`.
2. Candidates: `get_roadmap` rows that are operations, not skipped, whose `tier` is the project's tier
   or earlier.
3. For each candidate of the project's tier, `get_operation_definition` with the project id and the
   row's `operation.operationDefinitionId` — the tool takes that id, not the slug — and keep the ones
   whose `targetSlots` name the weakest block's code. Earlier tiers only when the project's tier
   yields none.
4. Pick the one with the latest completed run (`operation.latestRun.endedAtUtc`); with none completed,
   the first by position. Its row id is `nextOperationId`.
5. None at all: send no `nextOperationId`; say "no row of the plan writes to {block code}; the
   planner is the place to add one."

## The pivot to propose

From `list_pivot_definitions`: the one whose `revisits` include the next row's operation, or, when
several do, the one with the fewest revisits; when none does, say so and propose none. The tier to go
back to: the tier of the earliest row the pivot revisits on this plan, never below `T0`.

## The two escalations

Their rows are in `mentat-inbox`'s `reference/asks.md` ("Gate failed, pivot proposed" and "Gate
would pass with rows pending"), and both stand against the project with no subject row, as that
table has them. Through `mentat-inbox`, `raise_escalation`, `runId` set; then `observe_run` with the
`inboxItemId` it hands back, and `end_run` with the digest.

## Escalation: would pass with rows pending

No subject (the project itself):

Title: `The {gate name} gate would pass with {n} row(s) of {tier} still pending`

Body:
> {the scores table} Pending: {pending slugs}. Evaluating now moves the project under {to tier}'s
> policy before those rows are run. Reply "evaluate" to record it on the next firing, or "wait" to
> leave it until the rows are done.

## Escalation: the gate failed

No subject (the project itself, as `asks.md` has it):

Title: `The {gate name} gate failed: {weakest block name} at {confidence} against {threshold}`

Body:
> {the scores table} Recorded: evaluation {id}, {consecutive failures} consecutive failure(s) at this
> boundary. Next row proposed and recorded on the evaluation: {next operation slug}, because it fills
> {weakest block code}. Pivot I would propose: {pivot name} — {what changes}; it puts {revisit slugs}
> back on the plan and sends the project back to {tier}. Alternative: run {next operation slug} again
> and re-evaluate. Reply "apply {pivot name} to {tier}" and the next firing applies it, apply it
> yourself on the roadmap screen, or reply "no pivot". Acknowledging without a word applies nothing.

## Reading the answer, next firing

The brief's "answered since the last run" carries the escalation and the person's note. "evaluate" →
step 5 of the loop, unattended; acknowledged without a note → the proposed answer, "wait", stands.
"apply {pivot name} to {tier}" → `get_roadmap` for the version, `list_pivot_definitions` for the id
by name, `apply_pivot` with that id, that tier, notes "Applied on the person's answer to escalation
{id}", `runId`. Any other answer, and a bare acknowledgement, → nothing is applied; the digest says
what was answered. This is the one ask where `mentat-inbox`'s rule "acknowledged without a note, the
proposed answer stands" does not apply: a pivot is never applied without a person's instruction.
