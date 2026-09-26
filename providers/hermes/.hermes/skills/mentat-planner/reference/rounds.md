# The five rounds, field by field

Load this when running a round. The skill carries the shape; this carries what each round reads, asks and writes, with the exact arguments.

## Before round 1 — the reads

| Read | Fields kept |
| --- | --- |
| `list_goals(projectId)` | the mission node: `id`, `referenceCode`, `title`, `metricName`, `metricUnit`, `metricTarget`, `metricDirection`, `deadline`, `status`, `latest` (value, asOf, source); each child: the same plus `parentGoalId`, `servingItems` (id, referenceCode, kind, title). No description: a phase's `Starts:`, `Done when:` and `Measures:` lines are kept from the `create_goal` or `update_goal` answer that wrote them, and in a re-plan read back from the last plan decision's body |
| `get_roadmap(projectId)` | `thesis`, `roadmapVersion`, `projectTier`, every row: `id`, `kind`, `position`, `tier`, `isSkipped`, `skipReason`, `notes`, `servesGoalCodes`; on an operation row `slug`, `title`, `themeCode`, `validationLevel`, `status`, `prerequisites`, `targetBlockCodes`; on a move row `ideaId`, `ideaReferenceCode`, `ideaTitle`, `blockCode`, `ideaStatus` |
| `get_canvas(projectId)` | `canvasVersion`, `overallConfidence`; per block: `id`, `code`, `name`, `description`, `concern`, `coverage`, `confidence`, and the counts of `entries`, `hypotheses`, open `questions`, `risks`, `ideas`; each idea: `id`, `referenceCode`, `title`, `effort`, `impact`, `status` |
| `get_gate_status(projectId)` | `roadmap`: `isDue`, `fromTier`, `toTier`, `completedOperationsInTier`, `pendingOperationsInTier`, `pendingSlugs`; `gateName`, `wouldPass` |
| `list_decisions(projectId)`, and again with `status: "Proposed"` | codes and titles; the last one titled "Plan v…", accepted or still proposed |
| `list_work_items(projectId)` | open jobs: code, title, status |
| `list_operation_definitions(projectId)` | `id`, `slug`, `title`, `themeCode`, `validationLevel`, `source` |

## Round 1 — the mission

Show: "`G-01` — <title>. Counted in <metricName> (<unit>), target <target> <direction>, by <deadline>. Latest: <value> as of <asOf>, from <source>." or "This project has no mission."

Ask, one batch: does the mission stand as written; if not, the title, what is counted, the unit, the target and whether it is at least or at most, the date. For a project with none: the same, from your proposal.

Write:
- new: `create_goal(projectId, title, description?, metric: { name, unit?, target, direction: AtLeast | AtMost }, deadline: yyyy-MM-dd, runId)` — no `parentGoalId`.
- edit: `update_goal(projectId, goalId, title?, description?, metric?, deadline?, runId)` — a mission may not drop its metric, and the unit may not change once readings exist.

## Round 2 — the thesis

Show, one line per block: "<code> <name> — <coverage>, confidence <confidence>, <n> entries, <n> hypotheses, <n> open questions, <n> risks, <n> ideas." Then the block's description in one sentence.

Propose: one belief per block, "For <block> to hold, <belief>." Then the paragraph: the beliefs in the order of how much the business depends on each, opening with the mission's number.

Ask, one batch: the edits to each belief; the order; anything missing.

Write: `set_roadmap_thesis(projectId, roadmapVersion, thesis, runId)` — under 5,000 characters. Keep the answered `roadmapVersion`.

## Round 3 — blocks first, and the phases

Propose the block order — riskiest first, with the reason in a few words each — and three to six phases:

| Phase | Blocks | Starts | Deadline | Done when |
| --- | --- | --- | --- | --- |

Ask, one batch: confirm, rename, re-date, merge, split; whether any phase's blocks are tracked as objectives of their own.

Write, per phase, after the yes:

`create_goal(projectId, parentGoalId: <mission id>, title: "<Phase name>", description: "Starts: <when>\nDone when: <the condition>\nMeasures: (round 5)", deadline: <date>, runId)`

