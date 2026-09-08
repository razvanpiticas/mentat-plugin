---
name: mentat-update
description: Moves the installed Mentat plugin to the version published on the public mirror, and reports what changed. Use when asked to update, upgrade or refresh Mentat, the Mentat plugin or the Mentat skills, when a Mentat tool is missing or behaves unlike its documentation, or when the tools the server reports disagree with what the skills describe.
allowed-tools: Bash, mcp__mentat__server_info
---

# Updating Mentat

This plugin is published from Mentat's own repository to a public mirror, and every harness installs
it from there. Nothing arrives on its own unless the harness was told to look, so a skill that
disagrees with the server is almost always a plugin that was never refreshed.

Run this whole file top to bottom. It is one act, not a menu.

## 1. Find out whether there is anything to do

The published version is the `version` field of the manifest on the mirror's default branch:

```bash
# One HTTPS GET, no credentials, no gh CLI. Answers something like 0.2.0+e50c0e722efd.
curl -fsSL https://raw.githubusercontent.com/razvanpiticas/mentat-plugin/main/plugins/mentat/.claude-plugin/plugin.json | grep '"version"'
```

The installed version is in this plugin's own manifest, which the harness points at:

```bash
# CLAUDE_PLUGIN_ROOT is set for every command this plugin runs. Empty means the skill is loaded
# from somewhere other than an installed plugin, so there is no install to compare against.
cat "$CLAUDE_PLUGIN_ROOT/.claude-plugin/plugin.json" | grep '"version"'
```

The two strings are compared for equality, not ordered — the part after `+` is the source commit the
build was made from, so any difference at all means the installed copy is behind.

**Equal strings mean stop.** Say so and do nothing else. Reinstalling an identical version churns the
harness's cache and proves nothing.

## 2. Install the published version

Two commands, in this order. The first is the one that is usually missing: the marketplace is a git
clone on this machine, and it is never refreshed by installing or by reloading.

```bash
# Refresh the local clone of the marketplace, then move the plugin onto what it now holds.
claude plugin marketplace update mentat-plugins
claude plugin update mentat@mentat-plugins
```

Then tell the user to run `/reload-plugins`. Claude cannot reload the plugin it is
running inside; the harness reloads it, and until it does the old skills are still the loaded ones.

**Offer to switch auto-update on** if the user has just been surprised by a stale plugin. It is
`/plugin`, then Marketplaces, then `mentat-plugins`, then Enable auto-update. From
then on Claude Code refreshes the marketplace after each session starts and says when to reload, and
step 2 stops being something anybody has to remember. It is a menu, so the user does it, not you.

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
