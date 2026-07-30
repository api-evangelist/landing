---
name: Compare Landing markets and answer policy questions
description: Use Landing's market statistics to compare cities on cost and inventory, and answer deposit/pet/cancellation policy questions from Landing's grounded FAQ instead of guessing.
api: openapi/landing-public-openapi.json
operations: [markets, market_info, market_filters, policies_faq, status]
mcp_tools: [list_markets_tool, get_market_info, get_market_filters_tool, get_policies_faq_tool]
generated: '2026-07-19'
method: generated
source: openapi/landing-public-openapi.json
---

# Compare Landing markets and answer policy questions

Two research jobs that do **not** require paging through inventory. The API is
unauthenticated and read-only.

Base URL: `https://www.hellolanding.com`
MCP endpoint: `https://www.hellolanding.com/mcp/public`

## A. Compare markets on cost and availability

Use this for "is Austin or Dallas cheaper for a 1BR?" — do not answer it by
running `search` in both cities and averaging the cards.

1. Call `markets` (`GET /api/public/markets`, MCP `list_markets_tool`) to
   resolve each city name to a slug. `state` narrows the list.
2. Call `market_info` (`GET /api/public/market-info?market=<slug>`, MCP
   `get_market_info`) once per market. It returns average monthly rent by
   bedroom count, total published inventory, and the neighborhoods with the
   most availability.
3. Compare the like-for-like bedroom figures across markets, and report
   inventory depth alongside price — a cheaper market with thin inventory is a
   worse recommendation.
4. Optionally call `market_filters`
   (`GET /api/public/market-filters?market=<slug>`, MCP
   `get_market_filters_tool`) to tell the guest what is actually filterable in
   that market (amenities, bathroom options, neighborhoods, sort orders).
5. Hand off to the find-and-price skill (`search` → `quote`) once the guest
   picks a market. Market averages are context, **not** a quotable price.

## B. Answer a policy question

1. Call `policies_faq` (`GET /api/public/policies-faq?q=<question>`, MCP
   `get_policies_faq_tool`) for deposits, pets, parking, cancellation,
   qualification, LandingFlex, utilities, and move-in.
2. Return the grounded answer as given. Where the exact figure depends on the
   home, market, or reservation, the answer says so explicitly — **preserve
   that hedge; do not resolve it into a number.**
3. When a specific amount is needed, route to the real source: `quote` for
   pricing and first-month charges, the checkout flow, or Landing support at
   support@hellolanding.com.
4. If `policies_faq` has no grounded answer, say so and point to
   https://www.hellolanding.com/help-center rather than inferring policy.

## Rules

- **Never invent policy or price.** Both tools exist specifically so an agent
  does not have to guess; the FAQ is deliberately conservative.
- **Errors**: `{"error": {"code", "message"}}`, `400` bad input / `404` unknown
  market. On `not_found`, re-call `markets` for valid slugs.
- All operations here are safe `GET`s — retry freely.
- `status` (`GET /api/public/status`, returns `{"status":"ok"}`) is the health
  check if you need to confirm the service is reachable before a batch of calls.
