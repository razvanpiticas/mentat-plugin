---
name: "mentat-canvas"
description: "Reads and writes a Mentat project's business model canvas through the Mentat MCP tools — entries on blocks and inside other entries, the customer segments' trees with their parts, end user and context profiles with their gaps and high-value jobs, personas, value propositions with their value maps, fit and design — concept, use case, features, drawing, quantified value, brochure and promise — and our business as it is today, the Competitive Landscape's competitor records, Strategy Canvases and Core, the project's idea, hypotheses, experiments, evidence, questions, risks, ideas, contradictions and project insights — and the mechanics of a project's roadmap and runs: the plan's rows, the run tools, the method gates and pivots. The low-level skill every other Mentat skill reads and writes the canvas through: it knows the write loop, the versions, the run id and the refusals, and nothing of the method. Use it for any read or write by code on a canvas or a plan, whoever asks for it."
argument-hint: "<project name or id> <what to read or write>"
allowed-tools: "mcp__mentat__server_info, mcp__mentat__list_portfolios, mcp__mentat__list_projects, mcp__mentat__get_project, mcp__mentat__get_canvas, mcp__mentat__get_block, mcp__mentat__get_neighbours, mcp__mentat__get_hypothesis, mcp__mentat__get_experiment, mcp__mentat__list_experiment_definitions, mcp__mentat__get_experiment_definition, mcp__mentat__create_project, mcp__mentat__record_project_insight, mcp__mentat__add_entry, mcp__mentat__update_entry, mcp__mentat__retire_entry, mcp__mentat__delete_entry, mcp__mentat__choose_beachhead, mcp__mentat__create_hypothesis, mcp__mentat__update_hypothesis, mcp__mentat__recommend_experiment_definition, mcp__mentat__withdraw_experiment_definition, mcp__mentat__park_hypothesis, mcp__mentat__unpark_hypothesis, mcp__mentat__retire_hypothesis, mcp__mentat__decide_hypothesis, mcp__mentat__delete_hypothesis, mcp__mentat__design_experiment, mcp__mentat__add_metric, mcp__mentat__add_criterion, mcp__mentat__start_experiment, mcp__mentat__record_observation, mcp__mentat__judge_criterion, mcp__mentat__complete_experiment, mcp__mentat__abort_experiment, mcp__mentat__record_spend, mcp__mentat__delete_experiment, mcp__mentat__record_evidence, mcp__mentat__update_evidence, mcp__mentat__add_data_point, mcp__mentat__delete_evidence, mcp__mentat__add_question, mcp__mentat__update_question, mcp__mentat__resolve_question, mcp__mentat__add_what_was_learned, mcp__mentat__add_risk, mcp__mentat__update_risk, mcp__mentat__resolve_risk, mcp__mentat__add_idea, mcp__mentat__update_idea, mcp__mentat__resolve_idea, mcp__mentat__record_contradiction, mcp__mentat__rule_contradiction, mcp__mentat__get_roadmap, mcp__mentat__get_operation_package, mcp__mentat__get_gate_status, mcp__mentat__list_gate_evaluations, mcp__mentat__get_run, mcp__mentat__list_runs, mcp__mentat__list_operation_definitions, mcp__mentat__get_operation_definition, mcp__mentat__list_pivot_definitions, mcp__mentat__set_roadmap_thesis, mcp__mentat__place_roadmap_item, mcp__mentat__skip_roadmap_item, mcp__mentat__include_roadmap_item, mcp__mentat__annotate_roadmap_item, mcp__mentat__append_roadmap_operation, mcp__mentat__add_roadmap_move, mcp__mentat__remove_roadmap_move, mcp__mentat__apply_pivot, mcp__mentat__start_run, mcp__mentat__checkpoint_run, mcp__mentat__observe_run, mcp__mentat__pause_run, mcp__mentat__end_run, mcp__mentat__cancel_run, mcp__mentat__evaluate_gate, mcp__mentat__get_customer_segment, mcp__mentat__get_value_proposition, mcp__mentat__get_ideation, mcp__mentat__set_customer_segment_state, mcp__mentat__split_customer_segment, mcp__mentat__rate_customer_segment, mcp__mentat__test_market_conditions, mcp__mentat__write_comparison, mcp__mentat__write_gathering_places, mcp__mentat__write_customers_today, mcp__mentat__write_total_addressable_market, mcp__mentat__write_turned_away_segments, mcp__mentat__describe_end_user, mcp__mentat__write_purchasing_criteria, mcp__mentat__write_gaps, mcp__mentat__score_high_value_job, mcp__mentat__write_persona, mcp__mentat__request_picture_upload, mcp__mentat__remove_picture, mcp__mentat__erase_persona, mcp__mentat__move_end_user_profile, mcp__mentat__rerank_entries, mcp__mentat__aim_value_proposition, mcp__mentat__connect_value_map_entry, mcp__mentat__disconnect_value_map_entry, mcp__mentat__add_what_they_use_today, mcp__mentat__reach_fit, mcp__mentat__add_our_business_as_it_is_today, mcp__mentat__write_concept, mcp__mentat__write_use_case, mcp__mentat__draw_product, mcp__mentat__quantify_value_proposition, mcp__mentat__write_promise, mcp__mentat__get_competitor_record, mcp__mentat__classify_competitive_position, mcp__mentat__record_competitor_observation, mcp__mentat__read_the_field, mcp__mentat__draw_strategy_canvas, mcp__mentat__correct_strategy_canvas_score, mcp__mentat__read_strategy_canvas_lead, mcp__mentat__make_core, mcp__mentat__lock_core, mcp__mentat__propose_ideation_revision, mcp__mentat__write_starting_point, mcp__mentat__add_ideation_candidate, mcp__mentat__update_ideation_candidate, mcp__mentat__drop_ideation_candidate, mcp__mentat__add_ideation_signal, mcp__mentat__update_ideation_signal, mcp__mentat__raise_escalation"
---

