# Goals, measurements and what serves them

Load this before your first measurement or link of a session.

## The shape of a goal tree

One project has at most one `Mission`, and every other goal is an `Objective` under it or under
another objective. The mission is the only one that **must** carry a metric and a deadline; an
objective may be qualitative.

Codes are `G-01`, `G-02`, issued per project in the order the goals were written. Report the code.

## The metric

| Field | Rule |
| --- | --- |
| `name` | What is being counted: "Online revenue", "Median delivery time" |
| `unit` | "EUR", "hours", "orders/day". Null means a plain count |
| `target` | The number to reach |
| `direction` | `AtLeast` — bigger is better. `AtMost` — smaller is better |

A goal is met when its latest measurement meets the target **in its direction**. A delivery time of
three hours against an `AtMost` target of four is met; the same three against `AtLeast` is not. Read
the direction before you say a goal is on track.

## Measurements are append-only

`record_goal_measurement` takes a value, the date it was true (`asOf`), a **required** source, and an
optional note. Nothing edits or deletes one. A wrong number is corrected by recording a later, better
one — the older row stays, because when a number was believed is part of the record.

The goal's current value is the latest by `asOf`, so recording a measurement for last month today does
not overwrite this month's.

**The source is the point.** "PrestaShop orders export 2026-09-01", "Finestore's filing", "the founder
in conversation on 12 September" are all sources. "Estimated" is not, and neither is a number you
worked out yourself. If you cannot name where a figure came from, you do not have a figure — ask.

## Staleness

There is no staleness flag on a goal. You judge it: compare `latestAsOf` with today and with what the
metric measures. A revenue figure from three months ago on a goal due in six is stale. A headcount
from three months ago probably is not.

A stale number on a goal you own is the first thing to raise, before any proposal — a proposal made
against a number nobody has checked is a guess with a reference code.

## What serves a goal

A hypothesis under test or an experiment being run is put behind a goal with `link_to_goal`, naming
the item kind (`Hypotheses` or `Experiments`), the item, and the goal. `list_goals` then names the
serving items on the goal, by code.

- One item may serve several goals. Send `link_to_goal` once per goal.
- Drawing the same pair twice is refused with `CONFLICT`. That is the unique index, not a fault — the
  link is already there.
- `unlink_from_goal` removes one. It does not touch the hypothesis or the experiment.
- An entry of the canvas is the **model**, not an activity, and is refused. A goal serving a goal is
  the tree, drawn by the parent and not by this word.

A goal nothing serves is a goal nobody is working on. That is the second thing to raise.

## What you may not do

Creating a goal, editing a goal, assigning its owner and moving it along its status are the person's
work on the goals screen, and those tools are not in your list. You read goals, you measure them, and
you put work behind them.
