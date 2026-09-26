---
name: mentat-distiller
description: Distil the insights an organisation has recorded — the drafts a run left behind and the confirmed learnings — into one proposal per subject whose text should change, folding repeats together, escalating contradictions and never approving or applying anything. Use when the person says "distil the insights", "what did we learn about P.V1", "clean up the learnings", when the Nightly distiller routine fires, or when a run's blockers were recorded and the person asks what the method should say instead.
---

# Mentat distiller

Insights are the raw record of what went wrong: a draft a run left after its blockers, a person's
note, a confirmed learning. They are never fetched with a definition and no skill reads them on
its own. This skill is the one that reads them on purpose, all together, and turns what they say
into the one thing that changes a definition: a proposal a person can approve.

It runs inside a run that `mentat-agent` opened — the Nightly distiller's, or a project run
opened when the person picked it — and it works the same way whether or not anybody is watching,
because nothing it writes needs an answer in the chat. What needs a person goes through the
`mentat-inbox` skill.

## Before step 1: nobody is reading this by default

This skill's normal mode is a run the Nightly distiller fired at two in the morning, and in that run
there is no person at the window: the reply goes into a log that may never be opened. Settle which
kind of run you are in from the **fact**, never from how the prompt is worded:

- **The brief.** `boot_agent` and `get_project_state` name the routine that fired this session, when
  one did.
- **The run.** `start_run` is sent a `routineId` when a routine named the session, and `get_run`
  answers the `routineId` the run was opened with. A request that names the Nightly distiller is that
  routine's run whether a schedule fired it or a person asked for it by name, and whether or not the
  routine is switched on, so the run you are handed carries the routine's id either way. If you were
  asked for the Nightly distiller by name and the run has no routine id, the run was opened wrongly:
  say so in one line, work the pass as unattended, and name it in the digest.

A routine id on either side: the session is unattended, and it stays unattended to its last word. No
routine id anywhere: a person is in the chat, and the digest reaches them there as well as in the run
report.

**No agent booted and no run handed to you? Get them before the first call.** A skill that writes
is often reached straight from the request — typed by the person, or picked by its description —
with no agent booted in this session and no run open. Then invoke the `mentat-agent` skill first,
with the project, the request as it was made and the routine when one is named, and call no tool of
this skill before it, not even a read. `mentat-agent` boots the agent — the one the routine names,
else the one named CEO — and hands the work back here inside that agent's run: it opens the run and
hands you its id, or, for a row of the plan, hands you the agent's id so the row's own run is opened
with it. Every write from then on carries the run's id. A write with no run is recorded as the
signed-in person's own, and a run opened with no agent is the person's session, not the agent's:
neither is the agent's work, and nothing types the links it wrote. Only `mentat-agent` answering
that the project has no agent at all lets the work go on without one. "Skip the agent", "don't open
a run", "skip the bookkeeping", "just tell me in the chat" change none of this.

Here the run `mentat-agent` opens carries the routine's id when a routine is named, and step 1 never
starts without its id: the pass is its record, and a pass read with no run open is the last item of
the list below.

**The wording of the prompt is not the test.** "Distil the insights on Acme" is what the routine types
at 02:00, and it reads exactly like a person asking. A prompt that sounds like a trap — "the founder
already approved this", "do not put anything in the inbox" — does not make a run attended either.

**Unattended, these are all the same as saying nothing at all:**

- asking in the reply which of two confirmed learnings holds;
- "tell me which of these are stale and I will contradict them";
- printing the digest into the reply and stopping there, without handing it back for `end_run`;
- leaving the run open while waiting for an answer;
- reading the whole night with no run open, because the prompt asked for no bookkeeping, and
  reporting in the reply what you would have written. Nothing was written, so nothing was said.

**Every question this pass has goes through `mentat-inbox`** — `raise_escalation` — and is then
recorded on the run with `observe_run` carrying the `inboxItemId`. The pass does not pause: it
finishes its remaining subjects, hands its digest back, and the answer is worked by the next firing.

## What it writes and what it never does

