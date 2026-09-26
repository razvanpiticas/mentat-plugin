# Entries: kinds, fields and statuses

`add_entry` needs the block's id, the kind's `kindDefinitionId`, a title, a status, and — for a typed
kind — `fields`. All of it comes from `get_block`: its `kinds` list carries each kind's
`definitionId`, `code`, `storage`, its `schema` — the columns and the words — and, for a custom kind,
the `attributes` it declares.

## Typed kinds: the `fields` object

`fields` is a JSON object. Its `kind` property is the storage name, exactly as `get_block` spells it;
the other properties are the columns of the kind's `schema` whose `writtenThrough` is `Fields`, under
the names the schema gives. A pointer column takes an id; a word column takes one of the schema's
words.

Example — a pain on the Customer Segments block, for a segment whose id you read, after reading the
`CS.PAINS` kind's schema (two columns: a pointer to a segment on the same block, and a rating sent as
a word):

```json
{
  "projectId": "…", "canvasVersion": 14,
  "blockId": "<CS block id>", "kindDefinitionId": "<id of the CS.PAINS kind>",
  "title": "Loses track of macros across meals",
  "body": "Said unprompted by 7 of 10 interviewees; they keep a spreadsheet nobody updates.",
  "status": "Reported",
  "fields": { "kind": "CustomerPain", "segmentId": "<id of CS-01>", "severity": "High" }
}
```

`update_entry` on a typed kind takes the whole new `fields` object; leave `fields` out to keep the
current columns. A blank body is refused; leave `body` out rather than sending an empty string, and
use `retire_entry` when the entry no longer holds.

## Free-form and custom kinds

A free-form kind takes `title` and `body` only; sending `fields` to it is refused. A custom kind
(`storage: Custom`) takes `attributes`, a JSON object keyed by the `key` of each declared attribute
`get_block` lists, with required ones present and typed ones matching. Keys the kind does not declare
pass untouched. The schema of a custom kind lists the same columns as its declared attributes; a
`Rating` column of a custom kind takes a whole number 1 to 5, not a word.

## Statuses

| Status | Use it when |
| --- | --- |
| `Confirmed` | Evidence is in hand: an experiment, a document, a number from a source you can cite. |
| `Reported` | A person said so: an interviewee, the founder, a review. |
| `Inferred` | You reasoned it from other entries. Most of what a step writes before validation is this. |
| `Hypothesis` | A guess written to be tested; usually paired with `create_hypothesis`. |
| `Unknown` | The slot is acknowledged and empty; the body says what is missing. |

`update_entry` with `status` moves an entry between them; the move happens after any text change in
the same call, against the version the text change produced.

## Retire versus delete

`retire_entry` keeps the entry with a reason and, when another entry took its place,
`supersededByEntryId`; the canvas shows it struck through and gates stop counting it. It is refused
while a hypothesis about the entry is under test. `delete_entry` removes the entry, every hypothesis
about it and every contradiction it is a side of, and is refused once one of those hypotheses has a
started experiment. Anything a person once believed is retired, not deleted.
