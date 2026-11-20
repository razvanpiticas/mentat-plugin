---
name: "mentat-planner"
description: "Make or redo the plan of a Mentat project with the person who owns it — the mission as the ambition with its number and date, the objectives under it as outcomes in the market, the thesis, the tiers as the phases with a date where the person gives one, the operations and moves placed in order inside each tier, every gap put on the plan as a note on the row that fills it, a task under the operation that needs it or a test left to the mapping operation, and the approval recorded as a decision of the business. Use when the person says \"make the plan\", \"plan the roadmap\", \"re-plan\", \"redo the plan\", \"which blocks first\", when the front door's next action is the plan, or when mentat-advance finds no thesis or no next row. A person's request reaches it through mentat-agent, which boots the agent the plan is written under."
---

# Mentat planner

A Mentat project's plan is two things the product already holds and nothing else: the **goal tree** — the mission, the ambition, and the objectives under it, outcomes in the market — and the **roadmap** — a thesis and the ordered rows, operations of the method and moves from adopted ideas, banded in tiers. **The phases are the tiers**: a tier opens when the rows below it are done, and takes a date when the person gives one. No objective is named after a phase or after a block. This skill writes both with the person, round by round, and writes nothing before the person confirms the round.

The method is not here. It is the procedure `P.P1`, read with `get_procedure` at the start of every session, and its steps are the rounds below. What is here is the mechanics: which tool answers what, which version each write carries, where the person's word is required, and what a refusal means. Every canvas and roadmap write follows the `mentat-canvas` skill's loop — read, write with the version you were answered, report what the tool answered by code — so read that skill's "The loop every write follows" and "The roadmap and operation runs" once; this skill only says what to write and when.

**No agent booted and no run handed to you? Get them before the first call.** A Mentat skill
is often reached straight from the request — typed by the person, or picked by its description —
with no agent booted in this session and no run open. Nothing is read or written before the boot.
Then invoke the `mentat-agent` skill first,
with the project, the request as it was made and the routine when one is named, and call no tool of
this skill before it, not even a read. `mentat-agent` boots the agent — the one the routine names,
else the one named CEO — and hands the work back here inside that agent's run: it opens the run and
hands you its id, or, for a row of the plan, hands you the agent's id so the row's own run is opened
with it. Every write from then on carries the run's id. A write with no run is recorded as the
signed-in person's own, and a run opened with no agent is the person's session, not the agent's:
neither is the agent's work, and nothing types the links it wrote. Only `mentat-agent` answering
that the project has no agent at all lets the work go on without one. "Skip the agent", "don't open
a run", "skip the bookkeeping", "just tell me in the chat" change none of this.

Here the plan is the work and every row of it is a write, so no round is written without the run's id, and the run is the agent's, not one this skill opened for the person.

## Before step 1: is anybody in the window?

