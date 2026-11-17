# Value propositions, our product and its features

The Value Proposition block holds two kinds of card, and everything else sits inside one of them:
our product (`VP.PRODUCT`), written once whatever the number of value propositions built on it, and
each value proposition of ours (`VP`). A feature is never a card: it is inside the product. A concept
set aside, a barrier to adoption and the features tied to a value proposition are never cards: they are
read inside the value proposition.

## The block's cards

| Card | Kind | Read inside with |
| --- | --- | --- |
| our product: what we sell, holding its features and their sub-features | `VP.PRODUCT` | `get_product` |
| a value proposition of ours: ours to design, or our business as it is today | `VP` | `get_value_proposition`, by its sections; `get_entry` for its own entry |
| a note | `VP.NOTES` | — |

`get_block` on `VP` lists only the cards in `block.entries`. The product's `fields` carry its `card`:
how many features and sub-features it holds, how many are built, partly built and planned, the squares
feature by feature with every sub-feature's id, code and built state, and the value propositions built
on it with what it produces of each. A value proposition's `fields` carry its `card`: its fit on paper
with the three tallies, its concept's name, how many concepts were set aside, its barriers to adoption
and the features tied to it. A value proposition of what they use today is not a card: it is read
under `WhatTheyUseToday` on ours. A risk about a value proposition is among the block's risks with
`isCard` false, its title, the value proposition and the factor it arises at.

On `VP` the kinds' descriptions and schemas come only in a read of the kinds, apart from the cards —
`includeSchemas: true` every kind in short, `kindCode: "VP.FEAT"` one kind whole: make it before
writing an entry there. What is no longer in play — the retired entries, such as a retired value
proposition or a product written for its retired lines, the retired hypotheses and the rejected ideas —
comes only with `includeRetired: true`, and is counted in `leftOut` otherwise.

A card lands last among the block's cards. `rerank_entries` with the block's id as `blockId`, instead
of `ownerId`, puts them in a new order: our product first, then the value propositions built on it.
The list names every card of the block once, retired ones included, so read the block with
`includeRetired: true` first: "The order must list each of block VP's 2 cards exactly once, and nothing
else: an entry inside another is ranked by its owner." It answers the block in short — its id, code,
name and live entry count — and the canvas version.

## Reading a value proposition by its sections

`get_value_proposition` answers a header — the value proposition by id, code and title, whether it is
ours to design (`isOursToDesign`), our business as it is today (`isOurBusinessAsItIsToday`) or what
they use today, the profile it aims at by id, code and title, the canvas version — and the sections
named in `sections`, at least one. A read that names none is refused, "Name at least one section",
with the list. Name only the sections the work needs: each is shaped to be read at once, its rows in
short, and a row is read whole by naming it.

| Section | What it answers |
| --- | --- |
| `WhatTheyUseToday` | The value propositions of what they use today and our business as it is today aimed at the same profile: each with its competitor and one line, how well it does each top item (`Well`, `Badly`, `NotAtAll`) and the lines of its value map by code. Its value map in short: `ValueMap` on its own id |
| `ValueMap` | Products and services, pain relievers and gain creators in rank order, each with its body, status, relevance, measure, a count of citations and its connections; a line of ours with our product it points at; a reliever or creator with the codes of the features and sub-features that produce it |
| `FitCheck` | On ours: each job, pain and gain of the profile by id, code and title, addressed or not, the three tallies, whether it fits on paper and by which rules, the comparison with what they use today, the warnings, the offers tried and the next kind of fit (`nextKindOfFit`) |
| `Concept` | The chosen concept whole, with its napkin sketch, what it goes after and its assessment, and the concepts set aside inside it, each whole with its `id` |
| `UseCase` | The ten rows in short: today beside with ours, a count of citations, and on each factor its barriers — the risks about this value proposition at it, by title, exposure and status, and the pains marked as a barrier at it — and the codes of the features that serve it. `useCaseFactors` names up to three rows read whole in their place |
| `Features` | The features tied to it, read from the links: each feature of a product it builds on that produces one of its pain relievers or gain creators, with its headline, built state and what it produces here, its sub-features under it |
| `Drawing` | Its form and its frames in short. `drawingFrameIds` names up to five frames read whole, each with its wireframe's prompt |
| `QuantifiedValue`, `Promise` | Each whole; each ad-lib with the words of its slots, the sentence built from them, and the feature it opens with. An ad-lib written before the slots whose sentence did not read into them has every slot null and only its sentence |
| `LandingPageCopy` | Whole: the copy as one Markdown text (`markdown`), each part of the page under its own heading, and who wrote it last (`writtenBy`, `Person` or `Agent`) and when (`writtenAtUtc`) — what `write_landing_page_copy` takes back. The document compiled from it is read with `get_landing_page_document` |

