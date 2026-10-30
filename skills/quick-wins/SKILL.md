---
name: quick-wins
description: Find title/meta-description quick wins in a property's Google Search Console data through the Organiq MCP server (score_opportunities, find_page_poaching_opportunities, get_ctr_curve, get_breakdown). Use when the user asks for quick wins, low-hanging fruit, queries with CTR below expected, striking-distance keywords, what to optimize first this week, or pages close to page one.
license: Apache-2.0
metadata:
  author: Organiq
  version: "1.0"
---

# Quick wins (CTR gap + striking distance)

Goal: a short, ranked list of queries/pages where a title or meta-description rewrite, or a small content push, is likely to add clicks soon. Everything comes from the user's own Search Console data via the Organiq connector; never invent numbers.

## Steps

1. Resolve the property: `list_sites` if the name is not exact. Note `dataTo` (the last synced date; periods end there, not today).
2. Baseline: `get_site_summary` `{site, days: 28}` for totals and the comparison window.
3. CTR gaps: `score_opportunities` `{site, days: 28, minImpressions: 20, limit: 25}`. Keep `level` high/medium. `ctrGap` is a fraction; `lostClicks = impressions × ctrGap`; `ctrSource` says whether the expected CTR came from the property's own curve or the generic benchmark.
4. Striking distance: `find_page_poaching_opportunities` `{site, days: 28, minPosition: 4, maxPosition: 20, targetPosition: 3, sort: "gain"}`. `estimatedGain` is clicks per period if the query reached the target position's CTR.
5. Where titles underperform: `get_ctr_curve` `{site, days: 28}`; buckets with `reliable: true` and negative `deltaPoints` point to systematic title problems at that position range.
6. Context: `get_breakdown` `{site, days: 28}` for brand share and how much traffic sits in positions 4–10 and 11–20.
7. For the top candidates, `query_top_pages` with `query: "<the query>"` to see which URL ranks, then draft the rewrite.

## Output

A table: query, page, position, impressions, ctr, expected ctr, estimated gain, suggested action (title, meta, intro, internal links). State the period and comparison used (every response includes them) and whether the expected CTR was own or benchmark.

## Small sites

If few rows come back, lower `minImpressions` to 5–10 and widen `days` to 90 before concluding there is nothing to do.

## Units

`ctr` and `ctrGap` are fractions (0.052 = 5.2%); `position` is impression-weighted and lower is better; dates are yyyy-mm-dd.
