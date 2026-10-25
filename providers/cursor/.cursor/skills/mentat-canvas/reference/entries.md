# Entries: kinds, fields, owners and parts

`add_entry` needs the block's id, the kind's `kindDefinitionId`, a title, a status, and — for a typed
kind — `fields`; for a kind written inside another entry, the owner's id as `ownerId`; for a
confirmed or reported customer or value entry, its `source`. All of it comes from `get_block`: its
`kinds` list carries each kind's `definitionId`, `code`, `storage`, its `schema` — the columns and the
words — and, for a custom kind, the `attributes` it declares. The owner's id comes from the read that
answered the owner: `get_block` for a segment, `get_customer_segment` for a profile,
`get_value_proposition` for a value proposition.

## Typed kinds: the `fields` object

`fields` is a JSON object. Its `kind` property is the storage name, exactly as `get_block` spells it;
the other properties are the columns of the kind's `schema` whose `writtenThrough` is `Fields`, under
the names the schema gives. A pointer column takes an id; a word column takes one of the schema's
words. A kind's own kind is never called `kind`, because `kind` is the storage: a job's is `jobKind`,
a pain's `painKind`, a gain's `gainKind`.

Example — a pain inside the end user profile of beachhead `CS-03`, after reading the `CS.PAINS`
kind's schema and the profile's id from `get_customer_segment`:

```json
{
  "projectId": "…", "canvasVersion": 14,
  "blockId": "<CS block id>", "kindDefinitionId": "<id of the CS.PAINS kind>",
  "ownerId": "<id of CS-03's end user profile>",
  "title": "Waiting more than 10 minutes for a client reply",
  "body": "Said unprompted by 7 of 10 interviewees; they refresh the inbox between sessions.",
  "source": "Interviews with ten gym owners, 12–19 October, notes in the shared folder.",
  "status": "Reported",
  "fields": { "kind": "CustomerPain", "painKind": "Obstacle", "severity": "High", "measure": "more than 10 minutes" }
}
```

A new item lands last among its kind inside its owner; `rerank_entries` with every one of them, in
the new order, moves it. `update_entry` on a typed kind takes the whole new `fields` object; leave
`fields` out to keep the current columns. A blank body is refused; leave `body` out rather than
sending an empty string, and use `retire_entry` when the entry no longer holds.

## Inside an owner

| Kind | Written inside | `ownerId` |
| --- | --- | --- |
| `CS` | its parent segment, or nothing for the root of a tree — the market | the parent segment's id, or left out |
| `CS.PROFILE` | a segment: a shortlisted one or a beachhead | the segment's id |
| `CS.CONTEXT` | an end user profile | the profile's id |
| `CS.JOBS`, `CS.PAINS`, `CS.GAINS` | a profile, end user or context | the profile's id |
| `VP.PS`, `VP.PR`, `VP.GC` | a value proposition | the value proposition's id |

`CS.NOTES`, `VP`, `VP.NOTES` and the kinds of every other block are written on the block itself and
refuse an `ownerId`. On CS, `get_block` lists every segment of every tree in short (id, code, state,
parent, order, flags, `isCard`) and the notes. On VP, it lists our value propositions and the notes.
Profiles, their items and value map entries are read through their owner with
`get_customer_segment` / `get_value_proposition`. A first child
of a segment is refused until the segment says why it is split: `split_customer_segment` writes both.
An owner that still holds live entries is not deleted; retire it, or delete what it holds first.

## A segment's parts

Each part is written whole by its own tool, with the segment's id and `canvasVersion`; nothing of a
part is sent in `fields`.

| Part | Tool | Rule the server holds |
| --- | --- | --- |
| state: candidate, shortlisted or dropped | `set_customer_segment_state` | dropping says why; shortlisting needs the seven ratings; a dropped segment takes its branch out of the running |
| children | `split_customer_segment` | why it is split, then the children in order |
| the seven ratings, each with its reason | `rate_customer_segment` | the score is worked out from them |
| the three conditions of a market | `test_market_conditions` | each holds or not, with its evidence |
| the comparison | `write_comparison` | on a shortlisted segment only; a row of the founder's own is named once |
| customers today | `write_customers_today` | marks the segment served |
| the TAM | `write_total_addressable_market` | on a beachhead only |
| the beachhead part | `choose_beachhead` | on a leaf; why not on every other shortlisted segment; offers to try first is at least 1 |
| the segments it turns away | `write_turned_away_segments` | on a beachhead, the whole list |

