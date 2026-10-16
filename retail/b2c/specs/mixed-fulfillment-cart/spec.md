# One cart holding lines fulfilled several different ways

## Purpose

A retailer that sells from stores, warehouses and suppliers cannot force a shopper into one fulfillment method per order without either losing the basket or making them check out several times. Once lines differ, shipping charge and tax stop being basket-level facts, which is what makes this a data-model decision rather than a checkout screen.

## Requirements

### Requirement: One cart holding lines fulfilled several different ways

The system SHALL allow each line in one cart to carry its own fulfillment method and destination, and price shipping and tax per line accordingly, so a single order can combine home delivery, store collection and supplier-direct shipment.

#### Scenario: Two methods one cart
- **GIVEN** a cart with one line for home delivery and one for store collection
- **WHEN** the shopper reviews the cart
- **THEN** each line shows its own method and destination, and the basket shows one combined total

#### Scenario: Charge recalculated on change
- **GIVEN** a cart already carrying shipping charges
- **WHEN** the shopper changes one line's fulfillment method
- **THEN** the shipping charge and totals are recalculated and the cart is re-read before being shown

#### Scenario: Tax follows the destination
- **GIVEN** two lines going to destinations in different tax jurisdictions
- **WHEN** the cart is priced
- **THEN** each line carries the tax for its own destination rather than one basket-level rate

#### Scenario: Supplier direct line
- **GIVEN** a line fulfilled directly by a supplier rather than from the retailer's own stock
- **WHEN** the order is placed
- **THEN** the line carries its supplier-direct method and destination through to the order

#### Scenario: Method no longer possible
- **GIVEN** a line set to store collection at a store that can no longer fulfill it
- **WHEN** the cart is revalidated
- **THEN** the line's method is cleared with the reason and the remaining valid methods are offered

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Fulfillment method per line | `[MIDDLEWARE]` | Chosen on the line, not once for the basket |
| Several destinations on one cart | `[MIDDLEWARE]` | Home address and collection points together |
| Shipping charge per method | `[MIDDLEWARE]` | Recalculated whenever a line's method changes |
| Tax resolved per destination | `[MIDDLEWARE]` | A line's tax follows where it is going |
| Grouped review before payment | `[STATIC]` | Lines grouped by how and where they arrive |

## commercetools

**Entities:** `Cart`, `LineItem`, `ShippingMethod`, `Zone`, `Order`, `Channel`, `Store`

**Verified API surface**

- (concept) ShippingMode Multiple on the CartDraft is what allows individual items of one Order to use different Shipping Methods - the default Single mode cannot express a mixed basket — [docs](https://docs.commercetools.com/api/shipping-delivery-overview)
- (update-action) The sequence is: put the destinations in the Cart's itemShippingAddresses (on the draft, or with Add ItemShippingAddress), attach methods with Add ShippingMethod naming a shippingAddress, then assign them per line with Set LineItem ShippingDetails — [docs](https://docs.commercetools.com/api/shipping-delivery-overview)
- (concept) The same Shipping Method resource can be added to a Cart more than once with different shippingKey values, which is how lines sharing a logical method but going to different places are modeled — [docs](https://docs.commercetools.com/api/shipping-delivery-overview)
- (concept) Internal and external Shipping Methods can coexist in a single Cart, so a supplier-direct or externally rated method sits alongside the retailer's own methods — [docs](https://docs.commercetools.com/api/shipping-delivery-overview)

**Constraints that change the design**

- Under ShippingMode Multiple a Line Item carries perMethodTaxRate and taxedPricePortions instead of a single taxRate - code reading taxRate keeps working under Single mode and silently reads nothing once a cart goes mixed — [docs](https://docs.commercetools.com/api/shipping-delivery-overview)
- Setting a shipping address or method triggers recalculation of shipping cost, taxes and available options, so the storefront must re-read the cart after each change or show totals that no longer match — [docs](https://docs.commercetools.com/learning-implement-checkout/custom-checkout/shipping)

**Modeling notes**

Decide ShippingMode at cart creation, because it changes the shape of the data the rest of the checkout reads: under Multiple the tax lives in perMethodTaxRate and taxedPricePortions, and any code still reading the single taxRate field goes quietly empty rather than failing. That makes a late switch from Single to Multiple one of the more painful changes available, so if mixed fulfillment is on the roadmap at all, start in Multiple. A collection point is an address in itemShippingAddresses like any other destination, which keeps store collection and home delivery on one mechanism instead of two. Every method or address change recalculates, so treat the cart as read-after-write throughout checkout; caching a total across a method change is the defect this model produces most often.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-checkout`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Can one line be split across two fulfillment methods, or is a line indivisible?
- Is the shipping charge per method, per destination, or capped once per basket?
- Does a supplier-direct line need its own order record downstream, or only its own method?
