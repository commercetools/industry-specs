# Bundles priced as one line and adapting to component stock

## Purpose

Coordinated-outfit and starter-kit selling raises basket value, but a bundle is only worth offering if it can be honoured: half a bundle is a return. The components have to stay individually visible for picking, stock and returns while the shopper sees and pays one price.

## Requirements

### Requirement: Bundles priced as one line and adapting to component stock

The system SHALL add a bundle to the cart as one priced line carrying its components as individually tracked lines, and refuse or re-offer the bundle when a component is unavailable rather than adding an incomplete one.

#### Scenario: All components available
- **GIVEN** a bundle whose components are all in stock
- **WHEN** the shopper adds the bundle
- **THEN** one priced bundle line and one zero-priced line per component are added, each component line naming its parent

#### Scenario: Component unavailable substituted
- **GIVEN** a bundle slot with an ordered list of alternates and its first choice out of stock
- **WHEN** the shopper adds the bundle
- **THEN** the first available alternate fills the slot and the shopper is told which component changed

#### Scenario: No alternate available
- **GIVEN** a bundle slot whose every alternate is out of stock
- **WHEN** the shopper adds the bundle
- **THEN** nothing is added, and the reason names the unavailable slot rather than failing silently

#### Scenario: Quantity changed
- **GIVEN** a bundle already in the cart
- **WHEN** the shopper changes the bundle quantity
- **THEN** the component line quantities move with it and the cart total reflects the bundle price times the new quantity

#### Scenario: Bundle removed
- **GIVEN** a bundle in the cart
- **WHEN** the shopper removes the bundle line
- **THEN** every component line belonging to it is removed with it, leaving no orphans

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Bundle definition with component list | `[CACHED]` | Product type attribute holding component SKUs and quantities |
| Bundle price as one line | `[MIDDLEWARE]` | Bundle line carries the price; components carry zero |
| Component lines linked to their parent | `[MIDDLEWARE]` | Custom field on each component line naming the bundle line |
| Component substitution on unavailability | `[MIDDLEWARE]` | Ordered alternates per slot, evaluated at add time |
| Bundle quantity change | `[MIDDLEWARE]` | Parent and every component move together or not at all |

## commercetools

**Entities:** `ProductType`, `Product`, `Cart`, `LineItem`, `Type`, `InventoryEntry`

**Verified API surface**

- (update-action) Add the bundle with Add LineItem using externalPrice set to the bundle price, quantity 1 and inventoryMode None, then add each component as its own Line Item with externalPrice 0.00 and inventoryMode TrackOnly or ReserveOnOrder — [docs](https://docs.commercetools.com/tutorials/product-bundles)
- (concept) A parentLineItemId Custom Field of type String on the Line Item is what ties a component line back to the bundle line it belongs to — [docs](https://docs.commercetools.com/tutorials/product-bundles)

**Constraints that change the design**

- commercetools has no native bundle resource. The supported pattern models the component list as an attribute on the bundle's ProductType, and the cart composition is middleware logic you own — [docs](https://docs.commercetools.com/tutorials/product-bundles)
- Bundle quantity changes and removals are not handled by the platform: the middleware must validate component availability first, then change the parent and iterate every component line, or the cart drifts out of step — [docs](https://docs.commercetools.com/tutorials/product-bundles)
- An externally priced Line Item is not available to Product Search or Product Projection Search for filtering, faceting or sorting by price, so a bundle priced this way cannot be sorted by price alongside ordinary products — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)

**Modeling notes**

The bundle line carries the money and the component lines carry the stock: that split is what lets one price reach the shopper while the warehouse still sees real SKUs. It also means the bundle line must be inventoryMode None - it is not a thing that exists - while the components reserve or track normally. Decide early whether bundle contents are fixed or shopper-chosen, because a shopper-chosen bundle needs the price recomputed per configuration and cannot use a single stored bundle price. The price-search limitation is the one that surprises people: if bundles must appear in price-sorted listings, model the bundle as a real product with its own price and treat the components as fulfillment detail instead.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Are bundle contents fixed by a merchandiser, or chosen by the shopper at add time?
- When a component is substituted, does the bundle price hold, or move with the substitute?
- Can a component be returned on its own, or only the whole bundle?
