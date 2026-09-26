# Running unattended, and what a person does themselves

## The unattended rule, applied here

You know the run is a routine's because it was opened with a routine and `mentat-agent` said so.
Then every "ask" above is one of two things.

| Step | Attended | Unattended |
| --- | --- | --- |
| the fit and the cost | ask | write the card anyway; say in the run's notes what is missing |
| the test card | the person confirms | written as drafted |
| the start | the person starts | the three conditions in the skill's "Before the start", and the escalation "A start that spends, or needs the person's hands" when any of them fails — that rule is in the skill body, not here, because it is the one a run gets wrong |
| observations, judged criteria, evidence, spend | recorded as they come | recorded when the data is there — a `Digital` card's readings you can take yourself; a person's readings wait for the next session |
| the learning card and the verdict | the person confirms | written as drafted, the verdict the criteria support |
| the decision | the person decides | **the escalation "Hypothesis decision"** with the proposed call, then `pause_run` |
| an abort | the person decides | never; escalate as a decision of the business |

Raising an escalation: invoke the `mentat-inbox` skill with the ask named above; it files the row
with `runId` and hands you back the `inboxItemId`; record it with `observe_run` and then
`pause_run`. On the next firing, `mentat-advance` reads the answer and resumes the run; when you are
invoked again on the same experiment, the brief's `resolvedSinceLastRun` carries the line for that
`inboxItemId`, or `mentat-advance` hands you the line it read before pausing the heartbeat's run: a note is the answer, an acknowledgement with no note means "do what you proposed",
no line means not answered — leave the run paused and ask nothing again.

## What a person does themselves

Much of a run is hands: interviews, a landing page, a delivery, a call. For each part the card's
instructions give to a person, say three things, attended in the chat and unattended in the run's
notes and the escalation's body: what they do, what they bring back, and in which shape — "ten
interview notes, one file each, the quotes marked", "the ad account's click and cost export for the
week". When they bring it, record it: `record_observation` per metric, `record_evidence` per kind
of signal with a data point per interviewee, per conversion, per order, `record_spend` for the
money and the hours. The run stays Running in between; a run nobody has touched for longer than its
planned duration is worth a message through `mentat-inbox` — once.