# Mentat canvas

The canvas is a project's business model: twelve blocks, each holding entries of the kinds it
accepts, plus the hypotheses made about it, the experiments that test those hypotheses, the evidence
they produce, and the questions, risks and ideas pinned alongside. Every tool call is made as the signed-in
person inside their own organisation — the connection and sign-in are the `mentat` skill's job; this
skill is about what to write and how.

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

Here the run is handed to you by `mentat-agent` or by the skill that invoked you inside it, and every
write carries its id.

## The loop every write follows

1. **Find the project.** `list_portfolios`, then `list_projects` with the portfolio's id, or
   `get_project` when you hold the project id. `get_project` also answers the current canvas version.
   A new one is `create_project`, and it needs `isExistingBusiness`: ask the person "New idea or
   existing business?" and send their answer, true for a business that sells today; never guess it
   from the name or the description. An existing business starts on the `ExistingBusiness` route,
   and every later session maps the business as it is first.
2. **Read before you write.** `get_block` with the block's code answers the block's id, the kinds of
   entry it accepts (each with the `kindDefinitionId` and `storage` a write needs), everything already
   written on it, and the canvas version. Reading first is how you avoid writing a duplicate of an
   entry that already exists and how you learn which ids to address. On Customer Segments it answers
   the trees under `customerSegments`, not in `block.entries`: an `outline` row per live segment, the
   `beachheads` in full with their adjacent markets in short, the `shortlisted` segments of a tree with
   no beachhead in short, and `retiredCount`. It leaves out the kinds' schemas unless you send
   `includeSchemas: true`, the retired segments unless `includeRetired: true`, and every segment as a
   whole entry unless `allInFull: true`. `get_customer_segment` reads one segment whole,
   `get_value_proposition` one value proposition, whole or only the `sections` named, `get_ideation`
   the idea.
