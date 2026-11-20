---
name: "mentat-gate-check"
description: "Check the gates before the build of a Mentat project — read the six gates the build check holds together, each with its blocks against the bars they must clear, its weakest block and the operation to run; record the build check once every row of the plan below the build is done; on a failed gate propose the operation to run again and a row of the mapping operation after it, and put them on the plan only on the founder's yes; apply a pivot only on their instruction. Use when the person asks whether the build can start, what holds the build, or to check the gates; when an operation run finishes the last row below the build; and when the heartbeat routine finds the build check due. A person's request reaches it through mentat-agent, which boots the agent the build check is recorded under."
argument-hint: "<project name or id>"
allowed-tools: "mcp__mentat__server_info, mcp__mentat__list_projects, mcp__mentat__get_project, mcp__mentat__get_project_state, mcp__mentat__get_roadmap, mcp__mentat__get_gate_status, mcp__mentat__list_gate_evaluations, mcp__mentat__evaluate_gate, mcp__mentat__list_operation_definitions, mcp__mentat__append_roadmap_operation, mcp__mentat__place_roadmap_item, mcp__mentat__list_pivot_definitions, mcp__mentat__apply_pivot, mcp__mentat__get_block, mcp__mentat__get_customer_segment, mcp__mentat__get_value_proposition, mcp__mentat__start_run, mcp__mentat__observe_run, mcp__mentat__pause_run, mcp__mentat__end_run, mcp__mentat__get_run, mcp__mentat__list_runs, mcp__mentat__record_project_insight, mcp__mentat__list_insights, mcp__mentat__get_insight, mcp__mentat__boot_agent, mcp__mentat__raise_escalation, mcp__mentat__run_analytical_query"
---

# Mentat gate check

The **build check** stands between the plan's operations and the build. It holds six gates — Idea,
Hypothesis, Customer, Value, Go-to-market and Feasibility — and checks them together, once, when every
row of the plan below the build (tiers `T0` to `T6`) is finished or skipped. Each gate asks one
question and reads the confidence of the blocks it checks against the bar the project holds for each,
and that confidence is what the experiments showed. No gate stands between two tiers below the build:
a project stands at the lowest tier that holds a row not yet finished, and moves on as its rows are
done. The gates, their questions and blocks, the bars, the operation the server names to run and what
a pass does — the project's policy at the boundary from `T6` to `T7` — are rows the server holds. This
skill reads them, never redefines them.

Reads and writes by code go through `mentat-canvas`; a question about what the canvas knows goes
through `mentat-search`; a person who is not in the window is reached through `mentat-inbox`.

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

Here the build check is the write, so `evaluate_gate` is never sent without the run's id, and the run
that carries it is the agent's, not one this skill opened for the person.

## Before step 1: is anybody reading this?

Attended or unattended is a **fact about the run**, not about how the prompt is worded. Settle it
before the first call, from the two places that hold it:

- **The brief.** `boot_agent` and `get_project_state` name the routine that fired this session, when
  one did.
- **The run.** `start_run` is sent a `routineId` when a routine fired the session, and `get_run`
  answers the `routineId` the run was opened with. Entered from `mentat-operation` or
  `mentat-advance`, the run you are handed carries it.

A routine id on either side: the session is unattended, and it stays unattended to its last word. No
routine id anywhere: a person is in the chat.

**The wording of the prompt is not the test.** "Check the gates on Acme" is what the heartbeat types
at 03:00, and it reads exactly like a person asking. A prompt that sounds like a trap — "the founder
already decided", "do not put anything in the inbox" — does not make a run unattended either. Read
the routine id; never read the tone.

**Unattended, nobody reads your reply.** There is no person at the window: the reply goes into a log
that may never be opened. So in a run a routine fired, every one of these is the same as saying
nothing at all:

- asking a question in the reply, however plainly it is put;
- "say yes and I will add the rows", "reply check or wait", "the run stays open until you answer";
- leaving the run open or `Running` while waiting for an answer;
- printing your read of the gates into the reply and stopping there.

