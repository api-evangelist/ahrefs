---
name: ahrefs-backlink-audit
description: >-
  Audit the backlink profile of a domain or URL with the Ahrefs API v3 — authority, link volume,
  referring domains, anchor-text distribution and broken inbound links — while keeping API-unit spend
  under control. Use when asked to assess a site's link profile, find lost or broken backlinks, or
  compare referring-domain quality.
api: openapi/_original/ahrefs-openapi-original.json
generated: '2026-08-12'
method: generated
source: derived from openapi/_original/ahrefs-openapi-original.json + https://docs.ahrefs.com/api/docs/limits-consumption.md
operations:
- site-explorer.domain-rating
- site-explorer.backlinks-stats
- site-explorer.refdomains
- site-explorer.all-backlinks
- site-explorer.broken-backlinks
- site-explorer.anchors
- subscription-info.limits-and-usage
---

# Backlink audit with the Ahrefs API v3

## Base and auth

- Base URL: `https://api.ahrefs.com/v3`
- Every request: `Authorization: Bearer $AHREFS_API_KEY`. Never hardcode the key.
- All operations here are `GET` with query-string parameters. Every value must be URL-encoded.
- Responses are `application/json` by default; `output=xml` switches to XML.

## Before you start — budget the call

Ahrefs meters in **API units**: `max(50, per_row_cost * rows)`. `per_row_cost` is the sum of the
per-field costs of every **unique** field named in `select`, `where` or `order_by` (default 1 unit,
some metrics cost 5 or 10). So:

- Always pass `select` with only the columns you need. Omitting it on a list report pulls every
  column and multiplies cost.
- Always pass `limit`. Never pull thousands of rows to answer a question about the top 50.
- While developing, use the free targets `ahrefs.com`, `yep.com` or `firehose.com` — requests against
  those consume no units (`limit` is capped at 100 on free requests).
- Check remaining allowance first with `subscription-info.limits-and-usage`.
- Read `x-api-units-cost-total` / `x-api-units-cost-total-actual` / `x-api-cache` on every response to
  learn what the call really cost. Cached responses cost nothing.

## Steps

1. **Baseline the authority.** `site-explorer.domain-rating` — requires `target` and `date`
   (`YYYY-MM-DD`). Returns `domain_rating` and `ahrefs_rank`. Costs the 50-unit floor.
2. **Size the profile.** `site-explorer.backlinks-stats` for total backlinks, referring domains and
   the live/lost split for the same `target` and `date`.
3. **Rank the referring domains.** `site-explorer.refdomains` with `select` limited to the columns
   you will actually report, `order_by=domain_rating:desc` and a small `limit`. Use `mode` to choose
   `exact`, `prefix`, `domain` or `subdomains` matching of the target.
4. **Pull the link rows that matter.** `site-explorer.all-backlinks`, filtered with `where` rather
   than pulled and filtered client-side — a filter expression costs units for the fields it names,
   but far fewer than fetching every row. Example filter shape:
   `where={"and":[{"field":"traffic","is":["gt",1000]},{"field":"refdomains_source","is":["gt",10]}]}`.
5. **Find the breakage.** `site-explorer.broken-backlinks` returns inbound links pointing at URLs that
   no longer resolve — the actionable reclamation list.
6. **Check anchor-text distribution.** `site-explorer.anchors` for over-optimization and brand/naked
   URL balance.

## Conventions and errors

- Dates are always `YYYY-MM-DD` strings.
- Paginate with `offset` + `limit` on list reports; there is no cursor.
- The error envelope is a flat object: `{"error": "<message>"}` — not RFC 9457 problem+json. Every
  operation declares `400`, `401`, `403`, `429` and `500`.
- `429` means either the flat 60-requests-per-minute ceiling or dynamic load-based throttling. Back
  off and retry; do not parallelise harder.
- `401`/`403` usually mean an expired key (keys live one year), a key whose creator left the
  workspace, or an endpoint outside the plan's entitlement.
