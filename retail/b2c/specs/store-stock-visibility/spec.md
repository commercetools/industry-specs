# Stock shown per store before the shopper commits to one

## Purpose

Store stock is the figure a shopper trusts least and needs most, because being wrong costs them a wasted journey rather than a refresh. Trust here comes less from the number being perfect than from the shopper knowing how fresh it is, since shop-floor stock is provably imperfect.

## Requirements

### Requirement: Stock shown per store before the shopper commits to one

The system SHALL report availability for a product at each nominated store separately from online availability, and state how current that figure is, so a shopper can decide where to buy before committing to a fulfillment method.

#### Scenario: Per store availability
- **GIVEN** a shopper who has chosen a store
- **WHEN** they view a product
- **THEN** availability at that store is shown alongside online availability, each labeled

#### Scenario: Freshness stated
- **GIVEN** store availability derived from a periodic feed
- **WHEN** the figure is shown
- **THEN** how recently it was confirmed is shown with it, rather than being presented as live

#### Scenario: Listing narrowed to a store
- **GIVEN** a shopper who has chosen a store
- **WHEN** they narrow a listing to that store's stock
- **THEN** only products the store holds are listed, and the narrowing is visible and reversible

#### Scenario: Unavailable here available elsewhere
- **GIVEN** a product with no stock at the chosen store
- **WHEN** the shopper views it
- **THEN** that is stated plainly, with the nearest store holding it or the online option offered

#### Scenario: Display figure is not the contract
- **GIVEN** a store figure that has gone stale since it was shown
- **WHEN** the shopper tries to reserve or buy against it
- **THEN** the reservation is evaluated against current stock and refused if it cannot be honoured

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Stock held per location | `[MIDDLEWARE]` | One stock record per SKU and supply location |
| Nearby store resolution | `[MIDDLEWARE]` | From a chosen store or an entered location |
| Freshness statement | `[MIDDLEWARE]` | When the figure was last confirmed |
| Banded rather than exact counts | `[MIDDLEWARE]` | Low stock beats a precise number that will be wrong |
| Availability filter on listings | `[MIDDLEWARE]` | Narrow a listing to what the chosen store holds |

## commercetools

**Entities:** `InventoryEntry`, `Channel`, `Store`, `ProductVariant`, `Cart`

**Verified API surface**

- (concept) An InventoryEntry represents stock for a single SKU in an optional supply Channel, with exactly one entry per SKU and supply Channel pair - so a store is a supply Channel and per-store stock is one entry per store — [docs](https://docs.commercetools.com/api/inventory-overview)
- (concept) To fulfill one SKU from several locations, add multiple Line Items to the Cart, each naming a different supply channel — [docs](https://docs.commercetools.com/api/inventory-overview)

**Constraints that change the design**

- Supply channels hold stock and distribution channels select price; they are separate, and a Cart bound to a Store filters available stock and prices using that Store's configured channels — [docs](https://docs.commercetools.com/api/inventory-overview)
- The ProductVariant availability field is documented for non-critical checks such as showing a quantity on a listing page and may lag real-time stock by seconds; anything deciding a sale must query the InventoryEntry directly — [docs](https://docs.commercetools.com/api/inventory-overview)
- Fetching every InventoryEntry for a SKU to answer a store question is called out as the pattern to avoid - query only the supply channel availability or the specific stock detail needed — [docs](https://docs.commercetools.com/api/inventory-overview)

**Modeling notes**

One supply channel per store, one stock record per SKU and store. The scale question comes first: a large chain multiplied by a large assortment is a lot of stock records, and the query pattern - never fetch all entries for a SKU - matters more here than anywhere else in the catalog. Whether store stock is genuinely live or a periodic feed is a business fact, not an implementation detail, so surface the freshness rather than hiding a batch behind a live-looking figure. Prefer bands to exact counts for shop-floor stock: an exact number invites a trust it cannot support, and a band survives the inaccuracy that physical stock always has. Keep the display figure and the commitment check separate - the listing may use the cached availability view, but reserving or buying must read the stock record.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-platform`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Is store stock live from the point of sale, or a periodic feed, and at what cadence?
- What is the acceptable error rate on a store figure before it should not be shown at all?
- Should stock held for online fulfillment be visible as store stock, or ring-fenced?
