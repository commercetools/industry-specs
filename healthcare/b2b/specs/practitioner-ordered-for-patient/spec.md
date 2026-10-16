# Ordered by the practice, received by the person it is for

## Purpose

Ordinary commerce assumes one party who chooses, pays and receives. Care-team ordering separates all three, and the separation is not an edge case — it is the normal way clinical goods reach the person who uses them. A practice orders because it holds the entitlement and the professional judgment; it may be billed, or a scheme may be, or the recipient may pay their own share; and the goods go to the recipient's home, not the practice. Collapse these into one customer and something breaks immediately: bill the recipient and the practice's contract price is wrong; ship to the account holder and the goods go to a clinic that did not want them; put the recipient's details on the practice's account and every other person that practice orders for can see them. The individual within the practice matters too, and for a different reason. Where an order carries clinical judgment, "the practice ordered it" is not a sufficient record — afterwards, somebody always needs to know which member of staff acted, and reconstructing that from a shared login is not possible.

## Requirements

### Requirement: Ordered by the practice, received by the person it is for

The system SHALL record the ordering practice, the individual within it who placed the order, and the person the goods are for as three separate parties on an order placed on someone else's behalf, so that entitlement, billing and delivery can each resolve to a different one.

#### Scenario: Three parties recorded
- **GIVEN** a practice ordering goods for one of the people in its care
- **WHEN** the order is placed
- **THEN** the practice, the acting individual and the recipient are all recorded on it

#### Scenario: Billed to a party other than the recipient
- **GIVEN** an order whose cost falls to the practice
- **WHEN** it is placed
- **THEN** the practice is billed at its own terms and the recipient is not charged

#### Scenario: Delivered to the recipient
- **GIVEN** an order placed by a practice for a recipient at home
- **WHEN** it is fulfilled
- **THEN** it is delivered to the recipient's address, and the practice sees that it was

#### Scenario: Recipient without an account
- **GIVEN** a recipient who has never signed in and has no credentials
- **WHEN** an order is placed for them
- **THEN** the order completes and they receive dispatch information without needing an account

#### Scenario: Recipients isolated from one another
- **GIVEN** a practice that orders for many people
- **WHEN** any one recipient's record is read
- **THEN** no other recipient's details appear in it

#### Scenario: Acting individual is attributable
- **GIVEN** a completed order carrying clinical judgment
- **WHEN** it is examined afterwards
- **THEN** the individual who placed it is identifiable, not only the organization

#### Scenario: Recipient pays their own share
- **GIVEN** an order in which the recipient owes part of the cost
- **WHEN** payment is taken
- **THEN** the recipient's share is collected from them and the remainder from the billed party

## Components

Data source tags: `[STATIC]` served from CDN with no middleware call; `[CACHED]` one shared middleware call at build or cache expiry; `[MIDDLEWARE]` called per request because the response is session-specific.

| Component | Data Source | Notes |
| --- | --- | --- |
| Ordering organization on the order | `[MIDDLEWARE]` | The practice, clinic or team that holds the entitlement |
| Acting individual identified within it | `[MIDDLEWARE]` | Which member of staff placed it, not just which account |
| Recipient recorded as a party in their own right | `[MIDDLEWARE]` | Not merely a shipping address on the practice's order |
| Billing party resolved independently | `[MIDDLEWARE]` | The practice, a scheme, or the recipient — decided per order |
| Delivery to the recipient without exposing the account | `[MIDDLEWARE]` | Dispatch and tracking reach them without an account login |
| Recipient isolation from the practice's other orders | `[MIDDLEWARE]` | One recipient is never visible in another's record |
| Recipient who holds no account at all | `[MIDDLEWARE]` | Most recipients never sign in, and must not have to |

## commercetools

**Entities:** `BusinessUnit`, `Associate`, `AssociateRole`, `Customer`, `Cart`, `Order`, `Type`, `CustomObject`, `Store`, `ApprovalRule`

**Verified API surface**

- (concept) Associate Carts and Associate Orders allow Associates to view, create, update and delete Carts and Orders on behalf of a Business Unit, through as-associate/{associateId}/in-business-unit/key={businessUnitKey} endpoints, with the permitted actions depending on the roles and permissions that Associate holds — [docs](https://docs.commercetools.com/api/projects/associate-carts)
- (concept) Creating a Cart in a Business Unit as an Associate requires the CreateMyCarts or CreateOthersCarts Permission, and a missing permission returns an AssociateMissingPermission error — so acting for someone else is an explicit, separately granted right — [docs](https://docs.commercetools.com/api/projects/associate-carts)
- (concept) The Customer is the authentication identity and the Associate is that Customer's membership of a Business Unit with assigned roles; an Associate references exactly one Customer, but one Customer can be an Associate of many Business Units — [docs](https://docs.commercetools.com/learning-model-b2b-commerce/configure-associate-access/customers-and-associates)

**Constraints that change the design**

- The as-associate model gives an Order an owning Business Unit and an acting Associate, but it has no third party: there is no recipient distinct from the Customer and the shipping address, so the person the goods are for is Custom Field data the platform will not relate, scope or protect — [docs](https://docs.commercetools.com/api/projects/associate-orders)
- Every customizable resource can be extended with only one Type at a time, so the recipient reference shares its Type with every other custom field the Order carries, and that Type has to be designed once for all of them — [docs](https://docs.commercetools.com/learning-model-your-business-structure/extensibility/data-model-extensions)

**Modeling notes**

Use the as-associate endpoints for the two parties the platform understands. The Business Unit is the practice, the Associate is the member of staff, and the permission split between CreateMyCarts and CreateOthersCarts is already the distinction you need between acting for yourself and acting for another — do not rebuild it in middleware. The recipient is the part the platform does not model, and the temptation is to make them a Customer under the practice. Think hard before doing so: a Customer is an authentication identity, most recipients will never authenticate, and creating dormant identities for people who did not ask for them manufactures an access-control problem and a data-retention obligation at the same time. The lighter shape is a recipient reference in Custom Fields on the Order, resolvable in the system that actually owns the relationship, with the delivery address taken from there — and see the companion capability on keeping health data out of commerce, because this reference is exactly where it creeps in. Whichever you choose, do not rely on the Business Unit to isolate recipients from one another: it scopes the practice's staff, not the people they order for, so if any recipient can ever read their own order, scoping is yours to enforce. Record the acting Associate on the order at placement rather than deriving it later from an audit trail; trails get truncated and the question outlives them.

## commercetools skills

Load `commercetools-commerce-patterns` before implementing this capability. Supporting: `commercetools-platform`, `commercetools-storefront`. Any task generated from this spec carries `[SKILL: commercetools-commerce-patterns]`.

## Open questions

- Is the recipient ever a Customer in their own right, and what happens when a person is both a recipient and a direct buyer?
- Who may order for whom inside a practice, and is that a role, a per-recipient assignment, or both?
- Which party's terms and prices apply when the recipient pays their own share?
- What may a recipient see of an order placed for them, and how do they see it without an account?
