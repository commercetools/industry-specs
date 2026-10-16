# Markdown and clearance driven by age and sell-through

## Purpose

Seasonal retail carries a standing obligation to clear stock before it loses value, and doing it by hand means it happens late and unevenly across a large assortment. The reverse path matters as much as the forward one: goods marked down and never restored quietly erode margin for the rest of their life.

## Requirements

### Requirement: Markdown and clearance driven by age and sell-through

The system SHALL move a product onto its markdown price and into the clearance assortment when it crosses the age or sell-through threshold set for its category, and move it back out when the markdown is withdrawn.

#### Scenario: Crosses age threshold
- **GIVEN** a product whose days on hand exceed its category's threshold
- **WHEN** the markdown rule runs
- **THEN** the markdown price applies, the product joins the clearance assortment, and the rule that acted is recorded

#### Scenario: Crosses sell through threshold
- **GIVEN** a product whose sell-through rate is below its category's threshold for the season
- **WHEN** the markdown rule runs
- **THEN** the markdown price applies at the configured depth for that shortfall

#### Scenario: Markdown expires
- **GIVEN** a product on a markdown price whose end date passes
- **WHEN** the price is next resolved
- **THEN** the standard price applies again and the product leaves the clearance assortment

#### Scenario: Exempted line untouched
- **GIVEN** a product a merchandiser has exempted from automatic markdown
- **WHEN** it crosses a threshold
- **THEN** no markdown is applied and the exemption is the recorded reason

#### Scenario: Clearance page follows
- **GIVEN** products entering and leaving clearance
- **WHEN** the clearance page is requested
- **THEN** it lists exactly the products currently in the clearance assortment, with no manual curation

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Age and sell-through thresholds per category | `[MIDDLEWARE]` | Owned by merchandising, editable without a release |
| Markdown price with an end date | `[CACHED]` | A bounded price, so it expires rather than persists |
| Clearance assortment membership | `[CACHED]` | A derived attribute the clearance page selects on |
| Markdown provenance on the product | `[MIDDLEWARE]` | Which rule marked it down and when |
| Manual override | `[MIDDLEWARE]` | A merchandiser can exempt a line; the rule respects it |

## commercetools

**Entities:** `Price`, `StandalonePrice`, `Product`, `ProductType`, `InventoryEntry`, `Subscription`

**Verified API surface**

- (concept) A markdown is a Price with validFrom and validUntil: the price is only shown while the request timestamp falls inside that range, so the markdown withdraws itself when the window closes — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)
- (concept) Standalone Prices are managed and queried through their own API independently of the Product, and support up to 50000 prices per Product Variant - which is what makes a rolling markdown program across a large assortment tractable — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)

**Constraints that change the design**

- Within a fallback step a time-bound price takes precedence over a non-time-bound one, so a bounded markdown correctly overrides the standing price for its window without the standing price being edited or removed — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)
- Days on hand and sell-through are not commercetools concepts: InventoryEntry carries current stock, not its age or its rate of sale, so the threshold inputs come from an external merchandising or analytics system — [docs](https://docs.commercetools.com/api/inventory-overview)

**Modeling notes**

Express the markdown as a bounded price rather than an edit to the standing price: an edited price has to be edited back, and the restoring edit is the one that gets forgotten. Prefer standalone prices for a markdown program, both for the volume and because they can be prepared and published independently of product changes. The thresholds themselves are the part commercetools cannot supply - stock age and sell-through live in the merchandising or analytics system - so this capability is a scheduled job reading those signals and writing prices, and its cadence is a business decision, not a technical one. Model clearance membership as a derived attribute rather than a hand-curated category, so the clearance page and the markdown rule cannot disagree, and always record which rule marked a line down: without that provenance nobody can answer why a price moved.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-connect`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Which system owns days on hand and sell-through, and how often can it be read?
- Is markdown depth a fixed ladder per category, or calculated per line?
- Should a marked-down product be excluded from other promotions?
