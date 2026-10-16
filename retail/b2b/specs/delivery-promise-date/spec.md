# An arrival date promised per shipment before payment

## Purpose

Arrival date is the information shoppers most often leave a basket over, and a basket that will arrive in two shipments has two answers, not one. Recording the promise matters as much as showing it: without it, nobody can tell whether the promise was kept, and the estimate never improves.

## Requirements

### Requirement: An arrival date promised per shipment before payment

The system SHALL show an expected arrival date for each shipment a cart will produce before the shopper pays, and record the promised dates on the order so that what was promised can be compared with what happened.

#### Scenario: One shipment one date
- **GIVEN** a cart whose lines all ship together
- **WHEN** the shopper reaches checkout
- **THEN** one expected arrival date is shown for the whole basket

#### Scenario: Split basket two dates
- **GIVEN** a cart whose lines ship from different origins
- **WHEN** the shopper reaches checkout
- **THEN** each shipment is shown separately with its own expected arrival date

#### Scenario: Estimate before full address
- **GIVEN** a shopper who has given only a postal area
- **WHEN** they view a product
- **THEN** an estimate for that area is shown, marked as an estimate rather than a commitment

#### Scenario: Cut off passes
- **GIVEN** a shopper in checkout when the day's dispatch cut-off passes
- **WHEN** the cart is re-read
- **THEN** the expected dates move to the next dispatch and the change is shown before payment

#### Scenario: Promise recorded
- **GIVEN** an order placed with expected dates shown
- **WHEN** the order is created
- **THEN** the promised date for each shipment is recorded on the order

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Shipment grouping of the cart | `[MIDDLEWARE]` | Lines grouped by origin and method into shipments |
| Estimate per shipment | `[MIDDLEWARE]` | From the fulfillment system; commercetools carries it |
| Promise recorded on the order | `[MIDDLEWARE]` | Per shipment, at placement |
| Cut-off handling | `[MIDDLEWARE]` | The estimate moves when the cut-off passes mid-session |

## commercetools

**Entities:** `Cart`, `LineItem`, `Order`, `ShippingMethod`, `Channel`, `Type`, `Store`

**Verified API surface**

- (concept) Under ShippingMode Multiple, lines are already grouped by Shipping Method and destination through shippingKey and Set LineItem ShippingDetails - that grouping is the shipment boundary a promise is quoted against — [docs](https://docs.commercetools.com/api/shipping-delivery-overview)
- (concept) GET shipping-methods matching-cart returns only the methods valid for the current cart given its address and contents, so it is the right input to which shipments a cart will actually produce — [docs](https://docs.commercetools.com/api/projects/shippingMethods)
- (concept) Custom Fields defined by a Type are how a value the platform does not model - here the promised date per shipment - is carried on the Cart and the Order — [docs](https://docs.commercetools.com/tutorials/custom-types)

**Constraints that change the design**

- commercetools models shipping as Shipping Methods, Zones and rates and carries no delivery-date or promise resource - the estimate is external state, and the platform's role is to carry the answer onto the cart and the order — [docs](https://docs.commercetools.com/api/shipping-delivery-overview)
- Setting the shipping address or method triggers recalculation of shipping cost, taxes and available options, so a promise quoted before such a change has to be requoted after it — [docs](https://docs.commercetools.com/learning-implement-checkout/custom-checkout/shipping)

**Modeling notes**

commercetools does not compute delivery dates and should not be made to: the estimate comes from the distributed order management or carrier system that knows stock positions, cut-offs and transit times. What belongs here is the promise as a custom field on the cart and then on the order, quoted per shipment rather than per basket. The latency budget is the design constraint - a promise service called on every cart mutation will dominate checkout response time, so cache per postal area and origin and requote only when the address, the method or the contents change. Distinguish an estimate from a commitment in the wording as well as the data: a pre-address estimate keyed off a postal area is a different promise from one quoted against a confirmed address, and conflating them is how a retailer ends up measured against a number it never actually promised.

## commercetools skills

Load `commercetools-connect` before implementing this capability. Supporting: `commercetools-commerce-patterns`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-connect]`.

## Open questions

- Which system computes the promise, and what is its latency at peak?
- Is the shown date a commitment the retailer stands behind, or an estimate — and does that differ before and after the address is known?
- What happens to a promise if the order is edited during a grace window?

---

_Excluded for B2B: Estimate before an address is given._
