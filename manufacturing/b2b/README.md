# Industrials & Manufacturing · B2B

[Industrials & Manufacturing](../README.md) · [All industries](../../README.md)

Copy `specs/` into your project's `openspec/specs/`.

| # | Capability | Priority |
|---|---|---|
| 1 | [Authentication and identity](specs/authentication-and-identity/spec.md) | P1 |
| 2 | [Bulk and repeat ordering](specs/bulk-and-repeat-ordering/spec.md) | P1 |
| 3 | [Cart management](specs/cart-management/spec.md) | P1 |
| 4 | [Checkout](specs/checkout/spec.md) | P1 |
| 5 | [Company account setup](specs/company-account-setup/spec.md) | P1 |
| 6 | [Viewing contract and account pricing](specs/contract-and-account-pricing/spec.md) | P1 |
| 7 | [Discovery and browse](specs/discovery-and-browse/spec.md) | P1 |
| 8 | [Order approval](specs/order-approval/spec.md) | P1 |
| 9 | [Purchase order and payment terms](specs/purchase-order-and-payment-terms/spec.md) | P1 |
| 10 | [Quote request and negotiation](specs/quote-request-and-negotiation/spec.md) | P1 |
| 11 | [Requisition and shopping lists](specs/requisition-and-shopping-lists/spec.md) | P1 |
| 12 | [Managing team members and access](specs/team-members-and-access/spec.md) | P1 |
| 13 | [Account and self-service](specs/account-and-self-service/spec.md) | P2 |
| 14 | [Account-level order history](specs/account-level-order-history/spec.md) | P2 |
| 15 | [Free-text order description becomes a cart](specs/free-text-order-description-becomes-a-cart/spec.md) | P2 |
| 16 | [Order input becomes a named shopping list](specs/order-input-becomes-a-named-shopping-list/spec.md) | P2 |
| 17 | [Post-purchase order management](specs/post-purchase-order-management/spec.md) | P2 |
| 18 | [Switching region or language](specs/switching-region-or-language/spec.md) | P2 |
| 19 | [Subscriptions and recurring orders](specs/subscriptions-and-recurring-orders/spec.md) | P3 |
| 20 | [Cart with engine-calculated totals after every change](specs/cart-page/spec.md) | P1 |
| 21 | [Checkout re-reading totals after each shipping change](specs/checkout-page/spec.md) | P1 |
| 22 | [Landing page with session-resolved buyer context](specs/home-landing-page/spec.md) | P1 |
| 23 | [Order confirmation stating reference and true order state](specs/order-confirmation-page/spec.md) | P1 |
| 24 | [Product detail page showing the buyer's effective price](specs/product-detail-page/spec.md) | P1 |
| 25 | [Category listing with request-derived filter and page state](specs/product-listing-page/spec.md) | P1 |
| 26 | [Search results with exact part-number resolution](specs/search-results-page/spec.md) | P2 |
| 27 | [Account request held until the seller activates it](specs/account-registration-request/spec.md) | P1 |
| 28 | [Buyer sign-in with password or federated identity](specs/account-sign-in/spec.md) | P1 |
| 29 | [Address book with a default delivery location](specs/address-book/spec.md) | P1 |
| 30 | [Order history scoped to what the buyer may see](specs/order-history/spec.md) | P1 |
| 31 | [Company user and role administration](specs/user-and-role-management/spec.md) | P1 |
| 32 | [Session-scoped account dashboard with explicit empty states](specs/account-dashboard/spec.md) | P2 |
| 33 | [Order approval rules and the approver queue](specs/approval-workflows/spec.md) | P2 |
| 34 | [Saved payment methods and account payment terms](specs/payment-methods/spec.md) | P2 |
| 35 | [Quote acceptance at the negotiated price](specs/quote-negotiation/spec.md) | P2 |
| 36 | [Saved and requisition lists with bulk add to cart](specs/saved-lists/spec.md) | P2 |
| 37 | [Invoice history and outstanding balance on account](specs/invoice-and-billing-history/spec.md) | P1 |
| 38 | [Quick order and bulk upload with per-line results](specs/quick-order-entry/spec.md) | P1 |
| 39 | [Budget monitoring against committed order spend](specs/budget-and-spending-limits/spec.md) | P2 |
| 40 | [Company profile with permission-gated editing](specs/company-profile/spec.md) | P2 |
| 41 | [Contract pricing shown as the cart will price it](specs/contract-pricing/spec.md) | P2 |
| 42 | [Cost center codes assignable to a purchase](specs/cost-center-management/spec.md) | P3 |
| 43 | [Contact page that confirms only enquiries support has accepted](specs/contact-us/spec.md) | P1 |
| 44 | [Policy pages stating the effective date of the text shown](specs/policy-pages/spec.md) | P1 |
| 45 | [About page assembled entirely from published CMS content](specs/about-us/spec.md) | P2 |
| 46 | [FAQ answers readable and indexable without being expanded](specs/faq/spec.md) | P2 |
| 47 | [Articles addressable and indexable independently of the listing](specs/blog-resources/spec.md) | P3 |
| 48 | [Error pages that name the failure and route the buyer back](specs/error-pages/spec.md) | P1 |
| 49 | [Password reset gated on a valid, unexpired reset token](specs/password-reset/spec.md) | P1 |
| 50 | [Email verification by token with a recoverable resend path](specs/email-verification/spec.md) | P2 |
| 51 | [Parts resolved from the identity of one built unit](specs/fitment-by-unit-identity/spec.md) | P1 |
| 52 | [The buyer's own list of units, held per site](specs/installed-base-register/spec.md) | P1 |
| 53 | [Rival-brand equivalents named without their price or stock](specs/alternate-brand-cross-reference/spec.md) | P2 |
| 54 | [Stock across own and upstream locations, scoped per buyer](specs/network-stock-visibility/spec.md) | P2 |
| 55 | [A withdrawn part number resolving to what replaces it](specs/superseded-part-resolution/spec.md) | P2 |
| 56 | [One agreement pricing each of the buyer's sites differently](specs/agreement-price-by-site/spec.md) | P1 |
| 57 | [Prices owned outside the platform, held steady on the cart](specs/external-price-of-record/spec.md) | P1 |
| 58 | [One price per service territory, unchanged through checkout](specs/territory-resolved-price/spec.md) | P1 |
| 59 | [A minimum charge taking over when the computed total is lower](specs/minimum-charge-floor/spec.md) | P2 |
| 60 | [One order covering several of the buyer's own sites](specs/multi-site-order-composition/spec.md) | P1 |
| 61 | [Invoice recipient and paying account named per destination](specs/bill-to-and-payer-per-destination/spec.md) | P2 |
| 62 | [An order class chosen at submission that sets how it is served](specs/order-class-routing/spec.md) | P2 |
| 63 | [Amending a live agreement, with the part-period charge shown](specs/agreement-amendment-with-proration/spec.md) | P1 |
| 64 | [A base charge with an included allowance, and what exceeds it](specs/included-allowance-and-overage/spec.md) | P2 |
| 65 | [Equipment and its ongoing service bought in one checkout](specs/mixed-oneoff-and-recurring-cart/spec.md) | P2 |
| 66 | [One submitted order split to the back office that owns each part](specs/order-decomposition-by-business-unit/spec.md) | P1 |
| 67 | [An order staying open while its lines ship and backorder](specs/progressive-shipment-and-backorder/spec.md) | P1 |
| 68 | [An order recorded as provisional until the owning system accepts](specs/provisional-order-until-accepted/spec.md) | P1 |
