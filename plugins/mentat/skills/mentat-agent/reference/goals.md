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
worked out yourself. If you cannot name where a figure came from, you do not have a figure — ask. A
reading an experiment took names the experiment by its code: "T-07: 3 of 10 founders paid the tested
price". The server does not copy an experiment's metric onto a goal; you record it.

## Is the number met

`list_goals` answers `isMet` on every goal with a metric: whether its latest reading meets its target
in its direction. Read it rather than working it out; the goals page draws the goal from it — on
paper until it is met, the plate once it is.

## Staleness

There is no staleness flag on a goal. You judge it: compare `latestAsOf` with today and with what the
metric measures. A revenue figure from three months ago on a goal due in six is stale. A headcount
from three months ago probably is not.

A stale number on a goal you own is the first thing to raise, before any proposal — a proposal made
against a number nobody has checked is a guess with a reference code.

## What serves a goal, and what it rests on

`link_to_goal(projectId, itemKind, itemId, goalId, relation, figure, runId)` draws an item to a goal in one of
two ways:

| `relation` | `itemKind` | What it says |
| --- | --- | --- |
| `ServedBy` | `Hypotheses`, `Experiments`, `RoadmapItems` | the work taken on to move the goal: a hypothesis under test, an experiment, a row of the plan |
| `RestsOn` | `BlockEntries`, `Hypotheses` | what the goal's target is built on: the entry holding the total addressable market, the hypothesis that customers pay |

`list_goals` answers both on the goal, by code: `servingItems`, each with its state (an experiment
`Running` shows the goal being tested now), and `restsOn`, a hypothesis there with its `evidenceCount`,
the bundles of evidence bearing on it.

An entry the goal's target is reached through — revenue through the price — is drawn with the figure
the target uses of it: `figure: { magnitude: 39, unit: "USD a month", meaning: "the launch price" }`,
the number above zero in the target's own period, and on `RestsOn` to an entry only (anything else is
refused). `meaning` names which figure it is: the launch price and the founding price are two figures,
and the target uses the one the founder says it is reached at. `restsOn` answers it as the entry's `figureUsed`, and `worksOutTo` what the target
works out to beside each beachhead TAM the goal rests on: `customersNeeded` (target ÷ figure, rounded
up) and `shareOfEndUsers` (of 1) of the market's `endUsers`, called `endUsersAreCalled` — "Works out
to 129 customers at $39 a month · 0.5% of the beachhead". No figure, no target or no market: no line.
A link already drawn without its figure is taken off with `unlink_from_goal` and drawn again with it.

- One item may serve several goals. Send `link_to_goal` once per goal.
- Drawing the same pair twice is refused with `CONFLICT`. That is the unique index, not a fault — the
  link is already there.
- `unlink_from_goal` removes one, either relation. It does not touch the item.
- Rests on with anything but an entry or a hypothesis is refused: "A goal rests on an entry or a
  hypothesis, and {table} is neither." A task is never drawn to a goal. A goal serving a goal is the
  tree, drawn by the parent and not by this word.

A goal nothing serves is a goal nobody is working on. That is a thing to raise.

## To review

A goal is marked to review, with the reason, when an entry or a hypothesis it rests on is rewritten in
a run, when an experiment serving it completes, or when a hypothesis it rests on is decided `Pivot` or
`Kill`. `list_goals` answers each open review in `reviews`: the reason, what changed (its table and
id) and the run that changed it. A rewrite that moves a figure carries what changed
(`whatChanged: { from: "1,330 developers", to: "25,000" }`, written by the operation that rewrote it, on the entry's rewrite and on every write
of a segment's, a job's, a value proposition's or a Strategy Canvas's figures), and the reason then says it: "Total addressable market of the
beachhead changed: 1,330 developers became 25,000." The review keeps the two apart (`changedFrom`, `changedTo`), and the
row under the goal's Rests on reads "Changed on 31 Oct: it was 1,330 developers." The mark stays until a person accepts or rejects a proposed change, or
says on the goals page that the goal stands as it is.

## A change to a goal

A goal changes only through a decision a person accepts. `propose_goal_change(projectId, kind,
goalId?, parentGoalId?, title?, description?, metric?, deadline?, reason, figures, runId)`:

| `kind` | Sends | Becomes, once accepted |
| --- | --- | --- |
| `Change` | `goalId`, what changes (`title`, `description`, `metric`, `deadline`), `figures` — each `{ label, was, becomes }` | the goal rewritten, the line kept in its history |
| `NewObjective` | `parentGoalId`, `title`, `metric`, `deadline` | a new objective under that goal, `Planned` |
| `Abandon` | `goalId`, `reason` | the goal `Abandoned`, its card reading the day and the reason; the only way a goal is abandoned |

Every kind carries `reason`, naming what changed — the entry and the operation, the experiment, the
hypothesis — and answers a decision, `Proposed`, titled with the goal and the change; the goal reads as
it did until the person accepts it on the decisions tab, and a rejection leaves it as it is. Both clear
the mark to review. A goal counted in another unit is not changed: propose a new objective for the new
unit, which is what the refusal says when you try.

## What you may not do

Creating a goal outside the planner's interview, editing one, assigning its owner and moving it along
its status are the person's work on the goals screen; update_goal and set_goal_status are refused to
an agent, and set_goal_status refuses `Abandoned` to everybody. Accepting a decision is the person's, your own proposal included. You read goals, you measure
them, you put work behind them, you draw what they rest on, and you propose their changes.
