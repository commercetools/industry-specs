# Self-service changes to an order before it is picked

## Purpose

Most change requests arrive within minutes of placement, and every one handled by an agent costs more than the margin on the order. The reason retailers hesitate is that a post-placement change is a financial change against a catalog that has since moved, so the shopper has to see the new total before agreeing to it.

## Requirements

### Requirement: Self-service changes to an order before it is picked

The system SHALL allow a shopper to amend or cancel their own order without contacting support while it remains within a stated change window and fulfillment has not started, showing the resulting price change before the amendment is committed.

#### Scenario: Amendment previewed then committed
- **GIVEN** an order inside its change window
- **WHEN** the shopper proposes removing a line
- **THEN** the resulting order and its new total are shown, and nothing changes until they confirm

#### Scenario: Prices moved since placement
- **GIVEN** an order whose catalog prices or promotions have changed since it was placed
- **WHEN** the shopper previews an amendment
- **THEN** the preview reflects current prices across the whole order, not only the amended line, and the difference is stated

#### Scenario: Window closed by picking
- **GIVEN** an order still inside its stated window whose picking has started
- **WHEN** the shopper tries to amend it
- **THEN** the change is refused, the reason is that fulfillment has begun, and support is offered

#### Scenario: Canceled in window
- **GIVEN** an order inside its window that has not been picked
- **WHEN** the shopper cancels it
- **THEN** the order is canceled and the refund is raised against the instruments that paid

#### Scenario: Amendment costs more
- **GIVEN** an amendment that raises the order total
- **WHEN** the shopper confirms it
- **THEN** the additional amount is collected before the amendment is applied

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Change window on the order | `[MIDDLEWARE]` | Stated at confirmation; ends when picking starts |
| Amendment preview | `[MIDDLEWARE]` | The order as it would be, before committing |
| Self-service cancellation | `[MIDDLEWARE]` | Whole order, within the same window |
| Refund or additional charge | `[MIDDLEWARE]` | Settled against the original instruments |
| Fulfillment interlock | `[MIDDLEWARE]` | Picking closes the window even if time remains |

## commercetools

**Entities:** `Order`, `OrderEdit`, `LineItem`, `Payment`, `Cart`, `InventoryEntry`

**Verified API surface**

- (rest) Order Edits are the required route for any post-placement change that affects the order total: create the OrderEdit, add stagedActions - which run immediately as a dry run returning the Order as if applied - review, then Apply — [docs](https://docs.commercetools.com/api/carts-orders-overview)
- (concept) The OrderEditApplied message carries the before and after net total, gross total and tax portion, and is the audit record of what an amendment did to the money — [docs](https://docs.commercetools.com/api/carts-orders-overview)

**Constraints that change the design**

- An Order Edit can only be created when the InventoryMode of the Order and its Line Items is None, because the Order Edits API does not track or reserve inventory - so an order placed under ReserveOnCart cannot be edited at all — [docs](https://docs.commercetools.com/api/carts-orders-overview)
- Applying an Order Edit recalculates the whole Order against the current catalog, prices and promotions, and removes discounts that are no longer valid - so an edit made some time after placement can move lines the shopper never touched — [docs](https://docs.commercetools.com/api/carts-orders-overview)
- Re-evaluation of Shipping Methods and shipping rates happens only when the change is made through an Order Edit, not through a direct order update - so a direct address update leaves a stale shipping charge — [docs](https://docs.commercetools.com/api/carts-orders-overview)
- Inventory quantity limits are not enforced during Order Edits, so a purchase cap respected at placement can be exceeded by an amendment unless you re-check it yourself — [docs](https://docs.commercetools.com/api/inventory-overview)

**Modeling notes**

The inventory mode decision is made long before this capability is built and it decides whether this capability can exist: Order Edits require InventoryMode None, so a project that reserves stock on the cart to prevent overselling cannot also offer self-service amendment on those orders. That is a genuine either-or, and the right answer differs between a limited release and an everyday basket - it can be varied per line, which is usually the way out. Always show the dry-run preview rather than a locally computed total, because apply recalculates the entire order: prices that moved and promotions that expired will change lines the shopper did not touch, and a preview computed in your own code will not show that. The window has two ends and fulfillment is the authoritative one; a purely time-based window will eventually let someone amend an order already in a picker's hands.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-storefront`, `commercetools-connect`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- How long is the change window, and does it differ by fulfillment method?
- Which system signals that picking has started, and how quickly?
- Are amendments allowed to increase the order, or only reduce and cancel?
