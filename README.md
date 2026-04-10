# Job Scout — Claude Code Skill

Personalized job search agent for Claude Code. Scrapes your CV/portfolio, searches Google + LinkedIn (via Apify), scores roles against your profile, returns a ranked table of best-fit opportunities, and generates ATS-optimized tailored CVs for selected positions.

## How to Use

```
/job-scout
```

Or say: "find jobs", "job search", "search jobs", "scout jobs"

## What It Does

1. **Onboarding** — Conversational intake: CV/portfolio URL, target role, locations, company stage, domain, compensation, fit threshold
2. **Profile Building** — Scrapes your website/CV, extracts structured profile, saves for reuse
3. **Multi-Source Search** — Parallel Google web search + LinkedIn scraper (via Apify, optional) across all your target locations
4. **Scoring** — Each job scored 0-10 against your profile (role fit, responsibilities, skills, seniority, domain, location)
5. **Results Table** — Top 10 roles sorted by recency + fit score, with apply links
6. **CV Tailoring** — Select a role → scrapes full job + company details → generates ATS-optimized CV. Never fabricates facts.

## Requirements

- **Claude Code** with WebSearch and WebFetch access
- **Apify MCP** (optional) — for LinkedIn job scraping via `worldunboxer/rapid-linkedin-scraper`

## Files

- `SKILL.md` — The skill definition (6 phases)
- `PRD.md` — Full product requirements document
- `profile.md` — Generated after first run (your saved search profile)
- `cv-tailored-*.md` — Generated tailored CVs per company

## Key Principles

- **Strict scoring** — no inflated scores. A 5 is a 5.
- **Never fabricates** — CV tailoring only restructures, rewords, and emphasizes existing experience. Gaps are honestly disclosed.
- **Progressive enhancement** — works with just Google search, gets better with Apify connected
- **Reusable profile** — onboard once, search repeatedly
