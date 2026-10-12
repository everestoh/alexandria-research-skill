# Alexandria Research — a Claude skill

A Claude skill for research that has to be **accurate and traceable**. It makes Claude pull facts from Firecrawl Alexandria's official data providers first, fall back to the web only to fill gaps, and deliver a structured report where every claim carries a source.

Works for companies, people, jobs, finance and filings, clinical trials and papers, public records, real estate, prices, developer data, news, travel and more.

## What it does

- **Alexandria first, web second.** Searches Firecrawl's catalogue of data providers and its Research, Developer and Government indexes before touching ordinary web results.
- **Every claim is sourced.** Facts are tagged `[Provider: …]`, `[Index: …]` or `[Web: …]`. Anything from Claude's own memory is tagged `[Unverified]` and kept out of the conclusion.
- **No mental maths.** All sums, averages, growth rates, date gaps and conversions are computed in code.
- **Completeness is reported.** Each source shows "N of M records", and Claude never says something doesn't exist based on a partial result.
- **Constraints checked row by row.** Filters like dates, locations, prices or statuses are verified against each record, not trusted from the query.
- **Asks before spending.** Shows a cost estimate and asks before running more than 50 credits of provider calls.
- **Never accepts data terms or books anything** on your behalf.

Every report follows the same six sections: Conclusion, Key findings, Data table, Source ledger, Gaps and caveats, Quality check.

## Requirements

- **Claude** (claude.ai app or Claude Code) with skills enabled.
- **Firecrawl connector** (or the Firecrawl CLI in Claude Code), with an API key on a Firecrawl team that has **Alexandria** enabled. Without Alexandria, the skill still runs but delivers web-only results, labeled as such.
- Provider calls are billed at each provider's listed price in Firecrawl credits. Discovery is free.

## Install

### Claude app (claude.ai / desktop)
1. Download this repo: **Code → Download ZIP**, then unzip it.
2. Zip **only the `alexandria-research` folder** (the one containing `SKILL.md`).
3. Upload that zip in Claude's skills settings.

### Claude Code
Copy the `alexandria-research` folder into your skills directory, for example:

```bash
git clone https://github.com/everestoh/alexandria-research-skill.git
cp -r alexandria-research-skill/alexandria-research ~/.claude/skills/
```

## Usage

Just ask a research question. The skill triggers on tasks that need facts, records or data, for example:

- "Find Phase 2+ clinical trials for GLP-1 drugs that are currently recruiting."
- "List remote operations jobs posted in the last 7 days with salary bands."
- "Compare total funding raised by the top five AI note-taking startups."

Claude will first ask how you want the output (chat reply, Doc, Sheet, Word, Excel, PDF, slides or dashboard) and confirm the table columns, then run the research.

## Customising

Everything lives in `alexandria-research/SKILL.md`. Common tweaks:

- Change the **50-credit approval threshold** in Workflow step 4.
- Set a **default output format** instead of asking (Step 0).
- Adjust the **output sections** under "Output structure".

## Repo structure

```
alexandria-research-skill/
├── README.md
├── LICENSE
└── alexandria-research/
    └── SKILL.md
```

## Disclaimer

This is an independent community skill. It is not affiliated with or endorsed by Firecrawl or Anthropic. "Firecrawl" and "Alexandria" are names of Firecrawl's products and are used here only to describe what the skill works with.

## Author

Created by **Everest Ouko**.

## License

[MIT](LICENSE)
