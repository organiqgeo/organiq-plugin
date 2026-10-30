---
name: ai-traffic
description: Measure and interpret traffic coming from AI assistants (ChatGPT, Perplexity, Gemini, Copilot, Claude) and conversational search queries using GA4 and Google Search Console data through the Organiq MCP server (get_ai_traffic, query_top_queries, get_timeseries). Use when the user asks about GEO, AI traffic, LLM referrals, how much traffic comes from ChatGPT or Perplexity, AI Overviews impact, or question-style queries.
license: Apache-2.0
metadata:
  author: Organiq
  version: "1.0"
---

# AI traffic (GEO)

Goal: how many sessions AI assistants send to the site, which sources, how it trends, and which conversational queries the site already answers in Google.

## Steps

1. `list_sites`: `hasGa4` must be true for referral data; otherwise only the Search Console part applies.
2. `get_ai_traffic` `{site, days: 28, compare: "previous"}`: sessions, active users, engaged sessions and key events per source (chatgpt.com, perplexity.ai, gemini, copilot, claude…), totals and a daily series. `site: "all"` aggregates the account.
3. Trend: compare with `days: 90` and read the daily series for spikes (a single citation can cause one).
4. Conversational demand in Google: `query_top_queries` `{site, days: 90, query: "/^(como|o que|qual|quanto|por que|how|what|why|which)\\b/", sort: "impressions", limit: 50}` lists question-style queries; `query_top_queries` with `sort: "decline"` over 90 days vs the previous 90 can show queries losing clicks while impressions hold, a pattern consistent with AI Overviews taking the click (Search Console does not separate those clicks, so present it as a hypothesis).
5. `get_timeseries` `{site, days: 365, granularity: "month"}` for the long-run CTR trend.

## Output

Sessions from AI assistants (total and per source) with the change; share of all GA4 sessions when the `total` dimension is available; the top conversational queries with position and CTR; the pages those sessions land on if the user asks (GA4 landing pages are not exposed yet, say so). Keep the limitation explicit: referral traffic is measurable, clicks inside AI Overviews / AI Mode are not.

## Units

Sessions and users are integers; `keyEvents` are GA4 conversions; `ctr` fraction; dates yyyy-mm-dd; periods end on the last synced date.
