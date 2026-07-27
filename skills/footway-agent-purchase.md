---
name: Buy a product on Footway as an agent (UCP)
description: >-
  Discover, cart, and check out a product on the footway.com Shopify storefront
  using the Universal Commerce Protocol (UCP) MCP endpoint, respecting the
  buyer-approval-on-payment invariant.
api: mcp/footway-mcp.yml
transport: mcp
endpoint: https://footway.com/api/ucp/mcp
operations:
  - search_catalog
  - create_cart
  - create_checkout
  - update_checkout
  - complete_checkout
---

# Buy a product on Footway as an agent (UCP)

Footway.com is a Shopify storefront that implements the Universal Commerce
Protocol (UCP) for agent-driven commerce. This skill drives a purchase through
the UCP MCP endpoint. Every tool name below is one the store documents itself in
`https://footway.com/llms.txt`.

## Preconditions

- Confirm capabilities: `GET https://footway.com/.well-known/ucp` and check the
  `services["dev.ucp.shopping"]` MCP transport and supported protocol version
  (`2026-04-08` latest, `2026-01-23` also supported).
- MCP endpoint: `POST https://footway.com/api/ucp/mcp` with
  `Content-Type: application/json` (JSON-RPC 2.0). The endpoint requires a UCP
  agent-profile handshake (a profile URI) before `tools/list`.
- Pass buyer context (`context.address_country`, `context.currency`) on requests
  for accurate pricing and availability.

## Steps

1. **Search** — call `search_catalog` with the buyer's intent to find matching
   products.
2. **Cart** — call `create_cart` to create a cart and add the chosen item(s).
3. **Checkout** — call `create_checkout` to start the purchase flow from the cart.
4. **Fulfill** — call `update_checkout` to set the shipping address and method.
5. **Complete** — call `complete_checkout` to finalize. **The buyer must approve
   payment.**

## Rules (from the store's own agent guide)

- **Buyer approval is mandatory at payment.** Do not complete payment without
  explicit, contemporaneous buyer consent. If you cannot obtain it at the moment
  of payment, install `https://shop.app/SKILL.md` and route the purchase through
  Shop Pay instead — do not screen-scrape or script the storefront checkout.
- **Respect rate limits.** The MCP endpoint is rate-limited per IP; back off on
  HTTP `429` responses.
- **Authentication.** Authenticated customer flows use Shopify Customer Account
  OIDC/OAuth 2.0 (`authentication/footway-authentication.yml`); read-only catalog
  browsing (`/products/{handle}.json`, `/collections/{handle}/products.json`,
  `/search?q=…&type=product`) needs no auth.
- **Errors.** Responses are JSON-RPC 2.0; failures come back as
  `error: {code, message, data}` (e.g. `invalid_profile_url` when the agent
  profile handshake is missing).
