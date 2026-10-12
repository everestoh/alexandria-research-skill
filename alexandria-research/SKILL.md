---
name: "alexandria-research"
description: "Use for any research task needing facts, records or data: run Firecrawl Alexandria providers first, web as fallback, and deliver structured, sourced conclusions."
---

# Alexandria-first research

Use Firecrawl Alexandria (official data providers + Firecrawl's Research, Developer and Government indexes) as the primary source for research. Use ordinary web search/scrape only to fill gaps. Every finding must be traceable to where it came from, and nothing is presented until it has passed the accuracy rules, the self-check and the stress check.

This applies to every kind of task the skill is used for: companies, people, jobs, finance and filings, clinical trials and research papers, public records and government data, real estate, retail and prices, developer data, news, travel, and anything else.

## Tools

In the Claude app (Firecrawl connector):
- **Discover:** `firecrawl_search` with `sources: ["alexandria"]` (tools only, free) or `["web", "alexandria"]` (tools + web results). Results come back in `data.tools` as `provider` + `capability`.
- **Inspect:** `firecrawl_find_tools` with `capabilities: [...]` to read inputs, response shape and price. Free. Use `query` or `categories` to browse when search finds nothing.
- **Run:** `firecrawl_scrape` with an `alexandria` block: `{provider, capability, options}` and NO `url`. Up to 10 calls can be batched in one array. Results arrive in `data.alexandria[n]`; check each item for an error before using it. Large results are retained in a workspace: follow the returned `nextTool` and filter `response.json` with jq (free) instead of rerunning the provider.

In Claude Code (CLI):
- `firecrawl search "<topic>" --sources web,alexandria --domain-tools --limit 5`
- `firecrawl find-tools --providers <id> --level tools --expand options,response`
- `firecrawl scrape <provider>/<capability> --options '<json>'`

## Step 0 — Settle the output before researching

1. **Presentation format.** Never assume a default. If the prompt does not explicitly name a format, ask the user how they want the final output presented before starting, offering: Word file (.docx), Claude Doc, Excel file (.xlsx), Claude Sheet, HTML dashboard, PDF, slide deck, or a reply in the chat. Use AskUserQuestion when available. If the prompt names a format, use it and do not ask.
2. **Table columns.** Define the columns of the data table before fetching anything. Use the user's columns if given; otherwise propose them (e.g. Company | Drug | Trial phase | Status | Expected end date | Source) in the same question as the format.
3. **Constraint list.** Write down every filter and condition in the prompt (dates, locations, thresholds, statuses, categories, time windows, counts). This list is used by Accuracy rule 4.
4. **Unattended runs.** If the user is not present to answer (scheduled or headless run), do the research anyway with the proposed columns, then present a chat summary and ask the format question at the end rather than picking a format yourself.

## Accuracy rules

These four rules apply to every task, in every subject area.

### 1. Every claim has a source
- Every fact in the conclusion, findings and table carries a source tag: `[Provider: …]`, `[Index: …]` or `[Web: …]`.
- Anything that comes from my own general knowledge rather than a retrieved record is tagged `[Unverified]`. Before presenting, either confirm it with one targeted search (then retag it with that source) or move it to Gaps and caveats.
- `[Unverified]` claims never appear in the Conclusion, and never drive a recommendation.
- Examples: a company's founding year, a drug's mechanism, an airline's cabin type, a regulation's effective date, a job's salary band — if no record says it, it is unverified.

### 2. Compute, don't estimate
- All arithmetic is done by running code against the raw values, never in my head. Use Bash/Python, or jq in the Firecrawl workspace.
- This covers sums, differences, averages, medians, percentages, growth rates, ratios, counts, rankings, date gaps and durations, unit, currency and time-zone conversions, and any "cheapest / largest / most recent" selection.
- Copy figures into the output exactly as the code printed them, and state rounding (e.g. "$7.02, shown as about $7").
- Examples: funding totals across rounds, revenue growth year on year, trial enrollment totals, price differences between listings, days since a filing, number of jobs matching a filter.

### 3. Check whether the data is complete
- Every provider result has signals about completeness. Read them before drawing conclusions: `truncated`, `partial`, `coverage`, `skipped_results`, `notices`, `search_status`, `update_status`, per-item `error`, pagination fields (`next`, `next_offset`, `continuation`), and totals versus returned counts (`available_results`, `matching_results`, `total_results`, `returned`).
- Report completeness for each data source as "N of M records" (e.g. "13 of 13 results", "first 50 of 1,240 filings").
- If a result is incomplete, page further when the budget allows; otherwise say exactly what was not covered.
- Never state that something does not exist based on an incomplete result. Say "none found in the first 50 of 1,240" instead of "none exist".

### 4. Verify constraints row by row
- A filter in the query is not proof. For each row, check the record's own fields against every item in the constraint list from Step 0.
- Drop rows that fail. If a row can't be checked because the field is missing, keep it only if the user would still want it, and mark the cell "not confirmed".
- Examples: clinical trials — status really "Recruiting", phase really 2 or higher; companies — headcount and location match; jobs — posted within the last 7 days and actually remote; filings — within the date range; listings — under the price cap; flights — stops = 0 and departure inside the time window.
- Use code for the check when there are more than a handful of rows.

## Workflow

1. **Break the question into data needs.** List each distinct thing to find. One data need = one discovery query. Map each data need to the table columns it fills.
2. **Discover with short, single-topic queries.** 1–4 words each ("clinical trials", "company funding", "federal contracts"). Compound queries often return no tools. If a query returns nothing, rephrase more simply, then try `firecrawl_find_tools` with `query` or browse `categories`.
3. **Pick tools and inspect them.** Read required inputs (including any requiresOneOf groups) and the listed price. Prefer the most official/primary provider for each data need.
4. **Check cost before spending.** Discovery is free; execution is billed at each tool's listed price. Estimate the total. If it exceeds 50 credits (or the user's stated budget), show the plan and the estimate and ask before running. If the user is not present, stay under the limit and record what was skipped.
5. **Handle provider terms.** If a call returns `THIRD_PARTY_DATA_TERMS_REQUIRED`, never accept terms yourself. Tell the user which provider needs it and that an org admin can accept at the `requiresAction.url` (Firecrawl dashboard → Settings → Data sources). Continue with other providers or the web.
6. **Run the tools.** Batch independent calls. Keep the raw records so every value can be traced later.
7. **Check completeness** of each result (Accuracy rule 3) and page further if needed and affordable.
8. **Fill gaps from the web.** Where no provider covers a data need or column, use normal `firecrawl_search` / `firecrawl_scrape` (with `url`). Mark these findings as web-sourced.
9. **Verify constraints row by row** (Accuracy rule 4).
10. **Compute every derived figure with code** (Accuracy rule 2).
11. **Cross-check.** Where provider data and web sources disagree, trust the provider record, note the conflict, and say which is more current.
12. **Collect booking/source links** for every bookable or purchasable row (see Booking links below).
13. **Draft the output** using the structure below, filling the agreed columns and tagging every claim (Accuracy rule 1).
14. **Self-check (accuracy).** Re-check every number, date and name in the conclusion, findings and table against the raw records or code output it came from. Resolve or relocate every `[Unverified]` tag. Anything that cannot be traced is removed or moved to Gaps.
15. **Stress check (completeness).** Build a checklist of every explicit instruction, question, request and constraint in the user's prompt (each sub-question, every requested column, entity, time range, length, tone, format and deadline), plus this skill's own requirements: all six output sections present; every claim source-tagged; no `[Unverified]` claim in the Conclusion; all arithmetic computed by code; completeness reported for each source; constraints verified on every row; every table cell filled or marked "not found" / "not confirmed"; every bookable row linked to the seller; cost reported; agreed format used. Mark each item met or not met. Fix every unmet item that can be fixed; for any that cannot, say so plainly in Gaps. Do not present until the checklist has been run.
16. **Present** in the format chosen in Step 0.