3. **Write with the version you read.** Every write takes `canvasVersion` and answers with the new
   one. Carry the answered version into the next write. Two writes in a row against the same version
   means the second is refused as stale — that is the canvas protecting itself from a decision made on
   old state, not a fault. The idea keeps its own number: `get_ideation` answers it as the idea's
   `version`, its writes send it as `ideationVersion` and answer the next `version`; the plan's
   take `roadmapVersion`.
4. **Report what the tool answered**, by reference code (`CS-04`, `H-CS-002`, `T-3`, `E-7`), never
   what you intended. A refusal changed nothing unless its message says otherwise.

**`runId` on every write.** Every write tool on this server but `start_run` takes `runId`: the run
the call is being made from inside. When you are working inside a run, send the id `start_run` answered
on every write until that run ends, so the project's history records which session wrote each row. A
write without it is recorded as the person's own, not the agent's, so it is never left out. A write refused with a `CONFLICT` naming `runId` means the
run was ended, paused or opened by somebody else — nothing was written, every other call carrying it
will be refused the same way, and the fix is a new `start_run`, never a retry.

## Addressing

`get_block` and `get_neighbours` take **codes**. `get_customer_segment` takes `segmentId` and
`get_value_proposition` takes `valuePropositionId`: the ids `get_block` answered for that code or,
for a value proposition of what they use today, the id `get_value_proposition` answered under
`whatTheyUseToday`. `get_competitor_record` takes the Competitive Position's entry id from `get_block`
on `CL`. A
block's code comes from `get_canvas`, which lists every block with its code,
or from a refusal of `get_block`, which lists the codes that exist; never from memory, because a
project may hold custom blocks with codes of their own. Writes take **ids** a read answered:
`blockId`, `kindDefinitionId`, `entryId`, `hypothesisId`, `experimentId`, `evidenceId`, `metricId`,
`criterionId`. Never guess an id.

## Which kind, which shape

Every kind `get_block` lists carries its `schema`: the columns a write of that kind takes, each with
its type, whether it is required, what it points at and the words it accepts with what each word
means. On Customer Segments the schema comes only with `includeSchemas: true`; send it before writing
an entry there. Read the schema of the kind you are about to write; it is the only source of its
columns and words.

- A kind whose schema has no columns is free-form: `add_entry` takes `title` and `body` (markdown)
  only.
- A kind whose storage is `Custom` takes `attributes`: a JSON object keyed by the schema's column
  names.
- Every other kind takes `fields`: a JSON object whose `kind` property is the kind's `storage` name,
  exactly as `get_block` spells it, plus one property per schema column whose `writtenThrough` is
  `Fields`. A column whose `writtenThrough` is `OwnAction` or `ReadOnly` is never sent; its
  description says where it is written — a segment's ratings, its beachhead part, a persona are
  written by tools of their own, below.
- **A kind written inside another entry takes its owner's id as `ownerId`** on `add_entry`: a segment
  under its parent segment (none for the root of a tree, the market), an end user profile
  (`CS.PROFILE`) inside a segment, a context profile (`CS.CONTEXT`) inside an end user profile, a
  job, pain or gain inside a profile, end user or context, a product, reliever or creator inside a
  value proposition. Sent without one: "CS.PAINS is written inside a profile: send the profile's id
  as ownerId." A kind written on the block itself refuses an `ownerId`.
