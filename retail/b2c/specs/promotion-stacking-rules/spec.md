# Overlapping promotions resolving to one predictable outcome

## Purpose

Retail runs many overlapping offers at once, and the margin question is not what each offer does but what happens when four of them land on one basket. If that outcome is not deterministic and explainable, finance cannot forecast it, support cannot defend it, and shoppers assume the cheaper reading is the correct one.

## Requirements

### Requirement: Overlapping promotions resolving to one predictable outcome

The system SHALL resolve every promotion that qualifies for a cart into one deterministic set of applied discounts under a stated combination rule, and show the shopper which promotions applied and which were excluded.

#### Scenario: Two offers both apply
- **GIVEN** a cart qualifying for both a product-level and a basket-level offer under a stacking rule
- **WHEN** the cart is recalculated
- **THEN** both apply in the configured order and the cart names each one against what it discounted

#### Scenario: Best of the two
- **GIVEN** the same cart under a best-deal rule
- **WHEN** the cart is recalculated
- **THEN** only the discount type giving the lower total applies, and the cart states which one won

#### Scenario: Exclusive offer stops the rest
- **GIVEN** a cart qualifying for an exclusive offer and two ordinary ones
- **WHEN** the exclusive offer applies
- **THEN** no further basket-level offer is applied after it, and the excluded ones are named

#### Scenario: Order changes the total
- **GIVEN** a cart qualifying for both a percentage and a fixed-amount basket offer
- **WHEN** the cart is recalculated
- **THEN** the offers apply in rank order and the resulting total is reproducible for the same cart

#### Scenario: Rejected code explains itself
- **GIVEN** a shopper entering a code that does not qualify for their cart
- **WHEN** the code is submitted
- **THEN** the cart is unchanged and the reason it did not apply is stated

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Combination rule for the business | `[MIDDLEWARE]` | Project-wide: stack the discount types, or take the best of them |
| Ordered promotion evaluation | `[MIDDLEWARE]` | Rank; order changes the total when mixing percent and amount |
| Exclusivity for an offer | `[MIDDLEWARE]` | Stop further discounts after this one applies |
| Applied and excluded shown on cart | `[MIDDLEWARE]` | Named per line and at basket level |
| Code entry with a stated reason | `[MIDDLEWARE]` | A rejected code says why, not just "invalid" |

## commercetools

**Entities:** `CartDiscount`, `ProductDiscount`, `DiscountCode`, `Cart`, `Project`

**Verified API surface**

- (concept) The Project-level discountCombinationMode in DiscountsConfiguration decides whether Product Discounts and Cart Discounts both apply (Stacking) or only the type giving the better deal applies (BestDeal); set it via Set Discounts Configuration on the Project — [docs](https://docs.commercetools.com/learning-price-and-discount-your-products/cart-discounts/how-cart-discounts-work)
- (concept) Under BestDeal the platform calculates the cart total twice, once per discount type, applies whichever yields the lower total, and reports the outcome in the discountTypeCombination field — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)

**Constraints that change the design**

- discountCombinationMode is project-wide and affects every Cart in the Project; it cannot be varied per campaign, per banner or per cart, and it defaults to Stacking when unset — [docs](https://docs.commercetools.com/learning-price-and-discount-your-products/cart-discounts/how-cart-discounts-work)
- Cart Discount rank sets processing order and the order changes the final total when a percentage and a fixed amount both apply - 10% then $5 off $100 gives $85, the reverse gives $85.50 - so rank is a pricing decision, not a display preference — [docs](https://docs.commercetools.com/learning-price-and-discount-your-products/cart-discounts/how-cart-discounts-work)
- A Cart Discount's stackingMode of StopAfterThisDiscount prevents further Cart Discounts, but a Total Price discount still applies last regardless of sort order and ignores other discounts' StopAfterThisDiscount - so exclusivity is not absolute — [docs](https://docs.commercetools.com/learning-price-and-discount-your-products/cart-discounts/how-cart-discounts-work)
- Cart Discounts are subject to a per-Project limit, and Cart Discounts attached to Discount Codes are counted under a separate limit - a campaign calendar has to be planned against both — [docs](https://docs.commercetools.com/learning-price-and-discount-your-products/cart-discounts/how-cart-discounts-work)

**Modeling notes**

Choose the combination mode once, deliberately, and write down why: it is project-wide, so a single campaign that needs the other mode cannot have it, and discovering that mid-season is expensive. Rank is where the real behavior lives - treat the rank ladder as a maintained artefact with reserved bands per campaign class, not a number each merchandiser picks, because two offers sharing a rank make the total depend on evaluation order nobody controls. The total-price exception to StopAfterThisDiscount is the trap: an offer marketed as "cannot be combined" can still be followed by a basket-total discount, so if exclusivity must be absolute, enforce it in the qualifying predicate rather than the stacking mode. Both discount limits are project limits; a retailer running hundreds of concurrent offers should plan the calendar against them before the first campaign is built.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Does any campaign require the combination mode the project does not use? If so, how is it handled?
- How many concurrent offers does a peak trading period need, and does that fit the project limits?
- Should an excluded promotion be shown to the shopper, or only the applied ones?
