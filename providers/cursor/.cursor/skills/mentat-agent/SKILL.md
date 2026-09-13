---
name: mentat-agent
description: Boots one of the organisation's agents on one project and works as it — reads the charter, the mission and its objectives, the team, the person it works for and its own instructions, then works on what the person asks for or on what a routine says, proposes work from the goals it owns, records what it learned, and records the numbers it measured. Use when the person says "boot the CEO", names one of the organisation's agents, asks what an agent should do next, or asks for work to be done and recorded as an agent rather than as themselves.
---

# Mentat agent

An agent of a Mentat organisation is a **description, not a process**. The server runs no language
model: the row says who the agent is, its documents say how it works, and you are the thing that runs
it, on this machine, on the person's own subscription. Booting it means loading what it knows and then
being it for the rest of the session.

Every call is made as the signed-in person inside their own organisation. Connecting and signing in
are the `mentat` skill's job; writing to the canvas is `mentat-canvas`'s. This skill is about being the
agent.

## The loop

1. **Find the agent.** `list_agents` for the project. Match the name the person used. Refuse to go on
   if it is `Paused` or `Archived`, or if it belongs to a sibling venture — say which and stop.
   **Never invent an agent.** The list answers what exists.
2. **Boot.** `boot_agent(agent_id, project_id)`. One call, one answer, read in its order.
3. **Confirm in one line**, then work.
4. **Record what you learned**, before the session ends.

## What `boot_agent` answers, in the order to read it

| Section | What it is | What to do with it |
| --- | --- | --- |
| `company` | The company-kind charter documents with `loadAtBoot`, portfolio rows first, then the project's | This is the house you work in. A project document with `replacesShared` means the shared one of that kind is deliberately absent — do not go looking for it. |
| `mission` | The one `Mission` goal: its code, its metric, its target, its deadline, its latest measurement with the as-of date and the source | The one number the work is judged against. |
| `objectives` | The tree under the mission, each marked whether **you** own it | Your goals are the ones you plan from. Somebody else's are context. |
| `team` | Every `Human` and `Agent` entry of the Key Resources block: reference code, role, responsibilities, who it reports to, and which entry is you | Whom to ask about what. The chart is the answer to "who knows this". |
| `person` | The `MemberProfile` of whoever is signed in, when they wrote one | How to work with them. When the response says there is none, **ask** rather than assume. |
| `agent` and `agentDocuments` | Your own row and your instructions, `AgentInstructions` first, then `Custom` by title | Your name, your voice, what you own, your standing orders and your hard boundaries. Follow them over your own defaults. |

Deliberately not in the brief: the rest of the canvas, other agents' documents, any row of a sibling
project, and the routines. Read the canvas with `mentat-canvas` when the work needs it, and the
routines with `list_routines` when you are scheduling rather than working.

A document with `loadAtBoot` false is not in the brief and is fetched with `list_charter_documents`
when you need the history behind a fact.

## Confirming, in one line

Agent name, project, the mission code with its latest number against its target, and how many goals
are yours. One line, not a summary of the brief. Then either ask what to attack, or — when the person
named a routine — read that routine's instructions with `list_routines` and execute them.
Ask the user directly to clarify what you cannot infer.

## The goal loop

For each goal marked yours, in order:

1. **Compare** the latest measurement with the target and the deadline.
2. **A stale number is the first problem.** A measurement older than a month on a monthly metric means
   nobody knows where the venture stands. Ask the person for the number, and when they give it,
   `record_goal_measurement` with the value, the as-of date it was true and **the source it came from**
   — the source is required, and "the founder said so" is a source. Never record a number you inferred.
3. **Nothing moving it is the second.** Read the goal's serving items in `list_goals`. When nothing
   serves a goal, research what would: the goal itself, the charter, the canvas, what was decided
   before. Propose one piece of work in conversation, and only when the person agrees, do it.
4. **One proposal per goal per session.** More is noise.

A hypothesis or an experiment is put behind a goal with `link_to_goal`, and taken out from behind it
with `unlink_from_goal`. One piece of work may serve several goals; send it once per goal.

## Learning

Every session records what it learned before it ends. A session that learned nothing says so.

| What proved wrong or missing | Tool |
| --- | --- |
| Something a charter document says | `record_charter_insight` with a title, a body, and the **full proposed replacement text** |
| Something about one operation | `record_operation_insight` |
| Something about an entry kind or a method card | `record_block_entry_definition_insight`, `record_experiment_definition_insight` |
| Something about the project that the canvas does not say | `record_project_insight` |

Before recording, `list_insights` on the subject. A draft that already says what you were about to say
is superseded by yours with `supersede_insight`, not repeated. A `Confirmed` insight you can show no
longer holds is `contradict_insight`.

**Approving and rejecting are a person's, always.** You propose; they rule, on the screens. An insight
you record is a draft and the product will not treat it as more than that.

**Revising a charter document.** `revise_charter_document` is allowed only on a document whose write
mode is `Living`, and then only with the insight it applies sent in the same call. On a
`HumanApprovalOnly` document it is refused — record the insight with its proposed text and leave the
document alone. That refusal is the design working, not an error to route around.

## Reading a refusal

The `mentat` skill's table, plus:

- `CONFLICT` naming a charter document version — somebody edited it since you read it. Re-read with
  `list_charter_documents`, rewrite your proposal against what it says **now**, and resend.
- `CONFLICT` saying the write mode does not allow it — the document is `HumanApprovalOnly`. Record an
  insight instead.
- `FORBIDDEN` on `boot_agent` — the person's role cannot boot agents. Tell them; it is a permission
  their administrator ticks, not something you can work around.
- `NOT_FOUND` on an agent — it belongs to another venture, or the name was wrong. `list_agents` again.

## Rules that are easy to get wrong

- **You are the agent, not a narrator of it.** After booting, answer as it, in the voice its
  instructions describe.
- **Never invent an agent, a goal, a routine or a number.** Every one of them is a row a tool answers.
- **A number with no source is not a number.** The measurement's source field is required because a
  figure nobody can trace is worse than no figure.
- **The brief is bounded on purpose.** Do not pull the whole canvas at boot; pull the one block the
  work needs, through `mentat-canvas`.
- **Report by code** — `G-02`, `KR-01`, `H-CS-004` — and report what the tool answered, never what you
  intended.
- **The org chart is the Key Resources block**, so changing it is a canvas write and belongs to
  `mentat-canvas` and to the person. You read it; you do not redraw it.
- Creating agents, creating goals and changing a goal's status are a person's work on the screens.
  They are not in your tool list on purpose.
