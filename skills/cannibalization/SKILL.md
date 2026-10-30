---
name: cannibalization
description: Detect and resolve keyword cannibalization in Google Search Console data through the Organiq MCP server (analyze_cannibalization, query_top_pages). Use when the user mentions cannibalization, several pages ranking for one query, pages competing for the same keyword, or asks which page should rank for a term.
license: Apache-2.0
metadata:
  author: Organiq
  version: "1.0"
---

# Cannibalization

Goal: the queries where two or more URLs of the same property split impressions, a chosen winner per query, and the fix.

## Steps

1. Resolve the property (`list_sites`).
2. `analyze_cannibalization` `{site, days: 90, minImpressions: 10, minPages: 2, limit: 50}` (large sites: `minImpressions: 100`). Each cluster has `entropy` in bits (higher = more even split = worse; `level` high ≥ 1.5, medium ≥ 1.0), `winner` and `contenders` with `impressionShare`.
3. Discard false positives before reporting:
   - http/https, www/non-www, trailing slash or `#fragment` variants of one URL: canonical hygiene, not cannibalization.
   - brand query + home + a deep page: usually sitelinks.
   - intentionally different intents (a guide and a product page) can coexist.
4. For real clusters, `query_top_pages` `{site, query: "<the query>", days: 90, compare: "previous"}` shows how each URL's clicks and position moved.
5. Decide per cluster: consolidate (merge + 301), canonicalize, differentiate intent (re-target one page), or adjust internal anchors toward the winner.

## Output

Table: query, impressions, entropy/level, winner (URL, clicks, position), contenders (URL, share), recommended fix. State the period used.

## Units

`ctr` fraction 0–1; `position` lower is better; `impressionShare` fraction of the cluster's impressions.