A section not named answers null, and so does a part not written yet. `get_entry` reads one row whole
by its id: a line of the value map with its citations, and the value proposition's own entry — what it
forgoes, the fit it reached, its brochure. A feature is read with `get_product`.

## Value propositions

| What | Tool |
| --- | --- |
| our value proposition, aimed at a profile or a draft | `add_entry` on `VP`, then `aim_value_proposition` |
| our product, once, whatever the number of value propositions built on it | `add_entry` on `VP.PRODUCT`, no `ownerId`, its kind as `productKind` in `fields`; read the block first, since another value proposition of ours may already build on it, then put it before the value propositions with `rerank_entries` and `blockId` |
| its line in a value map of ours | `add_entry` on `VP.PS`, `ownerId` the value proposition, the product's id as `productId` in `fields` |
| its pain relievers and gain creators | `add_entry` with `ownerId` the value proposition |
| a connection to the job, pain or gain addressed | `connect_value_map_entry`, `disconnect_value_map_entry` |
| what they use today, mapped against a profile before or after ours aims at it, on a Competitive Position or an indirect competitor that exists or a new one | `add_what_they_use_today` with the profile's id (`profileId`) and exactly one of `competitorId` or `newCompetitivePosition` (`title`, `model`, `indirectCompetitorKind` on an indirect one, `oneLineSummary`, `targetsSegmentIds`); it answers the ids of the value proposition, its competitor and the profile's Strategy Canvas. Its lines name no product |
| how well what they use today does each top job, pain and gain of the profile it aims at | `write_top_item_coverages` with the value proposition's id and `coverages`, the whole list |
| our business as it is today, the offer an existing business sells now, one per profile | `add_our_business_as_it_is_today` with the profile's id (`profileId`), `title`, `body` and `citations` (empty while nothing shows it yet); its lines, relievers and creators then with `add_entry` and `ownerId` it, a line naming our product as on ours, connected as ours are |
| what ours forgoes and why, its brochure | `update_entry` on our value proposition, `whatItForgoesAndWhy` and `brochure` in its `fields` |
| where a pain reliever or gain creator comes from when not the products and services ("30-day money-back guarantee", "the price") | `offerPartItComesFrom` in its `fields` |
| the kind of fit reached | `reach_fit`: problem–solution fit with the reason to switch from what they use today, then product–market, then business model, one at a time |

**A line of ours points at our product.** A line of a value proposition of ours without `productId`
is refused: "<code> is ours; a line of its value map points at the product it builds on. Write the
product on the block first and send its id." A line of what they use today names none. What differs
from one value proposition to the next stays on the line: its relevance and the jobs it helps. A
rewrite keeps the product the line points at, and another `productId` is refused: write a new line
pointing at the other product, then retire the old. The only line pointing a value proposition at a
product is neither retired nor deleted while features of that product are tied to it: "<code> cannot
be removed: it is the only line pointing <code> at product <code>; …", naming the features, the ad-libs
and the frames that hold it. Take them off first, or point another line at the product.

A draft connects nothing until it is aimed; a value proposition with connections or a fit is not
retargeted. A new competitor named by `add_what_they_use_today` is a `CompetitivePosition` or an
`IndirectCompetitor`: "A named competitor holds no value proposition: write it with add_entry, and
promote it to a Competitive Position to map it." A competitor a value proposition maps as what they use
today is retired, never deleted. A rewrite with `update_entry` replaces every field: send the profile
it aims at now, what it forgoes and the brochure back with the one that changes.

`get_value_proposition` answers which of the three a value proposition is in its header. Our business
as it is today and what they use today carry a value map and sit in the fit check of ours on the same
profile; neither is designed nor reaches a fit, and a design write on either is refused "<code> is not
ours to design." "Profile <code> already has our business as it is today, <code>." means it is mapped:
write inside the one there.

