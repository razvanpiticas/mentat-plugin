# The shapes a walk may take

Load this before `run_analytical_query`. Every shape is one `SELECT` over the property graph
`canvas`; fill the placeholders and keep the project clause on the vertex you start from — the
server refuses a query that does not name the project there. The graph's words: vertices `block`,
`entry`, `hypothesis`, `experiment`, `evidence`, `question`, `risk`, `idea`, `run`, every one also
labelled `node` with `id`, `project_id`, `reference_code`; structural edges `on_block` (a row to
its block), `about` (a hypothesis to its entry), `tests` (an experiment to its hypothesis),
`from_experiment` (evidence to its experiment), `produced` (a run to what it wrote), `contradicts`,
`for_segment`, `part_of_value_proposition`, `relieves`, `creates`, `supplies`, `performs`,
`pays_for`; every accepted link is an edge `linked` with `link_type_id`, `origin`, `confidence`,
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

## S7 — the pains a value proposition relieves, and the segment they belong to

```sql
SELECT * FROM GRAPH_TABLE (canvas
  MATCH (vp IS entry WHERE vp.project_id = '<projectId>' AND vp.reference_code = '<code>')
        <-[IS part_of_value_proposition]- (pr IS entry)
        -[IS relieves]-> (pain IS entry)
        -[IS for_segment]-> (seg IS entry)
  COLUMNS (pr.reference_code AS pain_reliever, pain.reference_code AS pain, seg.reference_code AS segment))
```

## S8 — contradictions still open on one block

```sql
SELECT * FROM GRAPH_TABLE (canvas
  MATCH (a IS entry WHERE a.project_id = '<projectId>')
        -[c IS contradicts WHERE c.resolution_state = 'Open']-> (b IS entry)
  COLUMNS (a.reference_code AS entry_a, b.reference_code AS entry_b, c.contradiction_severity AS severity))
```

More hops than a shape has: run one shape, take the codes it answered, run the next from them.
Never a recursive query, never a table outside the graph, never a second statement.
