# Merchandising that reacts to live stock without manual intervention

## Purpose

Curated placements and campaign pages go stale within hours of a stock movement, and a retailer large enough to need them cannot re-merchandise by hand at that rate. The cost of not doing this is paid twice: promoting something unbuyable, and hiding something that came back into stock.

## Requirements

### Requirement: Merchandising that reacts to live stock without manual intervention

The system SHALL update a product's merchandising treatment — its stock badge, its eligibility for featured placement, and its position in curated collections — from stock movement, without a merchandiser editing the product.

#### Scenario: Crosses low threshold
- **GIVEN** a product whose available quantity falls below its category's low-stock threshold
- **WHEN** the stock movement is processed
- **THEN** the product carries a low-stock badge on listing, search and detail, and remains buyable

#### Scenario: Goes out of stock
- **GIVEN** a product holding a featured placement on a campaign page
- **WHEN** it goes out of stock
- **THEN** it is demoted out of the featured placement and a stocked alternative takes the position

#### Scenario: Comes back into stock
- **GIVEN** a product previously badged out of stock
- **WHEN** stock is replenished
- **THEN** the badge clears and the product regains normal eligibility without a manual edit

#### Scenario: Threshold changed by merchandiser
- **GIVEN** a category whose low-stock threshold is changed
- **WHEN** the change is saved
- **THEN** products in that category are re-evaluated against the new threshold

#### Scenario: Display lag is bounded
- **GIVEN** a shopper on a listing page
- **WHEN** a stock movement has just occurred
- **THEN** the listing may lag briefly, but the cart refuses a quantity the current stock cannot cover

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Stock movement subscription | `[MIDDLEWARE]` | Subscription on inventory change; the trigger, not a poll |
| Low-stock and out-of-stock badge | `[CACHED]` | Derived attribute written back on the product, so search can facet it |
| Featured placement eligibility | `[CACHED]` | Demote or drop on out-of-stock; promote a stocked substitute |
| Threshold configuration per category | `[MIDDLEWARE]` | A merchandiser owns the numbers, not a deploy |

## commercetools

**Entities:** `InventoryEntry`, `Channel`, `Product`, `ProductType`, `Subscription`, `Store`

**Verified API surface**

- (concept) InventoryEntry is the source of truth for stock, one per SKU and supply Channel pair, and direct updates through the Inventory API are strongly consistent and immediately visible — [docs](https://docs.commercetools.com/api/inventory-overview)
- (concept) Supply channels hold stock and distribution channels select price - they are distinct, and a Cart bound to a Store filters both using that Store's configured channels — [docs](https://docs.commercetools.com/api/inventory-overview)

**Constraints that change the design**

- Changes driven by ReserveOnCart, ReserveOnOrder or TrackOnly are eventually consistent: availableQuantity and quantityOnStock may take up to 10 seconds to reflect them, so a merchandising rule reading those fields is reacting to a value that can be seconds stale — [docs](https://docs.commercetools.com/api/inventory-overview)
- The ProductVariant availability field is explicitly for non-critical display such as a listing page and may lag real-time stock by a few seconds; anything that decides whether a sale can happen must query the InventoryEntry — [docs](https://docs.commercetools.com/api/inventory-overview)

**Modeling notes**

Drive this from a subscription on inventory change, never a catalog crawl: the crawl is what makes the reaction window minutes instead of seconds, and it scales with catalog size rather than with change rate. Write the outcome back as a product attribute rather than computing it in the storefront, because search has to facet and sort on it and a storefront-side rule cannot reorder a result set it did not build. Accept that badges are eventually consistent and that the cart is the place where truth is enforced - a shopper seeing a stale in-stock badge is a tolerable error, a shopper being sold stock that does not exist is not. Keep the thresholds in data a merchandiser can edit; a threshold in code becomes a deploy every season.

## commercetools skills

Load `commercetools-connect` before implementing this capability. Supporting: `commercetools-commerce-patterns`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-connect]`.

## Open questions

- What reaction window is acceptable between a stock movement and the badge changing?
- When a featured product is demoted, who picks the replacement — a rule, or a merchandiser's fallback list?

---

_Excluded for B2B2C: Local availability sort._
