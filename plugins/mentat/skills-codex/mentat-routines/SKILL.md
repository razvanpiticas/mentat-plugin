---
name: mentat-routines
description: Routines: switch on, run now. The routines menu — what a Mentat project has scheduled, switching a routine on or off in both places it lives (the server row and this machine's scheduler), adding a new one, rescheduling one, and running one now by hand. Use when the person asks what runs on its own, wants the Heartbeat, the Nightly distiller or the Nightly linker switched on or off, wants a new scheduled job, or wants a routine run right now.
---

# Mentat routines

A routine is a row on the server: a name, a five-field cron line, the zone it is read in, the agent
it boots, and instructions written to that agent. **The server never fires one.** This machine
does, and only if an entry for it exists in a scheduler here. So a routine lives in two places, and
switching it on means writing both — one without the other is a routine that never fires, or a
schedule the product shows as off.

Every call here is the person's, made outside any run: a schedule is set by somebody who has read
it, and the server refuses these calls from inside a run.

## The menu

With no argument: find the project (`list_portfolios`, `list_projects`, or `get_project` when the
id is known), then `list_routines` and, on this machine, what is installed (below). Show one line per
routine — name · cron line · read in · who does it · on or off · last fired · installed here or
not — then offer: switch one on, switch one off, reschedule one, add one, run one now.

Every project is born with three, all switched off until a person reads them: **Heartbeat** (twice
a day: advances the plan by one row), **Nightly distiller** (folds the day's learnings), **Nightly
linker** (gives the day's inferred links their word).

## Switching a routine on

1. `get_routine` and show the instructions whole. The person reads them and says yes; a routine is
   switched on by somebody who has read it. Offer to change what does not fit (reschedule, below).
   STOP and use Codex's structured user-input tool when available; if it is unavailable, ask directly in chat to clarify.
2. **Server first:** `enable_routine`. `CONFLICT` "already switched on" means somebody did it, or
   the entry is what is missing: go on to step 3 either way.
3. **Then this machine.** The entry runs the harness on its own with the boot prompt, in this
   project's directory:

   > Use the mentat-agent skill: boot the agent "<agent name>" on the project "<project name>" and
   > run its routine "<routine name>".

   Install an operating-system entry:

   ```
   pnpm dlx github:razvanpiticas/mentat-plugin schedule add --harness codex --routine <routineId> --name "<routine name>" --cron "<line>" --zone "<zone>" --project-dir "<this project's directory>" --prompt "<the boot prompt>"
   ```

   It writes a crontab line on macOS and Linux, or a Task Scheduler task on Windows, that runs
   `codex exec "<the boot prompt>" --sandbox workspace-write` in the project directory. Codex reuses
   the sign-in saved on this machine; whether that reaches a scheduled run is proved by step 5, not
   assumed.

   On a harness with no verified way to run a prompt on its own — Cursor, Gemini CLI, OpenCode,
   OpenClaw, Hermes — print the routine's cron line, its zone and the boot prompt, and say: "install
   this in a scheduler of your own that starts <harness> with that prompt; until then, run it by
   hand with *run now*". Do not guess a command.

   The command refuses a zone that is not this machine's: cron and Task Scheduler read the machine's
   clock, and a line cannot be moved between zones honestly. Then offer to set the routine's *Read
   in* to this machine's zone with `update_routine`, and install again.

4. **If step 3 fails**, `disable_routine` and say so: a routine that is on and not installed is one
   nobody notices never fires.
5. **Run it once now** (below) before trusting the schedule. The first run proves the sign-in
   reaches a run nobody started by hand, and on Claude Code it is where the tool approvals are
   saved.

Report: on, installed as `<what>`, next fires at `<time>` in `<zone>`.

## Switching a routine off

This machine first — remove the entry (`schedule remove --routine <routineId>`, or delete the
Desktop task, or tell the person which line to remove on a harness with no command) — then
`disable_routine`. In that order: if the second fails, the routine is on but not installed, which
never fires; the other way round is a schedule the product shows as off and still runs.

## Rescheduling or editing

`update_routine` takes all five fields together — send the current values for what is not changing.
When the routine is switched on and the line or the zone changed, reinstall the entry in the same
breath: `schedule remove` then `schedule add`, or edit the Desktop task. Everything the routine says
is in `get_routine`; rewrite instructions the way the seeded ones are written —
[reference/writing-a-routine.md](reference/writing-a-routine.md).

## Adding a routine

Ask for the five: a name the project has not used; the agent it boots (`list_agents`; or none, to
run as whoever fired it); the five-field cron line — numbers, `*`, lists, ranges and steps only;
the IANA zone it is read in (ask; never take the server's, it has none); the instructions, written
to the agent in the second person, every step naming something that exists. STOP and use Codex's structured user-input tool when available; if it is unavailable, ask directly in chat to clarify.
`create_routine`; it is born switched off. Then the switch-on flow.

## Running a routine now

Use the `mentat-agent` skill with the agent, the project and the routine's name. It reads the
routine with `get_routine`, opens a run naming the routine, and follows the instructions; the run's
summary lands in the person's inbox as the run report. This is how "run the linker", "run the
distiller" and "run the heartbeat" are done by hand, and how a routine is tested before it is
scheduled.

**A routine run by hand is still the routine's run**, and `start_run` carries its id — a routine that
is switched off still names the work, and this is the one way of running one that the schedule has
never run. What the run files, and where, is the same either way; the person being at the window adds
somewhere else to say it, and takes nothing away.

## Reading a refusal

- `CONFLICT` "already switched on" / "already switched off" — the server is where you wanted it;
  check the entry on this machine instead.
- `CONFLICT` naming the routine's name — the project already has one; pick another.
- `NOT_FOUND` on an agent — not one this project can boot; `list_agents` again.
- `INVALID_ARGUMENT` naming `schedule` or `timeZone` — the line does not parse, or the zone is not
  one the server knows; show the person the field.
- `400` saying only a person sets a project's schedule — the call was made inside a run. It never
  should be: this skill carries no `runId`.
- `schedule add` refusing the zone, or a line Task Scheduler cannot express — the message says
  what to change; [reference/scheduler.md](reference/scheduler.md) has the limits.

## Rules that are easy to get wrong

- **Both places or neither.** On: server, then machine. Off: machine, then server.
- **Nobody switches on what they have not read.** `get_routine` first, always.
- **Never inside a run.** These are the person's calls.
- **Never guess the zone.** Ask, and refuse to install a line the machine would read in another.
- **The server never fires anything.** A routine that is on and has no entry here is silent.
- **A first run by hand before a first run by schedule.**