Per block-objective the person wants: `create_goal(parentGoalId: <phase id>, title: "<Block name> — <what state>", description: "Done when: …")`.

Keep every goal id the answers give, and each phase's description: round 4 links rows to the ids, and round 5 rewrites the description whole.

## Round 4 — the rows of each phase

Candidates per phase: every unskipped operation row whose `targetBlockCodes` meet the phase's blocks. An operation that is not on the plan yet has no row to read them from: `get_operation_definition(projectId, operationDefinitionId)` for that one only, reading `targetSlots[].blockCode`.

The band: when no unskipped operation row sits in `projectTier`, the gate above the project can never be due. Propose moving the first phase's first rows into `projectTier`, and move them only on the person's word — `place_roadmap_item` with `tier: <projectTier>`.

Show, per phase:

1. Operations: position, slug, title, level, status; your proposed order and the ones you propose skipping, with a reason each.
2. Moves: the ideas on the phase's blocks, code, title, effort, impact, status; the ones you propose adopting.
3. Gaps: what the phase needs that the canvas does not hold, one line each; an open job that already covers one, by its code.

Ask, one batch, the three questions.

Write, in this order:

1. `skip_roadmap_item(projectId, roadmapVersion, itemId, reason, runId)` — reason under 2,000; `include_roadmap_item(projectId, roadmapVersion, itemId, runId)` for a row the person wants back.
2. `add_idea(projectId, canvasVersion, blockId, title, body, effort, impact, runId)` for a move that is not an idea yet, after `mentat-search` found none like it; `resolve_idea(projectId, canvasVersion, ideaId, resolution: "Adopted", runId)` for each idea the person adopted; then `add_roadmap_move(projectId, roadmapVersion, ideaId, tier: <the phase's rows' tier>, runId)`.
3. `append_roadmap_operation(projectId, roadmapVersion, operationDefinitionId, tier, runId)` for an invented operation, or one to run a second time.
4. The order: build the whole list — phase 1's rows in the agreed order, then phase 2's, …, then every other unskipped row in the order it holds — and for each row whose current position differs, `place_roadmap_item(projectId, roadmapVersion, itemId, position, tier: <its tier>, runId)` from position 1 upward. Each answer is the whole plan; take `roadmapVersion` and the new positions from it before the next call.
5. `link_to_goal(projectId, itemKind: "RoadmapItems", itemId: <row id>, goalId: <phase id>, runId)` per row of the phase; 409 means it is already there.
6. `annotate_roadmap_item(projectId, roadmapVersion, itemId, notes, runId)` where the person said what the runner must know; under 5,000.
7. `create_work_item(projectId, title, description, priority, effort, goalValue, dueOn?, aboutKind: "Goals", aboutId: <phase id>, runId)` per gap; or `aboutKind: "Questions"`, `aboutId: <question id>` when the gap is a question already on the canvas. Each lands `Proposed`.

Report after each phase: the rows placed with position and slug, the moves with their idea codes, the jobs with their `W-` codes, the skipped rows.

## Round 5 — measures and the approval

Show the whole plan as a table: phase, deadline, rows in order (position, slug or idea code), moves, gaps (job codes), skipped rows with reasons.

Ask, one batch: what each phase measures and how; is the plan approved.

Write:
- `update_goal(projectId, goalId: <phase>, description: "Starts: …\nDone when: …\nMeasures: <the measure>", metric?: { name, unit, target, direction } when the measure is one number, runId)` per phase.
- `record_decision(projectId, title: "Plan v<n> approved — roadmap version <v>", context: "<tier, mission and its number, per-block coverage and confidence as read, the previous plan's code if any>", decision: "<phases with dates and their rows in order, the moves, the block order and why>", consequences: "<rows skipped and why; the gaps proposed as jobs, by code; what waits for the person: the jobs' approvals, the first phase's activation, this decision's acceptance>", runId)` — only after the person, shown the whole plan, said it is approved.

`<n>` is 1 for a first plan, else the previous plan decision's number plus one; `<v>` is the `roadmapVersion` the last roadmap write answered.
