---
name: ahrefs-keyword-and-serp-research
description: >-
  Research keywords and the search results behind them with the Ahrefs API v3 — volume, difficulty and
  traffic potential for seed terms, expansion into matching and related terms, the live SERP for a
  keyword, and which competitors already rank. Use when asked to size a keyword opportunity, build a
  keyword list, or explain who owns a SERP.
api: openapi/_original/ahrefs-openapi-original.json
generated: '2026-08-12'
method: generated
source: derived from openapi/_original/ahrefs-openapi-original.json + https://docs.ahrefs.com/api/docs/free-test-queries.md
operations:
- keywords-explorer.overview
- keywords-explorer.matching-terms
- keywords-explorer.related-terms
- keywords-explorer.volume-by-country
- serp-overview.serp-overview
- site-explorer.organic-keywords
- site-explorer.organic-competitors
- batch-analysis.batch-analysis
---

# Keyword and SERP research with the Ahrefs API v3

## Base and auth

- Base URL: `https://api.ahrefs.com/v3`, bearer API key in `Authorization`.
- `GET` + query string for everything here except `batch-analysis.batch-analysis`, which is `POST`.
- Monetary fields (`traffic_value`, `org_cost`, `paid_cost`, `value`) come back in **USD cents** —
  divide by 100 before displaying.

## Free while developing

Requests are free when the only keyword passed is `ahrefs`, `yep` or `firehose` (Keywords Explorer and
SERP Overview) or when `target` is one of those domains (Site Explorer). Adding any other keyword to
the same request, or passing `keyword_list_id`, makes the whole request billable.

## Steps

1. **Size the seeds.** `keywords-explorer.overview` with `country`, `keywords` (comma-separated) and a
   tight `select` — e.g. `keyword,volume,traffic_potential,difficulty`. This is the cheapest way to
   triage a list before spending units on expansion.
2. **Expand.** `keywords-explorer.matching-terms` (contains the seed) and
   `keywords-explorer.related-terms` (semantically adjacent). Both take `select`, `where`, `order_by`,
   `limit`, `offset` and a `terms` mode. Order by `volume:desc` and cap `limit` — these reports are
   the single biggest unit sink in the API.
3. **Check seasonality and geography.** `keywords-explorer.volume-by-country` for one keyword.
4. **Read the SERP.** `serp-overview.serp-overview` with `country`, `date`, `keyword` and
   `top_positions` to bound the result set. A `type` parameter filters by SERP feature.
5. **See who already ranks.** `site-explorer.organic-keywords` for a target's ranking terms, and
   `site-explorer.organic-competitors` for the domains sharing those SERPs.
6. **Compare many targets at once.** `batch-analysis.batch-analysis` (`POST`) takes up to 100
   targets in one request — always prefer it to a loop of Site Explorer calls.

## Conventions and errors

- `select` is mandatory on most list reports and defines both the payload and the cost.
- `where` takes a JSON filter expression, URL-encoded; `order_by` takes `field:desc,field2:asc`.
  The field names valid in `where` may differ from those valid in `select` — check the per-endpoint
  parameter description in the spec.
- Errors are `{"error": "<message>"}` with `400`/`401`/`403`/`429`/`500` declared on every operation.
- Rate limit is 60 requests/minute plus dynamic throttling; both surface as `429`.
