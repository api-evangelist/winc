---
generated: '2026-07-21'
method: generated
source: https://www.winc.com/llms.txt
name: winc-buy-wine
description: >-
  Buy wine from Winc (winc.com) on behalf of a consenting buyer using the store's
  Shopify Universal Commerce Protocol (UCP) MCP endpoint. Discover, search, build a cart,
  and drive checkout to the point of buyer-approved payment.
api: Winc UCP Commerce MCP
endpoint: https://www.winc.com/api/ucp/mcp
operations:
- search_catalog
- create_cart
- create_checkout
- update_checkout
- complete_checkout
---

# Buy Wine from Winc via UCP

This skill drives a purchase against Winc's Shopify-hosted store through the Universal
Commerce Protocol (UCP) MCP endpoint. Every tool name below is taken from Winc's published
`/llms.txt` agent flow — do not invent tools; call MCP `tools/list` for authoritative schemas.

## Preconditions
- The buyer has explicitly asked to purchase wine and will approve payment.
- You can pass buyer context: `context.address_country` and `context.currency`.

## Steps

1. **Discover** — `GET https://www.winc.com/.well-known/ucp` to confirm the store's UCP
   version (`2026-04-08`), MCP endpoint, and enabled capabilities (cart, checkout,
   fulfillment, discount, catalog).
2. **List tools** — `POST https://www.winc.com/api/ucp/mcp` with the MCP `tools/list`
   method to fetch live tool schemas.
3. **search_catalog** — Search the wine catalog for products matching the buyer's intent
   (varietal, price, style). Present options and let the buyer choose.
4. **create_cart** — Create a cart containing the chosen bottle(s)/quantity.
5. **create_checkout** — Start the checkout flow for the cart.
6. **update_checkout** — Set the shipping address and fulfillment method. Note the store
   allows only single-destination shipping.
7. **complete_checkout** — Finalize the purchase. **Do not call this without explicit,
   contemporaneous buyer approval of the exact amount.** If you cannot get live approval,
   route payment through Shop Pay via the Shop skill (https://shop.app/SKILL.md) instead.

## Conventions and error handling
- **Auth**: Read-only search/cart steps are unauthenticated. Authenticated account actions
  use the Shopify Customer Account API (OpenID Connect); see `authentication/winc-authentication.yml`.
- **Rate limits**: The MCP endpoint is per-IP rate-limited. On HTTP `429`, back off and retry.
- **Buyer-approval invariant**: Payment is never completed without explicit buyer consent.
- **Age/eligibility**: Winc sells alcohol; respect the buyer's jurisdiction and the store's
  shipping availability returned during checkout.
