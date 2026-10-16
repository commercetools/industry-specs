# Regulated goods bought in their own cart and their own order

## Purpose

Some categories cannot legally be sold by the same entity, in the same transaction, or into the same territory as the rest of the assortment. Merging them into one order is not a usability improvement, it is a compliance failure — but forcing shoppers to a second site loses the sale, so the split has to live below one storefront.

## Requirements

### Requirement: Regulated goods bought in their own cart and their own order

The system SHALL keep goods sold under a separate license or legal entity in their own cart and their own order, settled separately from the general basket, while presenting both in one browsing session.

#### Scenario: Regulated item opens its own cart
- **GIVEN** a shopper with a general basket
- **WHEN** they add an item sold by the separately licensed entity
- **THEN** it goes into that entity's own cart and does not join the general basket

#### Scenario: Both carts visible
- **GIVEN** a shopper holding items in both carts
- **WHEN** they view their basket
- **THEN** both are shown with their own totals and their own selling entity, and no combined total is offered

#### Scenario: Settled separately
- **GIVEN** a shopper checking out with both carts populated
- **WHEN** they complete the purchase
- **THEN** each cart is paid for and placed as its own order, and either can fail without voiding the other

#### Scenario: Entity shown before adding
- **GIVEN** a listing mixing general and regulated products
- **WHEN** the shopper views it
- **THEN** each regulated product states its selling entity before it can be added

#### Scenario: Not available in this territory
- **GIVEN** a shopper in a territory where the regulated category may not be sold
- **WHEN** they browse it
- **THEN** it is not purchasable, and the reason is stated rather than the items silently disappearing

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Regulated assortment identification | `[CACHED]` | The selling entity is a property of the product |
| Parallel carts in one session | `[MIDDLEWARE]` | One cart per selling entity, both live at once |
| Selling entity shown before adding | `[STATIC]` | Who the shopper is buying from, on listing and detail |
| Separate settlement per entity | `[MIDDLEWARE]` | Each cart pays separately; totals never merge |
| Territory eligibility | `[MIDDLEWARE]` | The regulated cart is unavailable where it may not sell |

## commercetools

**Entities:** `Store`, `Cart`, `Order`, `Channel`, `Product`, `ProductSelection`, `Payment`

**Verified API surface**

- (concept) A Cart is bound to a Store, and that binding filters the stock and prices available to it through the Store's configured supply and distribution channels - which is what makes one cart per selling entity a platform-level separation rather than a storefront convention — [docs](https://docs.commercetools.com/api/inventory-overview)
- (concept) A Store carries its own Product Selections, countries and channels, so the regulated assortment and the territories it may be sold into are expressed as Store configuration rather than as storefront rules — [docs](https://docs.commercetools.com/learning-model-your-business-structure/stores-and-channels/apply-stores-and-channels)
- (concept) Product Tailoring lets the same catalog record be presented differently per Store, which is how a regulated listing can carry the selling entity's own naming and disclosures — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/design-b2b-catalogs/product-tailoring-for-b2b)

**Constraints that change the design**

- A Cart belongs to one Store, so two selling entities means two Carts - there is no supported way to partition one Cart by entity and settle the partitions independently — [docs](https://docs.commercetools.com/api/inventory-overview)

**Modeling notes**

One Store per selling entity, one cart per Store, and a session that holds both. Doing this with a flag on a line item instead is the tempting shortcut and it fails at the first audit: a single cart produces a single order with a single payment, and no amount of downstream splitting makes that two legally separate transactions. Because there are two carts, almost everything the storefront treats as singular becomes plural - the basket indicator, promotions, delivery charges, the checkout itself - and promotions in particular must not span the carts, since a discount funded by one entity applied to another's goods is its own problem. Say who the shopper is buying from before they add, not at checkout. Territory rules and age checks are related but separate: this capability keeps the transaction separable, and does not by itself verify anyone's age.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-checkout`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- How is age or eligibility verified, and at what point — order, dispatch or handover?
- Can a delivery combine goods from both entities physically, even though the orders are separate?
- Which entity owns the customer record and the order history the shopper sees?