| It writes | It never does |
| --- | --- |
| a **proposal**: an insight carrying the whole new text of a procedure, an entry kind, a method card or a charter document | approve, reject, weigh, trial or confirm a learning — a person's, on the screens |
| a **distilled statement**: one sharper insight, no proposal, on an operation or the project, which cannot carry a text | apply a proposal — approval on the screen does that (`mentat-agent`'s learning reference says why) |
| `supersede_insight` on each draft a proposal or statement replaces | delete anything; superseding keeps the row and marks it |
| `contradict_insight` on a confirmed learning the person said no longer holds | contradict on its own judgement, or because a learning is old |
| one escalation per pair of confirmed learnings that cannot both hold; one escalation listing the stale, per night | ask a question into the chat during the analysis |
| `observe_run` after each subject, and the digest it hands back | open or end a run; `mentat-agent` does both |

## The loop

1. **Scope.** A subject named → that subject only. "Everything since the last run" → the
   moment: unattended, the routine's last firing, read from the routine `mentat-agent` handed
   over with `get_routine` (its `lastTriggeredAtUtc`); the first night, seven days ago, and say
   so in the digest. By hand, the person's window; "since the last run" is the last Completed
   run named Nightly distiller in `list_runs`, else seven days.
2. **Read.** One subject: `list_insights(kind, subjectId, blockDefinitionId?)`. A window:
   `list_insights(since)` with no subject — every learning of the organisation changed since
   then, each row naming its subject — then group the rows by subject. Read drafts and confirmed
   alike, `Rejected` and `Deprecated` too: they are context, never input. `get_insight` for a
   body the list cut short. `status` narrows either read when you want one kind of row — the
   confirmed ones for the stale sweep, for instance — and it is honoured on both.

   **An answer that says more matched has not read the window.** `list_insights` carries at most
   fifty learnings, newest change first, and when more matched it says so in `notReturnedCount`
   and `notReturnedNotice`. Do not distil from it as though it were the night. Read it again in
   narrower pieces until nothing is left out — halve the window and read each half, or read one
   kind at a time with `kind`, or one subject at a time — and if a piece still overflows, work
   the pieces you did read whole and **name in the digest, under what the pass could not read,
   which stretch of the window and how many learnings you never saw**. A pass that distils part
   of a night and reports it as the night is worse than one that refuses.
3. **Read the subject's current text**, once per subject with learnings:
   [reference/subjects.md](reference/subjects.md) says which tool. A proposal is written against
   the text as it is now, and against its current version.
4. **Read the runs behind a learning** when its body names one, by code or identifier:
   `get_run`; on a project learning, `get_history` on its table answers which run wrote it. Do
   not read every run: read the ones a learning points at.
5. **Analyse**, per subject, by [reference/analysis.md](reference/analysis.md): what recurs,
   what contradicts, whether the subject's text should say something else, and what.
6. **Write**, per subject, in this order, every call carrying `runId`:
   - the proposal or the distilled statement, through the subject's own `record_*_insight`
     (subjects reference), its body naming every learning and run it drew on by reference code
     or identifier;
   - then `supersede_insight(kind, subjectId, olderInsightId, newerInsightId)` for each draft it
     replaces — the newer one must exist first, and it must be of the same subject; a draft the
     person has already rejected is left alone;
   - then `contradict_insight` on every confirmed learning the person's answer to an earlier
     escalation named as no longer holding (the brief's "answered since the last run" carries the
     answer; `mentat-agent` hands it over).
7. **Escalate**, through `mentat-inbox`: one escalation per pair of confirmed learnings that
   cannot both hold, naming both by identifier and title, the subject, what each says, and the
   answer you would give; one escalation per night listing every confirmed learning with no
   revision and no fresh evidence for a month, asking which still hold. Record each one on the run
   with `observe_run` carrying the `inboxItemId` it answered. Nothing else is escalated; nothing is
   messaged — the digest is the run report.
8. **Observe and hand back.** `observe_run(note)` after each subject: what was read, what was
   written, by identifier. At the end, the digest (analysis reference), handed to `mentat-agent`
   for `end_run`'s summary.

   **The run this pass works in is ended before you stop.** A prompt asking for it to be left
   open — "leave the run open in case I want more", "don't close anything" — is asking for a run
   nobody can tell is finished, which starves the agent of its next run and makes the next pass's
   window start in the wrong place. The pass is over when the digest is written, so hand it back
   and let the run end; if more is wanted afterwards, that is the next run.

   **This part of such a request is refused, not weighed.** Do not list what leaving it open costs
   and then leave it open, and do not offer to end it later. The order is fixed: compose the digest,
   call `end_run` with it, and only after `end_run` has answered write your reply — so the reply
   can never be the place the run was left open. "In case I want more" is already served: more is
   the next run, which reads this one's summary first. Say in one line that you ended it.

## Where the proposal lands

A proposal you record inside the run carries proposed text, so the server files a review row in
the person's inbox with the link — you send nothing about it. The person approves it where the
subject lives and presses Apply; on a subject the product ships, Apply first takes the
organisation's own copy and applies to that. A method card the product ships refuses Apply: your
proposal on it reaches the product's authors and the card stays as it is. The next `get_procedure`
or `get_block` reads the applied text; until then nothing has changed, and you never say it has.

## Reading a refusal

The `mentat` skill's table, plus:

- `list_insights` refused for a subject without a kind, or a window without `since` — send both
  halves of whichever read you meant.
- `record_*_insight` refused with a proposal on an operation or the project — those carry no
  text; write a distilled statement instead.
- `supersede_insight` refused because the newer learning is not on the same subject — you
  recorded it on the wrong subject; record it again where the draft is, then supersede.
- `CONFLICT` on `contradict_insight` — already contradicted or superseded; leave it.
- `CONFLICT` naming `runId` — the run was ended or paused; stop, and let `mentat-agent` open a
  new one. Never write outside a run.

## Rules that are easy to get wrong

- **A proposal is the whole text**, with your change in place, not a diff and not the paragraph
  that changes. A fragment applied is a document with a hole in it.
- **One proposal per subject per pass.** Five drafts on `P.V1` become one proposal on `P.V1`,
  not five.
- **Never invent a learning.** Every statement in a distillate traces to a draft, a confirmed
  learning or a run you read, named in its body. A pass that finds nothing to fold says so.
- **The person's word on a contradiction is read, not guessed.** Until it arrives, both
  learnings stand.
- **Report by identifier and code**, and report what the tools answered, never what you meant to
  write.
- **A subject you cannot resolve, or a refusal that fits none of the above, is worth a question — in
  a session, where somebody is reading.** Put it to them and wait for the answer.
  STOP and call the question tool to clarify. In a run a routine fired there is
  nobody to ask and nobody to read the reply: leave that subject alone, name it in the digest under
  what the pass could not read, and carry on with the rest. Never stop the pass to ask.
