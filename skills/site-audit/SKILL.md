---
name: site-audit
description: Run a full organic-search audit of one property from its own Google Search Console and GA4 data through the Organiq MCP server (get_site_summary, get_timeseries, get_breakdown, query_top_queries, query_top_pages, query_countries, query_devices, find_decaying_content, analyze_cannibalization, score_opportunities, find_page_poaching_opportunities, get_ctr_curve, get_annotations, get_ai_traffic). Use when the user asks for an SEO audit, a health check, a first look at a site, an onboarding report for a new client, or "what is wrong with my site".
license: Apache-2.0
metadata:
  author: Organiq
  version: "1.0"
---

# Site audit (one property, one pass)

Goal: a complete, dated diagnosis of one property in one pass: where the traffic is, what moved and why, the structural problems (decay, cannibalization, CTR gaps) and a prioritized action list. Run the tools in the order below; each answer already states the period and comparison used. Never invent numbers; if a tool returns `no_data`, say so and move on.

## Steps

1. Scope: `list_sites` (or `check_connection`) for the property, `dataFrom`/`dataTo` and whether GA4 is connected. Use `days: 28` with `compare: "previous"` for the recent view and `days: 365, granularity: "month"` for the trajectory.
2. Headline: `get_site_summary` `{site, days: 28, compare: "previous"}` and `{site, days: 28, compare: "year"}`.
3. Trajectory: `get_timeseries` `{site, days: 365, granularity: "month"}`; locate the month a decline or growth started.
4. Events: `get_annotations` `{site, days: 365}` (deploys, Google algorithm updates) near the inflection.
5. Mix: `get_breakdown` `{site, days: 28}` (brand vs non-brand; position bands 1–3, 4–10, 11–20), `query_countries` and `query_devices` `{site, days: 28, compare: "previous"}`.
6. Movers: `query_top_queries` and `query_top_pages` `{site, days: 28, compare: "previous", sort: "growth", limit: 10}` and `sort: "decline"`.
7. Decay: `find_decaying_content` `{site, days: 28, compare: "previous"}` and, when history allows, with a baseline 6–12 months earlier (`comparisonStartDate`/`comparisonEndDate`).
8. Cannibalization: `analyze_cannibalization` `{site, days: 90, minImpressions: 10, minPages: 2, limit: 25}` (large sites: `minImpressions: 100`); discard URL variants (http/https, www, trailing slash, fragments) and brand sitelinks.
9. Opportunities: `score_opportunities` `{site, days: 28, minImpressions: 20, limit: 15}` and `find_page_poaching_opportunities` `{site, days: 28, minPosition: 4, maxPosition: 20, targetPosition: 3, sort: "gain", limit: 15}`; `get_ctr_curve` `{site, days: 28}` to see whether whole position bands underperform (title problems).
10. AI assistants: `get_ai_traffic` `{site, days: 28, compare: "previous"}` when GA4 is connected.

## Output

One report with these sections, numbers always next to their period:

1. Headline (clicks, impressions, CTR, position; change vs previous period and vs last year).
2. Trajectory and events (when it started; nearby deploys or Google updates).
3. Mix (brand share; position bands; countries and devices worth noting).
4. What grew and what declined (top 5 queries and pages each way).
5. Decaying pages (clicks lost, pattern, likely cause) ordered by clicks at stake.
6. Cannibalization clusters (query, entropy, winner, contenders, fix), real ones only.
7. Quick wins (CTR gap and striking distance) with the suggested change per row.
8. Traffic from AI assistants (or "GA4 not connected").
9. Action plan: at most 10 items, each with the expected effect, the evidence row it comes from, and a horizon (2–4, 4–8 or 8–12 weeks). Order by clicks at stake.
10. Limits: data ends on `dataTo`; Search Console lags about 3 days; clicks inside AI Overviews are not separated; keyword volume and backlinks are not in this data.

## Small sites

If few rows come back, lower `minImpressions` to 5–10 and widen `days` to 90 before concluding that a section is empty.

## Units

`clicks`/`impressions` integers; `ctr` fraction 0–1 (write it as %); `position` impression-weighted, lower is better; `entropy` in bits; dates yyyy-mm-dd; periods end on the last synced date.
