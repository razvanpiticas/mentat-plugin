# The shapes a walk may take

Load this before `run_analytical_query`. Every shape is one `SELECT` over the property graph
`canvas`; fill the placeholders and keep the project clause on the vertex you start from — the
server refuses a query that does not name the project there. The graph's words: vertices `block`,
`entry`, `hypothesis`, `experiment`, `evidence`, `question`, `risk`, `idea`, `run`, every one also
labelled `node` with `id`, `project_id`, `reference_code`; structural edges `on_block` (a row to
its block), `about` (a hypothesis to its entry), `tests` (an experiment to its hypothesis),
`from_experiment` (evidence to its experiment), `produced` (a run to what it wrote), `contradicts`,
`for_segment` (a channel, relationship or revenue stream to its segment), `supplies`, `performs`,
`pays_for`; the customer and value edges, each from the entry written inside
or about another to that other: `child_of` (a segment to its parent), `profile_of` (an end user
profile to its segment), `context_of` (a context profile to its end user profile), `in_profile` (a
job, pain or gain to its profile), `copied_from` (an item to the one it was copied from),
`in_value_map` (a product, reliever or creator to its value proposition), `aims_at` (a value
proposition to its profile), `addresses` (a product, reliever or creator to the job, pain or gain it
is connected to), `value_proposition_of_what_they_use_today` (a value proposition to its competitor,
a Competitive Position or an indirect competitor), `coverage_of` (a value proposition of what they
use today to each top job, pain or gain it has a coverage row for), `resells_to` (a segment to the
one it sells on to), `adjacent_market_of` (a segment to the beachhead it is an adjacent market of),
`turned_away_by` (a segment to the beachhead that turns it away); the competitor edges: `targets` (a
competitor of any of the three models to a segment it targets), `for_profile` (a Strategy Canvas to
the profile it is for), `scores` (a Strategy Canvas to each competitor it scores),
`observation_of` (a `competitor_observation` vertex, one fact of a record, to its competitor, also
held as its `competitor_id`); every accepted link is an edge `linked` with `link_type_id`, `origin`, `confidence`,
`magnitude`, `unit`. Content vertices expose every column of their row: `status`, `severity`,
`reference_code` among them; the status words are the ones the glossary and the tool answers use.

Placeholders: `<projectId>` the project's id; `<code>` a reference code; `<status>` a status word
as a read answered it; `<n>` a number.

## S1 — open questions that block a hypothesis a high risk depends on (two hops)

The two `linked` edges carry no arrow: which way a link points is decided by the word it was typed
with, so a risk may be tied to its hypothesis either way round, and an arrow would miss half of them.

```sql
SELECT * FROM GRAPH_TABLE (canvas
  MATCH (r IS risk WHERE r.project_id = '<projectId>' AND r.severity >= <n>)
        -[IS linked]- (h IS hypothesis)
        -[IS linked]- (q IS question WHERE q.status = 'Open')
  COLUMNS (r.reference_code AS risk, h.reference_code AS hypothesis, q.reference_code AS question))
```

## S2 — every hypothesis and the entry it is about

```sql
SELECT * FROM GRAPH_TABLE (canvas
  MATCH (h IS hypothesis WHERE h.project_id = '<projectId>') -[IS about]-> (e IS entry)
  COLUMNS (h.reference_code AS hypothesis, h.status AS status, e.reference_code AS entry))
```

## S3 — the hypotheses of one block, with their experiments

A block vertex carries no code (its row holds `block_definition_id`, not a code); start from the
block's `id`, which `get_block` answers.

```sql
SELECT * FROM GRAPH_TABLE (canvas
  MATCH (b IS block WHERE b.project_id = '<projectId>' AND b.id = '<blockId>')
        <-[IS on_block]- (h IS hypothesis)
        <-[IS tests]- (x IS experiment)
  COLUMNS (h.reference_code AS hypothesis, h.status AS hypothesis_status, x.reference_code AS experiment, x.status AS experiment_status))
```

## S4 — what one run wrote

```sql
SELECT * FROM GRAPH_TABLE (canvas
  MATCH (run IS run WHERE run.project_id = '<projectId>' AND run.id = '<runId>') -[IS produced]-> (row IS node)
  COLUMNS (row.reference_code AS written))
```

## S5 — everything typed with one word, from one row outwards (one hop, filtered)

```sql
SELECT * FROM GRAPH_TABLE (canvas
  MATCH (a IS node WHERE a.project_id = '<projectId>' AND a.reference_code = '<code>')
        -[l IS linked WHERE l.link_type_id = '<linkTypeId>']-> (b IS node)
  COLUMNS (b.reference_code AS neighbour, l.confidence AS confidence, l.magnitude AS magnitude, l.unit AS unit))
```

(`linkTypeId` is the `id` of that word on the words list `list_untyped_links` answers; `get_neighbours`
answers a link's word by its code, not by its id.)

## S6 — the risks nothing mitigates

```sql
SELECT * FROM GRAPH_TABLE (canvas
  MATCH (r IS risk WHERE r.project_id = '<projectId>' AND r.severity >= <n>)
  COLUMNS (r.reference_code AS risk, r.severity AS severity))
```

then S5 from each risk with the `mitigates` word's id, `In` direction: `<-[l IS linked WHERE …]-`;
a risk with no row is unmitigated. Two queries, because a "not exists" is not a shape the walk
offers.

## S7 — what a value proposition's map addresses, and the profile it belongs to

```sql
SELECT * FROM GRAPH_TABLE (canvas
  MATCH (vp IS entry WHERE vp.project_id = '<projectId>' AND vp.reference_code = '<code>')
        <-[IS in_value_map]- (m IS entry)
        -[IS addresses]-> (item IS entry)
        -[IS in_profile]-> (profile IS entry)
  COLUMNS (m.reference_code AS value_map_entry, item.reference_code AS job_pain_or_gain, profile.reference_code AS profile))
```

The segment is one hop further: `-[IS profile_of]->` from an end user profile, or `-[IS context_of]->`
then `-[IS profile_of]->` from a context profile — a second shape run from the profile codes this
one answered.

## S8 — contradictions still open on one block

```sql
SELECT * FROM GRAPH_TABLE (canvas
  MATCH (a IS entry WHERE a.project_id = '<projectId>')
        -[c IS contradicts WHERE c.resolution_state = 'Open']-> (b IS entry)
  COLUMNS (a.reference_code AS entry_a, b.reference_code AS entry_b, c.contradiction_severity AS severity))
```

More hops than a shape has: run one shape, take the codes it answered, run the next from them.
Never a recursive query, never a table outside the graph, never a second statement.
