# Entries: kinds, fields, owners and parts

`add_entry` needs the block's id, the kind's `kindDefinitionId`, a title, a status, and — for a typed
kind — `fields`; for a kind written inside another entry, the owner's id as `ownerId`; for a
confirmed or reported customer or value entry, at least one citation in `citations`. All of it comes
from `get_block`: its
`kinds` list carries each kind's `definitionId`, `code`, `storage`, its `schema` — the columns and the
words — and, for a custom kind, the `attributes` it declares. On Customer Segments `get_block` answers
the schemas only with `includeSchemas: true`. The owner's id comes from the read that answered the
owner: `get_block` for a segment, `get_customer_segment` for a profile, `get_value_proposition` for a
value proposition.

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
  "citations": [
    { "reference": "Interview with Ana Pop, owner of FitHub Cluj", "who": "Ana Pop", "words": "I refresh my inbox between every session, and still they answer after ten minutes.", "seenOn": "2026-10-14" },
    { "reference": "https://www.reddit.com/r/gymowners/comments/…", "who": "u/liftlocal", "words": "Ten minutes waiting on a client reply is ten minutes I'm not coaching.", "seenOn": "2026-09-30", "upvotes": 212, "comments": 48 }
  ],
  "status": "Reported",
  "fields": { "kind": "CustomerPain", "painKind": "Obstacle", "severity": "High", "measure": "more than 10 minutes", "isMeasureNumberMissing": false, "barrierToAdoptionAtUseCaseFactor": null }
}
```

A new item lands last among its kind inside its owner; `rerank_entries` with every one of them, in
the new order, moves it. `update_entry` on a typed kind takes the whole new `fields` object; leave
`fields` out to keep the current columns. A blank body is refused; leave `body` out rather than
sending an empty string, and use `retire_entry` when the entry no longer holds.

## Citations: why we believe it

Evidence sits on the statement it backs. Every entry carries `citations`, a list; every fact inside a
part carries its own (below); a signal of the idea carries exactly one, `citation`.

| Member | What it holds |
| --- | --- |
| `reference` | Required. Where it came from, so anyone can check it: a link, "interview with Ana, 12 Oct", the business's own data, an experiment, the founder's starting point |
| `who` | Who said or did it: "u/indiedev42", "Starter Story" |
| `words` | What they said, quoted exactly, transcripts included; markdown |
| `seenOn` | The day it was said or measured, `YYYY-MM-DD` |
| `upvotes`, `comments`, `views` | The counts where it was posted, 0 or more; left out where the place counts none |

- One citation per quote or figure: ten quotes behind a pain are ten citations, never one reference
  holding ten links.
- A `Confirmed` or `Reported` customer or value entry, and a `Confirmed` or `Reported` fact inside a
  part, needs at least one: "A Reported entry needs at least one citation, so anyone can check it."
  Customers today need at least one.
- `update_entry` with `citations` replaces the whole list; leave it out to keep the list; `[]` clears
  it. Send the ones you keep along with the new one.
- A quote or a source is never a note. Write it on the statement it backs: the entry, or the fact
  inside a part.

## Inside an owner

| Kind | Written inside | `ownerId` |
| --- | --- | --- |
| `CS` | its parent segment, or nothing for the root of a tree — the market | the parent segment's id, or left out |
| `CS.PROFILE` | a segment: a shortlisted one or a beachhead | the segment's id |
| `CS.CONTEXT` | an end user profile | the profile's id |
| `CS.JOBS`, `CS.PAINS`, `CS.GAINS` | a profile, end user or context | the profile's id |
| `VP.PS`, `VP.PR`, `VP.GC` | a value proposition | the value proposition's id |

`CS.NOTES`, `VP`, `VP.NOTES` and the kinds of every other block are written on the block itself and
refuse an `ownerId`. On CS, `get_block` lists the notes in `block.entries` and the segments under
`customerSegments`:

| Member | What it holds |
| --- | --- |
| `outline` | Every live segment, parent before child: id, code, title, status, display order, and only what holds a value — parent code, state, score, the segment it resells to, `whySplitIntoChildSegments`, `whyDropped`, `whyNotBeachhead`, and the true flags of `isRoot`, `isLeaf`, `isBeachhead`, `isProposedBeachhead`, `isServed`, `isOutOfTheRunning` |
| `retiredCount` | How many segments are retired. `includeRetired: true` lists them in the outline with `isRetired`, `retirementReason` and `supersededByReferenceCode` |
| `beachheads` | Each accepted or proposed beachhead with what `get_customer_segment` answers but the end user profile — the whole entry, ratings, market conditions, comparison, customers today, TAM, market share, the beachhead part, offers tried — and `adjacentMarkets`: each live adjacent market's title, body, state, score and parts, in the order of the beachhead part, which says what changes to win each |
| `shortlisted` | On a tree with no beachhead yet, its shortlisted segments still in the running: title, body, state, score, parts and `whyNotBeachhead` |

`allInFull: true` adds every live segment as a whole entry to `block.entries` — body, citations,
timestamps, and `fields` with its card, parts and the refusals of a beachhead or an end user profile
on it. Read one segment whole with `get_customer_segment` instead. On VP, `get_block` lists our value
propositions and the notes.
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
| children | `split_customer_segment` | why it is split, then the children in order, each with `gatheringPlacesKept`: the names of the parent's gathering places it keeps, `[]` for none |
| the seven ratings, each with its reason | `rate_customer_segment` | the score is worked out from them |
| the three conditions of a market | `test_market_conditions` | each holds or not, with its evidence |
| the comparison | `write_comparison` | on a shortlisted segment, a beachhead or a beachhead's adjacent market; a row of the founder's own is named once. Where they gather is not a row: it is the segment's gathering places |
| the gathering places | `write_gathering_places` | the whole list; each name once; every measurement a kept place had is sent again; a place the persona uses stays |
| customers today | `write_customers_today` | marks the segment served |
| the TAM | `write_total_addressable_market` | on a beachhead only |
| the beachhead part | `choose_beachhead` | on a leaf; why not on every other shortlisted segment; offers to try first is at least 1 |
| the segments it turns away | `write_turned_away_segments` | on a beachhead, the whole list |

Every fact inside a part (a rating's reason, a condition's evidence, a cell of the comparison, a level
of the TAM, a characteristic, a criterion's why, a fact of the persona, how a place's end users were
measured) is `text`, `status` and its own
`citations`, the same shape as an entry's. The text is left out only when the status is `Unknown`.
A fact also carries `breakdowns` and `figures`, the numbers its text quotes, both `[]` when it has none
and always on an `Unknown` fact. The text is then the conclusion, one or two sentences on what the evidence means, and the numbers go beside it, not in it. A
breakdown is one source's buckets: `title` (what is counted, among whom, by whom, once per fact),
`measure` (`Percent` with no `unit`; `Number` with the `unit` it counts in; `Amount` with the currency
code as its `unit`), `sampleSize` or none, `bucketsAreOrderedRanges` (true for ages and incomes, false
for countries and job titles), at least two `buckets` (`label` once each, `value`, a percent 0 to 100,
and `fitWithSegment`: `InSegment`, `PartlyInSegment`, `OutsideSegment` or none), and its one
`citation`, and its `chart`: `Columns` for ranges in order, `Bars` for names, `StackedBar` or `Donut`
for shares of a whole (`Percent` only), `DotPlot` for the same buckets in several sources, `Line` for
a number over time (its buckets the dates oldest first, `bucketsAreOrderedRanges` true). A gender
breakdown is labelled only `Men`, `Women` and `Other`. A figure is one number: `label` once per fact,
`value`, `measure`, `unit`, its one `citation` and its `display`, `Tile` or `Ring` (a share from 0 to
100%). `steps` tell a path as a timeline, at least two, each a `lead` and its `text`; `tags` are the
short labels it names, each once. All four are `[]` when the fact has none. `measure`, `chart` and `display` are
required: a breakdown or figure sent without one is refused, naming it.
Customers today carry their `citations` beside the figures. `get_block` on CS answers how many
citations each entry has, not the citations; `get_customer_segment` answers them whole.

**The gathering places are the segment's own.** Each segment holds its own list; nothing is shared
with its parent or read from it. A place is `platform` (free text: "Reddit", "Hacker News",
"Substack", "the local gym"; reuse the spelling already on the segment, because the tally groups by
it), `name` ("r/ClaudeCode", "Show HN"), `link` or none, `measurements` (`count`, `unit` in words such
as "members" or "posters a year", `seenOn`, at least one citation), `whyTheyAreThere` (one or more of
`TalkToEachOther`, `ReadThere`, `ShowTheBehaviourThere`, `BuyThere`), `endUsersThere` once measured,
and `howMeasured`, a fact with its sample, citations and status. `write_gathering_places` replaces the
whole list, so read the segment with `get_customer_segment` first and send every place back. A new
count is a new measurement added to the place's list: the earlier ones are sent again unchanged, never
edited or dropped, and a list that leaves one out is refused. The latest audience, the growth between
two counts in the same unit, and the tally by platform are worked out by the server, never sent. The
tally adds each place's latest count in every unit it was counted in, one sum per unit, before
overlap: one person can be in several places.

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
| who the end user is: the twelve characteristics and any added. Where they gather, what they read and where they are counted are the segment's gathering places | `describe_end_user` |
| the team member who fits it, who else decides, other patterns and outliers | `update_entry` on the profile, in its `fields` |
| the purchasing criteria, in order, each pointing at one of the profile's own jobs, pains or gains | `write_purchasing_criteria` |
| the gaps left by what they use today, on the end user profile and on each context profile | `write_gaps` |
| a job's high-value score, on either profile | `score_high_value_job` |
| the persona: how it was made, its fact sheet in the profile's characteristic names, which of its segment's gathering places the person uses (`gatheringPlacesTheyUse`, by name), its words for the top pain, the last purchase for the top job (`lastPurchaseForTheTopJob`: what they bought, why that one, what else they looked at and who had a say, a fact with its citations, or none), its picture prompt | `write_persona` |
| the persona's picture: a PNG, JPEG, WebP or SVG of at most 25 MB, sent from its file with one PUT to the link `request_picture_upload` answers for the slot `PersonaPicture`, with the end user profile's id as `ownerId`; taken off, its prompt kept, by `remove_picture` with the same slot and owner | `request_picture_upload`, `remove_picture` |
| a real end user's persona, erased on their request | `erase_persona` |
| the profile moved onto a leaf when the beachhead moves | `move_end_user_profile` |

A real end user's persona says what the person agreed to be kept. The places it uses are names on
its segment's list. A move is refused while one of our
value propositions aimed at the profile has reached a fit: a new offer starts from the moved profile.

**The gaps are a ranked list, written whole.** `write_gaps` takes the profile's id (`profileId`, an
end user or a context profile: each holds its own) and `gaps` in rank order, the one that matters most
first, replacing the list there was; `[]` clears it. Each gap is `pointsAtProfileEntryId`, a live
top-ranked item of that same profile — its top job, or one of its first three pains or gains —
`coverageByWhatTheyUseToday` (`NothingAddressesIt` or `AllAddressItBadly`) and `inTheirUsersWords`, a
fact with its citations, link and date seen. One gap per item. `get_customer_segment` answers each
profile's `gaps` in rank order, each with the code and title of the item it points at. The free-text
gaps field of a profile is kept only for older plugins: never write it.

**A high-value job is scored, its sum worked out.** `score_high_value_job` takes the profile's id, the
job's id (`jobId`) and its marks, each a rating from `VeryLow` (1) to `High` (4) with its `reason`, a
fact with its citations; the call replaces the score there was.

| Mark | Sent as | Rule the server holds |
| --- | --- | --- |
| Important | read from the job's importance: 5 gives 4, 4 gives 3, 3 gives 2, lower gives 1; `importanceRaisedByItsPainsOrGains` only when a pain or gain of the job matters more, its reason naming it | a raise not above the job's own mark is refused |
| Tangible | `tangible`, always | 1 to 4 |
| Unsatisfied | read from the gap that points at the job: `NothingAddressesIt` gives 4, `AllAddressItBadly` 3; `unsatisfiedWhenNoGapPointsAtTheJob` exactly when no gap points at it | sent beside a gap, or left out without one, is refused; a gap written later on the job drops the written mark |
| Lucrative | `lucrative`, always | 1 to 4 |

The score is the sum, 4 to 16, never sent. `get_customer_segment` answers each scored job's
`highValueScore` (each mark, whether it was read from the profile, its reason, and the `score`) and
each profile's `highValueJobs`, highest sum first, ties in rank order. Write the gaps first: a job's
Unsatisfied mark comes from them. When a rewrite of the gaps takes away the gap that gave a job its
mark, that job reads no score until it is scored again.

**The openings sit beside the gaps.** Each profile some value proposition of what they use today aims
at answers `openings`: its top pains no alternative's value map relieves, each with the complaints in
the competitor records that name it and `isOpeningWithoutAGap` when no gap points at it — the field
and the gaps disagree, so look at both before writing the gaps again. Nothing writes them; they are
worked out.

**A pain's measure and its barrier.** A pain's `fields` carry `isMeasureNumberMissing`, true when the
measure is known and no source gives its number (the web reads "Number not found", not "Measure
missing"), and `barrierToAdoptionAtUseCaseFactor`, the step of the full life cycle use case at which
it stops them adopting the product (`DeterminesTheyHaveANeed` … `BuysMoreOrSpreadsTheWord`, the ten
factors of `write_use_case`), or none. Send both on every rewrite of a pain.

| Refused | Because |
| --- | --- |
| "A high-value job mark is 1 to 4." | a mark sent as `VeryHigh` |
| "Job <code>'s importance gives it Important <n>; a pain or gain raises the mark only above that." | `importanceRaisedByItsPainsOrGains` not above what the job's importance gives |
| "Gap <n> of <profile code> gives job <code> its Unsatisfied mark; leave the mark out of the score." | `unsatisfiedWhenNoGapPointsAtTheJob` sent while a gap points at the job |
| "No gap points at job <code>, so its score says how unsatisfied they are, with why." | no gap points at the job and the mark was left out |
| "Gap <n> points at no top-ranked job, pain or gain of profile <code>. …" | the item is not live and top-ranked on that same profile |
| "Two gaps point at <code>. Write one gap per item." | two gaps on one item |
| "A pain's number can be missing only when its measure is written." | `isMeasureNumberMissing` true with no `measure` |
| "Only an obstacle or a risk can be a barrier to adoption." | a barrier on an `UndesiredOutcome` pain |

**What a gap points at is not deleted.** `delete_entry` on a job, pain or gain a gap points at is
refused "<code> cannot be removed: gap <n> of <profile code> points at it." Write the gaps without it
first.

## Value propositions

| What | Tool |
| --- | --- |
| our value proposition, aimed at a profile or a draft | `add_entry` on `VP`, then `aim_value_proposition` |
| its products, relievers and creators | `add_entry` with `ownerId` the value proposition |
| a connection to the job, pain or gain addressed | `connect_value_map_entry`, `disconnect_value_map_entry` |
| what they use today, mapped against a profile before or after ours aims at it, on a Competitive Position entry that exists or a new one | `add_what_they_use_today` with the profile's id (`profileId`) and exactly one of `competitivePositionEntryId` or `newCompetitivePosition` (`title`, `alternativeKind`, `targetsSegmentIds`) |
| our business as it is today, the offer an existing business sells now, one per profile | `add_our_business_as_it_is_today` with the profile's id (`profileId`), `title`, `body` and `citations` (empty while nothing shows it yet); its products, relievers and creators then with `add_entry` and `ownerId` it, connected as ours are |
| what ours forgoes and why, its brochure | `update_entry` on our value proposition, `whatItForgoesAndWhy` and `brochure` in its `fields` |
| where a pain reliever or gain creator comes from when not the products and services ("30-day money-back guarantee", "the price") | `offerPartItComesFrom` in its `fields` |
| the kind of fit reached | `reach_fit`: problem–solution fit with the reason to switch from what they use today, then product–market, then business model, one at a time |

A draft connects nothing until it is aimed; a value proposition with connections or a fit is not
retargeted. A new Competitive Position named by `add_what_they_use_today` needs at least one segment:
"Pick at least one segment this alternative targets." A Competitive Position entry a value
proposition maps as what they use today is retired, never deleted. A rewrite with `update_entry`
replaces every field: send the profile it aims at now, what it forgoes and the brochure back with
the one that changes.

`get_value_proposition` answers which of the three a value proposition is: ours to design
(`isOursToDesign`), our business as it is today (`isOurBusinessAsItIsToday`), or what they use today
(neither). Our business as it is today and what they use today carry a value map and sit in the fit
check of ours on the same profile; neither is designed nor reaches a fit, and a design write on either
is refused "<code> is not ours to design." "Profile <code> already has our business as it is today,
<code>." means it is mapped: write inside the one there.

## Our value proposition's design

Each part is its own tool and is written whole: it replaces the part there was, so read it first with
`get_value_proposition` and `sections` naming it, and send back what stays. Every one takes the value
proposition's id (`valuePropositionId`), `canvasVersion` and `runId`.

| Part | Tool | What it takes |
| --- | --- | --- |
| the concept chosen | `write_concept` | `name`, `coreIdea`, `sketch` (`caption`, `whatTheDrawingShows`, `picturePrompt`), `whatStrengthensIt`, `whatUnderminesIt`, and `changeToTheCurrentValueProposition` (`Improve`, `Extend` or `Invent`) exactly when our business as it is today aims at the same profile |
| the full life cycle use case | `write_use_case` | `factors`: one row for each of the ten — `DeterminesTheyHaveANeed`, `FindsOutAboutIt`, `AnalysesIt`, `AcquiresIt`, `InstallsIt`, `UsesIt`, `DeterminesTheValueGained`, `PaysForIt`, `ReceivesSupport`, `BuysMoreOrSpreadsTheWord` — each with `today` and `withOurs`, facts with their citations, the `UsesIt` row telling the use in its steps; `visualPrompt` |
| the features | `add_entry` on `VP.FEAT`, `ownerId` a product or service of ours | `fields`: `kind: Feature`, `whatItDoes`, `howItWorks`, `isBuilt`, `servesUseCaseFactor` (one of the ten), `servesSupportingJobId` (a live supporting job of the aimed profile), `whereElseItShows`, `candidateToCutReason` (null while it is not one), `producesIds` (pain relievers and gain creators of the same value proposition it produces). Ranked with `rerank_entries` on the product; changed with `update_entry`, every field sent back |
| the drawing | `draw_product` | `form` (`SoftwareStoryboard`, `ServiceStoryboard`, `HardwareDiagram`), `frames` in order, at least one — each `id` (the one read, or none for a new frame), `title`, `whatHappens`, `showsFeatureIds`, `wireframePrompt` — and `diagramPrompt` on a `HardwareDiagram` only |
| the value quantified | `quantify_value_proposition` | `measuredOnProfileEntryId` (a live job, pain or gain of the aimed profile), `asIs` and `possible` (facts with their numbers and citations; `possible` stays `Hypothesis` until a pilot or a customer shows it), `difference`, `sentence`, `benefitCategory` (`Better`, `Faster`, `Cheaper`), `stageRows` (`stage`, `today`, `withOurs`, `change`; empty when there is no table) |
| the brochure | `update_entry` on the value proposition | `brochure` in its `fields`, markdown |
| the promise | `write_promise` | `adLibs` in rank order, at least one — each `sentence`, `painRelieverId`, `gainCreatorId` and `unlikeWhatTheyUseTodayValuePropositionId` (none when it names no alternative) — `finalNapkinSketch` shaped as the concept's sketch, and `objections`, each `inTheirWords` (a fact with its citations) and exactly one of `answer` or `writtenAsRiskId` (a risk of this canvas, written first with `add_risk`) |

A concept, use case, drawing and promise carry their pictures through their prompts: the part is
written first with the prompt, then the picture goes up through `request_picture_upload` with the
slot `ConceptNapkinSketch`, `UseCaseVisual`, `ProductDiagram`, `DrawingFrameWireframe` (naming the
frame's id from the drawing just written) or `PromiseNapkinSketch`, and the value proposition's id as
`ownerId`. A part written again with the same prompt keeps its picture; another prompt drops it.

| Refused | Because |
| --- | --- |
| "<code> is not ours to design." | the value proposition is what they use today or our business as it is today |
| "Say whether this concept improves, extends or replaces the current offer." | our business as it is today aims at the same profile, and the concept left `changeToTheCurrentValueProposition` out |
| "No current offer aims at this profile, so the concept changes nothing of it." | the concept named a change, and no business of ours as it is today aims at that profile |
| "The use case has no row for <factor>. It holds one row for each of the 10 factors." | a factor is missing; a factor sent twice is refused the same way |
| "Job <code> is not a supporting job of the profile <code> aims at; …" | the feature's job is another kind of job, or on another profile |
| "Only a HardwareDiagram has a diagram of the whole, …" | a diagram prompt on a storyboard |
| "An objection is either answered or written as a risk, …" | an objection sent with both, or with neither |

**What the design holds up is not deleted.** `delete_entry` is refused, naming every row that points
at the entry: "<code> cannot be removed: it holds features <codes>." on a product with features; "…
feature <code> produces it." on a reliever or creator; "… ad-lib <n> of the promise of <code> names
it." and "… frame '<title>' of the drawing of <code> shows it." on what they name; "… the quantified
value of <code> is measured on it." and "… feature <code> serves it." on a profile item. A risk an
objection became is held the same way: deleting it in the web is refused "<code> cannot be removed: an
objection in the promise of <code> became this risk." until the promise is written with that
objection answered instead.

## The Competitive Landscape

Three typed kinds on `CL`, each written with `add_entry` and its `fields`, and read back by
`get_block` on `CL`:

| Kind | `fields` | Written through its own tool |
| --- | --- | --- |
| `CL.POS`, a Competitive Position | `kind: CompetitivePosition`, `alternativeKind` (`StatusQuo`, `DirectCompetitor`, `OfferFromAnotherIndustry`, `LogicalFutureCompetitor`), `targetsSegmentIds` (live segments of this canvas, at least one once it has a kind) | `classify_competitive_position` (kind and segments together), `record_competitor_observation`, `read_the_field` |
| `CL.SC`, a Strategy Canvas | `kind: StrategyCanvas`, `comparedValuePropositionId` (a value proposition of ours to design) | `draw_strategy_canvas`, `correct_strategy_canvas_score`, `read_strategy_canvas_lead` |
| `CL.MOAT`, a moat or Core candidate | `kind: MoatOrCoreCandidate`, `guessedForCompetitivePositionId` (none on ours), `category`, `verdict` (`Core`, `Moat`, `NeitherYet`), `moatDurability` (`Wide`, `Narrow`, exactly on a `Moat`), `deliversBetterThanAnyoneIds` (pain relievers and gain creators of ours; `[]` on a guess) | `make_core`, `lock_core` |

**A Competitive Position.** One written before alternatives had kinds has no `alternativeKind` and its
old values in its body as a JSON block; `classify_competitive_position` gives it a kind and its
segments. "A Competitive Position targets a live segment of this canvas."

**Its record is appended, never edited.** `record_competitor_observation` adds one fact: its `type`
(one of the 22 sections: `Identity`, `OfferAndPrice`, `Promise`, `ProductVelocity`, `Financials`,
`TestPurchase`, `TrafficAndSearch`, `Demand`, `AiVisibility`, `Ad`, `LandingPageAndComparisonPages`,
`Email`, `ContentAndSocial`, `ReviewCountAndRating`, `Praise`, `Complaint`, `SwitchingStory`,
`ChannelsAndDistribution`, `SalesMotionAndSupport`, `Hiring`, `Teardown`, `SuppliersAndPartners`),
the day it was true (`observedOn`), the fact itself (a fact as inside a part: text, status, citations,
figures, tags), the ad on an `Ad` fact, the receipt on a `TestPurchase` fact, and on a `Complaint` the
top pain it names (`aboutProfileEntryId`), a live top pain of a profile this position's value
propositions aim at. Read the form of each type from the `CL.POS` kind's `observationForms` before
writing: the figures it needs under their labels ("price", "monthly visits") and the tags it needs,
each written `name: value` ("plan: Pro", "month: 2026-10"). A fact that is not `Unknown` carries a
citation with its link and the date it was seen. Each refusal names the type and what is missing:
"Ad id is required for an Ad fact.", "Order or invoice number is required for a Test purchase
fact.", "A Financials fact needs its revenue, profit or recurring revenue figure and the period it
covers." A section that could not be filled is an `Unknown` fact whose text says why. Sent with the
run's id, the fact is entered by the agent, and by the routine's collector in a routine's run; sent
without one, it is the person's. Ad running days, orders per 30 days and growth are worked out by
`get_competitor_record`, never typed; growth against ours needs the goal of ours that counts orders,
named as `comparedWithGoalId`, and without one the record says "No number of ours yet: pick the goal
that counts our orders."

**Our reading of it.** `read_the_field` writes the threat to ours (`Low`, `Medium` or `High`, with
why), where it is going and what works for it, each a fact with its citations; it replaces the last
reading.

**A Strategy Canvas** compares one value proposition of ours on the jobs, pains and gains of the
profile it aims at. `draw_strategy_canvas` replaces its factors whole, in order: each a `name` (what
it does for the customer), the job, pain or gain it matches, the pain reliever or gain creator it came
from (none when it came from the pain or gain itself), and its scores — one for ours (no Competitive
Position) and one per alternative, each 0 to 10 with its evidence; evidence with status `Hypothesis`
marks the score a guess, drawn dashed. Every alternative is a live, classified Competitive Position,
and among them a status quo that targets the aimed profile's segment. `correct_strategy_canvas_score`
rewrites one score from evidence; `read_strategy_canvas_lead` reads an alternative's lead on one
factor as `DeliberateTradeOff` or `ThingToImprove`, with why, only where it scores above ours.

| Refused | Because |
| --- | --- |
| "The Strategy Canvas needs a status quo alternative that targets <segment name>." | no classified status quo targets the aimed segment |
| "<code> is not ours to design." | the compared value proposition is what they use today or our business as it is today |
| "<code> already has Strategy Canvas <code>." | one live Strategy Canvas per value proposition of ours |
| "A factor matches a job, pain or gain of <profile code>, the profile <code> aims at." | the factor matches an item of another profile |
| "<code> is not classified yet. Classify it before a Strategy Canvas compares it." | an alternative has no kind |

**Moat and Core candidates.** Ours carry no `guessedForCompetitivePositionId` and show as cards; a guess
names the competitor and shows in that competitor's record. A competitor has one guessed Core: "<code>
is already our guess of <competitor>'s Core." "A moat says whether it is wide or narrow." / "Only a
moat has a durability." `make_core` makes a candidate of ours with verdict `Core` the working Core —
or the strongest moat while no candidate is a Core, keeping its verdict and durability. `lock_core`
locks the working Core with the plan for building it, every part required: team members with the
skills, leaders in the field, where they are found, owner, review cadence, skills plan, incentives.
A locked Core is not replaced: "The Core <code> is locked; another candidate cannot become the Core."

**What it holds up is not deleted.** `delete_entry` is refused, naming every row that points at the
entry: "<code> cannot be removed: Strategy Canvas <code> scores it." on a Competitive Position a
Strategy Canvas scores, "… moat guess <code> points at it." or "… value proposition of what they use
today <code> points at it."; "… Strategy Canvas <code> compares it." on our value proposition; "…
Competitive Position <code> targets it." on a segment; and on a job, pain, gain, reliever or creator a
factor matches or comes from, or the Core delivers.

## The idea

| What | Tool, each with `ideationVersion` |
| --- | --- |
| the founder's starting point, and the route the first time | `write_starting_point` |
| an idea weighed, its statement and ratings, why it was not chosen | `add_ideation_candidate`, `update_ideation_candidate`, `drop_ideation_candidate` |
| a fact read for an idea, with its one `citation`: who, their words, the day, and a web address as its `reference` (bare or `[Reddit](https://…)`) unless it is a hypothesis | `add_ideation_signal`, `update_ideation_signal` |
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
| `Confirmed` | Evidence is in hand: an experiment, a document, a number you can cite. Send at least one citation. |
| `Reported` | A person said so: an interviewee, the founder, a review. Send at least one citation. |
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
