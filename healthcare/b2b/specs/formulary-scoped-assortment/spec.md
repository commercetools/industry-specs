# The account's approved list, and the equivalent it points to

## Purpose

Clinical buying is not a free market inside the account. A committee decides which of several interchangeable products the organization will standardize on, for reasons of safety, training, contract and waste, and a purchase off that list is a governance failure even when the item is in stock and the price is right. So the approved list is not a merchandising filter; it is the boundary of what the account is permitted to do, and it has to hold on every surface, including the ones people reach for when the storefront says no — a reorder from last year's history, a saved list built before the list changed, a part number typed straight into quick order. The half that gets left out is the substitution. A buyer who wanted a specific item and is told only that it is unavailable will phone somebody, and that call is the entire cost of the control. The list is only usable if it can answer the question the buyer actually asked, which is not "may I have this" but "what do we use instead".

## Requirements

### Requirement: The account's approved list, and the equivalent it points to

The system SHALL restrict an account to the items on its approved list across browse, search, saved lists and cart alike, and where an item outside that list is requested, name the approved equivalent rather than returning nothing.

#### Scenario: Browse and search show only approved
- **GIVEN** an account with an approved list
- **WHEN** a buyer browses or searches
- **THEN** only items on the list are offered, at the prices that apply to that account

#### Scenario: Off list item names its equivalent
- **GIVEN** an off-list item with a defined approved equivalent
- **WHEN** the buyer requests it by part number
- **THEN** the equivalent is named and can be added in its place

#### Scenario: Off list item with no equivalent
- **GIVEN** an off-list item for which no equivalent is defined
- **WHEN** the buyer requests it
- **THEN** it is refused as off-list, and the refusal says so rather than reporting it as unavailable

#### Scenario: Reorder of an item since removed
- **GIVEN** an order history containing an item that has since left the list
- **WHEN** the buyer reorders from it
- **THEN** that line is not added, the reason is given, and the remaining lines still are

#### Scenario: Sub unit narrower than its parent
- **GIVEN** a department restricted further than the organization it belongs to
- **WHEN** a buyer in that department browses
- **THEN** they see the narrower list, not their parent's

#### Scenario: List change reaches existing baskets
- **GIVEN** a saved list and an open cart built before the approved list changed
- **WHEN** they are next opened
- **THEN** the lines that are no longer approved are reported before checkout

#### Scenario: Exception route where permitted
- **GIVEN** an account that permits exceptions
- **WHEN** a buyer asks for an off-list item
- **THEN** the request is raised for approval rather than being silently dropped

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Approved list held per account | `[MIDDLEWARE]` | Owned by the buying organization, not by the seller's merchandising |
| Narrower list inherited by a sub-unit | `[MIDDLEWARE]` | A department may be restricted further than its parent, never wider |
| One restriction applied to every surface | `[MIDDLEWARE]` | Browse, search, quick order, saved lists and reorder resolve the same list |
| Approved equivalent named for an off-list request | `[CACHED]` | Maintained as data; never inferred from category or description at read time |
| Reorder checked against the list as it stands now | `[MIDDLEWARE]` | History records what was bought, not what may be bought today |
| Off-list request routed for exception rather than refused outright | `[MIDDLEWARE]` | Where the account allows exceptions at all |
| List changes reflected in existing carts and saved lists | `[MIDDLEWARE]` | A list that only applies to new baskets is not a control |

## commercetools

**Entities:** `Store`, `ProductSelection`, `BusinessUnit`, `CustomerGroup`, `Channel`, `Product`, `ProductProjection`, `ShoppingList`, `Cart`, `Order`

**Verified API surface**

- (concept) A Product Selection specifies a subset of the catalog in one of two modes: Individual as an allowlist when the buyer should see a small curated slice, and IndividualExclusion as a denylist when the buyer should see most of the catalog with only a few restricted lines removed — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/design-b2b-catalogs/company-specific-product-catalogs)
- (concept) A Store can reference more than one Product Selection, so two buyers who share most of a catalog and differ at the edges are modeled as a shared Product Selection plus a buyer-specific one on each Store — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/design-b2b-catalogs/resource-clustering)

**Constraints that change the design**

- The mode is fixed when you create the Product Selection, so the choice between allowlist and denylist has to be made on how the assortment will be maintained over time and cannot be reversed later without rebuilding the Selection — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/design-b2b-catalogs/company-specific-product-catalogs)
- Changing the Store a Business Unit points at changes its assortment, pricing and transactional data together, so a Store cannot be used to vary the approved list alone without also moving the account's carts and orders — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/design-b2b-catalogs/resource-clustering)
- Restriction can alternatively be driven through search, with middleware passing the buyer's price Channel, inventory Channel or Customer Group as filter criteria on a Product Search query — a more advanced customization that leaves the restriction outside the platform and therefore outside every surface that does not go through that query — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/design-b2b-catalogs/company-specific-product-catalogs)
- A maximum of 10000 Customer Groups can be created per Project as a soft limit, which is the ceiling on any design that gives each buying organization its own group to express an approved list — [docs](https://docs.commercetools.com/api/projects/customerGroups)

**Modeling notes**

Put the list on the Store as Product Selections and let the Business Unit resolve to it. That is the one construction the platform enforces for you, and enforcement is the point — a list implemented as a search filter is not a list, because quick order, reorder and the Cart API do not go through search. Choose the mode deliberately and early: it cannot be changed afterwards, and a formulary is nearly always an allowlist even when it starts small enough to look like an exclusion. Compose rather than duplicate — a shared Selection for what the whole network standardizes on, plus one per account for its own additions — because a Selection per account over the full catalog becomes unmaintainable at the third revision. Note what the Store approach costs you: moving an account to a different Store moves its carts and orders too, so do not model a mid-life list change as a Store move. Equivalence is separate data and deserves to be: hold the approved-equivalent mapping as its own resource keyed by part number, do not derive it from category, and keep it directional, because the item people ask for and the item the account standardized on are not interchangeable in both directions. Finally, re-resolve saved lists and open carts on read rather than on write; a list changes rarely and is read constantly, and a stale saved list is where off-list items get in.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-platform`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Who owns the approved list — the buying organization, the seller, or a contract between them — and how is a change to it communicated?
- May an off-list item be bought at all with approval, and who in the account holds that authority?
- When a sub-unit's list conflicts with its parent's, which wins, and can a sub-unit ever be wider?
- How current must the list be at the moment of ordering, and what is the acceptable staleness for a cached copy?
