# A service agent acting on a shopper's cart, on the record

## Purpose

Support resolves faster when an agent can act on the actual cart rather than talking a shopper through it, and goodwill gestures are a normal part of retail service. Both are also the most direct route to unpriced giveaways and account takeover, so the same mechanism has to carry the limits and the audit trail.

## Requirements

### Requirement: A service agent acting on a shopper's cart, on the record

The system SHALL let an authorized service agent act on a named shopper's cart within the limits granted to them, recording on every resulting change which agent made it and under what authority.

#### Scenario: Agent acts on a cart
- **GIVEN** an authorized agent and a shopper who has asked for help
- **WHEN** the agent opens that shopper's cart and adds an item
- **THEN** the change lands on the shopper's own cart and records the agent who made it

#### Scenario: Adjustment within authority
- **GIVEN** an agent whose authority allows an adjustment up to a set limit
- **WHEN** they adjust a line within that limit and give a reason
- **THEN** the adjustment applies, and the agent, the reason and the original price are recorded

#### Scenario: Adjustment beyond authority
- **GIVEN** an agent attempting an adjustment above their limit
- **WHEN** they submit it
- **THEN** it is refused, the limit is stated, and an escalation path is offered rather than the cart being changed

#### Scenario: Shopper completes it
- **GIVEN** a cart an agent has assisted with
- **WHEN** the shopper returns to their own session
- **THEN** they see the assisted cart, including the adjustment and the reason, and can complete it themselves

#### Scenario: Assistance ends
- **GIVEN** an assisted session that has ended or timed out
- **WHEN** the agent attempts a further change
- **THEN** it is refused, and a new assisted session must be established

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Agent acting as a named shopper | `[MIDDLEWARE]` | An explicit act-as, never the shopper's credentials |
| Bounded price adjustment | `[MIDDLEWARE]` | Capped by amount or percentage per agent |
| Reason recorded with every change | `[MIDDLEWARE]` | From a controlled list, not free text alone |
| Shopper sees the assisted cart | `[MIDDLEWARE]` | The same cart, so the shopper can complete it |
| Session end and handback | `[MIDDLEWARE]` | Assistance expires; it does not linger |

## commercetools

**Entities:** `Cart`, `LineItem`, `Customer`, `Type`, `Order`

**Verified API surface**

- (update-action) A price adjustment on a line is expressed by setting the Line Item priceMode to ExternalPrice and supplying externalPrice through Add LineItem or Set LineItem Price; totalPrice is then externalPrice multiplied by quantity — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)
- (concept) Set LineItem TotalPrice with priceMode ExternalTotal sets the line total directly, which suits a fixed goodwill amount rather than a recalculated unit price — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)
- (concept) Custom Fields defined by a Type on the Cart and Line Item are where the acting agent, the authority and the reason code are carried so they survive onto the Order — [docs](https://docs.commercetools.com/tutorials/custom-types)

**Constraints that change the design**

- An external price overrides any Embedded or Standalone Price on the variant and takes the line out of normal price selection - so an agent-adjusted line will not re-price when catalog prices or the shopper's group change — [docs](https://docs.commercetools.com/learning-price-and-discount-your-products/price-calculation/price-selection)
- commercetools has no built-in service-agent role acting on a consumer's behalf: the act-as relationship, the adjustment limits and the audit are yours to build and enforce above the API — [docs](https://docs.commercetools.com/tutorials/custom-types)

**Modeling notes**

Build this as an explicit act-as with its own credential and its own audit, never by letting an agent authenticate as the shopper - the second is indistinguishable from account takeover in the logs, which is exactly when you need to tell them apart. Enforce the adjustment limits in the service layer, because nothing in the platform caps an external price: the API will accept any number the caller sends. Note that an externally priced line stops participating in price selection, so a goodwill adjustment silently survives later catalog or tier changes; decide whether adjusted lines should be re-evaluated at placement and, if so, do it deliberately. Carry agent, authority and reason as custom fields so they persist onto the order - an audit trail that lives only in the support tool is not available to the finance team asking why a line was discounted.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-platform`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- What is the adjustment limit per agent role, and who approves an escalation?
- May an agent complete the purchase, or only prepare the cart for the shopper?
- How long does an assisted session last, and is the shopper notified it started?