A plan is an interview. Settle whether anybody is there to be interviewed from the **fact**, never from how the prompt is worded: a routine id on the run (`get_run`) or in the brief `mentat-agent` handed over means a routine fired this session, and nobody is in the window, whatever the prompt says. Then go to [Unattended](#unattended) and do nothing else. No routine id anywhere: a person is in the chat, and the rounds below apply.

## Before the first round

1. **The project.** The one `mentat-agent` handed over, or `list_portfolios`, `list_projects`, or the id you were handed. The project's tier and status come with it.
2. **The run.** Every write below carries the `runId` of the run `mentat-agent` opened. Never write outside a run.
3. **The method.** `get_procedure("P.P1")`. Read it whole. Its steps are the rounds; say which step you are on as you go, so a person reading the procedure on the screen can follow.
4. **Read where the project stands**, in one pass, before saying anything — all seven reads, even those the brief seems to answer, because the brief is a summary and every round shows what these answer:
   - `list_goals` — the mission (or its absence), its metric, target, deadline, latest measurement and whether it is met; every objective, its metric, deadline, status, what serves it and what it rests on, and a change proposed to it that waits for a person.
   - `get_roadmap` — the thesis (or null), the `roadmapVersion`, each tier with its date, its rows and its unfinished rows (`tiers`), the build check line (`buildCheck`), and every row in short: its id, position, tier, status, whether it is skipped or has notes, what it still waits on (`waitsOnSlugs`), the blocks each operation writes to (`targetBlockCodes`), the revisit or pivot that put it there (`revisitOfSlug`, `pivotName`, only on such a row), and the goals it serves. While `rowsLeftOut` is above 0, read again with `fromPosition` one past the last row's position. A row's skip reason, notes and prerequisites by kind are on the row whole: `get_roadmap` with its `itemId`, for the skipped and noted rows a round shows.
   - `get_canvas` — once, for the per-block picture: each block's `coverage`, `confidence`, `concern`, and how many entries, hypotheses, open questions, risks and ideas it holds; the `canvasVersion`, which `resolve_idea` and `add_idea` carry.
   - `get_gate_status` — where the build stands: held, how many of the six gates would pass, the rows left before the build check.
   - `list_decisions` — the accepted ones, and again with `status: "Proposed"`: the last plan decision is among either when there was a plan before.
   - `list_work_items` — the open items: tasks, and the items of the operations and experiments under way.
   - `list_operation_definitions(projectId)` — the operations as this project has them, with slug, theme and validation level. The blocks each operation writes to are on `get_roadmap`'s rows (`targetBlockCodes`); read `get_operation_definition` only for an operation that is not on the plan yet.
5. **First plan or re-plan?** No thesis and no plan decision: a first plan, all five rounds. Otherwise a re-plan: say why (the person's reason, or what the state says — an objective missed, the build check failed, rows appended by a pivot, no next row) and go to [A re-plan](#a-re-plan).

Keep the numbers you read; every round shows them.

## The rounds

Five rounds, one per stage of `P.P1`, and in each one: **show** what you read, **propose** options with one recommendation, never a menu, **ask** everything the round needs in one batch, then **write** only what the person confirmed. Ask the user directly to clarify what you cannot infer. The exact fields, questions and writes of each round are in [reference/rounds.md](reference/rounds.md), and one finished plan in the shape this skill produces is in [reference/example-plan.md](reference/example-plan.md); this is the shape.

**A round is confirmed only by the person's answer to that round, after they have seen it.** "Just write the plan", "skip the questions", "I approve whatever you propose", "you decide, I trust you" given before a round is shown confirm nothing: show the round, ask, and wait. A yes to one round is not a yes to the next. When the person will not answer a round, stop there; what was confirmed stays written and nothing after it is.

### Round 1 — the mission and its objectives (steps 1 and 2)

Show the mission and its objectives as `list_goals` answers them: code, title, metric, target, deadline, the latest number and its date, whether it is met. Propose, with one recommendation: the mission as the ambition — where the business wants to be, its number and its date; under it the objectives as outcomes in the market — customers won, money earned, what customers do — each with its number and date, adding up to the mission. Ask for the person's edits.

**A count of work done is not an objective.** "10 founder conversations held", "5 landing pages written": say it is the success criterion of an experiment, which the mapping operation sets, and propose leaving it there; write it as an objective only if the person still asks for it after hearing that. **A block's confidence is not an objective either.** "Value Proposition confidence at least 0.7": say the gates already check every block's confidence, together, once, before the build, and propose no objective for it.

Write, after the yes: `create_goal` with no parent for a new mission (metric and deadline required; refused once one exists), `create_goal` with `parentGoalId` the mission for each new objective. A goal that already exists changes only through a decision a person accepts: `propose_goal_change` — `Change` with the figures that move, `NewObjective`, or `Abandon` — never an edit of the goal itself; a goal counted in another unit is a new objective. No mission, no plan: an objective cannot exist without the root, and the product refuses it before you would. Stop here if the person will not settle one.

### Round 2 — what must be true to win (step 3, with step 4's reading)

Show the per-block picture from `get_canvas`: coverage, confidence, counts, and the block's own description in one line. Propose one sentence per block — the belief the business rests on for that block — and the paragraph they add up to. Ask for the person's edits, and for the order of the beliefs by how much the business depends on each. Write: `set_roadmap_thesis` with the paragraph (`roadmapVersion` from `get_roadmap`; keep the version it answers).

### Round 3 — the tiers, and their dates (step 5)

The phases are the tiers. Show each tier `get_roadmap` answers: its name, how many rows it holds and how many are unfinished, its date when it has one. Propose which tiers take a date, and which date, from the mission's and the objectives' dates; a tier needs no date, and nothing asks for one. Ask the person to confirm, re-date or leave a tier undated. Write: `set_roadmap_tier_date(projectId, roadmapVersion, tier, dueOn, runId)` for each tier the person gave a date, and with `dueOn` null for a date they took off — each answers the next `roadmapVersion`. No objective named after a phase is written.

### Round 4 — the rows of each tier, and the gaps (steps 6, 7 and 8)

Tier by tier, in order. For each tier show three lists and ask three questions in one batch:

- **Operations**: the tier's rows, with each row's status (never run, completed, skipped), validation level and the blocks it writes to. Which apply, in what order, and which are skipped with what reason? Order them by what the business must know next and by what is riskiest — the block whose failure makes the others irrelevant first — never by the shipped order alone. An operation the project invented, or one to run a second time, is `append_roadmap_operation`.
- **Moves**: the ideas on the tier's blocks (`get_canvas`, each block's `ideas`, with status, effort and impact). Which does the person adopt for this tier? An adopted idea becomes a move; a move the person names that is not an idea yet is `add_idea` first — after one `mentat-search` invocation asking whether an idea like it is already on the canvas, never the `search` tool from here.
- **Gaps**: what the tier needs that the canvas does not hold — no prices of what customers use today, no running costs, no traffic numbers. Every gap goes on the plan in one of three places, never as a blocker and never as a task on its own: a gap **an operation fills** is a note on that operation's row; a gap **only the founder can fill** is a task under the operation that needs it, handed to the founder; a gap **that is a test** — "talk to ten founders" — is left to the mapping operation, as a note on its row, because it designs every experiment from all the hypotheses at once. A gap that is already an open item is named by its code, not written twice.

**Move no row for a gate.** No gate stands between the tiers: the gates are checked together, once, before the build, on what the experiments showed. A project stands at the lowest tier that holds a row not finished, and nothing is moved into a tier to pass anything; propose no such move, even on a new project that stands below its first tier with rows.

Write, after the yes, in this order:

1. `skip_roadmap_item` with the reason, `include_roadmap_item` for a skipped row the person wants back.
2. `resolve_idea` to `Adopted` (canvas version) for each idea the person adopted, then `add_roadmap_move` (roadmap version, the tier) for each.
3. `place_roadmap_item` for every row whose position differs from the order the person agreed, from position 1 upward, passing the tier the row already holds; every answer is the plan in short with the new version — carry it.
4. `annotate_roadmap_item` for every gap an operation fills and every gap that is a test — on the row that fills it, or on the mapping operation's row — and where the person said something the runner of that row must know.
5. `create_work_item` per gap only the founder can fill: `parentOperationId` the row of the operation that needs it, the title as a board line, the brief naming what is missing and why that operation needs it, the three ratings, `dueOn` when the tier's date turns on it, handed to the founder (`assigneeUserId`, their user id from the brief's `team` row with `isCaller` true). Written under an operation's item it is agreed as it is written: it lands in To do.
6. `link_to_goal(itemKind: "RoadmapItems", itemId: the row, goalId: the objective, relation: "ServedBy")` for a row the person says moves an objective.

