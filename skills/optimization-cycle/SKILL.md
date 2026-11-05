---
name: optimization-cycle
description: Run one optimization cycle on a website from inside its code repository with the property's own Google Search Console data through the Organiq MCP server (get_query_performance, get_page_performance, get_site_summary, get_ranking_changes, query_top_queries, analyze_query_shapes, score_opportunities, find_page_poaching_opportunities, analyze_cannibalization). It judges the changes logged in the previous cycle, picks at most three page edits and one new page that build on queries already gaining, question-style queries and the user's key queries, edits the site's source files, runs the project's checks and logs each change with its baseline in .organiq/cycle.md. Use when the user asks to optimize the site in the code, run the SEO or GEO cycle, act on Search Console data in the repository, build on the pages that already work, or check whether the last changes worked.
license: Apache-2.0
metadata:
  author: Organiq
  version: "1.0"
---

# Optimization cycle (data, edits in the repository, log)

Goal: one complete pass that ends with edited files, not a list of suggestions. Read the property's data through the Organiq connector, judge what the previous cycle changed, choose a few changes that build on what already works, edit the site's source files in this repository, verify them, log them with a measured baseline, and report. Nothing is scheduled: the user runs the cycle again when the log has changes old enough to judge. Never invent numbers; if a tool returns `no_data` or `plan_limit`, note it and continue with the other steps.

## 0. Before starting

1. File access: this cycle edits the repository of the site. Without access to the files (a chat client with no workspace), produce the same output as text: for each change, the URL, the file or page section if known, and the before and after text ready to paste, plus the ledger rows from step 7 for the user to save.
2. Property: `list_sites`. Pick the property and note `dataTo` (the last synced date; every window ends there, not today), `hasGa4`, `hasBing` and `brandedTerms`.
3. Same domain: confirm the repository builds that property's site. Look for the domain in the canonical tags, the base URL in the framework config (for example `site` in `astro.config.*`, `metadataBase` or a site URL constant in Next.js, `baseURL` in Hugo, `url` in Jekyll's `_config.yml`), the sitemap generator, a `CNAME` file or `.env.example`. If nothing matches, stop and ask the user which property belongs to this repository.

## 1. The cycle file

The cycle keeps its memory in `.organiq/cycle.md` at the repository root, so it travels with the code. If the file does not exist, create it with this template:

```markdown
# Organiq optimization cycle

Property: <site as in list_sites>
Dates are yyyy-mm-dd. Data windows end on the last synced Search Console date, not on the day of the run.

## Key queries

<!-- Up to 20 searches that matter to the business, one per line. Edit this list; every cycle reads it. -->

## Ledger

<!-- One row per changed page. date = day the change went live ("pending (edited yyyy-mm-dd)" until then).
baseline = page + target query, 28 days ending on the date shown. d+14 and d+28 = windows that start the day after date. -->

| date | page | target query | change | baseline (impr / pos / ctr, 28d to yyyy-mm-dd) | d+14 (impr / pos) | d+28 (impr / pos / ctr) | verdict |
|---|---|---|---|---|---|---|---|
```

If the key query list is empty, continue without it and, in the report, suggest up to 10 non-branded queries with the most impressions for the user to keep or replace.

## 2. Judge the previous cycle

1. Rows with `pending` in `date`: find the day the change reached production (the date the commit landed on the production branch in `git log`, or a deploy tag); if the repository does not show it, ask the user once. Write that date. If the change is not live yet, leave the row and do not judge it.
2. For each live row without a verdict, with D = `date`:
   - D + 14 ≤ `dataTo` and `d+14` empty: `get_query_performance` `{site, query: "<target query>", page: "<page>", startDate: D+1, endDate: D+14, compare: "none"}`. Write impressions and position. At d+14, only compare position and daily impressions with the baseline (baseline impressions ÷ 28 against d+14 impressions ÷ 14); change nothing.
   - D + 28 ≤ `dataTo`: the same call with `endDate: D+28`, and `get_site_summary` `{site, startDate: D+1, endDate: D+28, compare: "previous"}` to separate the page's movement from the whole site's. Write the d+28 cell and the verdict:
     - **revert**: impressions for page + query fell 30% or more against the baseline while the site's impressions changed less than 10% in either direction.
     - **keep**: impressions rose 10% or more, or position improved by 1.0 or more, or clicks rose, and impressions did not fall more than 10%.
     - **iterate**: anything else (flat, mixed, or a fall that matches a fall of the whole site).
     - Baselines under 20 impressions: judge on position only and write "small sample" in the verdict. For a new page (baseline 0), keep if it has impressions for the target query, iterate if it has none; a new page is never reverted.
