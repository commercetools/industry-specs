# Collection orders bound to a store that can actually fulfill them

## Purpose

Collection converts a delivery cost into a store visit, which is why retailers want it — but a collection order the store cannot fill is worse than no collection at all, because the shopper has already traveled. The store commitment therefore has to be part of the order, not a routing decision taken afterwards.

## Requirements

### Requirement: Collection orders bound to a store that can actually fulfill them

The system SHALL bind a collection line to the specific store that will fulfill it and confirm that store's stock at order placement, so a collection order is never confirmed against a store that cannot supply it.

#### Scenario: Bound to a store
- **GIVEN** a shopper choosing collection at a named store
- **WHEN** the order is placed
- **THEN** the line records that store as its fulfilling location and the order carries it downstream

#### Scenario: Stock gone at placement
- **GIVEN** a collection line whose store stock has been sold on the shop floor since selection
- **WHEN** the shopper submits the order
- **THEN** placement is refused for that line with the reason, and alternatives are offered rather than the order silently rerouting

#### Scenario: Ready when the store says so
- **GIVEN** a placed collection order
- **WHEN** the store confirms the goods are picked and waiting
- **THEN** the shopper is told it is ready, and not before

#### Scenario: Collected by someone else
- **GIVEN** an order the shopper has nominated another person to collect
- **WHEN** that person arrives and identifies themselves
- **THEN** handover is permitted and the order records who collected it

#### Scenario: Never collected
- **GIVEN** a ready order that passes its holding period
- **WHEN** the period expires
- **THEN** the stated outcome follows — return to sale, refund, or extension — and the shopper is told which

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Collection store on the line | `[MIDDLEWARE]` | The fulfilling location, carried to the order |
| Stock confirmed at placement | `[MIDDLEWARE]` | Re-checked against that store, not the network |
| Ready-for-collection notice | `[MIDDLEWARE]` | Sent when the store confirms, not at placement |
| Collector identity and authority | `[MIDDLEWARE]` | Who may collect, and what they must present |
| Uncollected order handling | `[MIDDLEWARE]` | A stated holding period and what happens after |

## commercetools

**Entities:** `Cart`, `LineItem`, `Order`, `Channel`, `Store`, `InventoryEntry`, `ShippingMethod`

**Verified API surface**

- (concept) A store is a supply Channel and holds its own InventoryEntry per SKU, so binding a line to a collection store means naming that supply channel on the Line Item — [docs](https://docs.commercetools.com/api/inventory-overview)
- (concept) Multiple Line Items each naming a different supply channel is the documented way to fulfill from more than one location, which is what lets a collection line and a delivery line coexist — [docs](https://docs.commercetools.com/api/inventory-overview)
- (concept) A collection point is expressed as a destination in the Cart's itemShippingAddresses with its own Shipping Method, so collection and delivery share one shipping mechanism instead of needing a parallel one — [docs](https://docs.commercetools.com/api/shipping-delivery-overview)

**Constraints that change the design**

- Reservation-driven availability is eventually consistent with up to a 10 second lag on availableQuantity, but reservations themselves are guaranteed at the time they are made - so a collection reservation is safe even though the reported figure trails it — [docs](https://docs.commercetools.com/api/inventory-overview)
- Store stock also moves on the shop floor, and those movements do not pass through the commerce API - so placement must re-read the store's InventoryEntry rather than trusting a figure captured when the shopper chose the store — [docs](https://docs.commercetools.com/api/inventory-overview)

**Modeling notes**

The store is a supply channel on the line, and that is what makes the commitment durable: routing decided later, outside the order, is exactly how a shopper ends up at the wrong store. Re-check at placement without exception - shop-floor sales never touch the commerce API, so store stock is the least trustworthy figure in the system and the one this flow depends on. Whether to reserve at selection or only check at placement is a real trade-off: reserving protects the shopper but strands stock the shop floor could have sold, and reserving on the cart also forecloses Order Edits on the resulting order. Decide it per business and write it down. The ready notice belongs to the store's picking system, not to order placement; sending it at placement is the single most common way this capability loses shopper trust.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-connect`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Is stock reserved when the shopper chooses collection, or only checked at placement?
- How long is an order held before it is returned to sale, and is the shopper refunded automatically?
- What identification is required at handover, and does a nominated collector need to be named in advance?
