---
name: mentat
description: The front door to Mentat and the connection under every other Mentat skill. Invoked with nothing, it signs in, wakes the venture's Chief Executive, says where the venture stands and what happened last, proposes the one thing that should happen next, and offers everything else the plugin can do. Use when the person types the Mentat command alone, asks "where do we stand", "what's next", "what can you do", or for a menu; when a Mentat tool refuses a call; or when connecting a harness to the Mentat server.
---

# Mentat

Mentat's MCP server fronts the product's BusinessIntelligence service. It holds no database of its
own: every tool call is an HTTP call to a service that already enforces the product's own rules,
made **as the person using the model** rather than as a service account.

That is the fact everything else follows from. The server never sees more than the caller does, so
a tool that refuses is usually reporting the caller's own access rather than a fault.

**A task typed after the command is routed before anything else is done**, by the test in
[The front door](#the-front-door): work that writes goes through `mentat-agent`, a question about
the canvas goes to `mentat-search`, and nothing goes straight to a method skill — not even when that
skill's description names the task, as `mentat-operation`'s names a row of the plan. The first skill
invoked for work that writes is `mentat-agent`, even when you already know which skill will do the
work: that skill is invoked from inside `mentat-agent`, after the boot, never from here.

## Connecting

The server is declared under `mcp_servers` in `~/.hermes/config.yaml`.

Sign-in is OAuth against the Keycloak realm at `keycloak.quantarcane.io`, using the public client
`mentat-public-client` and the loopback callback on port `8125`. Both halves are fixed: a different
port is a redirect-uri mismatch, not a preference.

**Call `server_info` first when anything is wrong.** It answers without touching any downstream
service, so it separates "the server is unreachable or I am not signed in" from "the server is fine
and the call was refused".

## The front door

`the mentat skill` with nothing after it is the way in for somebody who does not know what to ask.
Named with a task instead — after the command, or as the answer to step 2 or step 6 — the task skips
the status and the menu and goes one of two ways, and never straight to a method skill. One test
decides which, and no list of tasks does: **does this work write anything** — an entry, a hypothesis,
a run, a checkpoint, a verdict, an evaluation, a message? Then it goes through `mentat-agent`.
Does it only read, to answer a question about the canvas? Then `mentat-search`. A skill whose
description seems to match the task is not the test: the task is routed by the test first, and the
skill that does the work is reached from inside `mentat-agent`.

- **It only reads** — "what is KR-01 tied to", "what do we know about pricing", "is there already a
  hypothesis like this" — so it is invoked on `mentat-search` with the project and the question. It
  needs no agent and no run. Do not answer it here from `get_block` or the brief: which instrument
  answers is that skill's rule, and its answer names the one it used.
- **It writes** — so it goes through `mentat-agent`: pick the project as step 2 says when it is not
  known, then boot the CEO as step 3 says, and hand it the task. Running a row of the plan writes,
  the plan and "advance" write, a gate check writes its evaluation, the linker and the distiller write
  — and so does every task not named here that is not a question. `mentat-agent` boots the agent and
  invokes the skill that does the work; the run — opened by `mentat-agent`, or by `mentat-operation`
  for a row of the plan — carries the agent's id, every write carries the run's id, and the run ends
  with its links typed. A method skill invoked from here
  instead writes as the person, outside the agent's run — or opens one nobody booted — and nothing
  types its links.

The routines menu and the plugin update are the person's own and open no run: they are invoked
directly.

Six steps, in this order, and every one of them reads before it says anything.

1. **Sign in.** Call `list_portfolios`. A 401 means sign in again, as [Connecting](#connecting) says;
   a 403 is explained, not retried; any other refusal — `server_info` first, then
   [Reading a refusal](#reading-a-refusal). Nothing else is called until this answers.
2. **Pick the project.** One portfolio with one project: take it, say which. More than one: list them
   by name with their tier and ask. Ask the user directly to clarify what you cannot infer. Never pick for the person; a status of the
   wrong venture is worse than a question. Keep both identifiers the reads answered: the portfolio's
   and the project's. `list_runs` takes both.
3. **Boot.** `list_agents` for the project; boot the agent named "CEO" — every organisation is born
   with one — through the `mentat-agent` skill, which calls `boot_agent` and reads the brief in its
   order. When there is no agent called CEO, boot the one shared agent the list answers; when the list
   is empty, say so and continue as the signed-in person with `get_project_state` instead. Do not call
   `get_project_state` as well as `boot_agent`: the brief carries the same section.
4. **Status.** Five lines, each read from the brief or from one more call, never invented — the fields
   are in [reference/front-door.md](reference/front-door.md):
   - where the venture stands: tier, status and overall confidence (`get_project`), the mission's
     latest number against its target and deadline, or "no measurement yet" (the brief);
   - the plan: the row last finished, the one in flight or next, how many operations of this tier are
     done out of how many and whether the gate would pass (`get_gate_status`), whether the gate is due
     (the brief's plan position) or the project awaits a person's unlock (its status);
   - what happened last: the changes since the CEO's last run, from the brief, in one sentence — who
     wrote what, by reference code — and the last three runs with their one-line summaries
     (`list_runs`, page size 3); `list_activity` only when the brief says its delta was truncated;
   - what waits for the person: the inbox count from the brief, the open work, the answers given since
     the last run;
   - what is open on the canvas: which blocks are still empty, open questions, top risks, hypotheses by
     status, running experiments, open contradictions — counts, from the brief.
5. **The next action.** One sentence, from the next-action rule in
   [reference/front-door.md](reference/front-door.md), naming the skill that does it and asking whether
   to do it now.
6. **The menu.** The lines in [reference/front-door.md](reference/front-door.md), verbatim, then do
   what the person picks the way a named task goes, above: work that writes through the CEO booted in
   step 3, a question through `mentat-search`. A line marked "not in this version yet" answers one
   sentence and shows the menu again.

The whole thing is read-only. Nothing is written, no run is opened, until the person picks something;
then the run is opened through `mentat-agent`, never by a method skill invoked from here.

## What access means here

Two permissions gate everything:

| Permission | Covers |
| --- | --- |
| `mcp:tools.read` | Every tool that only reads |
| `mcp:tools.write` | Every tool that changes something |

They are granted through Mentat's own roles screen, per tenant, like every other permission in the
product. A session where reads succeed and writes answer 403 is a role problem, not a connection
problem — and re-authenticating does not fix it, because the token has to be minted again *after*
the role is granted.

Every call is also tenant-scoped from the caller's token. A caller who belongs to no tenant gets
403 on everything, which is correct rather than broken.

## The tools

One hundred and twenty-seven tools, all over BusinessIntelligence: thirty-nine reads (portfolios, projects,
where a project stands, the canvas, one block by code, one hypothesis, one experiment, the method
cards, one procedure of the method catalogue, a project's plan, its runs, its agents, routines and
goals, its learnings, its board, its change log, its decision ledger, and the three ways of finding a
row nobody knows the code of) and eighty-eight writes (entries, hypotheses, experiments, evidence, questions, risks,
ideas, contradictions, insights, the plan's rows, runs, method gates, agents, routines and goals, the
board's jobs, escalations, messages and decisions). The full table with permissions and which tools take which version
is in [reference/tools.md](reference/tools.md). **How to use them — which kind, which fields, what
order BusinessIntelligence enforces — is the `mentat-canvas` skill; load it for any read or write on a
canvas or a plan.** Being one of the organisation's agents and working its board is the `mentat-agent`
skill.

`server_info` reports the tool list; when it disagrees with this file or any document, it is right.

## Reading a refusal

A refused call names the arguments at fault and says whether retrying is worth anything. Read it
rather than resending the same call.

- **The refusal names an argument** — fix the argument and retry once.
- **The refusal is a 403** — a permission or tenant problem. Retrying cannot fix it. Tell the user
  which permission is missing.
- **The refusal is a 401** — the token expired or was never minted. Sign in again.
- **The refusal is a CONFLICT naming a canvas version** — the canvas moved under you. Re-read
  and resend with the version it names; the `mentat-canvas` skill explains the version rule.
- **The refusal is a CONFLICT naming a roadmap version** — somebody else changed the project's plan.
  That is a different number from the canvas version: re-read with `get_roadmap` and resend with the
  one it names.
- **The refusal is a CONFLICT naming `runId`** — the run you are working inside was ended, paused or
  opened by somebody else, so nothing was changed. Every other call carrying that run is refused the
  same way. Stop working it, say so, and `start_run` a new one before writing anything else.
- **The refusal is a CONFLICT quoting a rule of the business model** — the call is not allowed in the
  canvas's current state. Change what you asked for; do not resend.
- **The refusal says a downstream holds nothing at that address** — an id or a code named nothing.
  Read what exists and address it by what the read answered.

A refusal that fits none of these is worth a question rather than a second attempt.
Ask the user directly to clarify what you cannot infer.

For symptoms that survive a retry, read
[reference/troubleshooting.md](reference/troubleshooting.md).

## Rules that are easy to get wrong

**Never invent a tenant.** No tool takes a tenant argument, and none should — the tenant comes from
the token. A call that appears to need one is a sign the wrong tool was chosen.

**Do not cache what a read returns across turns.** It is live state that another person in the
same tenant can change between calls, and every write is checked against the canvas version you
last read.

**A write is not confirmed until the tool says so.** Every write returns the row it wrote; if the
call refused, nothing was written unless the message says otherwise, and reporting otherwise to
the user is worse than reporting the failure.
