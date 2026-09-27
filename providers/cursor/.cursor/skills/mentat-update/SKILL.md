---
name: mentat-update
description: Update the plugin: moves the installed Mentat plugin to the version published on the public mirror, and reports what changed. Use when asked to update, upgrade or refresh Mentat, the Mentat plugin or the Mentat skills, when a Mentat tool is missing or behaves unlike its documentation, or when the tools the server reports disagree with what the skills describe.
---

# Updating Mentat

This plugin is published from Mentat's own repository to a public mirror, and every harness installs
it from there. Nothing arrives on its own unless the harness was told to look, so a skill that
disagrees with the server is almost always a plugin that was never refreshed.

Run this whole file top to bottom. It is one act, not a menu.

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

Here `mentat-agent` boots the agent and hands you the session; nothing below calls a Mentat tool but
`server_info`, and nothing carries the run's id. The one exception to the boot: when `mentat-agent`
answers that the Mentat server cannot be reached or signed in to, the update goes on without it,
because an out-of-date plugin is the usual cause and this file is the repair. Say so in the report.

## 1. Find out whether there is anything to do

The published version is the `version` field of the manifest on the mirror's default branch:

```bash
# One HTTPS GET, no credentials, no gh CLI. Answers something like 0.2.0+e50c0e722efd.
curl -fsSL https://raw.githubusercontent.com/razvanpiticas/mentat-plugin/main/plugins/mentat/.claude-plugin/plugin.json | grep '"version"'
```

This harness installs the skills as plain files, with no manifest and no recorded version, so there
is nothing local to compare. Treat the published version as the answer and go on to step 2.

The two strings are compared for equality, not ordered — the part after `+` is the source commit the
build was made from, so any difference at all means the installed copy is behind.

**Equal strings mean stop.** Say so and do nothing else. Reinstalling an identical version churns the
harness's cache and proves nothing.

## 2. Install the published version

Re-run the installer from the project directory. It rewrites the files it wrote before, so there is
nothing to uninstall first.

```bash
pnpm dlx github:razvanpiticas/mentat-plugin install
```

Reload the window afterwards so the rewritten skill files are read again.

## 3. Say what actually happened

Report the version it moved from and to, and name the reload the user still owes. An update that is
installed but not reloaded looks exactly like an update that did not happen, and the user is the only
one who can tell the difference from the outside.

If a command failed, give its output rather than a summary of it. The two failures worth naming:

- **The marketplace is not registered.** Nothing to update; the plugin was installed some other way,
  or under another marketplace name. Install it as the mirror's README describes rather than
  guessing at a name.
- **The refresh succeeded and the version did not move.** The publish has not finished. Mentat's
  workflow mirrors within about a minute of a push; wait and run this again.

## What this does not do

It does not touch the Mentat server, the canvas, or any project data — it only replaces the files
this plugin installed. `server_info` is the check for whether the *server* is reachable and current,
and it answers with the deployment that responded. When the tool list it reports disagrees with what
any skill says, the tool list is right and this file is what needs running.