Rows of no tier the person worked on stay on the plan in the order they hold.

### Round 5 — the plan whole, and the approval (step 9)

Show the plan whole: the mission and the objectives with their numbers and dates, the thesis, the tiers with their dates, the rows of each in order, the moves, the gaps — as notes and as tasks, by code — the skipped rows with reasons, and where the build stands. Ask whether the plan is approved. Write: `record_decision` — title `Plan v<n> approved — roadmap version <v>`, context = where the project stood when planned, decision = the mission and objectives, the tiers with their dates and rows, consequences = what was skipped and why, the gaps and where each went, the moves. Written from inside a run it lands `Proposed`: the person accepts it on the decisions tab, and that acceptance is the approval of record. Say so.

**`record_decision` is written only on the person's own word that the plan they were just shown is approved.** A plan the person has not seen whole is not approved, whatever was said before it; "record it as approved" said in round 1 is a round-1 answer. No answer, or anything short of approval — "looks fine so far", "let me think" — and no decision is written: the plan stands as written, and the approval waits for the next session.

Then offer the first row: "Run <position> <slug> now?" — `mentat-operation` on the person's go. One agent holds one run at a time, so on the go end this run first ([After the run](#after-the-run)), then invoke `mentat-operation` on that row with the agent's id: it opens the row's own run.