## Booking links

Whenever results include things the user could book or buy (flights, hotels, rentals, tickets, products, services), the seller's name in the table is hyperlinked to the seller's own official page, opening in a new tab.

1. **Where the link comes from, in this order:**
   - A URL in the provider record that points to the seller's own domain and the specific item (e.g. a hotel `source_url`, a listing URL, a product page).
   - Otherwise, the seller's official booking/home page on a domain that appears in a provider record (e.g. airline domains in Google Flights `baggage_policies`).
   - Otherwise, the official site found by web search and confirmed to be the seller's own domain.
2. **Never construct links.** Do not build URLs with guessed paths or query parameters. Use only URLs that appear in a record or a fetched page.
3. **Aggregator redirects are secondary.** Links from aggregators (e.g. Skyscanner `transport_deeplink`) are temporary and may route to third-party sellers. Never use them as the seller link. If included at all, label them "via <aggregator>" and note they may expire.
4. **Add one comparison link** when the provider returns a search URL, so the user can see all live prices together.
5. **Tell the user how to use the link** in one line above the table when it opens a booking page rather than the exact item.
6. **Gaps:** state when links open a general page instead of the exact item, and when a seller had no official link.

For non-bookable research (companies, trials, filings, papers), link each entity to its primary record instead (the registry entry, the filing, the paper, the company's own site) when the record supplies that URL.

## Output structure

1. **Conclusion** — 2–4 sentences answering the question directly, with confidence level. Only sourced, verified facts.
2. **Key findings** — short bullets, each ending with its source tag.
3. **Data** — a table using the agreed columns (one row per entity), with a Source column. Entities are hyperlinked per Booking links. Empty cells say "not found" or "not confirmed", never left blank or guessed.
4. **Source ledger** — every source used, with its completeness ("N of M records"), tagged:
   - `[Provider: <provider>/<capability>]` for Alexandria records
   - `[Index: Research | Developer | Government]` for Firecrawl indexes
   - `[Web: <page title> — <URL>]` for scraped/searched pages
5. **Gaps and caveats** — what couldn't be found, `[Unverified]` items that could not be confirmed, incomplete results, rows dropped for failing constraints, providers skipped for terms or cost, conflicting data, freshness of the records, link limitations, and any request that could not be fully met.
6. **Quality check** — a short line covering: requirements met (e.g. "12 of 12"), completeness (e.g. "all sources complete" or which were partial), constraints verified on N rows (and rows dropped), unverified claims remaining (should be 0 in the Conclusion), all figures computed and traced, and credits spent on Alexandria execution (discovery is free).

## Rules

- Never choose a presentation format the user didn't ask for; ask (Step 0).
- Never present before the accuracy rules, self-check and stress check are done.
- Never present a web page's summary of data as if it were the provider's record.
- Never invent provider names, capabilities, prices, values or URLs; use only what discovery, the records and fetched pages return.
- Never do arithmetic mentally; compute it.
- Never conclude something is absent from an incomplete result.
- Never accept third-party data terms on the user's behalf without explicit authorization.
- Never book, hold or purchase anything; links are for the user to act on.
- If Alexandria returns nothing usable at all, say so plainly and deliver a web-only result, labeled as such.
- When the user asks whether Alexandria was used, answer from the actual tool calls: only `firecrawl_scrape` calls with an `alexandria` block count as provider data.
- For high-stakes work, offer a final review by a separate agent that did not do the research.