3. Act on the verdicts:
   - revert: find the commit that made the change (`git log -- <file>` around the date, or `git log -S "<text from the change>"`) and undo only that change in the working tree (`git revert --no-commit <sha>` when the commit holds only that change, otherwise by hand). Write `revert (yyyy-mm-dd)` in the verdict. If the change is not in the history, undo it from the `change` description and say so in the report.
   - iterate: the page is a candidate in step 4 with a hypothesis different from the one logged.
   - keep: no action; the page's pattern can be replicated (rule 1 in step 4).
4. A page with a change younger than 28 days is not judged and not edited again in this cycle.

## 3. Read the data

About 12 calls. Default window: `days: 28`, `compare: "previous"`, ending on `dataTo`. Small sites (few rows): `days: 90` and lower `minImpressions`.

1. Context: `get_site_summary` `{site, days: 28, compare: "previous"}`.
2. Gaining queries: `get_ranking_changes` `{site, days: 28, compare: "previous", dimension: "query", status: "improved", minImpressions: 10, limit: 50}` and `query_top_queries` `{site, days: 28, compare: "previous", sort: "growth", minImpressions: 10, limit: 25}`.
3. Question-style and AI-style queries:
   - `query_top_queries` `{site, days: 90, query: "/^(how|what|why|which|when|where|who|can|is|does|should|como|o que|qual|quais|quanto|quando|por que|onde)\\y/", sort: "impressions", limit: 50}`.
   - `analyze_query_shapes` `{site, days: 28, compare: "previous", examples: 10}` for the AI-style split and example queries; optionally `get_query_shape_trend` `{site}` to see whether that share is growing.
   - `get_genai_impressions` `{site, days: 28, compare: "previous"}` (property-level impressions inside AI Overviews and AI Mode; `hasData: false` means nothing was imported) and, when `hasGa4` is true, `get_ai_traffic` `{site, days: 28, compare: "previous"}`. Both are context for the report; neither is per query.
4. Key queries, all in one call: `query_top_queries` `{site, days: 28, compare: "previous", query: "/^(key one|key two|…)$/", limit: 50}` with the list from the cycle file (escape regex characters). A key query missing from the answer had no impressions in the window.
5. Gaps:
   - CTR below the curve: `score_opportunities` `{site, days: 28, minPosition: 1, maxPosition: 10, minImpressions: 20, limit: 25}`.
   - Positions 4–30: `find_page_poaching_opportunities` `{site, days: 28, minPosition: 4, maxPosition: 30, minImpressions: 10, sort: "gain", limit: 50}`.
   - Pages splitting one query: `analyze_cannibalization` `{site, days: 90, minImpressions: 10, minPages: 2, limit: 25}`; drop protocol, host, trailing-slash and fragment variants of one URL, and brand query + home + deep page (sitelinks).
   - Sections: `breakdown_by_path` `{site, days: 28, compare: "previous", depth: 1}` (use `depth: 2` for deeper folders) to see which section each candidate belongs to and how the section moves.
6. Optional, when `hasBing` is true: `get_bing_performance` `{site, view: "queries", weeks: 4, sort: "growth"}`.

For the candidates you consider in step 4 (not for every row), find the ranking page and its queries: `get_query_performance` `{site, query, days: 28, compare: "previous", counterparts: 5}` and `get_page_performance` `{site, page, days: 28, compare: "previous", counterparts: 20}`. Sibling pages: `query_top_pages` `{site, days: 28, pageContains: "<folder>", sort: "impressions", limit: 25}`.

## 4. Decide

At most three page edits and one new page per cycle, one change per page. Excluded: pages changed less than 28 days ago, pages reverted in this cycle and queries under 10 impressions in 28 days (key queries excepted). For each candidate, the first rule that applies decides the change.

**Do not break what already works.** These limits apply to every rule below and win over them:
- Before editing a page, read what it ranks for: `get_page_performance` `{site, page, days: 28, compare: "previous", counterparts: 20}`. Clicks at risk = the page's clicks in that window.
- A page with clicks at risk keeps its title, H1, URL and first paragraph. It receives additions only, below the first paragraph: a section, a table, an answer block, links.
- Title, meta description and first paragraph are rewritten only on a page at position 10 or better with CTR below 1%, where little traffic is at risk. The new text keeps the words of every query that brought the page a click or 20 or more impressions in the window.
- Nothing that ranks is removed or shortened. Text is added or replaced, never deleted.
- A change that cannot meet these limits is left out of the cycle and listed in the report.

