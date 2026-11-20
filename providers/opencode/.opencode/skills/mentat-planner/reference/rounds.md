# The five rounds, field by field

Load this when running a round. The skill carries the shape; this carries what each round reads, asks and writes, with the exact arguments.

## Before round 1 — the reads

| Read | Fields kept |
| --- | --- |
| `list_goals(projectId)` | the mission node: `id`, `referenceCode`, `title`, `metricName`, `metricUnit`, `metricTarget`, `metricDirection`, `deadline`, `status`, `latest` (value, asOf, source), `isMet`; each child: the same plus `parentGoalId`, `servingItems` (id, referenceCode, kind, title, state), `restsOn`, `reviews` and `proposedChange` |
| `get_roadmap(projectId)`, again with `fromPosition` while `rowsLeftOut` is above 0 | `thesis`, `roadmapVersion`, `projectTier`, `tiers` (each `tier`, `dueOn`, `daysOverdue`, `rowCount`, `unfinishedRowCount`), `buildCheck` (`standing`, `isDue`, `rowsLeftBeforeTheCheck`), every row in short: `id`, `kind`, `position`, `tier`, `isSkipped`, `hasNotes`, `servesGoalCodes`; on an operation row `slug`, `title`, `status`, `waitsOnSlugs`, `targetBlockCodes`, and only where they hold one, `revisitOfSlug` and `pivotName`; on a move row `ideaId`, `ideaReferenceCode`, `ideaTitle`, `blockCode`, `ideaStatus`. The level of each operation is `list_operation_definitions`' `validationLevel`; a skipped or noted row's `skipReason` and `notes`, and a row's prerequisites by kind, come from `get_roadmap(projectId, itemId)`, the row whole |
| `get_canvas(projectId)` | `canvasVersion`, `overallConfidence`; per block: `id`, `code`, `name`, `description`, `concern`, `coverage`, `confidence`, and the counts of `entries`, `hypotheses`, open `questions`, `risks`, `ideas`; each idea: `id`, `referenceCode`, `title`, `effort`, `impact`, `status` |
| `get_gate_status(projectId)` | `standing`, `isDue`, `rowsLeftBeforeTheCheck`, `gatesThatWouldPass`, `consecutiveFailures`; per gate `name`, `wouldPass`, `weakestBlockCode` |
| `list_decisions(projectId)`, and again with `status: "Proposed"` | codes and titles; the last one titled "Plan v…", accepted or still proposed |
| `list_work_items(projectId)` | open items: code, type, title, status, and an operation's item's plan row |
| `list_operation_definitions(projectId)` | `id`, `slug`, `title`, `themeCode`, `validationLevel`, `source` |

## Round 1 — the mission and its objectives

Show: "`G-01` — <title>. Counted in <metricName> (<unit>), target <target> <direction>, by <deadline>. Latest: <value> as of <asOf>, from <source>." then one line per objective in the same shape, or "This project has no mission."

Propose, one recommendation: the mission as the ambition — "<number> <unit> by <date>", where the business wants to be — and two to five objectives as outcomes in the market — customers won, money earned, what customers do: sign up, pay, come back, tell others — each with its number and date, adding up to the mission.

Ask, one batch: does each stand as written; if not, the title, what is counted, the unit, the target and whether it is at least or at most, the date.

When the person asks for an objective that is a count of work done — "10 founder conversations held" — say: "A count of work done is the pass line of an experiment: the mapping operation sets it as an experiment's success criterion. I would leave it there." Write it only when the person still asks for it. When they ask for a block's confidence — "Value Proposition confidence at least 0.7" — say: "The gates already check every block's confidence, together, once, before the build." Propose no objective for it.

Write, after the yes:
- a new mission: `create_goal(projectId, title, description?, metric: { name, unit?, target, direction: AtLeast | AtMost }, deadline: yyyy-MM-dd, runId)` — no `parentGoalId`.
- a new objective: `create_goal(projectId, parentGoalId: <mission id>, title, description?, metric: { name, unit?, target, direction }, deadline, runId)`.
- a goal that exists, changed: `propose_goal_change(projectId, kind: "Change", goalId, title?, description?, metric?, deadline?, reason, figures: [{ label, was, becomes }], runId)`; abandoned: `kind: "Abandon"` with `goalId` and the reason. A goal counted in another unit is not changed: `kind: "NewObjective"` with `parentGoalId`. Each lands as a proposed decision the person accepts on the decisions tab; until then the goal reads as it did.

## Round 2 — the thesis

Show, one line per block: "<code> <name> — <coverage>, confidence <confidence>, <n> entries, <n> hypotheses, <n> open questions, <n> risks, <n> ideas." Then the block's description in one sentence.

Propose: one belief per block, "For <block> to hold, <belief>." Then the paragraph: the beliefs in the order of how much the business depends on each, opening with the mission's number.

Ask, one batch: the edits to each belief; the order; anything missing.