**The coverage rows say how well what they use today does each top item.** `write_top_item_coverages`
takes a value proposition of what they use today (`valuePropositionId`) and `coverages`, one row per
top item of the profile it aims at — its top job, its first three pains, its first three gains — each
named once, replacing the rows there were. Each row is `pointsAtProfileEntryId`,
`howWellItAddressesIt` and `inTheirUsersWords`, its users' words as a fact with its citations, link and
date seen:

| `howWellItAddressesIt` | When |
| --- | --- |
| `Well` | an entry of its value map is connected to the item, and its users say it works; their words required |
| `Badly` | an entry of its value map is connected to the item, and its users say it falls short; their words required |
| `NotAtAll` | no live entry of its value map is connected to the item; their words optional |

Connect the value map first: "<code> does <item> not at all, yet an entry of its value map is connected
to it. Disconnect it, or say how well it does it." and "<code> does <item> well, yet no entry of its
value map is connected to it. Connect the entry that does it, or say it does it not at all." A row
missing or named twice is refused naming the item. Ours is never covered: "<code> is ours to design;
ours is scored on the Strategy Canvas, not covered item by item." The rows answer `topItemCoverages`
on the value proposition's `whatTheyUseToday` line, and each gap's coverage is read from them.

## Our product and its features

A feature (`VP.FEAT`) is written with `add_entry` inside our product, the product's id as `ownerId`. A
sub-feature is a feature inside a feature: the same kind and the same fields, its feature's id as
`ownerId`, one level only. Neither is ever a card. A feature belongs to the product, not to one value
proposition: it produces pain relievers and gain creators of any value proposition of ours with a line
pointing at its product, and each value proposition reads the features tied to it from those links.

| `fields` member | What it holds |
| --- | --- |
| `kind` | `Feature` |
| `dreamOutcomeHeadline` | The benefit in one line, at most 160 characters |
| `whatItDoes` | What it does, in a sentence the customer could follow |
| `howItWorks` | The sentence; null only when at least one step is sent |
| `howItWorksSteps` | The named steps in order, each `{ "name": "…", "note": "…" or null, "text": "…" }`: a name of at most 80 characters, a note of one line of at most 160 or none; `[]` when the sentence says it all |
| `buildState` | `Built`, `PartlyBuilt` or `Planned` |
| `noteOnTheBuildState` | A note on the built state, in any state: what is missing on a partly built one, what is built of it, how many ship today; null when there is nothing to add |
| `whyItIsNeeded` | Always: the pain relievers or gain creators it supports, or the other value it has |
| `servesUseCaseFactors` | The factors of the use case it serves, a list of the ten `write_use_case` names |
| `servesJobIds` | Live jobs, of any kind, of a profile one of our value propositions built on the product aims at |
| `whyItServesNoJob` | Only when `servesJobIds` is empty: the other value it has |
| `whereElseItShows` | Where else it shows, or null |
| `whatItsStatusRestsOn` | One short line beside its status, at most 160 characters, such as "read from the product", or null |
| `candidateToCutReason` | Null, unless the founder marked it a candidate to cut, with why |
| `producesIds` | The pain relievers and gain creators it produces, of any of our value propositions with a line pointing at its product |

Example — a sub-feature inside feature `VP.FEAT-01` of our product, after reading its id with
`get_product`, the `VP.FEAT` kind's schema with `get_block` VP and `kindCode: "VP.FEAT"`, and the ids
of the pain reliever and the job with `get_value_proposition` (`ValueMap`) and `get_customer_segment`:

```json
{
  "projectId": "…", "canvasVersion": 52,
  "blockId": "<VP block id>", "kindDefinitionId": "<id of the VP.FEAT kind>",
  "ownerId": "<id of VP.FEAT-01>",
  "title": "Receipt reminders",
  "citations": [{ "reference": "https://receiptchaser.example/changelog", "seenOn": "2026-11-02" }],
  "status": "Confirmed",
  "fields": {
    "kind": "Feature",
    "dreamOutcomeHeadline": "Stop chasing: reminders run until each receipt arrives.",
    "whatItDoes": "Sends each client a reminder for every receipt still missing.",
    "howItWorks": null,
    "howItWorksSteps": [
      { "name": "Pick the missing receipts", "note": null, "text": "The month's bank lines with no receipt are listed." },
      { "name": "Remind", "note": "every three days", "text": "The client gets one message naming each missing receipt." }
    ],
    "buildState": "PartlyBuilt",
    "noteOnTheBuildState": "Email ships today; text messages are planned.",
    "whyItIsNeeded": "Supports the pain reliever 'No more chasing receipts'.",
    "servesUseCaseFactors": ["UsesIt"],
    "servesJobIds": ["<id of CS.JOBS-01>"],
    "whyItServesNoJob": null,
    "whereElseItShows": null,
    "whatItsStatusRestsOn": "read from the product",
    "candidateToCutReason": null,
    "producesIds": ["<id of VP.PR-02>"]
  }
}
```

