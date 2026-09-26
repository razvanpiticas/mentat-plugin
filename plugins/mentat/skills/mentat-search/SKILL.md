---
name: mentat-search
description: Finds what nobody knows the code of on a project's canvas, and says how it found it. Three instruments and one rule — exact reads when the code or the block is known, search by meaning through the embeddings when only the subject is known or a duplicate is being checked, the graph when a row is known and what hangs on it is wanted, a shaped walk for a question with more than one hop. Use when a skill or a person asks what the canvas knows about something, whether a hypothesis like this already exists, what a row is tied to, or a question with a shape such as which open questions block a hypothesis a high risk depends on. Reads only; every answer names its instrument and quotes the query it ran.
argument-hint: <project> <the question>
allowed-tools: ReadMcpResourceTool, mcp__mentat__search, mcp__mentat__get_neighbours, mcp__mentat__run_analytical_query, mcp__mentat__get_block, mcp__mentat__get_canvas, mcp__mentat__get_hypothesis, mcp__mentat__get_experiment, mcp__mentat__get_run, mcp__mentat__get_project, mcp__mentat__list_projects, mcp__mentat__list_portfolios
---

# Mentat search

Reads by code are `mentat-canvas`'s: a block by its code, a hypothesis by its id. This skill is
for every other read — the ones where nobody holds the code, or holds one and wants what is tied
to it. It chooses the instrument, runs it once, and answers with the rows found, by code, and the
query it ran, so the person and the calling skill can see what was asked. It writes nothing.

Two of the three instruments cost something: a search by meaning spends one embedding against the
organisation's allowance, and a walk is a query with a timeout. The exact read is free. That is why
the rule below is applied in this order and nowhere else in the plugin.

## The steps

Every question, whoever asks it, goes through these four steps in this order.

1. **Read the glossary — first, before any instrument is chosen.** The product's words — gate,
   confidence, validated, tier, retired, related, operation, run, checkpoint, verdict, pivot and the
   rest — are one short file on the server, the resource `mentat://glossary`. Read it once per
   session, before the first call of any instrument, so a question is asked in the product's words
   and an answer is read in them. Already read in this session: go to step 2. If this harness lists
   the `mentat` server's resources, read `mentat://glossary`. If it does not, skip it: the tool
   descriptions carry the words the three instruments need.
   Here, reference it as `@mentat:mentat://glossary`, or read it with the resource tool the harness
   provides for the `mentat` server: `ReadMcpResourceTool` with server `mentat` and uri
   `mentat://glossary`. That call is this step. It comes before `search`, `get_neighbours`,
   `run_analytical_query` and every exact read, and it is not made again in the session.
