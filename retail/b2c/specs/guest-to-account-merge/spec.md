# A guest session joining an existing account without duplicating it

## Purpose

Most retail shoppers buy as guests and register later, so duplicate customer records are created continuously rather than as a one-off migration problem. Every duplicate splits the order history, the loyalty balance and the marketing consent for one real person, and each of those is far harder to reconcile after the fact than to prevent at sign-in.

## Requirements

### Requirement: A guest session joining an existing account without duplicating it

The system SHALL attach a guest session's basket and completed purchase to the shopper's existing account when they sign in or register with an identity that account already holds, rather than creating a second account for the same shopper.

#### Scenario: Guest basket follows sign in
- **GIVEN** a guest with items in a basket who has no other saved basket
- **WHEN** they sign in
- **THEN** the basket is attached to their account and survives onto their next session and device

#### Scenario: Two baskets one shopper
- **GIVEN** a guest basket and a saved basket already on the account
- **WHEN** the shopper signs in
- **THEN** the configured merge rule is applied consistently and the shopper is shown the resulting basket

#### Scenario: Registration finds the existing account
- **GIVEN** a guest registering with an identity an account already holds
- **WHEN** they submit the registration
- **THEN** no second account is created, and they are routed to sign in or recover the existing one

#### Scenario: Guest order claimed
- **GIVEN** a shopper who bought as a guest and later registers with the same identity
- **WHEN** the account is created
- **THEN** the guest purchase appears in their order history

#### Scenario: Prompt after guest purchase
- **GIVEN** a guest who has just completed a purchase
- **WHEN** the confirmation is shown
- **THEN** creating an account from that purchase is offered without re-entering the details

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Guest basket carried into the account | `[MIDDLEWARE]` | The anonymous cart, resolved at sign-in |
| Merge rule for two live baskets | `[MIDDLEWARE]` | Decided once, applied every time |
| Duplicate detection on the identity | `[MIDDLEWARE]` | Before the account is created, not after |
| Guest orders claimed onto the account | `[MIDDLEWARE]` | Past purchases made with the same identity |
| Register prompt after a guest purchase | `[STATIC]` | Offered at confirmation, while intent is high |

## commercetools

**Entities:** `Customer`, `Cart`, `Order`, `Store`

**Verified API surface**

- (concept) Pass anonymousCart or anonymousId on CustomerSignin or CustomerDraft to have the platform attach the guest cart at sign-in or sign-up; the older anonymousCartId field still exists but is deprecated — [docs](https://docs.commercetools.com/learning-implement-carts-and-shopping-lists/implement-carts/manage-cart-sessions-and-merges)
- (concept) anonymousCartSignInMode decides between MergeWithExistingCustomerCart and UseAsNewActiveCustomerCart - which of the two live baskets wins is a configured choice, not a default to be discovered in production — [docs](https://docs.commercetools.com/learning-implement-carts-and-shopping-lists/implement-carts/manage-cart-sessions-and-merges)
- (concept) Where the customer has no pre-existing cart the case is simply setting customerId on the guest cart; the built-in merge only matters when two carts exist — [docs](https://docs.commercetools.com/learning-implement-carts-and-shopping-lists/implement-carts/manage-cart-sessions-and-merges)

**Constraints that change the design**

- If the built-in behaviors do not match the business rule, the documented route is to omit anonymousCart and anonymousId entirely and run your own merge after authentication - a partial reliance on both is what produces inconsistent baskets — [docs](https://docs.commercetools.com/learning-implement-carts-and-shopping-lists/implement-carts/manage-cart-sessions-and-merges)
- A guest cart that is never attached to a customer is eventually deleted, so claiming it at sign-in is what prevents it being silently orphaned — [docs](https://docs.commercetools.com/learning-implement-carts-and-shopping-lists/implement-carts/manage-cart-sessions-and-merges)

**Modeling notes**

Pick one merge strategy and use it everywhere: either the platform's, by always passing the anonymous cart identifiers and setting the sign-in mode, or your own, by never passing them and merging after authentication. Half of each is the configuration that produces baskets nobody can explain. Duplicate prevention has to happen before the account is written, because afterwards you are merging customer records rather than preventing them - and merging carries consequences the platform cannot decide for you, such as which loyalty balance and which marketing consent survive. Note that identity matching is a business rule, not a technical one: matching on an email address alone will both miss real duplicates and merge two people who share a household address, so agree the rule with the business before implementing it.

## commercetools skills

Load `commercetools-storefront` before implementing this capability. Supporting: `commercetools-platform`, `commercetools-connect`. Any task generated from this spec carries `[SKILL: commercetools-storefront]`.

## Open questions

- What identity defines the same shopper — email alone, or email with another factor?
- When two baskets exist, does the guest basket win, the saved one, or the union?
- Which record wins for loyalty balance and marketing consent when duplicates are reconciled?
- Are duplicates already in the data, and does this need a one-off reconciliation as well?