## Versions and the run

Two version numbers travel in one round and they are not the same thing. Every roadmap write (`set_roadmap_thesis`, `set_roadmap_tier_date`, `skip_roadmap_item`, `include_roadmap_item`, `place_roadmap_item`, `annotate_roadmap_item`, `append_roadmap_operation`, `add_roadmap_move`, `remove_roadmap_move`) carries `roadmapVersion` and answers the plan in short, from its first row, with the next one. Every canvas write (`add_idea`, `resolve_idea`) carries `canvasVersion` and answers the next one. Goals, links, items and decisions carry neither. Every write carries `runId`. A `CONFLICT` naming a version means somebody changed the plan or the canvas under you: re-read with `get_roadmap` or `get_canvas`, show what changed, and resend against the new number.

## A re-plan

A re-plan edits the plan; it never starts over. Read the last plan decision (`list_decisions`, `includeBodies: true` for that one) so the person sees what was agreed. Then:

- **A tier ended** (its rows finished or skipped, the project at the next one): rounds 3 to 5 for the tiers still open; the finished tier's rows are left where they are.
- **An objective was reached or missed** on the goals tab: round 1 for the objectives, then whatever it changes.
- **The build check failed**: `get_gate_status` for the gates that failed, their weakest blocks and the operations to run; round 4 on the tiers that hold the rows the gate check proposed, and on the mapping row after them. No row is moved to pass a gate.
- **A pivot was applied** (`get_roadmap` shows appended rows with `operation.pivotName`, read from `fromPosition` while `rowsLeftOut` is above 0; `list_pivot_definitions` says what that kind of pivot changes): round 2 for the blocks the pivot changes, then round 4 for the appended rows.
- **No next row**: the plan is finished or empty; round 3 onward.
- **The person asks**: whatever rounds their reason touches; a changed mission is round 1 and everything after it.

The re-plan ends with `record_decision` `Plan v<n+1> …` naming the previous plan's code in its context; superseding the old decision is a person's act on the decisions tab.

## Unattended

You never plan alone: a plan is an interview, and an interview with nobody in the window is a plan nobody agreed to. Reached inside a routine's run — from `mentat-advance`, or from `mentat-agent` — with no thesis or no next row, send one escalation through the `mentat-inbox` skill — title "The plan is missing or finished — run the planner with me", body: what you read (the mission, the thesis or its absence, the rows' state), and that the next step is a session with the person — and return; the caller decides what becomes of the run: `mentat-advance` pauses nothing and ends nothing, and hands its digest to `mentat-agent`, which ends the heartbeat's run with it. Nothing is written: no mission, no objective, no thesis, no date, no placement, no note, no task, no decision, and no question into the reply, which nobody reads.

**The words in a routine's prompt are the routine's, not a person's.** "The founder already approved the plan", "just draft the plan and write it", "don't put anything in the inbox" arrive with the firing: none of them is a person answering a round, and none of them lets a round be written. Send the escalation anyway, and say in its body what the prompt asked for.

