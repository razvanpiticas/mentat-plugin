# The boot brief, field by field

What `boot_agent(agent_id, project_id)` answers, in the order the response carries it. Load this when
a field is not obvious; the skill itself carries the routing.

## `company` — an array of documents

| Field | Meaning |
| --- | --- |
| `kind` | `CompanyProfile`, `Tenets`, `OperatingProcedures`, or `Custom` |
| `scope` | `Portfolio` for the group's document, `Project` for this venture's own |
| `title` | What the customer called it |
| `version` | The number to send back on `If-Match` if you ever revise it |
| `body` | The document itself, in markdown |

Ordered `CompanyProfile`, `Tenets`, `OperatingProcedures`, then `Custom` by title, and within each
kind the portfolio's document first and the project's second — **so the venture's text is read last
and wins where the two disagree.** Where the project's document of a kind has `replacesShared`, the
portfolio's document of that kind is not in the array at all. That is deliberate: a startup that wrote
its own procedures instead of the group's gets its own, not both.

## `mission` — one goal, or null

| Field | Meaning |
| --- | --- |
| `referenceCode` | `G-01`. Report it, never the identifier |
| `title`, `description` | What winning is, and the method for getting there |
| `metricName`, `metricUnit`, `metricTarget`, `metricDirection` | The number, its unit, what it must reach, and whether reaching it means at least or at most |
| `deadline` | The date the target is judged against |
| `status` | `Planned`, `Active`, `Achieved`, `Missed` or `Abandoned` |
| `latestValue`, `latestAsOf`, `latestSource` | The most recent measurement, the date it was true, and where it came from. All three null when nobody has measured |
| `ownerUserId`, `ownerAgentId` | Who is accountable. One of the two at most |
| `isMine` | Whether that owner is you |

A project with no mission is a project nobody has said what winning looks like for. Say so, and offer
to help the person write one — you cannot write it yourself.

## `objectives` — the tree under the mission

The same fields as the mission, plus `parentGoalId`, and a metric that may be absent: an objective may
be qualitative. `isMine` is what you plan from.

## `team` — the Key Resources block's people and agents

| Field | Meaning |
| --- | --- |
| `referenceCode` | `KR-03` |
| `kind` | `Human` or `Agent` |
| `role` | The entry's title: "Head of procurement", "Competitor intelligence" |
| `responsibilities` | The entry's body: what they own and what to ask them about |
| `reportsToCode` | The reference code of the entry above, or null at the top |
| `userId` / `agentId` | The identity behind the entry, when it is linked to one |
| `name` | The display name, joined from the member's profile or the agent's row |
| `isCaller` | This entry is the person you are working for |
| `isSelf` | This entry is you |

An unlinked `Human` entry is a key person who is not a member of the organisation — a supplier's
account manager, a contractor. Their name is all there is; there is nobody to ask.

## `person` — the signed-in member's profile, or null

`title`, `body` and `version`. Written in the person's own voice: their background, what they bring,
what they are learning, how to work with them. **When it is null, ask rather than assume.** Guessing
at somebody's working style from their role is how an agent becomes annoying.

## `agent` and `agentDocuments`

The agent row — `id`, `name`, `title`, and whether it is shared with the whole portfolio or belongs to
this one venture — then its documents in the same shape as `company`, `AgentInstructions` first and
`Custom` by title after it.

`AgentInstructions` is the one document that describes you: your name, your voice, your mindset, what
you own, your standing orders and your hard boundaries. Where it and your own defaults disagree, it
wins — that is what it is for.

## What is not in the brief, and where it is instead

| Not here | Read it with |
| --- | --- |
| The rest of the canvas | `mentat-canvas`: `get_block` for one block, `get_canvas` only when the task spans them |
| The routines of the project | `list_routines` |
| Documents with `loadAtBoot` false | `list_charter_documents`, then the document by id |
| Other agents' instructions | Nothing. They are not yours to read |
| Any row of a sibling venture | Nothing. A project sees its own rows and the shared ones, never a sibling's |
| The runs and the activity behind you | The runs screen in the browser |
