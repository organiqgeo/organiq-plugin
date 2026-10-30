---
name: content-decay
description: Find, triage and build a recovery plan for decaying content in Google Search Console data through the Organiq MCP server (find_decaying_content, get_timeseries, query_top_queries, get_annotations). Use when the user asks about pages losing traffic, declining clicks, content refresh candidates, which pages are dying, or why a page dropped.
license: Apache-2.0
metadata:
  author: Organiq
  version: "1.0"
---

# Content decay

Goal: the pages (or queries) that lost the most clicks against a baseline, a diagnosis per page, and a refresh plan ordered by clicks at stake.

## Steps

1. Resolve the property with `list_sites` if needed; note `dataFrom`/`dataTo` so the baseline exists.
2. Trajectory: `get_timeseries` `{site, days: 365, granularity: "month"}` to see when the decline started.
3. Decaying pages: `find_decaying_content` `{site, days: 28, compare: "previous"}` for recent decay, and `{site, days: 28, comparisonStartDate, comparisonEndDate}` with a window 6–12 months earlier for slow decay. `severity` (severe > 50%, moderate 20–50%, mild) and `pattern` guide the diagnosis:
   - `position_slipped`: rankings fell → content freshness, competitors, internal links.
   - `impressions_collapsed`: demand fell or the page lost coverage → check indexing and seasonality.
   - `ctr_dropped`: SERP changed (AI Overviews, new features) or title lost appeal.
   - `disappeared`: no traffic at all now → indexing, redirect, canonical.
4. Causes: `get_annotations` `{site, days: 180}` for deploys and Google algorithm updates (`category: "algo_update"`) near the inflection date.
5. Per page: `query_top_queries` `{site, pageContains: "<path>", compare: "previous", sort: "decline"}` to see which queries left.

## Output

Per page: clicks lost (absolute and %), pattern, likely cause, recommended action (refresh, consolidate, re-target, fix technical issue), and priority by clicks at stake. Always state the recent and baseline periods.

## Notes

Compare like with like: a 28-day window against the same dates one year earlier (`compare: "year"`) removes seasonality. For small sites set `minBaselineClicks` to 1–3. `ctr` is a fraction; `position` lower is better.