Write: `set_roadmap_thesis(projectId, roadmapVersion, thesis, runId)` — under 5,000 characters. Keep the answered `roadmapVersion`.

## Round 3 — the tiers and their dates

Show, one line per tier that holds rows: "<tier> <its name> — <rowCount> rows, <unfinishedRowCount> not finished, <by <dueOn> | no date>." The tier names are in `mentat-canvas`'s `reference/roadmap-and-runs.md`.

Propose: a date for the tiers whose work the mission's or an objective's date turns on, worked back from those dates; no date for the rest. A tier needs no date.

Ask, one batch: confirm, re-date, or leave a tier undated.

Write, per tier the person dated, after the yes: `set_roadmap_tier_date(projectId, roadmapVersion, tier: "T2", dueOn: "2027-01-15", runId)`; to take a date off, the same with `dueOn: null`. Each answers the plan in short with the next `roadmapVersion`: carry it. No `create_goal` for a tier.

## Round 4 — the rows of each tier, and the gaps

Candidates per tier: every unskipped row of that tier, and the moves on its blocks. An operation that is not on the plan yet has no row to read from: `get_operation_definition(projectId, operationDefinitionId)` for that one only, reading `targetSlots[].blockCode`.

Show, per tier:

1. Operations: position, slug, title, level, status, the blocks it writes; your proposed order and the ones you propose skipping, with a reason each.
2. Moves: the ideas on the tier's blocks, code, title, effort, impact, status; the ones you propose adopting.
3. Gaps: what the tier needs that the canvas does not hold, one line each, with where you propose it goes — "note on <position> <slug>", "task under <position> <slug>, for you", or "note on the mapping row, as a test"; an open item that already covers one, by its code.

Propose no move of a row into another tier to pass a gate: the gates are checked once, before the build.

Ask, one batch, the three questions.

Write, in this order:

1. `skip_roadmap_item(projectId, roadmapVersion, itemId, reason, runId)` — reason under 2,000; `include_roadmap_item(projectId, roadmapVersion, itemId, runId)` for a row the person wants back.
2. `add_idea(projectId, canvasVersion, blockId, title, body, effort, impact, runId)` for a move that is not an idea yet, after `mentat-search` found none like it; `resolve_idea(projectId, canvasVersion, ideaId, resolution: "Adopted", runId)` for each idea the person adopted; then `add_roadmap_move(projectId, roadmapVersion, ideaId, tier, runId)`.
3. `append_roadmap_operation(projectId, roadmapVersion, operationDefinitionId, tier, runId)` for an invented operation, or one to run a second time — the plan may hold an operation twice.
4. The order: build the whole list — the first tier's rows in the agreed order, then the next tier's, … — and for each row whose current position differs, `place_roadmap_item(projectId, roadmapVersion, itemId, position, tier: <its tier>, runId)` from position 1 upward. Each answer is the whole plan; take `roadmapVersion` and the new positions from it before the next call.
5. `annotate_roadmap_item(projectId, roadmapVersion, itemId, notes, runId)` — under 5,000 — for each gap an operation fills, on that operation's row ("No prices of what customers use today: read them while mapping what they use"), for each gap that is a test, on the mapping row (`hypotheses-mapping-and-experiments-design`: "Test: talk to 10 founders about how they bill"), and where the person said what the runner must know. A row that already has notes is read whole first and its notes sent back with the new line: the call replaces them.
6. `create_work_item(projectId, parentOperationId: <the row's id>, title, description, priority, effort, goalValue, dueOn?, assigneeUserId: <the founder's user id>, runId)` for each gap only the founder can fill — "Enter monthly running costs" under the `cost-structure` row. It lands in To do as a sub-item of that operation's item.
7. `link_to_goal(projectId, itemKind: "RoadmapItems", itemId: <row id>, goalId: <objective id>, relation: "ServedBy", runId)` per row the person says moves an objective; 409 means it is already there.

Report after each tier: the rows placed with position and slug, the moves with their idea codes, the notes by row, the tasks with their `W-` codes, the skipped rows.

## Round 5 — the plan whole, and the approval

Show the whole plan as a table: tier, its date, rows in order (position, slug or idea code), moves, gaps (notes by row, tasks by code), skipped rows with reasons; above it the mission and the objectives; below it the build check line.

Ask, one batch: is the plan approved.

Write: `record_decision(projectId, title: "Plan v<n> approved — roadmap version <v>", context: "<tier, mission and its number, per-block coverage and confidence as read, the previous plan's code if any>", decision: "<the mission and objectives with their numbers and dates; the tiers with their dates and rows in order, the moves, the block order and why>", consequences: "<rows skipped and why; each gap and where it went — a note on which row, a task by code, a test on the mapping row; what waits for the person: the proposed goal changes, this decision's acceptance>", runId)` — only after the person, shown the whole plan, said it is approved.

`<n>` is 1 for a first plan, else the previous plan decision's number plus one; `<v>` is the `roadmapVersion` the last roadmap write answered.
