---
name: Find and price a Landing stay
description: Search Landing's furnished apartments in a US market for specific dates, pick a home, and get a real, bookable all-in quote — then hand the guest a checkout link.
api: openapi/landing-public-openapi.json
operations: [markets, search, home, quote]
mcp_tools: [list_markets_tool, search_apartments_tool, get_apartment_tool, get_quote_tool]
generated: '2026-07-19'
method: generated
source: openapi/landing-public-openapi.json
---

# Find and price a Landing stay

Landing rents fully-furnished apartments in 250+ US markets. The public API is
**unauthenticated and read-only** — send no keys, no tokens, no headers.

Base URL: `https://www.hellolanding.com`
MCP endpoint: `https://www.hellolanding.com/mcp/public` (JSON-RPC over POST)

Use the MCP tools if your environment has an MCP client; otherwise use the
identical HTTP GET mirror. Dates are always `YYYY-MM-DD`.

## Steps

1. **Resolve the market.** Call `markets` (`GET /api/public/markets`, MCP
   `list_markets_tool`). Match the guest's city to a `slug` — never guess a
   slug. Optionally filter with `state`.

2. **Optional — validate the filters.** Call `market_filters`
   (`GET /api/public/market-filters?market=<slug>`, MCP
   `get_market_filters_tool`) before searching if the guest named amenities,
   bedroom counts, or a sort order. It returns the valid vocabulary for that
   market, so you don't send a filter the market will reject.

3. **Search.** Call `search`
   (`GET /api/public/search?market=<slug>&check_in=YYYY-MM-DD&check_out=YYYY-MM-DD`,
   MCP `search_apartments_tool`). `market` is required; `check_in`,
   `check_out`, `bedrooms` and `max_price` are optional.
   - For an open-ended **LandingFlex** stay pass `check_out=indefinite` and
     pair it with a committed-nights tier (step 4a).
   - Results are paginated: `page` (1-indexed, default 1) and `per_page`
     (default 25, max 50). The response carries `total_count` and
     `total_pages`; keep incrementing `page` while `page <= total_pages`.
   - Pass `include_hero_images: false` to shrink the payload when you are not
     rendering photos.

4. **Fetch the home.** Call `home` (`GET /api/public/homes/{slug}`, MCP
   `get_apartment_tool`) for the chosen slug. This already returns the full
   availability calendar (bookable windows, earliest available date) and a
   pre-filled `reservation_link` — do **not** probe dates one by one to
   discover availability.

   4a. **Flex only.** If the stay is open-ended, call `flex_options`
   (`GET /api/public/homes/{slug}/flex-options`, MCP `get_flex_options_tool`)
   to get the commitment tiers *that home* accepts, and only offer those.

5. **Price it.** Call `quote`
   (`GET /api/public/homes/{slug}/quote?check_in=…&check_out=…`, MCP
   `get_quote_tool`). Add `cats` / `dogs` when the guest has pets. This is the
   real anonymous quote from the same pricing engine as checkout — monthly rent
   after seasonal and length-of-stay adjustments, the first-month charge, the
   ongoing monthly cost, and an estimated total. **Quote from this call only;
   never estimate a price yourself and never extrapolate from a search result.**

6. **Hand off.** The API cannot complete a reservation — payment happens on the
   website. Give the guest `checkout_url` (straight to payment) or `home_url`
   (the home's page with dates pre-filled). Over MCP you may instead call
   `booking_intent_tool` to record their interest and get a direct checkout link.

## Rules

- **No authentication.** Adding an `Authorization` header is never required.
- **Errors** come back as `{"error": {"code", "message"}}` with `400` for bad
  input and `404` for an unknown home or market. The messages name the recovery
  tool — on `not_found` for a market, re-call `markets`; for a home, re-run
  `search`, because published inventory changes as homes are booked.
- **Retries.** All the operations above are safe `GET`s and can be retried
  freely. `booking_intent_tool` is the one write and has **no idempotency key**
  — call it once per guest intent; do not blind-retry it.
- **Freshness.** Inventory and pricing are live. Re-run `search` and re-`quote`
  rather than reusing results from earlier in a long conversation.
- **Tracing.** Responses carry `x-request-id`; quote it when escalating to
  support@hellolanding.com.
- **Don't scrape.** `robots.txt` steers agents to `/api/public` and
  `/mcp/public` specifically so that scraping the site is unnecessary.