2. **Choose the instrument** by [the rule](#the-rule): the first row that fits.
3. **Run it once**, as its own section below says.
4. **Answer** in [the answer's shape](#the-answer): the instrument, the query, the rows by code.

## The rule

Apply in this order; the first row that fits is the instrument.

| The question | Instrument | Call | Cost |
| --- | --- | --- | --- |
| names a block, or a code you hold | **exact** | `get_block(code)`; a code that is not a block's → `get_neighbours(code)` for the row's id and kind, then `get_hypothesis`, `get_experiment` or `get_run` by id; the whole model only when the question spans blocks and nothing narrower answers: `get_canvas` | free |
| names a subject and no code — "what do we know about pricing"; or is a duplicate check before a write — "is there already a hypothesis like this" | **by meaning** | `search(projectId, question, limit)` | one embedding per call |
| holds a code and asks what is tied to it — "what hangs on KR-45" | **by connection** | `get_neighbours(projectId, code)` | a query |
| has a shape with more than one hop — "open questions that block a hypothesis a high risk depends on" | **by connection, shaped** | `run_analytical_query` with a shape from [reference/shapes.md](reference/shapes.md) | a query, five seconds, 200 rows |
| has two halves — "what is tied to whatever we know about pricing" | **both** | by meaning to find the rows, then by connection from each | one embedding, then queries |

Never by meaning when a code is in hand. Never more than one `search` per question: one call, the
question written once as a sentence, not one call per phrasing. [reference/instruments.md](reference/instruments.md)
has the rule with examples and the answer's shape.

## Exact

`get_block` answers the block whole: its kinds with their shape, every entry, the hypotheses,
questions, risks and ideas on it. An entry's code names its block before the first `.` or `-`
(`KR-01` → `KR`, `CS.JOBS-03` → `CS`). For any other code, `get_neighbours(code)` answers first
the row itself — its kind, its id, its title, its status — which is the id `get_hypothesis`,
`get_experiment` or `get_run` takes; one free read, and it brings the row's surroundings with it.

## By meaning

The glossary is read before this call — step 1. A session that has not read it yet reads it now,
as step 1 says, and only then sends `search`: the question is written in its words.

`search(projectId, question, limit)`: the server embeds the question once, finds the nearest
passages of the project, and answers their rows — each by kind, code and title, with the passage
that matched quoted and a similarity from 0 to 1. Write the question as one sentence about the
subject: "pricing pressure from the parallel importers", "a hypothesis that gift buyers reorder in
December". Five rows by default, up to twenty. A hit below 0.5 is named as weak. For a duplicate
check, the answer says whether any hit says the same thing as what is about to be written, and
quotes it.

## By connection

`get_neighbours(projectId, code)`: everything one hop from the row — the structural edges (a
hypothesis's entry, a job's segment, an experiment's hypothesis, what a run wrote) and every
accepted link (`references`, `related`, and the typed words), each with the edge's name, its
direction from the row, and for a link its word, origin and confidence. Passages are never
neighbours; a rejected link is never one. Structural edges come first, then links by confidence.

A question with a shape — two hops, a filter on the far end — is a query you write against the
property graph, named `canvas`, run under a read-only role with a five-second limit and a cap of
200 rows, the query echoed with the answer. Take the shape from
[reference/shapes.md](reference/shapes.md), fill its placeholders, and name the project on the
vertex you start from. A question needing more hops than any shape has is answered by running a
shape twice, from the rows the first answered; never by a query the server would refuse.

**A walk of more than one hop is bounded.** `get_neighbours` on the row named, then at most two more
calls, a shape first, run from the rows that answer named. When the shaped walk is refused — the
analytical query is off, or it failed twice — the fallback is at most two more `get_neighbours`, each
on a row the first answer named by its code, never on a block's code (a block is not a row
`get_neighbours` finds). Then stop
and answer with what those calls found, and say the shaped walk was refused and what it said. Never
walk on row by row until the question feels covered: the answer names the hops it reached.

## Both halves

By meaning first, then `get_neighbours` on each row found, at most the `limit` rows. The answer is
grouped by the row found, each with what hangs on it.

## The answer

Always in this shape, whoever asked:

```
By meaning — search("pricing pressure from the parallel importers", 5):
  RS-02 Cascade pricing (entry, Inferred) 0.83 — "…undercut by parallel importers on the same SKUs…"
  H-09 We believe that matching Finestore's price on the top 50 SKUs… (hypothesis, Testing) 0.71 — "…"
  R-04 Price war on whisky (risk) 0.64 — "…"
```

The instrument, the query as sent, then the rows by code with kind and status, the number, and
the passage quoted. For a walk, the query echoed as the server returned it, then the rows. Say
"nothing found" when the answer is empty, and say which instrument said so.

## Reading a refusal

- `CONFLICT` whose text names `embedding-allowance-spent`, or says search by meaning is off — the
  organisation's allowance for this period is spent, or no allowance authority is configured.
  Answer the question by exact reads and by connection instead, and say "search by meaning is off
  for this organisation" in the answer's first line. Do not retry and do not rephrase.
- `NOT_FOUND` on `get_neighbours` — no row of this project carries that code; read the code again
  from a list rather than guessing another.
- `INVALID_ARGUMENT` on `run_analytical_query` — the guard: the query must begin with `SELECT`,
  read `GRAPH_TABLE (canvas …)`, name the project on the start vertex, hold one statement, and stay
  under 4000 characters. Fix what the message names and send once more.
- `CONFLICT` "The analytical query is off" — no shaped walk runs for this organisation. The bounded
  fallback of [By connection](#by-connection): at most two more `get_neighbours`, on rows the first
  answer named by code, never a block's code; then answer and say the shaped walk was refused.
- `CONFLICT` "ran longer than 5 seconds" — narrow the walk: start from one row by its code, fewer
  hops, the project clause on the start vertex.
- `CONFLICT` "PostgreSQL refused the query at position …" — a syntax error; fix it at the position
  named. Two failures on one question: answer by `get_neighbours` and say the shape did not run.
- `FORBIDDEN` — the person's role cannot read the canvas; say so and stop.

## Rules that are easy to get wrong

- The exact read is always first. A code in hand and a `search` sent is an embedding spent on
  something a free read answers.
- One search per question. Two phrasings are two embeddings for one answer.
- Name the instrument and quote the query in every answer, including the empty ones. An answer
  that hides how it was found cannot be checked.
- `get_canvas` is the whole model and large; reach for it last.
- In a run a routine fired, the fallback on a spent allowance goes into the run report, not into a
  message: "search by meaning was off; answered by code".
- This skill writes nothing. A duplicate it finds is reported to the calling skill, which decides
  what to write.