**A feature with sub-features holds no link of its own.** Write it `Built` or `Planned` with
`servesUseCaseFactors`, `servesJobIds` and `producesIds` empty, no `noteOnTheBuildState` and no
`whyItServesNoJob`; then write those on its sub-features. From its first sub-feature on, its built
state, its factors, its jobs and what it produces are read from its live sub-features, and
`get_product` answers its state with their counts. A first sub-feature is refused while the feature
still holds one of them: "Feature <code> holds 1 pain reliever or gain creator, 1 use case factor of
its own; a feature with sub-features reads those from them. Take them off the feature first, then write
them on its sub-features." Sent to a feature that has sub-features, a link, a factor, a job, another
built state, a note or a reason it serves no job is refused, naming the sub-features: "Feature <code>
has sub-features <codes>: its built state is their counts, and it carries no note of its own. …"

A new feature lands last among its product's features, a new sub-feature last among its feature's:
`rerank_entries` with the product (or the feature) as `ownerId` and every one of them in the new order
moves it. `get_product` with `productId` reads the product with its card and its features in short —
name, headline, built state, the codes of what each produces; with `featureId` as well, that feature or
sub-feature whole, its `fields` as `update_entry` takes them back, what it produces and serves by id,
code and title, and its sub-features in short.

**Linking a feature.** `update_entry` on a feature or sub-feature takes its whole `fields` as
`get_product` answered them. A pain reliever or gain creator — one of a later value proposition built on
the same product among them — is linked by sending `producesIds` with its id added, and unlinked by
leaving it out; a job and a factor the same way. On a feature with sub-features link the sub-feature that
produces it, never the feature.

| Refused | Because |
| --- | --- |
| "VP.FEAT is written inside a product, or a feature for a sub-feature, and <code> is not one: send the product or feature's id as ownerId." | the owner sent is a line of a value map, or another entry |
| "Feature <code> is itself a sub-feature; a sub-feature has no sub-features. Write it inside the feature instead." | a sub-feature inside a sub-feature |
| "A feature says how it works: write the sentence, its named steps, or both. The sentence is left out only when at least one step is written." | `howItWorks` null with no step |
| "The note of a step is one line, at most 160 characters. Put the rest in its text." | a step's note too long |
| "What a feature's status rests on is one short line, at most 160 characters. Put the rest in its citations." | `whatItsStatusRestsOn` too long |
| "Why a feature serves no job is said only on one that serves none, and this one serves 1 job. Leave it out, or take the jobs off." | `whyItServesNoJob` beside a job |
| the pain reliever, gain creator or job named | it belongs to a value proposition of ours that does not build on the feature's product, or the job is not live |
| "The order must list each of the 12 sub-features of feature VP.FEAT-02 exactly once, and nothing else." | `rerank_entries` leaving one out or naming one the owner does not hold |

**What the product holds up is not deleted.** `delete_entry` is refused, naming every row that points
at the entry: "<code> cannot be removed: it holds features <codes>." on a product with features, "… it
holds sub-features <codes>." on a feature with sub-features, "… product or service <code> points at it."
on a product a line points at, "… feature <code> produces it." on a reliever or creator and "… feature
<code> serves it." on a job.

## Our value proposition's design

Each part is its own tool and is written whole: it replaces the part there was, so read it first with
`get_value_proposition` and `sections` naming it, and send back what stays. Every one takes the value
proposition's id (`valuePropositionId`), `canvasVersion` and `runId`, and answers the header and the
section it wrote, never the value proposition whole.

