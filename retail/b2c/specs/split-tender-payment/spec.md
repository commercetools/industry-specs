# One order settled by several payment instruments together

## Purpose

Gift cards and stored value are a large share of retail tender and rarely match the basket total, so a shopper who cannot combine them with a card abandons or under-redeems. The risk this capability manages is a partially settled order: value taken from a gift card against an order that never completes is the worst outcome for both sides.

## Requirements

### Requirement: One order settled by several payment instruments together

The system SHALL settle one order from several payment instruments applied in a defined order against a shrinking balance, confirming the order only once that balance reaches zero.

#### Scenario: Gift card plus card
- **GIVEN** a shopper with a gift card that covers part of the basket
- **WHEN** they apply it at checkout
- **THEN** the remaining balance is shown and a second instrument is requested for exactly that amount

#### Scenario: Several gift cards
- **GIVEN** a shopper holding two gift cards
- **WHEN** they apply both
- **THEN** each is applied in turn against the remaining balance and the order records both

#### Scenario: Instrument covers everything
- **GIVEN** stored value that covers the full basket
- **WHEN** it is applied
- **THEN** no further instrument is requested and the order completes on stored value alone

#### Scenario: Second instrument declines
- **GIVEN** stored value already applied and a card that is then declined
- **WHEN** the decline is returned
- **THEN** the order is not confirmed and the stored value is released rather than left redeemed

#### Scenario: Checkout abandoned
- **GIVEN** stored value applied to a cart the shopper then abandons
- **WHEN** the cart expires
- **THEN** the redeemed value is returned to the instrument

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Balance check before redemption | `[MIDDLEWARE]` | Shows what the instrument will actually cover |
| Ordered application of tenders | `[MIDDLEWARE]` | Stored value first, card for the remainder |
| Several stored-value instruments | `[MIDDLEWARE]` | More than one gift card on one order |
| Remaining balance shown live | `[MIDDLEWARE]` | Updated as each instrument is applied |
| Release on abandonment or failure | `[MIDDLEWARE]` | Redeemed value returns if the order does not complete |

## commercetools

**Entities:** `Payment`, `Cart`, `Order`, `Transaction`, `Store`

**Verified API surface**

- (concept) Gift card Connectors support checking a balance in real time, applying gift card value toward a purchase, partial payment where the gift card covers part and another method covers the remainder, and applying several gift cards to one transaction — [docs](https://docs.commercetools.com/certifications/frontend-developer-refresher-2026/checkout-refresher-2026/payment-features-and-capabilities)
- (concept) Gift card lifecycle events - balance check started, balance success carrying amount and isBalanceSufficient, redeem success, and their error counterparts - are published as Checkout messages the storefront subscribes to through the Browser SDK — [docs](https://docs.commercetools.com/certifications/frontend-developer-refresher-2026/checkout-refresher-2026/payment-features-and-capabilities)
- (concept) Non-standard tenders that no processor covers are modeled as a Custom payment type backed by an external module you provide, which returns only success or cancel to the checkout — [docs](https://docs.commercetools.com/instore/integration/payments/overview)

**Constraints that change the design**

- A gift card Payment Integration should always be configured alongside another Payment Integration, so that a gift card with insufficient balance can be topped up by a second method rather than dead-ending the checkout — [docs](https://docs.commercetools.com/certifications/frontend-developer-refresher-2026/checkout-refresher-2026/payment-features-and-capabilities)

**Modeling notes**

Treat the split as a sequence over a shrinking balance rather than a set of parallel payments, and fix the order deliberately: stored value first is almost always right, because it is the instrument that cannot be re-presented later. The hard part is not the happy path, it is the unwind. Redeeming a gift card is an external, immediately effective act, so any later failure or abandonment leaves real value stranded unless there is a compensating release with its own retry and its own alerting - build that before the redemption path, not after. Always pair a gift card integration with at least one other payment integration, or an insufficient balance becomes a dead end. Where a tender has no connector at all, the custom payment type is the supported route, but note it returns only success or cancel: any richer failure detail has to come from your own module.

## commercetools skills

Load `commercetools-checkout` before implementing this capability. Supporting: `commercetools-connect`, `commercetools-commerce-patterns`. Any task generated from this spec carries `[SKILL: commercetools-checkout]`.

## Open questions

- In what order are tender types applied, and can a shopper change it?
- How long may value stay redeemed against an incomplete order before it is released?
- Are loyalty points a tender here, and if so who holds the ledger?
- How is a refund apportioned back across the instruments that paid?
