# Each discount recording who funds it, so it can be claimed back

## Purpose

In a distribution network two parties run promotions on the same catalog, and they pay for them out of different pockets: the reseller's own offer is margin it chose to give away, while the manufacturer's program is money the reseller is owed back. The reseller's incentive to promote at all depends on being able to reclaim the second kind, and a reclaim built by recomputing discounts after the fact never reconciles — the prices, programs and eligibility have all moved on. The funding attribution has to be captured at the moment the discount is applied, because that is the only moment when it is unambiguous.

## Requirements

### Requirement: Each discount recording who funds it, so it can be claimed back

The system SHALL record on every discounted line which party funds that discount and under which program, so a reseller can claim back the manufacturer-funded amounts from the recorded figures without recalculating them.

#### Scenario: Funding recorded at application
- **GIVEN** a line discounted under a manufacturer-funded program
- **WHEN** the order is placed
- **THEN** the line records the funding party, the program and the funded amount as applied

#### Scenario: Both parties fund one line
- **GIVEN** a line carrying both a manufacturer-funded and a reseller-funded discount
- **WHEN** the order is placed
- **THEN** the two funded amounts are recorded separately and neither is inferable only as a remainder

#### Scenario: Claim reads the order
- **GIVEN** a period of placed orders containing manufacturer-funded discounts
- **WHEN** the reseller assembles a claim
- **THEN** the claimable total is taken from the amounts recorded on those orders, not recalculated from current prices or programs

#### Scenario: Program not offered to this reseller
- **GIVEN** a manufacturer program restricted to a subset of resellers
- **WHEN** a buyer of an excluded reseller reaches a qualifying line
- **THEN** the program does not apply and no funded amount is recorded against it

#### Scenario: Program withdrawn after the order
- **GIVEN** an order placed while a program was live, and the program since withdrawn
- **WHEN** the reseller assembles a claim covering that order
- **THEN** the recorded amount remains claimable, because the record is of what was applied rather than of what currently applies

#### Scenario: Buyer sees the program
- **GIVEN** an order line discounted under a named program
- **WHEN** the buyer reviews their order detail
- **THEN** the program name and the discounted amount are shown on that line

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Funding party recorded per applied discount | `[MIDDLEWARE]` | The manufacturer, the reseller, or shared |
| Program identity and funded amount per line | `[MIDDLEWARE]` | Name and amount, not a percentage to re-derive |
| Program name and amount on the buyer's order detail | `[MIDDLEWARE]` | The buyer sees what they saved and under what offer |

## commercetools

**Entities:** `CartDiscount`, `DirectDiscount`, `DiscountCode`, `Cart`, `LineItem`, `Order`, `Type`, `CustomObject`

**Verified API surface**

- (concept) A DirectDiscount can be applied to a cart as a whole or to specific Line Items, which is how an externally decided funded amount is placed on a line rather than expressed as a rule the platform re-evaluates — [docs](https://docs.commercetools.com/api/quotes-overview)
- (concept) Custom Fields declared through a Type can be attached to the Cart and its Line Items and are carried onto the Order, which is where the funding party, program identity and funded amount are recorded at the moment of application — [docs](https://docs.commercetools.com/api/projects/carts)
- (concept) Subscriptions send notifications to a message queue when a resource is modified and are used to trigger asynchronous background processes, which is the mechanism for feeding placed orders to the settlement system that owns the claim — [docs](https://docs.commercetools.com/api/projects/subscriptions)

**Constraints that change the design**

- Cart Discounts are evaluated in a fixed order — line item targets, gift line items, non-participating Direct Discounts, shipping cost, then cart total price — and sort order and stacking mode take effect only among discounts with those targets, so two parties' programs on one line resolve inside one ordering the platform owns — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)
- Under best-deal selection the platform calculates the cart total with the Product Discount and with the Cart Discounts and applies whichever gives the lower total, and in doing so an individual Line Item's price may increase relative to the other calculation — so a per-line funded amount cannot be derived from the final total alone — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)
- By default Direct Discounts and Cart Discounts are mutually exclusive on a cart: when a Direct Discount is present, matching Cart Discounts are ignored and Discount Codes cannot be applied — so mixing a directly placed funded amount with rule-based programs on one cart suppresses the latter — [docs](https://docs.commercetools.com/api/quotes-overview)

**Modeling notes**

The whole capability rests on recording the funded amount at application time, and the reason is a platform behavior that surprises people: best-deal selection computes two whole-cart outcomes and picks one, and an individual line's price can be higher in the winning outcome than in the losing one. A funded amount reverse-engineered from the final total therefore does not correspond to any discount that was applied. Write it down instead — a Custom Field per discounted line holding party, program and amount — at the point the discount goes on. Be careful combining mechanisms: placing a Direct Discount on the cart suppresses matching Cart Discounts and blocks Discount Codes entirely, so a design that uses Direct Discounts for manufacturer-funded amounts and Cart Discounts for the reseller's own offers will silently drop the reseller's. Pick one mechanism for both parties and distinguish them by the recorded attribution. Settlement itself belongs to the manufacturer's finance system; feed it orders through a Subscription and hold only the claim's state here.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-connect`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Who owns settlement, and what claim states must the portal reflect?
- When both parties fund one line, is the split agreed per program or per transaction?
- What is the claim window, and what happens to a claimable amount when an order is later returned or credited?

---

_Excluded for B2B2B: Program eligibility by reseller or reseller group; Reseller's own offers layered over the manufacturer's; Claimable amounts assembled per period; Claim submitted and its state tracked._
