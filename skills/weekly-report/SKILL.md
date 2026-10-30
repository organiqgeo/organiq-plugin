---
name: weekly-report
description: Produce a weekly organic-search report for a property from Google Search Console and GA4 data through the Organiq MCP server (get_site_summary, query_top_queries, query_top_pages, get_annotations, get_ai_traffic). Use when the user asks for a weekly or monthly SEO report, a status update on a site, what changed this week, or a summary for a client.
license: Apache-2.0
metadata:
  author: Organiq
  version: "1.0"
---

# Weekly report

Goal: one page a site owner or client can read in two minutes: what moved, why, and the three actions for next week.

## Steps

1. `list_sites` to confirm the property and `dataTo` (the report week ends there).
2. `get_site_summary` `{site, days: 7, compare: "previous"}` and `{site, days: 7, compare: "year"}`: clicks, impressions, CTR, position with `change`.
3. Movers: `query_top_queries` `{site, days: 7, compare: "previous", sort: "growth", limit: 10}` and `sort: "decline"`; the same with `query_top_pages`.
4. Brand context: `get_breakdown` `{site, days: 7}` (brand share, position bands).
5. Events: `get_annotations` `{site, days: 30}` (deploys, Google updates).
6. AI assistants: `get_ai_traffic` `{site, days: 7, compare: "previous"}` when `hasGa4` is true; `site: "all"` for an account-wide line.
7. Optional: `score_opportunities` `{site, days: 28, limit: 5}` for next week's quick wins.

## Output

Sections: headline numbers with week-over-week and year-over-year change; top gainers and losers (queries and pages); brand vs non-brand; traffic from AI assistants; events that may explain changes; three actions. Quote the period (start–end) next to the numbers. Use percentage points for CTR change (`ctrPoints`) and say "position improved" when the number went down.

## Units

`ctr` fraction 0–1 (write it as %); `position` impression-weighted, lower is better; `clicksPct` is a fraction.
