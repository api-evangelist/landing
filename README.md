# Landing

Landing rents fully-furnished apartments for flexible monthly, short-term, and open-ended
(LandingFlex) stays across 250+ US markets — booked entirely online, no security deposit.

Landing publishes a **public, unauthenticated, read-only API** in two transports that expose the
same capabilities: a plain HTTP GET REST API (OpenAPI 3.1) at
[`/api/public`](https://www.hellolanding.com/api/public) and a public MCP server at
[`/mcp/public`](https://www.hellolanding.com/mcp/public). Discovery is wired end to end — an
RFC 9727 [api-catalog](https://www.hellolanding.com/.well-known/api-catalog), an
[llms.txt](https://www.hellolanding.com/llms.txt) agent guide, an
[agent-skills index](https://www.hellolanding.com/.well-known/agent-skills/index.json),
schema.org JSON-LD, and a `robots.txt` that steers agents to the API instead of the scraper.

Reservations are not exposed: read tools return `checkout_url` and a person completes payment on
the website.

Backed by: foundry-group — https://www.hellolanding.com