**The beachhead is proposed.** `choose_beachhead` takes the leaf segment, the part — why this one,
what it gives up, when to switch, the offers to try first, whether it is the hardest side of a
platform — the adjacent markets in order, the beachhead it replaces or none, and why not on every
other shortlisted segment, with `canvasVersion` and `runId`. It writes the part and a decision in the
ledger: an agent's lands proposed, and the segment reads "Proposed beachhead" until a person accepts
it on the decisions tab; a person's is accepted as written. Accepting one that replaces a beachhead
removes the replaced part; rejecting removes the proposed one. A beachhead is never cleared: another
is chosen to replace it.

## An end user profile's parts and the persona

| Part | Tool |
| --- | --- |
| who the end user is: the fifteen characteristics and any added | `describe_end_user` |
| the team member who fits it, who else decides, other patterns and outliers, the gaps left by what they use today | `update_entry` on the profile, in its `fields` |
| the purchasing criteria, in order, each pointing at one of the profile's own jobs, pains or gains | `write_purchasing_criteria` |
| the persona: how it was made, its fact sheet in the profile's characteristic names, its words for the top pain, its picture prompt | `write_persona` |
| the persona's picture: a PNG, JPEG, WebP or SVG of at most 25 MB, its bytes in base64 | `upload_persona_picture`, `remove_persona_picture` |
| a real end user's persona, erased on their request | `erase_persona` |
| the profile moved onto a leaf when the beachhead moves | `move_end_user_profile` |

A real end user's persona says what the person agreed to be kept. A move is refused while one of our
value propositions aimed at the profile has reached a fit: a new offer starts from the moved profile.

## Value propositions

| What | Tool |
| --- | --- |
| our value proposition, aimed at a profile or a draft | `add_entry` on `VP`, then `aim_value_proposition` |
| its products, relievers and creators | `add_entry` with `ownerId` the value proposition |
| a connection to the job, pain or gain addressed | `connect_value_map_entry`, `disconnect_value_map_entry` |
| what they use today, on a Competitive Position entry that exists or a new one, once ours is aimed at the profile (`add_entry` on `VP`, then `aim_value_proposition`) | `add_what_they_use_today` with our value proposition's id (`valuePropositionId`) |
| what ours forgoes and why | `update_entry` on our value proposition |
| the kind of fit reached | `reach_fit`: problem–solution fit with the reason to switch from what they use today, then product–market, then business model, one at a time |

A draft connects nothing until it is aimed; a value proposition with connections or a fit is not
retargeted. A Competitive Position entry a value proposition maps as what they use today is retired,
never deleted.

## The idea

| What | Tool, each with `ideationVersion` |
| --- | --- |
| the founder's starting point, and the route the first time | `write_starting_point` |
| an idea weighed, its statement and ratings, why it was not chosen | `add_ideation_candidate`, `update_ideation_candidate`, `drop_ideation_candidate` |
| a fact read for an idea, with its link unless it is a hypothesis | `add_ideation_signal`, `update_ideation_signal` |
| a statement of the idea, with its riskiest hypothesis | `propose_ideation_revision` |

The route is set once: "The route is set once, and this project's is Idea." A dropped idea says why
it was not chosen.

## Free-form and custom kinds

A free-form kind takes `title` and `body` only; sending `fields` to it is refused. A custom kind
(`storage: Custom`) takes `attributes`, a JSON object keyed by the `key` of each declared attribute
`get_block` lists, with required ones present and typed ones matching. Keys the kind does not declare
pass untouched. The schema of a custom kind lists the same columns as its declared attributes; a
`Rating` column of a custom kind takes a whole number 1 to 5, not a word.

## Statuses

| Status | Use it when |
| --- | --- |
| `Confirmed` | Evidence is in hand: an experiment, a document, a number from a source you can cite. Send the `source`. |
| `Reported` | A person said so: an interviewee, the founder, a review. Send the `source`. |
| `Inferred` | You reasoned it from other entries. Most of what a step writes before validation is this. |
| `Hypothesis` | A guess written to be tested; usually paired with `create_hypothesis`. |
| `Unknown` | The slot is acknowledged and empty; the body says what is missing. |

`update_entry` with `status` moves an entry between them; the move happens after any text change in
the same call, against the version the text change produced.

## Retire versus delete

`retire_entry` keeps the entry with a reason and, when another entry took its place,
`supersededByEntryId`; the canvas shows it struck through and gates stop counting it. It is refused
while a hypothesis about the entry is under test. `delete_entry` removes the entry, every hypothesis
about it, every contradiction it is a side of and every connection it is an end of, and is refused
once one of those hypotheses has a started experiment, while live entries sit inside it, or while
other entries point at it. Anything a person once believed is retired, not deleted.
