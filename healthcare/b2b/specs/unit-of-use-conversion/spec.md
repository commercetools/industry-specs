# Ordered by the pack, used and counted by the single unit

## Purpose

Clinical supplies are bought in one unit and used in another. A ward consumes single items and counts them that way; the catalog sells cases of boxes of those items; and the number typed into a quantity box means whichever of the two the person typing had in mind. This is not a presentation nicety. Ordering a hundred cases when a hundred pieces were wanted is a well-known, entirely routine error, and its consequences run all the way down the line — the warehouse flags an implausible order and it stalls, or it does not flag it and a storeroom receives a pallet it must now return. Nor is it symmetric: the small mistake is an inconvenience and the large one ties up a budget. What removes the error is showing both numbers at the same time, so the quantity can be checked against the intent while it is still being typed. The conversion then has to travel with the order, because every system downstream counts in a different unit — the warehouse picks cases, the ward counts pieces, the contract prices per piece and the invoice bills per case — and each conversion done independently is a chance for the four to disagree.

## Requirements

### Requirement: Ordered by the pack, used and counted by the single unit

The system SHALL name the selling unit on every quantity for an item bought in a unit different from the one it is consumed in, and state how many use-units that quantity delivers before the order is placed.

#### Scenario: Both figures shown while ordering
- **GIVEN** an item sold by the case and used by the piece
- **WHEN** a quantity is entered
- **THEN** the selling unit is named and the number of pieces that quantity delivers is shown alongside it

#### Scenario: Price comparable across pack sizes
- **GIVEN** the same item offered in two pack sizes
- **WHEN** they are compared
- **THEN** a price per use-unit is shown for both, not only the price per pack

#### Scenario: Minimum expressed in the selling unit
- **GIVEN** an item with a minimum order quantity
- **WHEN** less than the minimum is requested
- **THEN** the refusal states the minimum in the same unit the quantity was entered in

#### Scenario: Conversion present on the order
- **GIVEN** a placed order for goods bought by the case
- **WHEN** it is read by fulfillment, by the receiving location or by billing
- **THEN** the selling unit, the conversion and the resulting use-unit count are all present on the line

#### Scenario: Selling unit changed before checkout
- **GIVEN** a line ordered by the case
- **WHEN** the buyer switches it to single pieces
- **THEN** the quantity, the price and the stated use-unit count are all recomputed together

#### Scenario: Implausible quantity is questioned
- **GIVEN** a quantity far outside the normal range for its selling unit
- **WHEN** it is entered
- **THEN** it is questioned before the order is placed rather than after the warehouse sees it

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Selling unit named on the quantity | `[CACHED]` | Case, box or single piece, stated rather than assumed |
| Conversion to the use-unit held as catalog data | `[CACHED]` | One number, maintained once, not recomputed per integration |
| Both figures shown together while ordering | `[MIDDLEWARE]` | Ordered quantity and the pieces it delivers, side by side |
| Price expressed per use-unit as well as per selling unit | `[MIDDLEWARE]` | How two pack sizes are actually compared |
| Minimums and multiples stated in the selling unit | `[CACHED]` | A minimum in the wrong unit is worse than none |
| Conversion carried onto the order | `[MIDDLEWARE]` | So the warehouse, the ward and the invoice do not each derive their own |
| Switching the selling unit reprices and re-explains the line | `[MIDDLEWARE]` | Same goods, different unit, different number and different price |

## commercetools

**Entities:** `Product`, `ProductVariant`, `ProductType`, `StandalonePrice`, `PriceTier`, `Cart`, `LineItem`, `Order`, `InventoryEntry`

**Verified API surface**

- (concept) The variant-based approach models the Product as variants that each represent a discrete quantity — a pack of 10, 25 or 50 — each carrying a normal Standalone Price for its increment, which keeps pricing entirely within native price selection — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/configure-b2b-pricing/modeling-unit-based-pricing)
- (concept) The smallest-unit approach prices the smallest indivisible unit as a Standalone Price and has a Custom Field or middleware layer multiply up to the display unit, with the Cart carrying the order in the smallest unit — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/configure-b2b-pricing/modeling-unit-based-pricing)
- (concept) A Standalone Price can carry Price tiers through its tiers field, each with a minimumQuantity, so a volume break is a property of the Price and participates in price selection like any other Price — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/configure-b2b-pricing/company-specific-pricing)

**Constraints that change the design**

- commercetools has no native unit-of-measure price type and no unit-of-measure resource to configure; unit-based pricing is a modeling exercise and is primarily a display concern, because money is stored as integers — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/configure-b2b-pricing/modeling-unit-based-pricing)
- Price tiers are evaluated against the Line Item quantity, which is counted in whatever unit the variant represents — so a tier threshold set against pieces will not fire for a variant that is a case, and the two must be authored in the same unit — [docs](https://docs.commercetools.com/learning-price-and-discount-your-products/price-calculation/price-selection)

**Modeling notes**

A variant per selling unit is the pattern that works, and it works because it keeps the whole problem inside native price selection: the case and the single piece are two SKUs, each with its own price, its own inventory and its own minimum, and nothing has to be converted at pricing time. Carry the conversion factor as a Product Type Attribute on each variant so the storefront can display the use-unit count without a second lookup, and copy both the factor and the resulting count onto the Line Item at add time — the order is the contract between four systems that each count differently, and a factor that lives only in the catalog will be read at a different version by each of them. Watch the quantity unit in every rule you write: price tiers, inventory limits and minimum quantities are all evaluated against the Line Item quantity, which is cases for a case variant, so a threshold authored in pieces silently never fires. This is a different problem from goods sold by variable weight, where the quantity is continuous and the billed amount is settled by the weighing system after shipment. Here the conversion is exact and known in advance, so there is no reason for the order to be provisional — and treating it as though it were introduces a reconciliation step nobody needs.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-platform`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Which unit does the buying contract price in, and does it match the unit the catalog sells in?
- Who maintains the conversion factor, and what happens to open orders when a pack size changes?
- What counts as an implausible quantity, and is that threshold per item or per account?
- Are part-packs ever supplied, and if so how is the residue priced and counted?
