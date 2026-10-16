# Running one product at two prices across regions or banners

## Purpose

Price elasticity is guessed far more often than it is measured, and the reason is usually that the platform cannot hold two live prices for one product without duplicating it. Recording the applied price on the order is what separates an experiment from a change nobody can evaluate.

## Requirements

### Requirement: Running one product at two prices across regions or banners

The system SHALL let the same product carry different prices for different regions or banners over a bounded period, and record on each order which price applied, so the result can be measured after the period closes.

#### Scenario: Two regions two prices
- **GIVEN** one product with a price for each of two regions during the experiment window
- **WHEN** shoppers in each region view it
- **THEN** each sees their region's price, and each cart charges the price that region was shown

#### Scenario: Window closes itself
- **GIVEN** an experiment price whose validity period has ended
- **WHEN** a shopper views the product
- **THEN** the standard price applies again without anyone withdrawing the experiment by hand

#### Scenario: Applied price recorded
- **GIVEN** an order placed during the experiment
- **WHEN** the order is created
- **THEN** the order records which experiment price applied, so results can be attributed afterwards

#### Scenario: Stable for the shopper
- **GIVEN** a shopper who saw one experiment price
- **WHEN** they return within the window
- **THEN** they see the same price, rather than a different one on a different visit

#### Scenario: Experiment and promotion together
- **GIVEN** a product under an experiment price that also qualifies for a promotion
- **WHEN** the cart is priced
- **THEN** the promotion applies to the experiment price and the order records both

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Price scoped to region or banner | `[CACHED]` | Country and distribution channel on the price |
| Bounded validity window | `[CACHED]` | Valid-from and valid-until; the experiment ends itself |
| Applied price recorded on the order | `[MIDDLEWARE]` | Which variant of the price the order was placed at |
| Consistent assignment per shopper | `[MIDDLEWARE]` | A shopper does not see the price move between visits |
| Experiment register | `[MIDDLEWARE]` | What is running, where, until when |

## commercetools

**Entities:** `Price`, `StandalonePrice`, `Channel`, `Store`, `Cart`, `Order`, `Type`

**Verified API surface**

- (concept) A Price can be scoped by country and by distribution channel, so a region-specific or banner-specific price is a second Price on the same Product Variant rather than a second product — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)
- (concept) Standalone Prices are managed independently of the Product through their own API and can be staged before publishing, which suits an experiment price that should be prepared before its window opens — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)

**Constraints that change the design**

- A price is shown only when the request timestamp falls within its validFrom and validUntil range, and within a fallback step a time-bound price takes precedence over a non-time-bound one - which is what makes an experiment self-terminating — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)
- Where several prices match the same fallback step, the cheapest before Cart Discounts is chosen - so two overlapping experiment prices at the same scope do not give a random split, they give the lower price to everyone — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)

**Modeling notes**

Split by a scope the platform already understands - country or distribution channel - rather than by a synthetic cohort, because scope-based selection is what keeps the cart and the catalog agreeing without any experiment logic in the pricing path. That constrains what you can test: this gives a geography or banner split, not a random split of shoppers within one region. A within-region split needs a customer group per arm and an assignment service, and then the group assignment becomes the thing that must be stable per shopper. Do not create two prices at the same scope hoping for a split; the cheapest wins for everyone. Record the applied price on the order at placement rather than reconstructing it later from price history - the prices will have changed by the time anyone analyzes the result.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-platform`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Is a geography or banner split enough, or is a random within-region split required?
- How long must the applied-price record be retained for analysis?
- Who approves a price experiment, and what is the rollback if it moves margin the wrong way?
