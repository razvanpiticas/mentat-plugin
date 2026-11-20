---
name: "mentat-advance"
description: "Do the next row of the plan of a Mentat project, or the task a person handed to the agent — read where the plan stands, pick the one thing that comes next by the next-action rule, do it through the skill that owns it, and stop at every point that is a person's, leaving what waits in their inbox. Use when the Heartbeat routine fires, when the person says \"advance\", \"do the next one\" or \"what's next, do it\", and when a run paused on a person's answer has its answer."
---

# Mentat advance

A project's plan is a sequence of rows — operations of the method and moves from adopted ideas — and at any moment exactly one thing comes next. This skill finds that thing and does it, once, through the skill that owns it: a row of the plan is `mentat-operation`'s, the build check is `mentat-gate-check`'s, a plan that is missing is `mentat-planner`'s, a paused experiment is `mentat-experiment`'s, and a task a person handed to the agent is worked here, in a run of its own. It writes nothing on the canvas itself and asks nobody anything when nobody is in the window: what is a person's goes to their inbox through the skill that hit it, and the run waits.

The rule that says what comes next is in [reference/next-action.md](reference/next-action.md). It is the one copy; the front door reads the same file to say the next action without doing it.

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

Here the run `mentat-agent` hands you is the firing's own — the run it opened on the project naming the routine, or the person's session run — and every call below that writes carries its id. Handed a run, you are already inside it: do not invoke `mentat-agent` again and do not open a second run on the project.

## Invoked with

