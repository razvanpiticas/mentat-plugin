# The digest

What `mentat-advance` hands back, and what `mentat-agent` writes as the firing's summary at `end_run`. It becomes the run report the person reads in the morning, so it is written for somebody who was not here.

Four short blocks, in this order:

1. **Where the plan stood** — tier, status, the row last finished, the next row as the rule found it (position, slug or idea code), the gate line. One or two sentences.
2. **What this firing did** — the rule that fired (by number and in words); the row worked and its run id; what was written, by code, grouped by block; hypotheses raised (codes); the evaluation recorded, if any; the job written, if any. "The plan is finished" when rule 10 applied.
3. **Where it stopped and why** — the ask raised, by title, and the row it is about; or "ran to its end".
4. **Waiting:** — one line per open ask, machine-readable, the last lines of the summary:

```
Waiting: <ask title> · inbox <inboxItemId> · since <date> · <what unblocks it>
```

Repeat every `Waiting:` line of the previous digest whose ask is still unanswered, marked "still waiting". A firing with nothing waiting writes `Waiting: nothing`.

A line that names a `message` rather than an `inbox` ask — `Waiting: … · message <id> · …` — is `mentat-agent`'s goal loop's: a measurement or a mission it asked for. Leave it out of what you hand back; `mentat-agent` repeats it or drops it in the summary itself, because only it reads whether the figure has arrived.

An answer the next firing still has to honour after the brief has stopped carrying it — "wait" on a gate that would pass early — goes on a line of its own beside them, and is repeated while it holds:

```
Decided: <what was decided> · inbox <inboxItemId> · answered <date>
```

Two run reports land when a row was run to its end: the row run's own summary (what it wrote) and this digest. The digest names the row run by id, and the row run's summary names the firing's run by id — you hand the owner the id of the half you paused — so either report leads to the other. Say that paused half's id in the digest too ("resumed from <id>"): it is the id the row's report carries, so the two reports share it. When the firing's run was paused around the row and resumed, hand `mentat-agent` the resumed run's id with the digest: that is the run it ends.

**`mentat-linker` runs on that run before `end_run`, on every firing and every path** — a row worked, a stop at rule 1 or 2a, a planner escalation, a row run left paused, nothing done at all. A run that wrote nothing is typed in one call that answers nothing; that answer is the record that the typing was not forgotten. The row run is not the one typed here: a row run the owner left `Paused` refuses every call that names it, the linker's included, and its rows are typed when a later firing resumes it. That is a reason to type the run you hand back, never a reason to skip it.
