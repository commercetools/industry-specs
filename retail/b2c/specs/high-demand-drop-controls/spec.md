# Limited releases that hold their limits under a demand spike

## Purpose

A timed release concentrates a week of traffic into minutes, and it is the one event where overselling is both most likely and least forgivable: every oversold order is a cancellation against the product the retailer most wanted to be trusted on. Limits enforced only in the storefront are not limits, because the spike is exactly when the storefront is bypassed.

## Requirements

### Requirement: Limited releases that hold their limits under a demand spike

The system SHALL enforce the per-shopper purchase limit and the real stock ceiling for a limited release at the moment an order is created, so that a demand spike cannot produce more confirmed orders than there is stock to honour.

#### Scenario: Cap enforced on add
- **GIVEN** a limited release with a maximum quantity per shopper
- **WHEN** a shopper tries to add more than the cap
- **THEN** the cart update is refused and the cap is stated, rather than silently trimming the quantity

#### Scenario: Cap enforced at order
- **GIVEN** a cart assembled before the cap was tightened
- **WHEN** the shopper submits the order
- **THEN** order creation fails against the current cap, so the stale cart cannot bypass it

#### Scenario: Stock exhausted mid checkout
- **GIVEN** the release sells out while a shopper is in checkout without a reservation
- **WHEN** they submit the order
- **THEN** placement is refused with the reason, and no order is confirmed that stock cannot honour

#### Scenario: Not open yet
- **GIVEN** a release whose window has not opened
- **WHEN** a shopper reaches the product by a direct link
- **THEN** it is visible but not purchasable, and the opening time is stated

#### Scenario: Split lines do not defeat the cap
- **GIVEN** a shopper who has the same item on two separate cart lines
- **WHEN** the cart is evaluated
- **THEN** the combined quantity is treated as one shopper's purchase and the cap still holds

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Per-line purchase cap | `[MIDDLEWARE]` | Maximum cart quantity held against the stock record |
| Stock reservation during checkout | `[MIDDLEWARE]` | Reserve on cart, so an abandoned cart releases |
| Release window | `[MIDDLEWARE]` | Not purchasable before it opens, regardless of route in |
| Honest sold-out state | `[MIDDLEWARE]` | Distinguishes sold out from reserved-by-others |
| Automation and resale defense | `[MIDDLEWARE]` | External; commercetools carries the outcome, not the decision |

## commercetools

**Entities:** `InventoryEntry`, `Cart`, `LineItem`, `Order`, `Channel`, `Product`

**Verified API surface**

- (concept) InventoryEntry carries minCartQuantity and maxCartQuantity, set at creation or through the Set Inventory Limits update action, and they are enforced by the platform rather than by the storefront — [docs](https://docs.commercetools.com/api/inventory-overview)

**Constraints that change the design**

- Quantity limits are enforced on Cart creation, on Cart update and on Order creation from a Cart - including for a cart assembled before the limit changed - but they are NOT enforced on Order Edits or Order Imports — [docs](https://docs.commercetools.com/api/inventory-overview)
- Each Line Item is evaluated independently against the limit, so two Line Items for the same SKU distinguished by Custom Fields each get the full allowance - this is documented as the workaround for exceeding a cap, and it is therefore also the hole in it — [docs](https://docs.commercetools.com/api/inventory-overview)
- Quantity limits are independent of InventoryMode, so a cap holds under None, TrackOnly, ReserveOnOrder and ReserveOnCart alike; the inventory mode governs only when stock is reserved — [docs](https://docs.commercetools.com/api/inventory-overview)
- ReserveOnCart reservations are guaranteed at the moment they are made even though availableQuantity is only eventually consistent, so a reservation-based release does not oversell because of the up-to-10-second reporting lag — [docs](https://docs.commercetools.com/api/inventory-overview)

**Modeling notes**

Put the cap on the InventoryEntry rather than in storefront logic: the platform then enforces it on cart write and on order creation, which is the only place enforcement survives a spike. Know the two holes before you rely on it - per-line evaluation means the same SKU on two lines gets the allowance twice, so if a shopper can create a second line for the same SKU you must collapse or reject it yourself; and Order Edits skip the check entirely, so a post-placement edit can exceed a cap the original order respected. ReserveOnCart is the right mode for a timed release because reservations are honoured immediately, but it forecloses Order Edits later, which conflicts with any grace-window change flow on the same order. Decide which of the two the release needs and record it. Queueing and automation defense sit in front of the platform; commercetools records who succeeded, it does not decide who gets through.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-platform`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Is the cap per order, per account or per household, and what identifies a household?
- Does this release need Order Edits afterwards? If so, ReserveOnCart cannot be used on it.
- Who owns queueing and automation defense in front of the storefront, and what does it hand over?
