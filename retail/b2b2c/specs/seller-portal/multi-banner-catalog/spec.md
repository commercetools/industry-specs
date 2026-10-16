# One catalog serving several banners with per-banner overrides

## Purpose

A retail group runs several banners off one merchandising team, and the whole reason to replatform is to stop maintaining the same product N times. The banners share most of their assortment and differ in a minority of products, prices, names and imagery — so the split has to be expressible as an override on a shared record, not as a second catalog.

## Requirements

### Requirement: One catalog serving several banners with per-banner overrides

The system SHALL resolve assortment, price, language and product presentation from the banner the shopper is on, so that one shared catalog record can carry banner-specific overrides without being duplicated per banner.

#### Scenario: Shared product reads through
- **GIVEN** a product in the shared assortment with no banner override
- **WHEN** a shopper opens it on any banner
- **THEN** the canonical name, copy and imagery are shown, identical across banners

#### Scenario: Banner override wins
- **GIVEN** a product carrying a banner-specific name and image for one banner
- **WHEN** a shopper opens it on that banner
- **THEN** the banner-specific values are shown, and fields with no override fall back to the canonical ones

#### Scenario: Exclusive not leaked
- **GIVEN** a product that belongs only to one banner's assortment
- **WHEN** a shopper browses or searches another banner
- **THEN** the product is absent from listing, search and direct retrieval on that banner

#### Scenario: Price follows banner and country
- **GIVEN** one product sold by two banners in two countries
- **WHEN** a shopper views it in a given banner and country
- **THEN** the price for that banner and country is the one shown, and it is the price the cart charges

#### Scenario: Override without availability shows nothing
- **GIVEN** a banner override written for a product not in that banner's assortment
- **WHEN** a shopper browses that banner
- **THEN** the product remains invisible, because an override changes presentation and never availability

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Banner resolution for the session | `[MIDDLEWARE]` | Store key from host or route; every catalog read is Store-scoped thereafter |
| Shared plus banner-exclusive assortment | `[CACHED]` | One inclusion Product Selection for shared, one per banner for exclusives |
| Banner-specific name, copy and imagery | `[CACHED]` | Product Tailoring per Product and Store; unset fields fall back |
| Banner and country price | `[MIDDLEWARE]` | Distribution channel plus country on the Store |

## commercetools

**Entities:** `Store`, `ProductSelection`, `ProductTailoring`, `Product`, `Channel`, `Cart`

**Verified API surface**

- (concept) A Store carries its own productSelections, languages, countries, distributionChannels and supplyChannels - it is the unit a banner maps onto, and a Store key is what a storefront resolves per request — [docs](https://docs.commercetools.com/learning-model-your-business-structure/stores-and-channels/apply-stores-and-channels)
- (concept) Model shared assortment as one inclusion Product Selection assigned to every banner's Store, plus one Product Selection per banner for its exclusives - a Store may hold several selections at once — [docs](https://docs.commercetools.com/learning-model-your-business-structure/stores-and-channels/apply-stores-and-channels)
- (concept) Product Tailoring overrides presentation - name, description, slug, images - for one Product and Store pair, and is selective: fields left untailored fall back to the canonical Product values — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/design-b2b-catalogs/product-tailoring-for-b2b)
- (concept) Price selection resolves over customer group, distribution channel and country, so a per-banner price is a Price scoped to that banner's distribution channel and the Store's country — [docs](https://docs.commercetools.com/api/pricing-and-discounts-overview)

**Constraints that change the design**

- Tailored data is returned ONLY by the Store-scoped Get ProductProjection in Store endpoints, never by a normal Product query - a storefront reading the plain projection silently shows canonical content on every banner — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/design-b2b-catalogs/product-tailoring-for-b2b)
- Tailoring changes presentation, not availability: tailoring a Product that no active Product Selection makes available in that Store changes nothing the shopper can see — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/design-b2b-catalogs/product-tailoring-for-b2b)

**Modeling notes**

One Store per banner, and one Store per banner-and-country pair where the countries differ in currency or language. Keep the shared assortment in a single inclusion Product Selection that every Store references, so adding a product to all banners is one write. Reach for Product Tailoring only for presentation; anything that changes what is sellable belongs in a Product Selection instead, and confusing the two is the most common way this model goes wrong. Every catalog read on the storefront must go through the in-store projection endpoints - a single plain Product query anywhere in the stack reintroduces canonical content on a tailored banner and the bug is invisible until a merchandiser notices the wrong name in production.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-platform`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Do the banners share a customer account and a cart, or is a shopper a separate customer per banner?
- Which banner owns a product's canonical content when two banners disagree about the same shared product?

---

_Excluded for B2B2C: Banner switcher._
