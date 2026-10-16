# A submitted order held for the seller to release

## Purpose

A reseller stands between the buyer and the supply, and it is the reseller who knows the things the storefront does not: that this buyer's credit is stretched, that the part in question is about to be superseded, that the quantity ordered is a typo. Releasing straight to fulfillment removes the only place that judgment can be applied. The counterpart matters just as much: a hold that no buyer can see is indistinguishable from a system failure, and a seller who trusts a given buyer needs to be able to turn the hold off rather than rubber-stamp every order.

## Requirements

### Requirement: A submitted order held for the seller to release

The system SHALL hold a submitted order for the selling party to review before it is released to fulfillment where that seller requires it, allowing the seller to amend, substitute or reject lines during the hold while the buyer can see the order is awaiting release.

#### Scenario: Order held and released
- **GIVEN** a seller who requires review for a given buyer
- **WHEN** that buyer submits an order
- **THEN** the order is held, appears in the seller's review queue, and reaches fulfillment only when the seller releases it

#### Scenario: Buyer sees the hold
- **GIVEN** a held order
- **WHEN** the buyer views it
- **THEN** it is shown as awaiting the seller's release rather than as confirmed or as failed

#### Scenario: Hold disabled for a trusted buyer
- **GIVEN** a seller who has turned review off for a particular buyer
- **WHEN** that buyer submits an order
- **THEN** the order is released without review, and orders from buyers still under review are unaffected

#### Scenario: Seller amends during the hold
- **GIVEN** a held order containing a line the seller must substitute
- **WHEN** the seller substitutes it and releases the order
- **THEN** the change and its reason are recorded, and the buyer can see what was changed after release

#### Scenario: Seller rejects
- **GIVEN** a held order the seller will not fulfill
- **WHEN** the seller rejects it with a reason
- **THEN** the order is marked rejected, the buyer is told the reason, and nothing is handed to fulfillment

#### Scenario: Hold expires unreviewed
- **GIVEN** a held order nobody has reviewed within the agreed window
- **WHEN** the window expires
- **THEN** the configured outcome is applied — released or escalated — and both parties can see which

#### Scenario: Buyer cannot change a held order
- **GIVEN** a held order under the seller's review
- **WHEN** the buyer attempts to change it
- **THEN** the attempt is refused with the state explained, rather than producing a change the seller has already reviewed past

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Hold applied according to the seller's setting | `[MIDDLEWARE]` | Per seller and per buyer, not one global switch |
| Awaiting-release state shown to the buyer | `[MIDDLEWARE]` | Plainly, not as a silent pause |
| Expiry when nobody reviews it | `[MIDDLEWARE]` | A hold cannot be indefinite |

## commercetools

**Entities:** `Order`, `State`, `OrderEdit`, `Subscription`, `BusinessUnit`, `AssociateRole`, `Type`, `Message`

**Verified API surface**

- (concept) States manage the lifecycle of Orders with the application defining the States, the allowed transitions and when they are triggered, which is how a held-pending-release stage is introduced between submission and fulfillment — [docs](https://docs.commercetools.com/learning-model-your-business-structure/state-machines/state-machines-page)
- (concept) Order Edits hold one or more proposed financial changes to an Order and can be previewed before being applied, so a seller's amendment during a hold can be shown in its effect before it is committed — [docs](https://docs.commercetools.com/api/projects/order-edits)
- (concept) Associates hold Associate Roles on a Business Unit and the Associate endpoints validate an Associate's permission, which is how the authority to release or reject a held order is evaluated server-side — [docs](https://docs.commercetools.com/api/projects/business-units)
- (concept) Subscriptions notify a message queue when a resource is modified and are used to trigger asynchronous processes, which is how arrival in the review queue and expiry of a hold reach the people who must act on them — [docs](https://docs.commercetools.com/api/projects/subscriptions)

**Constraints that change the design**

- If Embedded or Standalone Prices have changed since the Order was created, applying an Order Edit updates the prices of all Line Items even those the edit does not touch, and Line Items without a matching price are removed during the edit — [docs](https://docs.commercetools.com/api/projects/order-edits)
- Where no financial aspect of an Order changes, the Order should be updated directly through the Update Order endpoint rather than through an Order Edit — [docs](https://docs.commercetools.com/api/projects/order-edits)

**Modeling notes**

The hold is a State on the Order and release is a transition, which keeps the whole control in one auditable place. The trap is amendment: an Order Edit reprices every Line Item from current prices and drops any line whose price no longer resolves, so a seller substituting one line can silently reprice the rest of the order the buyer already agreed to. Preview the edit, compare the resulting totals against the submitted ones, and surface any change the buyer did not ask for rather than applying it. For non-financial corrections use Update Order directly, as the documentation advises, and keep Order Edits for changes that genuinely move money. Make the hold setting per seller and per buyer and resolve it from the Business Unit, since the commercial point is that trust is earned per relationship. Give the hold an expiry with a configured outcome; an unbounded hold is the failure this capability most often produces in practice, because the queue is checked daily until the day it is not.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-connect`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- What may a seller change during a hold, and what requires the buyer's agreement?
- How long may an order be held, and what happens when the window expires?
- Is payment authorized at submission or at release, and who carries the exposure during the hold?

---

_Excluded for B2B2B: Queue of orders awaiting release; Amend or substitute lines during the hold; Release to fulfillment; Reject with a reason; Notification when an order arrives for review._