- **A customer or value entry that is `Confirmed` or `Reported` carries at least one citation** in
  `citations`: each with its `reference` (the link, the interview, the business's own data), and, for
  a quote, `who`, their `words`, `seenOn` and its counts. A fact inside a part carries its own the
  same way. Without one: "A Reported entry needs at least one citation, so anyone can check it."
- A `Pointer` column takes the id of an entry of the storage its `pointer.targetStorages` names, on
  the same block when `pointer.scope` is `SameBlock` and anywhere on the canvas when it is
  `SameCanvas`. Write the entry pointed at first; the document answers its reference code beside the
  id, under the member `pointer.referenceCodeMember` names.
- A `Choice` or `Rating` column takes one of its `words`, spelled exactly. A misspelt word is refused
  naming the column and the words that would have worked, so it costs a round trip, not a wrong row.

When the kind you want does not exist on the block, write to the block's free-form Notes kind and say
in the body what it is; do not force content into a kind it does not fit. A quote or a source is never
a note: it is a citation on the statement it backs. Worked examples, the
statuses and every tool of the segments, profiles, value maps and the idea are in
[reference/entries.md](reference/entries.md).

## Segments, profiles and the persona

The Customer Segments block holds trees: a market at the root, segments split under it, and on a
beachhead or a shortlisted segment its end user profile with its jobs, pains and gains, its purchasing
criteria, its persona and its context profiles. A segment's parts have tools of their own, each
taking the segment's or the profile's id and `canvasVersion`: `set_customer_segment_state`,
`split_customer_segment`, `rate_customer_segment`, `test_market_conditions`, `write_comparison`,
`write_gathering_places`, `write_customers_today`, `write_total_addressable_market`, `choose_beachhead`,
`write_turned_away_segments`; `describe_end_user`, `write_purchasing_criteria`, `write_persona`,
`erase_persona`, `move_end_user_profile`; on an end user or a context profile, `write_gaps` and
`score_high_value_job`. Each writes its part whole: send everything the part holds,
not the change. A picture is the exception, the persona's as every other: `request_picture_upload`
names the slot and its owner and answers a link, the file goes to it from disk with one `curl` PUT, and
`remove_picture` takes it off, keeping its prompt. `rerank_entries` puts every
entry of one kind inside one owner in a new order; the order is the rank.

Where a segment's end users gather is one list on the segment, its gathering places, and nowhere else:
not a row of the comparison, not a characteristic of the end user or of the persona. Each segment holds
its own; a split names the places each child keeps (`gatheringPlacesKept`), copied onto the child, and
the persona names the places the person uses (`gatheringPlacesTheyUse`). `write_gathering_places`
replaces the whole list: read it with `get_customer_segment`, send every place back, and add a new count
as a new measurement beside the earlier ones, which are never edited or dropped.

The gaps left by what they use today are one ranked list on each profile, end user or context, and
nowhere else: `write_gaps` replaces the whole list, each gap pointing at a live top-ranked job, pain or
gain of that profile — its top job, its first three pains or gains — saying whether nothing they use
today addresses it or all address it badly, in their users' words. A job is scored as a high-value job
with `score_high_value_job`: four marks of 1 to 4, of which Important is read from the job's
importance and Unsatisfied from the gap that points at it, so only Tangible and Lucrative are always
sent; the sum is worked out, and `get_customer_segment` lists the high-value jobs highest first beside
the gaps and the openings. A pain says when its measure is known and no source gives the number
(`isMeasureNumberMissing`), and an obstacle or a risk the use case step it stops them adopting at
(`barrierToAdoptionAtUseCaseFactor`); the persona carries the last purchase for the top job. The rules
and the refusals are in [reference/entries.md](reference/entries.md).

A beachhead is proposed, never marked: `choose_beachhead` writes its part and its decision, and an
agent's decision waits on the decisions tab for a person to accept it. A beachhead is replaced by
choosing the one that replaces it; it is never cleared.

## Value propositions and fit

Our value proposition is aimed at one profile (`aim_value_proposition`); its products, relievers and
creators are written inside it and connected to the jobs, pains and gains they address
(`connect_value_map_entry`, `disconnect_value_map_entry`). What they use today is a value proposition
of its own on a Competitive Position entry, mapped against a profile whether or not ours aims at it
yet: `add_what_they_use_today` takes the profile's id (`profileId`) and names its Competitive Position
exactly once — an existing one by its entry id (`competitivePositionEntryId`), or a new one
(`newCompetitivePosition`) by its `title`, its `alternativeKind` and the segments it targets
(`targetsSegmentIds`, at least one). `get_value_proposition` answers the fit check; `reach_fit`
records the kind of fit reached, problem–solution fit with the reason to switch from what they use
today.

An existing business maps the offer it sells now with `add_our_business_as_it_is_today`: the
profile's id, its title, body and citations, one per profile. Its products, relievers and creators
are written inside it and connected as ours are, and the fit check of ours on that profile shows it
beside what they use today; it is never designed and never reaches a fit. Only a value proposition
whose `isOursToDesign` is true is designed: every design write on what they use today or on our
business as it is today is refused "<code> is not ours to design."

**Our value proposition's design** is written part by part, each part whole, replacing what was
there: `write_concept` the concept chosen with its napkin sketch, `write_use_case` the ten rows of
its full life cycle use case, `draw_product` the drawing frame by frame, `quantify_value_proposition`
the value quantified, `write_promise` the ad-libs, final napkin sketch and objections. Its features
are entries (`VP.FEAT`) written with `add_entry` inside a product or service of ours (`ownerId` the
product's id) and put in order with `rerank_entries` on that product; its brochure is a field of the
value proposition, written with `update_entry`. `get_value_proposition` with `sections` reads only the
parts named — `Promise` and `Features`, say — beside the header; before a part is rewritten, read it,
so what is kept is sent back. The shapes, the pictures each part carries and the refusals are in
[reference/entries.md](reference/entries.md).

## The Competitive Landscape

Every alternative the customer has is a Competitive Position entry (`CL.POS`) on the Competitive
Landscape: what it is to the customer (`alternativeKind`: `StatusQuo`, `DirectCompetitor`,
`OfferFromAnotherIndustry` or `LogicalFutureCompetitor`) and the segments it targets. One written
before alternatives had kinds reads unclassified until `classify_competitive_position` gives it both.
Its record is a list of facts appended one at a time with `record_competitor_observation`, each
through the form of its type, which the kind's schema serves as `observationForms`: a fact is never
edited or deleted, and the one true on the latest day is its section's latest. `read_the_field` writes our
reading of it: its threat to ours, where it is going and what works for it. `get_competitor_record`
reads the record whole, by section with history and the numbers worked out from its ads and test
purchases; `get_block` on `CL` answers each position's latest fact per section beside our candidates,
the Strategy Canvases and the openings.

A Strategy Canvas (`CL.SC`) compares one value proposition of ours with the alternatives: `add_entry`
names it, then `draw_strategy_canvas` writes its factors and scores whole, `correct_strategy_canvas_score`
corrects one score from evidence and `read_strategy_canvas_lead` reads an alternative's lead over ours.
Our moat and Core candidates, and our guesses of a competitor's, are `CL.MOAT` entries;
`make_core` makes one of ours the working Core, and `lock_core` locks it with the plan for building it
when the founder locks it. The rules each holds, the shapes and the refusals are in
[reference/entries.md](reference/entries.md).

## The idea

The project's idea is not a block: the founder's starting point, the candidates weighed with their
signals, and every statement of the idea are read with `get_ideation` and written with the idea
tools, each taking `ideationVersion`. Never write the idea as a block entry. The statement names its
riskiest part as a hypothesis: `create_hypothesis` on the block that part belongs to first, then
`propose_ideation_revision` with its id. An agent's statement is proposed; a person accepts it on the
decisions tab.

## Status says how true an entry is, so choose it honestly

`Confirmed` means evidence is in hand. `Reported` means someone said so. `Inferred` means you
reasoned it from other entries. `Hypothesis` means it is a guess to be tested. `Unknown` means the
slot is acknowledged and empty. An entry written from your own reasoning is `Inferred`, not
`Confirmed`; a gate later reads these statuses, so an inflated one misleads the person who
relies on the canvas.

## Hypotheses, experiments, evidence

A hypothesis is one testable sentence, "We believe that …", on a block, optionally about one entry,
with a polarity and the business concern it belongs to (Desirability, Feasibility or Viability). Score it
(importance and evidence, −5 to 5) and mark its quality checks with `update_hypothesis`; then
`list_experiment_definitions` narrowed by its concern shows the method cards that can test it, and
`recommend_experiment_definition` puts one on its shortlist. `design_experiment` plans a run in one
call: test card, metrics and criteria. Start it, record observations and judge criteria as the run
goes, `record_evidence` for each bundle gathered, then `complete_experiment` with the learning card
and a verdict that agrees with the judged criteria. The details, the order BusinessIntelligence
enforces, and worked examples are in [reference/hypotheses.md](reference/hypotheses.md) and
[reference/experiments-and-evidence.md](reference/experiments-and-evidence.md).

Two calls record a person's decision and never yours: `decide_hypothesis` (Persevere, Pivot, Kill,
ContinueTesting) and `rule_contradiction`. When a decision is needed, present the evidence and ask.
STOP and call the AskUserQuestion tool to clarify.

## Questions, risks, ideas, contradictions

When you cannot write an entry with confidence because something is unknown, `add_question` on the
block rather than guessing; what a later step learns towards an open question is appended to it with
`add_what_was_learned`, and the answer that closes it is `resolve_question`. When you notice
something that could go wrong, `add_risk` with likelihood and severity. When you have a thought
that is not yet an entry, `add_idea` with effort and impact.
When two entries cannot both be true, `record_contradiction`; a Critical one that nobody has ruled
on blocks the canvas gates, which is the point. Verbs and states are in
[reference/ledgers.md](reference/ledgers.md).

## The roadmap and operation runs

A project's **roadmap** is the plan it runs: forty-eight operations from the method, in order, banded
by tier, plus the moves it has decided to make. Start with `get_roadmap` — it answers the thesis, every
row with its id, tier, status and prerequisites, the gate line above the plan, and the `roadmapVersion`
every change to the plan carries. **That version is not the canvas version.** Changing the plan and
writing on the canvas are two different aggregates with two different numbers; send each where it
belongs.

Changing the plan: `set_roadmap_thesis`, `place_roadmap_item` (one row to one position and one tier, in
one call — every row after it is renumbered), `skip_roadmap_item` with a reason and
`include_roadmap_item` to undo it, `annotate_roadmap_item`, `append_roadmap_operation`,
`add_roadmap_move` for an idea a person has adopted, `remove_roadmap_move`.
Each answers the whole plan with the version it produced; carry that into the next change.

**Running an operation** is `mentat-operation`'s loop; these are the calls it is made of:

1. `get_operation_package` with the row's id, or with `operationSlug` when the operation is named by
   its slug: a slug the plan does not hold, a retired one among them, is refused naming the
   operations it does hold. It answers the method text as this project has it, the
   procedures in order with their ids and what each asks the founder, the places on the canvas or the
   idea the operation writes to, the operations it goes back to, on an `Experiment` operation its
   hand-over and the steps it shares, what it waits on, and any run left paused.
2. `start_run` with `operationId` set to the row. A refusal naming operations that have not run is the
   prerequisite check: report which ones and ask whether to run them first or to force past them.
   STOP and call the AskUserQuestion tool to clarify. Never set `forcePastPrerequisites` on your own judgement. Keep the run id it
   answered: every write for the rest of the run carries it as `runId`.
3. Do the procedures. Write what they produce with the canvas tools above — `add_entry`,
   `create_hypothesis`, `add_question` — each carrying `runId`, and keep the ids those writes answered.
4. `checkpoint_run` after **each** procedure, naming the procedure and those ids, with the founder's
   words as `founderAnswer` when the procedure asked them, and `revisitOperationIds` when it sends the
   project back to earlier operations. A run interrupted between checkpoints resumes from the last
   one; an uncheckpointed procedure is one somebody repeats. `observe_run` records what belongs to no
   procedure.
5. `end_run` with `outcome: "Completed"` and a summary written for whoever reads this project later,
   and a verdict **only when the package says the validation level is `Checkpoint`** — an `Experiment`
   operation is judged by its experiment and a `None` operation by nothing, and a verdict on either is
   refused. `Invalidated` sends the project back to the operations the package lists, the first of
   them when none is named: the server puts them right after the row; say so when you report it. A run that could not be carried out at all is `end_run` with `outcome: "Failed"` and
   the reason, not an invalidated verdict.

Pause with `pause_run` and continue by starting the next run with `resumedFromRunId` set to the paused
one. `get_run` reads one run whole and `list_runs` — which takes the portfolio beside the project —
lists a venture's runs of every kind with the summary each left. `cancel_run` abandons a run nobody intends to
finish and is a person's call, not yours.

A run is opened on other subjects too — an experiment, a job off the board, or the project itself —
by sending `experimentId`, `workItemId` or none of the three instead of `operationId`. Only an attempt
at a row of the plan checkpoints; every other shape of run records its progress with `observe_run`.

**Gates and pivots have a skill of their own.** `get_gate_status` answers what the gate would say now
— the question, every checked block with the confidence it holds against the bar it must clear, and
whether it would pass — and records nothing. `evaluate_gate` records an evaluation for life and moves
the project as the outcome and its boundary policy say; `apply_pivot` sends it back a tier and appends
the rows the pivot revisits. The `mentat-gate-check` skill is what calls them: in a session on the
person's word, from a routine on the rule that skill carries. From here, present the numbers and hand
over; never evaluate or pivot on your own judgement.

The exact tier names, row statuses, the verdict rule per validation level, the checkpoint payload
convention and two worked runs are in [reference/roadmap-and-runs.md](reference/roadmap-and-runs.md).

## Reading a refusal

A refused call names its code, what happened, the arguments at fault, and what to do next. Do what
the recovery says rather than resending.

- `INVALID_ARGUMENT` — fix the named arguments. Nothing was read or changed.
- `NOT_FOUND` — an id or code named nothing; read again rather than guessing another.
- `CONFLICT` naming a canvas version — the canvas moved: re-read what you are changing with
  `get_block`, then resend with the version the refusal names.
- `CONFLICT` naming a roadmap version — somebody else changed the plan: re-read it with `get_roadmap`,
  check that the row you meant is still where you thought, then resend with the version the refusal
  names. Row positions may have moved.
- `CONFLICT` naming an ideation version — somebody else changed the idea: re-read it with
  `get_ideation`, then resend with the version the refusal names.
- `CONFLICT` naming `runId` — the run you are working inside cannot be worked any more. Nothing was
  written. Open a new one with `start_run` and send the write again; do not resend it unchanged.
- `CONFLICT` quoting a rule of the business model — the call is not allowed in the canvas's current
  state (a run with no criterion cannot start, a scored hypothesis cannot be reworded). Change what you
  asked for.
- `SERVER_MISCONFIGURED` or `UPSTREAM_UNAVAILABLE` — not about your call; follow the `mentat` skill's
  troubleshooting.

Composed writes (`design_experiment`, `record_evidence`) may be refused part-way: the row created
before the refused part exists. The message says so; read it back with `get_experiment` and finish
with `add_metric`, `add_criterion`, `update_evidence` or `add_data_point` rather than designing again.

## Rules that are easy to get wrong

- Never invent a tenant, a portfolio or a project. Lists answer what exists.
- `get_canvas` is large: every block's cards and the segments in short, never what is written inside
  a segment or a value proposition, which comes through `get_customer_segment` and
  `get_value_proposition`. Reach for `get_block` unless the task spans blocks.
- Retire what was once believed (`retire_entry`, `retire_hypothesis`); delete only what should never
  have existed. Deletes cascade to hypotheses and contradictions about the entry.
- One `update_*` call per changed thing; each takes the version the previous answered.
- Changes to the plan take `roadmapVersion`; writes on the canvas take `canvasVersion`; writes to the
  idea take `ideationVersion`; writes to a run take none of them. Sending the wrong one is refused, not
  silently accepted.
- A nested kind is added inside its owner, with `ownerId`; a part is written with its own tool,
  never in `fields`.
- `runId` is not a version and is not one of those three: it says which run a write was made from
  inside, and it is sent on every write of a run including the canvas ones.
- Do not cache what a read answered across turns: another person in the same organisation may have
  written since.
