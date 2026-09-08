# Entries: kinds, fields and statuses

`add_entry` needs the block's id, the kind's `kindDefinitionId`, a title, a status, and — for a typed
kind — `fields`. All of it but the words comes from `get_block`: its `kinds` list carries each kind's
`definitionId`, `code`, `storage` and, for a custom kind, the `attributes` it declares.

## Typed kinds: the `fields` object

`fields` is a JSON object. Its `kind` property is the storage name, exactly as `get_block` spells it;
the other properties are that storage's columns. Enum-valued columns take the member name as
written below. Ids point at entries anywhere on the canvas unless the column says the same block.

| Storage (`kind`) | Columns | Allowed words |
| --- | --- | --- |
| `CustomerSegment` | `marketType` | MassMarket, NicheMarket, Segmented, Diversified, MultiSidedPlatform |
| `CustomerJob` | `segmentId` (same block), `jobKind`, `importance` | jobKind: Functional, Social, Emotional, Supporting · importance: VeryLow, Low, Medium, High, VeryHigh |
| `CustomerPain` | `segmentId` (same block), `severity` | VeryLow, Low, Medium, High, VeryHigh |
| `CustomerGain` | `segmentId` (same block), `relevance` | VeryLow, Low, Medium, High, VeryHigh |
| `ValueProposition` | `segmentId` (null while the fit is open) | — |
| `ProductService` | `valuePropositionId` (same block) | — |
| `PainReliever` | `valuePropositionId` (same block), `painId` (null while no pain is named) | — |
| `GainCreator` | `valuePropositionId` (same block), `gainId` (null while no gain is named) | — |
| `Channel` | `segmentId` (null = every segment), `phase`, `isOwned`, `isDirect`, `tactic` (null when none fits) | phase: Awareness, Evaluation, Purchase, Delivery, AfterSales · tactic: WarmOutreach, FreeContent, ColdOutreach, PaidAdvertising, LeadMagnet, CustomerReferral, AffiliateOrAgency |
| `CustomerRelationship` | `segmentId` (null = every segment), `relationshipKind` | PersonalAssistance, DedicatedPersonalAssistance, SelfService, AutomatedService, Community, CoCreation |
| `RevenueStream` | `segmentId` (null = every segment), `revenueKind`, `pricing` | revenueKind: Subscription, OneTimePurchase, UsageFee, Licensing, Advertising, MarketplaceCommission, Freemium · pricing: ListPrice, FeatureDependent, SegmentDependent, VolumeDependent, Negotiated, YieldManaged, RealTimeMarket, Auction |
| `KeyResource` | `resourceKind`, `owner` (null when nobody is named) | Physical, Intellectual, Human, Financial |
| `KeyActivity` | `activityKind`, `owner` (null when nobody is named) | Production, ProblemSolving, PlatformNetwork |
| `KeyPartner` | `partnershipKind`, `suppliesResourceId`, `performsActivityId` (both nullable) | StrategicAlliance, JointVenture, Supplier, Agency, Affiliate |
| `CostItem` | `costKind`, `driverKind` + `driverId` (together or neither), `amount` + `currency` (together or neither), `period` | costKind: Fixed, Variable · driverKind: Resource, Activity, Partner · period: OneOff, PerMonth, PerYear, PerUnit |

Example — a pain on the Customer Segments block, for the segment `CS-01` whose id you read:

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
pass untouched.

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
while a claim about the entry is under test. `delete_entry` removes the entry, every claim about it
and every contradiction it is a side of, and is refused once one of those claims has a started
experiment. Anything a person once believed is retired, not deleted.