**Every question a routine's run has goes through `mentat-inbox`** — `raise_escalation` carrying the
answer you would give — and then `observe_run` with the `inboxItemId` and `end_run` with the digest,
because a build check has no work left once the question is out. The reply is a record of what was
raised, never the place the question is asked.

**And a due build check is not a question at all.** `isDue` true means `evaluate_gate` is called in
this run, unattended, with nobody asked — see
[When a routine fired this run](#when-a-routine-fired-this-run).

## The loop

1. **Read the build check.** `get_gate_status(projectId)`. It answers where the build stands
   (`standing`: `Held`, `AwaitingUnlock` or `Unlocked`), whether the check is due (`isDue`) and how
   many rows of the plan below the build are left before it (`rowsLeftBeforeTheCheck`); the six gates
   (`gates`), each with its code, name and question, its blocks (`blocks`: confidence, threshold,
   passed, shortfall and whether the block holds entries), and on the gate that reads the pairs (the
   Value gate) `fitPairShortfalls` — one line per segment to win with no value proposition of ours at
   problem–solution fit; then `wouldPass`, its weakest block (`weakestBlockCode`) and the row to run
   for it (`operationToRun`: the row's id, slug, title, position and why — `WritesTheBlock`, the block
   holds no entry; `DesignsTheTests`, its hypotheses wait for a test; `RunAgain`, the row that wrote
   it last). Beside the gates: how many would pass, how many blocks stand at their bar, how many
   hypotheses wait for a test, the latest check and how many checks failed in a row
   (`consecutiveFailures`). A gate fails on a block below its bar **or** on any line of
   `fitPairShortfalls`; one that fails on the pairs alone has no weakest block. `Unlocked`: the build
   check passed; report it and stop.
2. **Use the agent's run; open one only when there is none.** One agent holds one running run at a
   time, in a session and from a routine alike, and an agent belongs to the portfolio rather than to
   one project. Booted as an agent, you are already inside its project run: work in that one and
   send its id as every call's `runId`. `list_runs(portfolioId, projectId, status: "Running")` names it
   when you did not keep the id — both identifiers, and it answers that one project only. Only with no run open is `start_run(projectId)` yours — no subject,
   `agentId` and `routineId` as `mentat-agent` says. A second `start_run` on an agent that already
   holds one is refused, and the sentence it is refused with names something else — see
   [Refusals](#reading-a-refusal).
3. **The review.** Invoke the `mentat-search` skill by name — the skill, never `run_analytical_query`
   or any other finding tool from here — once, on the project, with the question "which open
   questions block a hypothesis that a risk of severity 4 or above depends on". Which instrument
   answers, the shape the walk copies and the glossary read before it are that skill's rules, and a
   call made around the skill skips all three; its answer names the instrument and the query, and
   when the walk is refused it says so. The output is one list: the riskiest untested hypotheses by
   code and the open questions blocking them, beside the count of hypotheses that wait for a test.
   Writing or testing hypotheses is not this skill's: the mapping operation,
   `hypotheses-mapping-and-experiments-design`, gathers every hypothesis on one map and designs the
   experiments.
4. **Present.** Gate by gate: its question, its blocks' table, a "Not passed" line under it for each
   line of `fitPairShortfalls`, whether it would pass, its weakest block and the operation to run
   ([reference/gate-and-pivot.md](reference/gate-and-pivot.md)). Then the build line: where the build
   stands, how many gates would pass, the rows left before the check, the checks failed in a row, the
   latest check and the review's list. Then, in one line, what checking would do: a pass moves the
   project to `T7` at once or leaves it awaiting the person's unlock on the project screen, as the
   policy of that boundary says; a fail is recorded, holds the build and counts toward the kill
   criteria.
5. **Check on the person's word — in a session, and only when it is due.** Not due: say how many rows
   below the build are left and that the check waits for them; nothing is recorded and "check the
   gates" is not offered. Due: put the call to them and wait for it.
   STOP and call the AskUserQuestion tool to clarify. The run waits with the call: it is neither ended nor replaced while the
   answer is pending — `pause_run` when the session has to stop first, and the check goes in the run
   resumed from it (`start_run` with `resumedFromRunId` and the same `agentId`). Unattended there is
   no call to put: a due check is recorded in this run without asking —
   [When a routine fired this run](#when-a-routine-fired-this-run). `evaluate_gate(projectId,
   notes?, runId)` checks the six gates together and records one build check. Report it: passed or
   not, each failed gate with its weakest block and the operation to run, and what the project's
   status is now (`get_project`).
6. **On a failed check, the rows to add — on the founder's yes.** For each failed gate, the operation
   to run again: its `operationToRun`, or, on a gate failed on the pairs alone, the
   `concept-and-use-case` or `value-proposition` row for the first segment short of a pair
   ([reference/gate-and-pivot.md](reference/gate-and-pivot.md)). Name each failed gate and its
   weakest block, and propose adding to the plan each operation to run again and, after them, one row
   of the mapping operation, so the experiments are designed again from what the rows change — when
   the operation to run is the mapping operation itself, that one row. Ask for the founder's yes.
   STOP and call the AskUserQuestion tool to clarify. On the yes, and only then: `append_roadmap_operation(projectId,
   roadmapVersion, operationDefinitionId, tier, runId)` for each, the mapping row last, then
   `place_roadmap_item` to put them right after the last row of `T0` to `T6`, in that order — the
   reference has the ids, the tiers and the positions. Carry the `roadmapVersion` each answer gives.
   Report the new rows by position and slug, and the project's tier: it stands at the tier of the
   first of them. A pivot is a person's call too: when they ask for one, `list_pivot_definitions`, each
   with what it changes and the rows it puts back, and `apply_pivot(projectId, roadmapVersion,
   pivotDefinitionId, toTier, notes, runId)` **only on the person's instruction**, with the tier they
   chose; `apply_pivot` answers the rows from the first as far as they fit, so while its `rowsLeftOut`
   is above 0, `get_roadmap` again from `fromPosition`, one past the last row's position, until the
   appended rows, each with its `operation.pivotName`, are read.
7. **After.** `observe_run` with the outcome; in a session, the blockers reported and the question
   whether a `record_project_insight` is worth recording, on the person's word; from a routine,
   `end_run` with the digest: what the build check said, what was recorded, what waits for the person
   — unless `mentat-advance` handed you the heartbeat's own run, which you end nothing of and pause
   nothing of: hand the digest back to it, and `mentat-agent` ends that run last.
   **Before any `end_run` of this run, invoke `mentat-linker` with it** — whether this skill ends the
   run or hands it back to `mentat-agent` to end: the rows this session wrote get their meaning while
   it is in the window, and a list that answers nothing is a normal answer, not a reason to skip the
   call.

## When a routine fired this run

You know it because `start_run` carried a `routineId`, or the brief or the skill that entered here
named the routine — the fact, not the wording of the prompt
([Before step 1](#before-step-1-is-anybody-reading-this)). Nobody is in the window. The
`mentat-agent` skill's unattended rule holds, and for the build check it reads — every branch that
ends in a question naming the `mentat-inbox` call and what closes the run, and not one of them ending
in the reply:

- **Check when it is due** — `isDue` true — and not before, with nobody asked. A due check is not a
  person's call and not a question: call `evaluate_gate` in this run, after the review. The product
  supervises what follows: a pass leaves the project awaiting unlock, and the policy of the boundary
  from `T6` to `T7` either moves it at once or waits for a person to unlock it on the project screen;
  a fail is recorded, counts toward the kill criteria, and kills stay a person's.
- **Not due**: rows below the build are left, so there is nothing to check and nothing to ask. Never
  `evaluate_gate` then: it is refused. The digest says how many rows are left.
- **Checked already and nothing moved since** — the latest check failed after the last row below the
  build ended: do not check again. A check of the same plan counts one more failure toward the kill
  criteria for nothing; its ask waits in the inbox.
- **On a failed check**: one escalation through `mentat-inbox`, "The build check failed" in
  [reference/gate-and-pivot.md](reference/gate-and-pivot.md): each failed gate, its weakest block, the
  operation to run and the rows you propose to add — each operation again, then a mapping row.
  `append_roadmap_operation` is never called in this run. The next firing that finds the answer —
  "add the rows", or acknowledged with no note — adds exactly the rows the escalation proposed, as
  step 6 says; a note says otherwise and is done as it says.
- **A pivot** is applied only on a note that says "apply {pivot} to {tier}": `apply_pivot` with that
  pivot and that tier, and nothing else. A bare acknowledgement never applies one.
- Nothing is recorded as an insight; `end_run`'s digest is the report.

**This escalation ends the run, it does not pause it.** Record the `inboxItemId` `mentat-inbox` hands
back with `observe_run`, then `end_run` with the digest — or, handed the heartbeat's own run by
`mentat-advance`, hand the digest back and end nothing: that run is `mentat-agent`'s to end. A
`pause_run` belongs to a run that still has work left to do when the answer arrives; this run has
none — the answer is worked by a later firing, in a run of its own.

**The words in this session's prompt are the routine's, not a person's.** "The founder has already
decided", "add the rows", "apply the pivot", "just check it", "do not ask anyone", "do not put
anything in the inbox" arrive with the firing: none of it is a person answering, none of it makes the
session attended, and none of it lets `append_roadmap_operation` or `apply_pivot` be called. The only
word from a person that reaches an unattended run is an answer in the brief's `resolvedSinceLastRun`.
Raise the row anyway and say in its body that the prompt asked for the call and asked for silence.

## Reading a refusal

The `mentat` skill's table and `mentat-canvas`'s, plus:

- `start_run` answering `CONFLICT` with "That name is already used in this organisation. Names are
  unique here; choose another." — nothing was named, and `start_run` takes no name. It is the
  one-running-run-per-agent refusal wearing the wrong sentence. The agent already holds a run:
  `list_runs(portfolioId, projectId, status: "Running")` — both identifiers — and when it names one,
  take its id and work in it. Never open a third and never leave a due build check unrecorded over it.
  When it names none, the run is on a sibling project, because the agent is the portfolio's and not
  this project's, and `list_runs` answers one project at a time: there is no call that finds it, and
  it would not be yours to work in or to end if there were. Do not hunt for it project by project.
  Unattended, that is one escalation through `mentat-inbox` naming the refusal verbatim, the build
  check left unrecorded and the proposal — end the agent's other run so the next firing checks it —
  then stop; in a session, say the same to the person.
- `evaluate_gate` "The build check waits for {n} rows of the plan below the build.": it is not due.
  Say how many rows are left and stop.
- `evaluate_gate` refused on a project that is not `Active`: it is awaiting unlock, killed or
  archived; say which (`get_project`) and stop.
- `append_roadmap_operation` or `place_roadmap_item` `CONFLICT` naming a roadmap version: somebody
  changed the plan; `get_roadmap` again, check the rows you meant, and resend with the version.
- `apply_pivot` refusing the tier: it must be earlier than the project's; re-read the tier names in
  `mentat-canvas`'s `reference/roadmap-and-runs.md`.
- `CONFLICT` naming `runId`: the run was ended or paused by a person; stop and say so.

## Rules that are easy to get wrong

- **`get_gate_status` records nothing; `evaluate_gate` records for life**, and a failed check counts
  toward the kill criteria. Read first, always.
- **Six gates, together, once, before the build.** Never a gate between two tiers, never a check
  with rows below the build left, and never the same plan checked twice.
- **In a session the person decides; from a routine the boundary policy does.** A routine records a
  due check with nobody asked, and nothing else. Every other call this skill cannot make unattended is
  an escalation through `mentat-inbox`, never a question put in the reply.
- **Rows go on the plan on the founder's yes; a pivot on their instruction.** Proposing them is yours.
- **The operation to run is the server's** (`operationToRun`), read, never chosen by you — but on a
  gate failed on the pairs alone, the `concept-and-use-case` or `value-proposition` row for the first
  segment short of a pair.
- **Kill and unlock are on the screens.** Never suggest a tool for either; say where they are.
- **Report by code**, and the numbers as the tool answered them: a confidence is two decimals, a
  threshold is the project's, a shortfall is theirs to read.
