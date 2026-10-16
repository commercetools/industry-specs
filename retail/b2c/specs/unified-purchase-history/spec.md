# Store and online purchases in one history for one shopper

## Purpose

A shopper does not experience the till and the website as two retailers, and the service questions that follow a purchase — return it, reorder it, claim on it — do not care where it happened. The honesty constraint is that channels differ in what they can support afterwards, so a single list must not imply a uniformity that does not exist.

## Requirements

### Requirement: Store and online purchases in one history for one shopper

The system SHALL present a shopper's in-store and online purchases as one history against one customer record, marking which channel each purchase came from and what post-purchase actions each supports.

#### Scenario: Both channels one list
- **GIVEN** a shopper with purchases made online and in a store under the same loyalty identity
- **WHEN** they open their purchase history
- **THEN** both appear in one list, each marked with its channel and, for a store purchase, which store

#### Scenario: Actions match the record
- **GIVEN** a store purchase whose record does not support online returns
- **WHEN** the shopper views it
- **THEN** online return is not offered, and what they can do instead is stated

#### Scenario: Bought without identifying
- **GIVEN** a store purchase made without the shopper identifying themselves
- **WHEN** they later present proof of that purchase
- **THEN** it can be claimed onto their account and joins the history

#### Scenario: Store purchase not re transacted
- **GIVEN** a purchase already completed and paid at the till
- **WHEN** it is brought into the history
- **THEN** it appears as a completed purchase without any payment being taken or stock being moved again

#### Scenario: Channel record arrives late
- **GIVEN** a store purchase whose record arrives some hours after the sale
- **WHEN** the shopper checks their history before it lands
- **THEN** the delay is expected and stated rather than the purchase appearing to be missing

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Identity linking across channels | `[MIDDLEWARE]` | Loyalty identity presented at the till |
| Store purchases brought in | `[MIDDLEWARE]` | Imported from the point of sale, not re-transacted |
| Channel shown on each purchase | `[MIDDLEWARE]` | Where it happened, and which store |
| Actions offered per purchase | `[MIDDLEWARE]` | Only what that channel's record can support |
| Unlinked store purchases | `[MIDDLEWARE]` | Bought without identifying; claimable later |

## commercetools

**Entities:** `Order`, `Customer`, `Store`, `Channel`, `Payment`, `Type`

**Verified API surface**

- (rest) The Order Import API creates an Order from an external system without a Cart, through Create Order by Import - which is the supported route for a sale that was already transacted at a till — [docs](https://docs.commercetools.com/api/projects/orders-import)
- (concept) Pre-calculated taxedPrice values can be supplied on imported line items, custom line items and shipping info and are stored as sent rather than derived from taxRate, which avoids rounding discrepancies against an external tax or point-of-sale system — [docs](https://docs.commercetools.com/api/projects/orders-import)
- (concept) For bulk rather than single-order loads, the Import API's Import Orders endpoint is the route, though it does not support beta features of the commerce APIs — [docs](https://docs.commercetools.com/api/projects/orders-import)

**Constraints that change the design**

- An imported Order's total price must be set explicitly and is not calculated from the sum of its line item totals, and no error is raised for negative prices or quantities - so the import is only as correct as the data fed to it — [docs](https://docs.commercetools.com/api/projects/orders-import)
- When taxedPrice is supplied on an imported item the values are stored as sent and are not checked for consistency, so totalGross minus totalNet need not equal totalTax - the platform will not catch a broken feed — [docs](https://docs.commercetools.com/api/projects/orders-import)
- Inventory quantity limits are not enforced on Order Imports, which is correct here - the goods already left the shelf - but means the import path cannot be relied on for any validation — [docs](https://docs.commercetools.com/api/inventory-overview)

**Modeling notes**

Import store sales as Orders rather than replaying them through cart and checkout: they are finished transactions, and re-transacting them would move stock and take payment a second time. Because the import validates almost nothing - totals are taken as given, negatives pass silently, supplied tax figures are never checked for internal consistency - reconciliation has to be built alongside the feed rather than assumed from it, and a broken point-of-sale export will otherwise show up as wrong numbers in a shopper's history rather than as an error. Mark the channel on every order and drive the available post-purchase actions from it, since an imported record usually lacks the payment detail an online refund needs. The identity link is the part that decides whether this works at all: a store purchase only joins the history if the shopper identified themselves at the till, so plan for the unlinked case as the normal one rather than the exception.

## commercetools skills

Load `commercetools-connect` before implementing this capability. Supporting: `commercetools-platform`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-connect]`.

## Open questions

- What identifies a shopper at the till, and what share of store sales carry it?
- What is the acceptable delay between a store sale and it appearing in the history?
- Can a store purchase be returned online, and what does the refund settle against?
- How far back is history imported at launch, and is that a one-off load or a backfill?