| Part | Tool | What it takes |
| --- | --- | --- |
| the concept chosen and the concepts set aside | `write_concept` | `name`, `coreIdea`, `goesAfterProfileItemIds` (the live jobs, pains and gains of the aimed profile it goes after, by id, each once; `[]` when none yet), `sketch` (`caption`, `whatTheDrawingShows`, `picturePrompt`), `whatStrengthensIt`, `whatUnderminesIt`, `assessment`, `conceptsSetAside` (the whole list, `[]` when none), and `changeToTheCurrentValueProposition` (`Improve`, `Extend` or `Invent`) exactly when our business as it is today aims at the same profile |
| the full life cycle use case | `write_use_case` | `factors`: one row for each of the ten — `DeterminesTheyHaveANeed`, `FindsOutAboutIt`, `AnalysesIt`, `AcquiresIt`, `InstallsIt`, `UsesIt`, `DeterminesTheValueGained`, `PaysForIt`, `ReceivesSupport`, `BuysMoreOrSpreadsTheWord` — each with `today` and `withOurs`, facts with their citations, the `UsesIt` row telling the use in its steps; `visualPrompt`. It answers the ten rows in short with the barriers and the features on each factor |
| the features | `add_entry` on `VP.FEAT` inside our product | above: [Our product and its features](#our-product-and-its-features) |
| the drawing | `draw_product` | `form` (`SoftwareStoryboard`, `ServiceStoryboard`, `HardwareDiagram`), `frames` in order, at least one — each `id` (the one read, or none for a new frame), `title`, `whatHappens`, `showsFeatureIds` (live features and sub-features of a product it builds on; name the sub-feature when the frame shows one), `wireframePrompt` — and `diagramPrompt` on a `HardwareDiagram` only. Read every frame whole first, five at a time in `drawingFrameIds`: the frames in short leave the prompts out |
| the value quantified | `quantify_value_proposition` | `measuredOnProfileEntryId` (a live job, pain or gain of the aimed profile), `asIs` and `possible` (facts with their numbers and citations; `possible` stays `Hypothesis` until a pilot or a customer shows it), `difference`, `sentence`, `benefitCategory` (`Better`, `Faster`, `Cheaper`), `stageRows` (`stage`, `today`, `withOurs`, `change`; empty when there is no table) |
| the brochure | `update_entry` on the value proposition | `brochure` in its `fields`, one object, never Markdown: `why` (the persona's own words), `promise` (one sentence), `howItWorks` (one to three steps), `sections` in order, at least one — each `featureId` (a live feature tied to this value proposition; left out when it is about a product or service), `heading` (its dream outcome headline), `whatItDoes`, `benefit` (left out only when its points carry theirs) and `points`, each `featureId` (a live sub-feature tied to it; left out when the point is not one), `name`, `whatItDoes`, `benefit`. Its cover picture is the promise's final napkin sketch, never sent here; the server writes the Markdown the PDF reads |
| the landing page copy | `write_landing_page_copy` | `markdown`: the copy whole, one Markdown text, each part of the page under its own heading, exactly as the operation you run says. Nothing checks the parts, and no entry is named: a feature stays removable. Refused only when blank |
| the landing page document | `get_landing_page_document` | read only: compiled on every read, never stored and never written by hand — Part 1 the build prompt of `write-the-landing-page-document`, word for word, then "What a good page is" from the method as the project runs it; Part 2 the copy exactly as written. Answered in parts (`markdownPart` of `markdownParts`) to read in order and join exactly, and its `fileName`, `landing-page-{code}.md` |
| the promise | `write_promise` | `adLibs` in rank order, at least one — each the words of its slots, never the sentence, which the server builds as "Our {featureWords} {help/helps} {who} who want to {job}, by {painRelieverWords}, and by {gainCreatorWords} (unlike {unlikeWords})." — `featureWords` (the feature that creates the value, by its name), `helps` (`Help` or `Helps`, agreeing with it), `who`, `job`, `painRelieverWords`, `gainCreatorWords`, `unlikeWords` (left out exactly when it names no alternative), `opensWithFeatureId` (a live feature or sub-feature tied to this value proposition; left out only when its pain reliever comes from another part of the offer), `painRelieverId`, `gainCreatorId` and `unlikeWhatTheyUseTodayValuePropositionId` (none when it names no alternative) — `finalNapkinSketch` shaped as the concept's sketch, and `objections`, each `inTheirWords` (a fact with its citations) and exactly one of `answer` or `writtenAsRiskId` (a risk about this value proposition, written first with `add_risk`) |

**Every concept carries its assessment**, chosen or set aside: `questionsAnswered` of `questionsAsked`
(6 for the six questions of choose-the-concept), `founderCanDeliverIt` (true or false), and
`scoreOnTheFoundersCriteria` of `highestPossibleScore`. Each of the five is required, and refused by
name when left out: zero and false are answers.

**The concepts set aside are inside the concept**, never ideas on the block and never value
propositions of their own. Each in `conceptsSetAside` is `name`, `coreIdea`, `goesAfterProfileItemIds`
(as on the chosen one), its own `sketch`, `whatStrengthensIt`, `whatUnderminesIt`, `whyItWasSetAside`
(whole, however long; nothing of it moves into what undermines it) and its own `assessment`. The answer
gives each its `id`: whenever the concept is written again, send every one back with its `id`. One left
out is deleted with its sketch, and one sent back without its `id` is written as new and loses its
sketch. No tool makes a concept set aside the chosen one: a person does that on the value
proposition's page, which keeps both sketches.

**A barrier to adoption is read on its factor**, inside the use case, never as a card on the block. One
that is the customer's own pain is a pain of the profile with `barrierToAdoptionAtUseCaseFactor` in its
fields ([entries.md](entries.md)). One that is not is a risk about our value proposition:
`add_risk` with the Value Proposition block's id, a `title`, our value proposition's id as
`valuePropositionId` and the factor as `arisesAtUseCaseFactor` ([ledgers.md](ledgers.md)). An objection
nothing answers is a risk about the value proposition with no factor, named by `write_promise`.

A concept, use case, drawing and promise carry their pictures through their prompts: the part is
written first with the prompt, then the picture goes up through `request_picture_upload` with the
slot `ConceptNapkinSketch`, `ConceptSetAsideNapkinSketch` (naming the concept set aside's `id` as
`drawingFrameId`), `UseCaseVisual`, `ProductDiagram`, `DrawingFrameWireframe` (naming the frame's id
from the drawing just written as `drawingFrameId`) or `PromiseNapkinSketch`, and the value
proposition's id as `ownerId`. A part written again with the same prompt keeps its picture; another
prompt drops it.

| Refused | Because |
| --- | --- |
| "<code> is not ours to design." | the value proposition is what they use today or our business as it is today |
| the assessment's member named | an assessment left out, a member left out, or a member out of its bounds |
| "The concept '<name>' goes after <code>, which is not a live job, pain or gain of the profile <code> aims at." | a concept, chosen or set aside, goes after an item of another profile |
| "Say whether this concept improves, extends or replaces the current offer." | our business as it is today aims at the same profile, and the concept left `changeToTheCurrentValueProposition` out |
| "No current offer aims at this profile, so the concept changes nothing of it." | the concept named a change, and no business of ours as it is today aims at that profile |
| "The use case has no row for <factor>. It holds one row for each of the 10 factors." | a factor is missing; a factor sent twice is refused the same way |
| "Frame '<title>' shows <code>, which is not a live feature or sub-feature of a product <code> builds on. …" | a frame shows a feature of another product |
| "Only a HardwareDiagram has a diagram of the whole, …" | a diagram prompt on a storyboard |
| "An ad-lib fills every slot of its template, and <slot> was not written." | a slot of an ad-lib left out or blank |
| "Ad-lib '<sentence>' opens with no feature. …" | an ad-lib names no feature while its pain reliever comes from the products and services |
| "Ad-lib '<sentence>' opens with <id>, which is not a live feature tied to <code>. …" | the feature produces none of this value proposition's pain relievers or gain creators |
| "An objection is either answered or written as a risk, …" | an objection sent with both, or with neither |
| "Not a feature of this value proposition: section '<heading>' of the brochure is about <id>, …" | a section of the brochure, or one of its points, names a feature that is not live and tied to this value proposition |
| "Section '<heading>' of the brochure says no benefit and has no point that does: …" | a section with no benefit and no point to carry one |

**What the design holds up is not deleted.** `delete_entry` is refused, naming every row that points
at the entry: "… ad-lib <n> of the promise of <code> names it." and "… frame '<title>' of the drawing of
<code> shows it." on what they name; "… the quantified value of <code> is measured on it.", "… the
concept of <code> goes after it." and "… the concept set aside '<name>' of <code> goes after it." on a
profile item. A risk an objection became is held the same way: deleting it in the web is refused
"<code> cannot be removed: an objection in the promise of <code> became this risk." until the promise
is written with that objection answered instead.
