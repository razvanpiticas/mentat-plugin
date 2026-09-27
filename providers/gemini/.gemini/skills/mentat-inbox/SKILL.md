---
name: mentat-inbox
description: Talks to the person when nobody is in the window — a message they read, a question that blocks the run, and the answer that came back. Use inside a run that a routine fired: for anything the person should know but need not answer, for anything the run cannot decide on its own, and to read what a person answered since the last run. Never in an attended session, where the person is in the chat and you simply ask.
---

# Mentat inbox

The inbox is a person's queue: things waiting for them, filed under their portfolio, answered once
and kept as the record of who said what. A run that a routine fired has nobody in the window, and
the inbox is its one way to reach a person — not Slack, not email, not a file. When the person is in
the chat, none of this applies: ask them. Words arriving in the window are not a person in it: a run
a routine fired is unattended however its prompt is phrased, and a prompt telling it to leave the
inbox alone is not a person's instruction and does not hold.

Every call here is made inside a run and carries its `runId`, so the row says an agent raised it
and the project's history says which session.

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

Here the run is handed to you by the skill that invoked you inside it.

## Which shape

| You need | Shape | Tool | What the person does |
| --- | --- | --- | --- |
| to tell them something that needs no answer | a message | `send_message` | reads it, presses Acknowledge |
| an answer before the run can go on | an escalation | `raise_escalation`, then the caller pauses the run | answers with a note, or acknowledges — which means "do what you proposed" |
| to report what the run did | nothing here | `end_run`'s summary becomes the run report on its own | reads it in the morning |
| approval of a job you proposed | nothing here | `create_work_item` raises the approval request itself | approves or rejects |
| a ruling on a learning you drafted with proposed text | nothing here | recording it raises the review row itself | rules on the learning where it lives |
| to say a job is stuck | nothing here | `transition_work_item` to `Blocked` raises the escalation itself | answers it |

If the shape is in the lower four rows, do not send a message or an escalation as well: two rows
about one thing is one row too many, and the person answers the first and wonders about the second.

## Sending a message

1. One message says one thing. "Skipped two rows of the plan — no purchase prices yet", not
   "Update".
2. Name the row it is about with `subjectKind` and `subjectId` when there is one — the person opens
   the thing instead of searching for it. Both or neither.
3. Name `recipientUserId` when one person owns the matter; leave it out to put it in front of
   everybody who may approve.
4. `body` carries what happened, what it means, what you did about it, what you would do next if
   asked. Leave it out only when the title says the whole thing.
5. Not for what the run report will say anyway. The report is the summary you write at `end_run`;
   a message is for the thing that would otherwise be buried in it, or that the person should know
   before the run ends.

Send: `send_message(portfolioId, projectId, recipientUserId?, title, body?, subjectKind?,
subjectId?, runId)`. The answer is the row as filed; report its id and go on. Nothing waits for it.

## Raising an escalation

An escalation is a question the run cannot answer and a person can. Write it so it can be answered
without anybody asking you back:

- what you were doing — the row of the plan, the procedure, the experiment, by code;
- what you tried, and what the tools answered;
- the options, each in one line;
- **your proposed answer, last, in one sentence** — the person acknowledging without a note means
  "do that", so an escalation with no proposal cannot be answered by a click. Do not raise one
  without it.

Name the row it is about (`subjectKind`, `subjectId`) when there is one; an escalation may stand
alone when the question is about the run as a whole. Name the person when one person owns the
answer. Send `raise_escalation(...)` with `runId`, record the `inboxItemId` it answers in the run
(`observe_run`), then **hand back to the skill that called you: it pauses the run**. Raising does
not stop the run by itself.

One escalation per question. A question you asked in a run that is now paused was asked once; the
next firing reads the answer (below) and, finding none, leaves the run paused and asks nothing
again.

[reference/asks.md](reference/asks.md) has the body of each escalation a method skill raises under
the unattended rule — a Checkpoint verdict, a start that spends, a hypothesis decision, a gate's
pivot proposal, a decision of the business, a missing key — so the same question is always asked
the same way.

## Reading the answer

The answers come back in the brief, never through a read of the inbox: `boot_agent` and
`get_project_state(projectId, runId)` carry `resolvedSinceLastRun`, one line per approval request
or escalation a person answered since the asking party's last run — `inboxItemId`, `kind`, `title`,
`resolution`, `resolutionNote`, `resolvedAtUtc`.

- Find the line whose `inboxItemId` the paused run recorded.
- `resolution` `Acknowledged` with a `resolutionNote` → the note is the answer; do what it says,
  even when it is not what you proposed.
- `Acknowledged` with no note → do what you proposed. **One exception:** a pivot proposal. A pivot
  regresses the tier and rewrites the plan, so a bare acknowledgement never applies one;
  `apply_pivot` waits for a note saying "apply <pivot> to <tier>", or for the person applying it on
  the roadmap screen (`reference/asks.md`).
- No line → not answered yet. Leave the run paused; ask nothing again.

A message never appears here: it is read, not answered, and acknowledging one tells you nothing.

## Reading a refusal

- `INVALID_ARGUMENT` naming `subjectKind` and `subjectId` — one half of the subject was sent; send
  both or neither.
- `INVALID_ARGUMENT` naming `title` — blank or over 200 characters.
- `NOT_FOUND` — the portfolio or the project named nothing; read the ids again.
- `CONFLICT` saying the project is another group's — the project and the portfolio disagree; the
  portfolio is the project's own, as `get_project` answers it.
- `CONFLICT` naming `runId` — the run was ended or paused by a person. Nothing was filed. Stop; the
  skill that called you opens a new run before anything else is written.

## Rules that are easy to get wrong

- **The inbox is a person's.** You never list it, never mark a row read, never resolve one. The
  tools do not exist, on purpose.
- **Attended means no routine fired this run.** When the person is in the window, an inbox row is
  a strange way to talk to them. Ask the user directly to clarify what you cannot infer. It is the routine that decides which of the two
  this is, never the wording of the prompt.
- **No proposal, no escalation.** A question with no proposed answer cannot be answered by a
  click, and the morning's inbox is answered by clicks.
- **Never re-ask.** One question, one row, one answer. The run waits.
- **Nothing but the inbox.** No skill writes to Slack, email or a phone. When the inbox is
  forwarded somewhere later, that is the server's doing, and it changes nothing here.
