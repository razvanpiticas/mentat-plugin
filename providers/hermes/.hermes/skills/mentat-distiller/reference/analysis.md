# Analysing a subject's learnings

Load this before writing anything. The reading is done; this is what to make of it.

## Group

Within one subject, put learnings together when they are about the same thing: the same step of
a procedure, the same field of a kind, the same paragraph of a document. A learning may be about
two things; then it is in two groups, and the digest says so.

## Recurring

A group with two or more learnings from different runs or different people, saying the same thing
in different words, **recurs**. One learning, however strongly worded, does not: it is left where
it is, unless it already carries a proposal the person can rule on.

## Contradicting

Two learnings that cannot both hold contradict. Two **drafts** that contradict are both left
standing and named in the digest — a person has not looked at either. A draft that contradicts a
**confirmed** learning is left standing, and the digest says which confirmed learning it
challenges. Two **confirmed** learnings that contradict are the one case that is escalated:
neither can be folded away, and only a person may say which survives.

## What the text should say instead

A recurring group on a subject with a text yields a proposal when the learnings say the text
misled, omitted or contradicted what happened. The proposal is the whole current text with the
change in place — read it once more as the agent that will follow it next week. It keeps the
subject's voice (a procedure's numbered steps and its pitfalls line; a kind's description's
plain sentences), names no book, no tool and no column, and changes nothing the learnings do not
support.

A recurring group on an operation or the project yields a distilled statement: one insight whose
title is the finding and whose body says what recurred, in which runs, and what to watch for;
on an operation, `pitfall` and `appliesWhenNiche` when they fit.

Every distillate's body ends with a line "Distilled from: " followed by the identifiers and
titles of the learnings it replaces and the runs it read.

## Stale

A confirmed learning whose `updatedAtUtc` is older than a month, with no evidence recorded
against it since (`evidenceCount` unchanged, no run in the window naming it), is stale. Stale
is a question for the person, not a verdict: list every stale learning of the pass in one
escalation, by identifier, subject and title, asking which still hold.

## The digest

Handed to `mentat-agent` for the run's summary; the person reads it in the inbox as the run
report. In this order, plain lines, counts first:

- the window or the subject, and how many learnings were read, of which how many drafts;
- how many were folded together, and into what: each distillate by identifier, subject and title,
  and the identifiers it superseded;
- every proposal, by subject and identifier, with one sentence on what changes and why;
- every escalation raised: the contradictions, each pair by identifier; the stale list, its count;
- every confirmed learning contradicted on the person's word, by identifier;
- what was left alone and why: single learnings, drafts that contradict, subjects with nothing
  to fold;
- what the pass could not read: a refusal, a run it could not open, a subject it could not
  resolve.

No recommendation the person did not ask for, and no learning of the pass's own.