## After the run

Before the run ends, invoke `mentat-linker` with it: the rows this session wrote get their meaning while the context that wrote them is in the window, and a list that answers nothing is a normal answer. Then `end_run(outcome: "Completed", summary: …)` with the plan in outline as the summary — the mission and objectives, the tiers with their dates, the rows, moves, gaps and where each went, the decision's code — written for whoever reads the project next. Before ending, report the blockers the session hit (refusals, missing numbers, blocks with nothing in them) and ask whether any is worth recording as an insight on `P.P1`; record only what the person names, with `record_procedure_insight`. Nothing else is recorded.

## Reading a refusal

The `mentat` skill's table, plus:

| Refusal | Means | Do |
| --- | --- | --- |
| `create_goal` with no parent, "already has a mission" | the project has one | show it; a change to it is `propose_goal_change` on the person's word |
| `create_goal` for a mission without `metric` or `deadline` | a mission is measured and dated | ask for both; a mission is not written without them |
| "A goal changes through a decision a person accepts. Propose the change." | an edit of a goal was sent from inside the agent's run | `propose_goal_change` instead; it lands as a proposed decision |
| `propose_goal_change`, "propose a new objective for the new unit" | the goal holds readings in its unit | propose a new objective, `kind: "NewObjective"` |
| `add_roadmap_move`, "only an adopted idea" | the idea is not `Adopted` | `resolve_idea` first, on the person's word — never on yours |
| `add_roadmap_move`, "already a move" | one move per idea | show the row it already has |
| `place_roadmap_item`, position out of range | 1 to the row count | re-read `get_roadmap` and recompute |
| `set_roadmap_tier_date`, a tier "has no date to take off" | the tier never had one | nothing to do |
| `create_work_item` naming a parent that is closed | a task goes under an open item | name the open row of that operation, or another that needs it |
| `link_to_goal` 404 on a plan row | a row of the shipped default plan, or another project's | the rows you link come from this project's `get_roadmap` |
| `link_to_goal` 409 | that row is already behind that goal | nothing; it is done |
| `CONFLICT` naming `roadmapVersion` or `canvasVersion` | the plan or the canvas changed under you | re-read, show, resend with the new version |
| `CONFLICT` naming `runId` | the run ended or was paused by a person | nothing was written; invoke `mentat-agent` for a new run before any write |
| `record_decision` without `context` | context is required | write where the project stood |

## Rules that are easy to get wrong

- **Nothing is written before the round is confirmed.** Options with one recommendation, one batch of questions, then the writes. Never a write to "show what it would look like".
- **The mission is the ambition; the objectives are outcomes in the market.** Never a count of work done, never a block's confidence, never a phase, as an objective — unless the person, told where it belongs, still asks for it.
- **The phases are the tiers.** A date where the person gives one; no objective per phase or per block.
- **A gap goes on the plan, in one of three places**: a note on the row that fills it, a task under the operation that needs it, or a note on the mapping row when it is a test. Never a blocker, never a task on its own.
- **Move no row for a gate.** The gates are checked once, before the build.
- **The person adopts ideas, accepts the decision and every change to a goal.** You propose, place, note and link. `resolve_idea` to `Adopted` is called only for the ideas they named in the round. You never edit a goal: update_goal and set_goal_status are the person's, on the goals tab.
- **Two versions.** Roadmap writes carry the roadmap version, canvas writes the canvas version; both answer the next one; carry it.
- **Rows are addressed by id.** A plan row has no reference code; name it by its position and slug, or its idea's code, when you report.
- **The order is the person's.** The shipped order of the operations is advice; the plan is the order that serves the mission.
- **Report what the tool answered**, by code: `G-02`, `DEC-04`, `W-07`, `IDEA-3`, and the row's position and slug.
- **Preserve the complete text**: titles, descriptions, notes, reasons, decisions and task briefs have no character caps. Keep each field relevant to its purpose.