1. **Gaining query or page** (position better by 1 or more, or more clicks than the previous period): open the page in the repository and identify what it does that its siblings do not (title format, a direct answer near the top, a section, a table, structured data, internal links). Apply that pattern to sibling pages of the same section or template that rank worse for similar queries, and link the gaining page and the siblings to each other with descriptive anchors. On the gaining page itself, add links only.
2. **Question-style or AI-style query with impressions**, on a page that does not answer it directly: add the question as an H2, then a 40–60-word answer that stands on its own (names the subject and gives the fact), then the detail. Use citable facts (figures with their date and source) that already exist on the site or that the user provides. Keep JSON-LD consistent with the visible text; add `FAQPage` only for questions and answers that are visible on the page.
3. **Position 10 or better, CTR below the curve, 20 or more impressions**: when the page's CTR is below 1%, rewrite the title, the meta description and the first paragraph so they contain the query (or its closest natural form) and state what the page delivers. Otherwise, add a section near the top that answers the query, leaving the title and first paragraph as they are.
4. **Position 11–30, 10 or more impressions**: add a section that answers the query and two or three internal links to the page from pages that already get impressions, with anchor text close to the query.
5. **Real cannibalization**: give each page a role. One page owns the query (the winner from the tool unless another page matches the intent better); the other keeps its own angle with a distinct title and H1; link them to each other, with the owner linked using the query as anchor. Do not delete pages, add redirects or change canonicals here; list those as suggestions in the report.
6. **Key query with no page that owns it** (no impressions, or no page ranking in the top 30 and no matching page in the repository): create one new page.

Ties: key query, then non-branded, then branded (`brandedTerms` from `list_sites`), then more impressions.

## 5. Edit the project

1. Detect the framework from `package.json` dependencies and config files (Next.js, Astro, Nuxt, SvelteKit, Remix, Gatsby, Eleventy, Docusaurus, Hugo, Jekyll, plain HTML). Content that lives in a CMS or database (for example WordPress) cannot be edited here: deliver it as text to paste.
2. Map each URL to its source file through the routing conventions (`app/` or `pages/` folders, content collections, `slug` or `permalink` front matter). Otherwise search the repository for the page's current title or a distinctive sentence.
3. Follow the repository's conventions: its components for answer and FAQ blocks, its front matter fields, its language versions (when the site keeps translations side by side, change each version and keep the hreflang alternates pointing to each other) and its sitemap (a new page must be in it). Match the existing tone and formatting.
4. A new page uses an existing template of the same kind, links from at least two related pages and contains only facts from the site or the user. If there is not enough material for a useful page, create it as a draft (the framework's draft flag, or outside the routes) and list what is missing.
5. Never change a slug, URL, canonical, robots directive or redirect; never invent a fact, price, number, date, quote or review; never remove content that ranks (move it lower if needed); do not repeat the query beyond the title, H1 or first paragraph and one H2; touch only the files of the planned changes.

## 6. Verify

1. Run the project's existing checks: the build, typecheck, test and lint scripts in `package.json` (or the Makefile). Fix what the edit broke. A check that already failed before the edit is reported, not fixed.
2. For each edited page, in the source or the built HTML: exactly one H1; canonical unchanged; title up to about 60 characters; meta description about 70–160 characters; JSON-LD parses and matches the visible text; new internal links point to existing URLs; hreflang pairs intact.
3. An edit that fails a check and cannot be fixed is undone and left out of the ledger.

## 7. Log

Append one ledger row per changed page:

- `date`: `pending (edited yyyy-mm-dd)` with today's date; the next cycle replaces it with the day the change went live.
- `baseline`: `get_query_performance` `{site, query, page, days: 28, compare: "none"}` measured now, written as impressions / position / ctr with the end date. For a new page, measure the query for the whole site (`get_query_performance` without `page`) and start the baseline cell with `new page:`.
- `change`: one line with the rule used and what changed (for example "answer block: H2 'How long does X take?' + 52-word answer").
- Update the verdict cells and the dates written in step 2.

## 8. Stop before publishing

Do not commit, push or deploy unless the user asks. Keep `.organiq/cycle.md` in version control so the next cycle can read it. After the user deploys, `request_indexing` `{site, urls: [<changed URLs>]}` can submit the changed URLs (it needs the `mcp:write` scope or a write API key; a submission is a request, not a confirmation of indexing).

## 9. Report

Short, with the period next to every number:

1. Data used: property, `dataTo`, the windows.
2. Verdicts: page, target query, baseline → d+28, the site's change in the same window, verdict, action taken.
3. Changes this cycle: page, target query, rule, baseline (impressions, position, CTR), what changed, clicks at risk and what could fall.
4. Files touched, and the checks run with their result.
5. What needs the user: deploy, missing facts, CMS-only pages, suggested redirects or canonicals, key query suggestions when the list is empty.
6. Next cycle: the earliest date a change completes 14 days of data, that is the live date + 14 days plus about 3 days of Search Console lag (for pending rows: 17 days after the deploy).

## Units

`clicks`/`impressions` integers; `ctr` fraction 0–1 (write it as %); `position` impression-weighted, lower is better (an improvement is a negative change); dates yyyy-mm-dd; periods end on the last synced date.
