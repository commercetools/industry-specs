# Loyalty member prices that the cart actually charges

## Purpose

Member pricing only earns sign-ins if the member sees the benefit before committing. The failure that destroys that trust is a price that appears on the product page and then changes in the cart, which happens whenever display pricing and cart pricing are resolved by different rules.

## Requirements

### Requirement: Loyalty member prices that the cart actually charges

The system SHALL show a signed-in loyalty member the price for their tier wherever a price is displayed, and charge that same price in the cart, so that the browsing price and the checkout price cannot disagree.

#### Scenario: Member sees member price
- **GIVEN** a signed-in shopper in a loyalty tier with a tier price on a product
- **WHEN** they view that product on a listing or detail surface
- **THEN** the tier price is the price shown, marked as a member price

#### Scenario: Cart charges the same
- **GIVEN** the same shopper adding that product
- **WHEN** the cart is priced
- **THEN** the cart charges the tier price shown while browsing, with no change at checkout

#### Scenario: Guest sees the offer
- **GIVEN** a shopper who is not signed in
- **WHEN** they view a product carrying a tier price
- **THEN** the standard price is charged, and the member price is shown as what signing in would give

#### Scenario: Tier changes mid session
- **GIVEN** a shopper whose tier changes while they have an open cart
- **WHEN** the cart is next read
- **THEN** the cart re-prices to the new tier rather than holding the price from the old one

#### Scenario: No tier price falls back
- **GIVEN** a member viewing a product with no price for their tier
- **WHEN** the price is resolved
- **THEN** the standard price applies, and no member badge is shown

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Tier membership on the shopper | `[MIDDLEWARE]` | Customer group assignment, sourced from the loyalty system |
| Tier price on the product | `[CACHED]` | Price scoped to the customer group |
| Member price shown while browsing | `[MIDDLEWARE]` | Price selection applied to listing and detail reads |
| Non-member savings prompt | `[STATIC]` | Shows the member price and what joining would save |
| Tier change propagation | `[MIDDLEWARE]` | A tier change re-prices the open cart, not just future ones |

## commercetools

**Entities:** `Customer`, `CustomerGroup`, `Price`, `StandalonePrice`, `Cart`, `ProductProjection`

**Verified API surface**

- (concept) Product price selection is driven by priceCurrency, priceCustomerGroup, priceChannel and priceCountry on the Product Projections and Product Projection Search APIs; Line Item price selection uses the Cart's currency, customerGroup, country and the Line Item's distributionChannel — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)

**Constraints that change the design**

- Display pricing and cart pricing are resolved by DIFFERENT APIs with parallel but separate parameters - passing the customer group to the catalog read but not setting it on the Cart is what makes the shown price and the charged price diverge — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)
- Price selection follows an eight-step fallback over customer group, channel and country, and within each step a time-bound price with validFrom and validUntil takes precedence over a non-time-bound one — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)
- With multiple Customer Groups, eligible groups come from Customer customerGroupAssignments and the Cart customerGroup; if customerGroupAssignments is an empty list the base price applies, and if it is unset the Cart must carry a customerId or the base price applies — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)
- Product Projection Search does not support multiple Customer Group prices, so a shopper holding several tiers cannot have all of them resolved in a search-driven listing the way they are in the cart — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)
- Where several prices match the same fallback step the cheapest before Cart Discounts is selected, and tiered prices are ignored when picking that lowest price — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)

**Modeling notes**

Model a tier as a customer group and the member price as a price scoped to it, so the platform resolves it rather than the storefront overriding it - an override in the storefront is exactly the divergence this capability exists to prevent. The single most common defect is passing the customer group on catalog reads and forgetting it on the cart, or the reverse; make the session's pricing context one object that both paths must take. If shoppers can hold more than one tier, know that search-driven listings cannot resolve multiple groups the way the cart can, and either restrict a shopper to one effective tier for display or accept that a listing shows a conservative price. Loyalty tier is owned by the loyalty system, so the group assignment is a sync with a staleness budget: decide what happens to an open cart when the tier changes, because doing nothing means charging yesterday's tier.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-storefront`, `commercetools-connect`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Can a shopper hold more than one tier at once, and which one wins for display?
- How stale may tier membership be before a cart must be re-priced?
- Do member prices stack with promotions, or is the member price already the final one?
