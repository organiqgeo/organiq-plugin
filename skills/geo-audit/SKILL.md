---
name: geo-audit
description: Audit a property's visibility in generative AI (GEO) with its own data through the Organiq MCP server (get_ai_traffic, query_top_queries, query_top_pages, get_timeseries, get_breakdown, get_annotations). Use when the user asks how much traffic comes from ChatGPT, Perplexity, Gemini, Copilot or Claude, which pages AI assistants send people to, which conversational (question-style) queries the site already answers in Google, whether AI Overviews are taking clicks, or for a monthly GEO report.
license: Apache-2.0
metadata:
  author: Organiq
  version: "1.0"
---

# GEO audit (traffic and demand from generative AI)

Goal: one dated report with (1) sessions referred by AI assistants, per source and over time, (2) the conversational queries the site already answers in Google, with impressions, position and CTR, (3) pages that look affected by AI Overviews (clicks falling while impressions hold), and (4) three concrete actions. Everything comes from the user's own Search Console and GA4 data through the Organiq connector; never invent numbers, and keep every metric's nature separate (sessions ≠ impressions ≠ citations).

## Steps

1. `list_sites`: confirm the property, `dataTo` (periods end there) and `hasGa4`. Without GA4, skip steps 2–3 and say the referral part is not measurable for this property.
2. `get_ai_traffic` `{site, days: 28, compare: "previous"}`: sessions, active users, engaged sessions and key events per source (chatgpt.com, perplexity.ai, gemini.google.com, copilot.microsoft.com, claude.ai…), totals, change and daily series. `site: "all"` aggregates the account.
3. Trend: `get_ai_traffic` `{site, days: 90}` and read the daily series for spikes (a single citation can cause one). Compute the share of all GA4 sessions when the `total` dimension is present.
4. Conversational demand in Google: `query_top_queries` `{site, days: 90, query: "/^(como|o que|qual|quanto|quando|por que|onde|how|what|why|which|when|where|can|is|does)\\y/", sort: "impressions", limit: 50}`. These question-style queries are the ones generative engines also answer; note position and CTR for each.
5. AI Overviews hypothesis: `query_top_queries` `{site, days: 28, compare: "previous", sort: "decline", limit: 25}` and `query_top_pages` with the same arguments. Flag rows where clicks fell more than 20% while impressions fell less than 5% and position did not worsen. Search Console does not separate clicks inside AI Overviews or AI Mode, so present this as a hypothesis, not a measurement.
6. Context: `get_breakdown` `{site, days: 28}` (brand share: brand queries are rarely affected by AI answers) and `get_annotations` `{site, days: 90}` (Google updates near any inflection).
7. Long run: `get_timeseries` `{site, days: 365, granularity: "month"}` for the CTR trend.

## Output

Sections, each with the period (start–end) next to the numbers:

1. Traffic from AI assistants: total sessions and change vs the previous period; table per source (sessions, engaged sessions, key events); share of all sessions when available; notable spikes with the date.
2. Conversational queries the site already answers: table (query, impressions, clicks, position, CTR), grouped by topic when there are many.
3. Pages and queries consistent with AI Overviews taking the click: table with clicks change, impressions change, position; state explicitly that this is a hypothesis.
4. Three actions, ordered by impressions at stake: for example, add a direct answer in the first 60 words of the page that ranks for a question-style query; add a dated figure with its source; consolidate two pages competing for one question.
5. Limits: GA4 only sees clicks; impressions inside AI Overviews / AI Mode are in Search Console's generative-AI report, not in this connector; Bing Webmaster Tools has per-page citations. Do not add metrics of different nature together.

## Units

Sessions, users and key events are integers; `ctr` is a fraction 0–1 (write it as %); `position` is impression-weighted, lower is better; dates yyyy-mm-dd; periods end on the last synced date.
