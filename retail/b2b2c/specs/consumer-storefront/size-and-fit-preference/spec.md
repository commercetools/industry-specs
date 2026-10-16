# A stored size preference that narrows the whole storefront

## Purpose

In apparel and footwear most of the assortment does not exist in the shopper's size, so an unfiltered listing is mostly disappointment. A preference stored once is worth more than a filter re-applied per visit, and the categories differ — a shoe size is not a shirt size.

## Requirements

### Requirement: A stored size preference that narrows the whole storefront

The system SHALL let a shopper store a size preference per product category and apply it as the default filter on listing, search and recommendation surfaces, while still allowing the shopper to see everything on demand.

#### Scenario: Preference applied
- **GIVEN** a signed-in shopper with a stored footwear size
- **WHEN** they open a footwear listing
- **THEN** the listing defaults to products available in that size, and says so

#### Scenario: Preference reversible
- **GIVEN** a listing narrowed by the stored preference
- **WHEN** the shopper turns the narrowing off
- **THEN** the full assortment is shown for that request without the stored preference being changed

#### Scenario: Category specific
- **GIVEN** a shopper with a stored footwear size and no stored apparel size
- **WHEN** they open an apparel listing
- **THEN** the apparel listing is unnarrowed, and the footwear preference is not applied to it

#### Scenario: Captured in flow
- **GIVEN** a shopper choosing a size on a product
- **WHEN** they choose it
- **THEN** storing that size as their preference for the category is offered without leaving the page

#### Scenario: Survives sign in
- **GIVEN** a guest who set a size preference in the session
- **WHEN** they sign in or register
- **THEN** the preference is carried onto the account and applies on their next visit

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Size preference per category | `[MIDDLEWARE]` | Custom field on the customer; several categories at once |
| Preference applied as default filter | `[MIDDLEWARE]` | Search filter on the size variant attribute |
| Visible, reversible filter control | `[STATIC]` | Says the list is narrowed, and turns it off in one action |
| Preference capture in the flow | `[STATIC]` | Offered where size is already being chosen, not as a separate chore |
| Guest-session preference | `[MIDDLEWARE]` | Held on the session and carried onto the account at sign-in |

## commercetools

**Entities:** `Customer`, `Type`, `Product`, `ProductType`, `ProductVariant`

**Verified API surface**

- (concept) A Type with resourceTypeIds customer adds Custom Fields to the Customer resource; a stored preferred size is the documented worked example for exactly this, used to preselect on a detail page and to filter search results — [docs](https://docs.commercetools.com/tutorials/custom-types)
- (concept) Size is a Product Variant attribute, so applying the preference is a filter over that attribute at search time rather than a separate stored assortment per shopper — [docs](https://docs.commercetools.com/tutorials/custom-types)

**Constraints that change the design**

- Custom Fields are defined by a Type before they can be written, so adding a new preference category is a Type change and not a free-form write — [docs](https://docs.commercetools.com/tutorials/custom-types)

**Modeling notes**

Store the preference against the customer, and apply it at query time - never materialize a per-shopper assortment, which multiplies the catalog by the customer base. Because sizes are variant attributes, the filter narrows to products having an in-size variant; decide explicitly whether an in-size variant that is out of stock should still show, because the two answers give very different listings and shoppers notice. Size vocabularies are not comparable across categories or across regions, so hold one preference per category and normalize before comparing; a single stored "size" field is the mistake that makes this feature untrustworthy. Keep the narrowing visible: a filter the shopper did not knowingly apply reads as a broken site.

## commercetools skills

Load `commercetools-storefront` before implementing this capability. Supporting: `commercetools-platform`, `commercetools-commerce-patterns`. Any task generated from this spec carries `[SKILL: commercetools-storefront]`.

## Open questions

- Should a product whose in-size variant is out of stock still appear in a narrowed listing?
- Do size vocabularies need normalising across regions, and who owns that mapping?
- Does one account hold one set of preferences, or one per household member?