- **By the Heartbeat**, through `mentat-agent`, inside the run it opened naming the routine: the routine's text says "advance the plan by one row". You are handed the project, the agent's id, the run id and the brief's `project` section. Nobody is in the window.
- **By the person**: "advance", "do the next one", "what's next, do it". `mentat-agent` has booted an agent and opened a run on the project and hands it to you. The person is in the window: see [In a session, the question comes first](#in-a-session-the-question-comes-first) before anything else.

One thing per call. A second is the next heartbeat's, or the next "advance".

## In a session, the question comes first

No routine named the run, so a person is reading. "Do it" in their request does not answer a question you have not asked yet. In this order, and nothing between:

1. The reads of loop step 1 and the rule of loop step 3 — reads only.
2. Say the next action in one sentence naming the skill that does it, and ask "Do it now?". STOP and use Codex's structured user-input tool when available; if it is unavailable, ask directly in chat to clarify.
3. Stop and wait for the answer.

**Until the person answers, you call no run tool at all**: no `observe_run`, no `pause_run`, no `start_run`, no handover, no read of the owner's package. The session's run stays `Running` while you wait — waiting for an answer in the chat is not a pause, and a run paused before the person said yes is a run paused for nothing. On yes, go on at loop step 4. On no, write nothing and hand back.

## The loop

1. **Read where the plan stands.** The brief's `project` section when you were handed one, else `get_project_state(projectId, runId)`: `status`, `roadmap` (`lastCompleted`, `inFlight`, `next`, `buildCheck`, `hasThesis`), `openItems`, `resolvedSinceLastRun`, `lastRun`. Then `get_roadmap(projectId)` once — the rows of both kinds in short, in order, the `roadmapVersion`; again from `fromPosition` only while `rowsLeftOut` is above 0 and no outstanding row was found yet — `get_gate_status(projectId)` — `isDue`, `standing`, `latestCheck` — and `list_work_items(projectId, type: "Task", assigneeAgentId: <the agent>)`, the tasks handed to the agent. Four reads, never more before deciding.
2. **Remember the last firing.** `lastRun.summary` is the previous firing's digest. Its `Waiting:` lines name every ask that firing left open, each with its inbox row id; its `Decided:` lines name answers still in force. For each `Waiting:` line, look for the line with that id in `resolvedSinceLastRun`: found, the answer is in; not found, the ask still waits and is **not raised again**.
3. **Decide, by the rule.** Read [reference/next-action.md](reference/next-action.md) now, whole, with the file tool — every call, never from memory of an earlier one. Walk its numbered rules from the top; the first that applies is the action, and there is exactly one action per call.
4. **Do it, through its owner** — in a session, only after the person said yes to the question, and not one call earlier. Hand over what the rule says and nothing else. Record what you decided with `observe_run(projectId, runId, note)` on the firing's run before handing over — one line: the rule that fired, by number, and the row, task or build check it named.
5. **Take back what the owner hands back**: the row run's id and status, what was written by code, the ask it raised and the inbox row id, or "nothing to type yet". A run another skill opened is ended or paused by that skill, never by you.
6. **Hand back the digest** ([reference/digest.md](reference/digest.md)) to `mentat-agent` — naming the run you hand back, which is a new one when you paused the firing's run around a row or a task — or say it to the person. What comes next is the same on **every** path, whichever rule fired: `mentat-linker` on that run, and only then `end_run` with the digest. That includes a call that stopped and raised nothing (rule 1, rule 5a), a row run the owner left paused, a planner escalation and a finished plan: "this run wrote nothing" is what the linker answers, never a reason to skip it. Say so in the hand-back: "Next: `mentat-linker` on <run id>, then `end_run` with this digest."

## The firing's run and a row's own run

One agent holds one running run. `mentat-operation` opens a run of its own on the row, `mentat-experiment` resumes an experiment's own run, and a task is worked in a run of its own on the task, all with the agent's id — so while the firing's run is `Running` the agent cannot open theirs, and the refusal arrives wearing the sentence "That name is already used in this organisation." Before you hand to either of those two, or open a task's run:

1. Keep the answered line from `resolvedSinceLastRun` that the owner will need (its `inboxItemId`, `resolution`, `resolutionNote`). The brief measures "since the last run" from the last run that stopped, and the firing's run is about to be that run, so the owner cannot read the answer again — you hand it over.
2. `observe_run` on the firing's run with the rule and the row or task, then `pause_run(projectId, runId)` on the firing's run.
3. Hand over: the project, the agent's id, the routine's id when one fired the session, the row or the experiment, the answered line, and **the firing's run id** you just paused — the owner's own run summary names it, so the row's report and the firing's report each lead to the other — and that the owner hands back once its own run is ended or paused, without going on to the build check: the check is the next firing's.
4. When the owner hands back, or the task's run is ended or paused, `start_run(projectId, agentId, routineId, resumedFromRunId: <the firing's paused run>)` — the same agent and routine, no subject — and carry the new run's id to the end: the digest, and every later write of this firing, name it. The paused half stays behind as the record; it waits on nobody.

This pause is the one `pause_run` this skill makes on the firing's own run, around a row or a task, and you resume it yourself. Everything the owners hand to — `mentat-hypothesis`, `mentat-canvas`, `mentat-inbox` — works inside the owner's run. `mentat-gate-check`, `mentat-planner` and a move's job work inside the firing's run and need no pause.

## Resuming

A run this skill resumes is `Paused`; a `Running` one is another session's and is only reported. Find them with `list_runs(portfolioId, projectId, status: "Paused")`; each has a kind — an attempt at a row, an experiment run, a run on a task, a run on the project itself. `get_run(projectId, runId)` and read its events: the skill that paused it recorded the inbox row id of the ask it raised (`observe_run`, "escalation <id> raised"). Then:

- the line for that id is in `resolvedSinceLastRun` → the answer is in. An attempt at a row: `mentat-operation` resumes it (`get_run`, `start_run` with `resumedFromRunId`, on from the last checkpoint); when the ask was a verdict proposal, the answered verdict goes on `end_run`; when it was a start that spends or needs the person's hands, the operation hands the answer to `mentat-experiment`, which calls `start_experiment` (or leaves the card designed on "do not"), and the operation then ends the row run. An experiment run of its own: `mentat-experiment` resumes it with the answer's line. Either way the firing's run is paused around it first, as the section above says, and the answered line travels with the handover.
- no line → still waiting. Leave the run paused; one `Waiting:` line in the digest, and the call goes on down the rule: a task handed to the agent is still worked (rule 3). It stops at rule 6 when the next row is that paused row: the person's answer is the bottleneck.
- an attempt at a row paused on a step's question to the founder, or on a verdict proposal, names no ask in its events — the raise itself paused it — and `get_run` shows the step that asked, or the digest's `Waiting:` line names the proposal's inbox row: holding the answer, it is in and `mentat-operation` resumes the run from it; without one, still waiting, as above.
- a run on a task that waits on its `Blocked` escalation: the task's own row in the inbox; a `Waiting:` line, nothing else, even once that row is answered. Its answer reaches the task when a person moves it back to `Todo` or `InProgress`, and rule 3 then works it by resuming this run ([A task](#a-task), step 2), so it never stays `Paused` for good. A task a person closed instead leaves its paused run to them: cancelling a run is a person's.
- a run on the project itself whose events name no ask is the paused half of an earlier firing, or a person's session: it waits on nobody. Leave it and say nothing of it.

The answer is read as `mentat-inbox` says: a note is the answer; acknowledged without a note, the proposed answer stands — except a pivot, which is applied only on a note saying so, and only by `mentat-gate-check`.

## A task

Rule 3 picked it: an item of type `Task`, handed to this agent, in `Todo` or `InProgress`, the highest `priority` first. One per firing. In these steps, in this order:

1. **Read it.** `get_work_item(projectId, workItemId)`: the brief, the thread — what somebody already tried — its parent, and what it is about.
2. **Open its run.** The firing's run is paused first ([The firing's run and a row's own run](#the-firings-run-and-a-rows-own-run)); then `start_run(projectId, workItemId, agentId, routineId)`, the agent's and the routine's ids — with `resumedFromRunId` the task's `Paused` run when it has one (`list_runs(status: Paused)`: it was `Blocked` and a person moved it back), read first with `get_run` — and carry that run's id on every write until it ends.
3. **Start it.** A `Todo` task: `transition_work_item(projectId, workItemId, status: "InProgress", runId)`. An `InProgress` one stays as it is.
4. **Do the work** the brief asks, through the skill that owns each part of it — a canvas write through `mentat-canvas`, a hypothesis through `mentat-hypothesis`, a question about the canvas through `mentat-search` — never the canvas tools from here.
5. **Comment.** `comment_on_work_item(projectId, workItemId, body, runId)`: what you did, by code; what you found; what did not work; what is left.
6. **Leave it.** The work done: `InReview` (`transition_work_item`), and the run ended — `mentat-linker` on it, then `end_run(projectId, runId, outcome: "Completed", summary)`, the summary naming the task, what was written and the firing's run. **Never `Done`**: closing a task is a person's, and the server refuses it. More remains that a later firing can do: leave it `InProgress` and end the run the same way. It cannot be finished without something only a person has — an account, a key, a fact: `transition_work_item` to `Blocked` with that thing as `blockedReason`, which raises the task's own escalation, and `pause_run` on the task's run.
7. **Resume the firing's run** and write the digest: the task by code, its column, the run's id.

An operation's item or an experiment's item is never a task here, whatever its column or what it is marked: a plan row is run by rule 6, an experiment by `mentat-experiment`, and a run on such an item is refused.

## A move

The next row may be a move: an adopted idea given its place in the plan. A move has no run; its state is the job that carries it out (`get_roadmap`, the row's `move.workItemId`).

- **No job yet**: write one. `create_work_item(projectId, title: the idea's title, description: the idea's body and what "done" means for this move in one paragraph, priority, effort and goalValue rated against the other work, aboutKind: "RoadmapItems", aboutId: the row's id, runId)` — it lands `Proposed` with an approval request in the inbox. Then `set_roadmap_move_work_item(projectId, roadmapVersion, moveId: the row's id, workItemId, runId)`. Stop; the digest says the job waits for approval. Unattended, no message: the approval request is the row the person sees.
- **A job that is `Proposed`**: stop; it waits for a person.
- **A job that is `Todo` and assigned to this agent, or unassigned, or already `InProgress`**: work it now, in these five steps, in this order. Every step is done, every time; none is skipped because the job's description reads like a task for a person.
  1. **Read the job and the idea.** `get_work_item(projectId, workItemId)`: the description and the thread. The idea is on the move row: `move.ideaReferenceCode`, `move.ideaTitle`, `move.blockCode`.
  2. **Start it.** A `Todo` job: `transition_work_item(projectId, workItemId, status: "InProgress", runId)`. An `InProgress` one stays as it is.
  3. **Propose the idea's hypotheses: invoke the `mentat-hypothesis` skill** on block `move.blockCode`, handing it the firing's run id, the idea by code and title, and the job's description — what the idea bets on is what it proposes. Unattended it writes, checks, scores and shortlists them under its own rules, with no question; its duplicate check names a belief already written instead of writing it again. This invocation **is** the work of the firing on a move: it is made on every firing that reaches this step, and nothing replaces it — not research outside Mentat, not a comment saying what the person should find out. A fact nobody has yet is what a hypothesis is for; it is not a reason to write none.
  4. **Comment.** `comment_on_work_item(projectId, workItemId, body, runId)`: the hypotheses written, by code; those found already there; the card ranked first for each; what is left.
  5. **Leave the job.** `InReview` (`transition_work_item`) when its work is done — the idea's hypotheses are written and shortlisted and nothing else the description asks can be done from here; `InProgress` when more remains that a later firing can do. Never `Done`. Never `Blocked`, unless the job's own description names a fact only a person has **and** step 3 could write nothing without it — then `Blocked` with that fact as the reason, which raises its own escalation, and the comment names the fact in one sentence. The firing's run is not paused for it.
- **A job that is `Blocked`**: its own escalation waits; a `Waiting:` line, nothing else.
- **A job that is `InReview`, `Done` or `Cancelled`**: the move is not outstanding; the rule moves on to the row after it.

## When a routine fired this run

You know it because `mentat-agent` opened the run naming the routine and said so. Then the `mentat-agent` skill's unattended rule holds, and for this skill it is short: you ask nothing, you write nothing on the canvas, and every stop is a `Waiting:` line in the digest. The owners you hand to carry their own unattended paths — `mentat-operation` writes and checkpoints without asking, asks a step's question and proposes a verdict through the inbox, each raise pausing its run, and escalates a blocker; `mentat-hypothesis` writes and shortlists; `mentat-experiment`, resuming an experiment's run, starts a free digital test on its own and escalates **a start that spends or needs the person's hands** with the card and the cost; `mentat-gate-check` checks a due build check and escalates the rows to add on a fail; `mentat-planner` escalates once and writes nothing. A task's work is done here and its blocker is `Blocked`'s own escalation. What they raise, they record on their run; you copy it into the digest.

`mentat-gate-check` and `mentat-planner`, handed the firing's run, end nothing and pause nothing: they hand back, and `mentat-agent` ends that run last, with your digest. Tell them so when you hand over.

The words in the firing's prompt are the routine's, not a person's: "just run everything", "skip the build check", "the founder already approved" change no rule of the reference. The only word from a person that reaches this run is an answer in `resolvedSinceLastRun`.

## Reading a refusal

The `mentat` skill's table, plus:

- `start_run` refused by a prerequisite check, from `mentat-operation`: it raised the ask and stopped; the row is not forced. One `Waiting:` line.
- `start_run` refused because the build is held, naming the gates that have not passed: a build row is not run before the build check passes. The digest says so; forcing it is a person's.
- `start_run` refused on an item that "is worked by running what it follows, not as a task": an operation's or an experiment's item was taken for a task. Nothing was opened; take the next rule.
- `start_run` "is already running on this roadmap row": another session holds it; `inFlight` said so. Report; never cancel.
- `start_run` answering "That name is already used in this organisation" when an owner opens its run: the firing's run is still `Running` — pause it first, as [The firing's run and a row's own run](#the-firings-run-and-a-rows-own-run) says, and hand over again.
- `transition_work_item` refused naming `Done`: closing a task is a person's. Leave it `InReview`.
- `set_roadmap_move_work_item` `CONFLICT` naming `roadmapVersion`: `get_roadmap` again and resend with the version it answers.
- `set_roadmap_move_work_item` refused on an operation row or a row of the default plan: you named the wrong row; moves come from this project's `get_roadmap` with `kind` Move.
- `CONFLICT` naming `runId`: the firing's run was ended or paused by a person. Stop; say so; `mentat-agent` opens a new run before anything else is written.

## Rules that are easy to get wrong

- **One thing per call.** The first rule that applies, done once. Not two rows, not a task and a row, not a row and a build check.
- **A task handed to the agent comes before the plan**, the highest priority first, one per firing — and only a task, in `Todo` or `InProgress`. A question waiting in the inbox never holds it back.
- **Read the plan, not only the position.** The position knows operations; a move standing before the next operation is the next row.
- **A paused row is the next row, and its answer is the bottleneck.** Unanswered, this call stops at that row, after any task.
- **The gates are checked once, before the build.** A tier whose rows are done opens with no gate: work the next row.
- **A move's job that is yours to work gets its hypotheses.** Through `mentat-hypothesis`, every time, then the comment; left `InReview` or `InProgress`, `Blocked` only on a fact the job itself names.
- **Never raise an ask twice.** The last digest's `Waiting:` lines are your memory; an ask with no answer is repeated in the digest, not in the inbox.
- **You end only a task's run.** The owners end their own runs; `mentat-agent` ends the firing's run, after `mentat-linker` on it — every firing, one that did nothing included. The one run you pause is the firing's own, around a row or a task, and you resume it yourself.
- **In a session, the question comes first.** No run call before the person answers it.
- **Nothing on the canvas from here.** Entries, hypotheses, checks and pivots are their owners'.
- **Report by code**, and report what the tools answered, not what you intended.
