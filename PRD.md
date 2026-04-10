# Job Scout — Claude Code Skill PRD

## Problem Statement

Job searching as a senior/founding designer is time-consuming and fragmented. Candidates must manually check multiple platforms daily, read through irrelevant listings, and assess fit themselves. For niche roles like "founding product designer at an AI startup," the signal-to-noise ratio is extremely low. A candidate spends 1-2 hours daily across LinkedIn, Wellfound, YC, Google — and still misses relevant postings.

This skill automates the entire workflow: profile extraction, multi-source job scraping, fit scoring, and ranked output — in a single conversational session.

## Goals

1. **Reduce search time from hours to minutes** — a complete personalized job search in under 5 minutes
2. **Surface high-fit roles only** — strict scoring eliminates noise, user sees only roles worth applying to
3. **Cover multiple sources in parallel** — Google web search + LinkedIn (via Apify) combined for maximum coverage
4. **Reusable profile** — extract once, search repeatedly without re-entering info
5. **Zero-config start, progressive enhancement** — works with just WebSearch; gets better with Apify API key

## Non-Goals

- **Not an auto-apply tool** — outputs links, does not submit applications
- **Not a CRM / tracker** — does not persist job listings across sessions or track application status
- **Not a resume builder** — extracts profile data, does not generate or improve CVs
- **Not a recruiter outreach tool** — does not contact companies or hiring managers
- **Not a salary negotiation advisor** — reports listed compensation, does not advise on negotiation

## User Stories

### Job Seeker (Primary Persona)
- As a job seeker, I want to describe my ideal next role conversationally so that the search matches my actual preferences, not just keywords
- As a job seeker, I want my CV/portfolio scraped automatically so that I don't have to manually list my skills and experience
- As a job seeker, I want to see only jobs that genuinely match my profile so that I don't waste time on poor fits
- As a job seeker, I want results from multiple sources combined so that I don't miss postings on platforms I forgot to check
- As a job seeker, I want to save my preferences so that next time I can just say "run job scout" without re-onboarding
- As a job seeker, I want to optionally connect Apify for LinkedIn scraping so that I get deeper coverage beyond Google results

### Returning User
- As a returning user, I want to skip onboarding and jump straight to search so that daily checks take seconds
- As a returning user, I want to update just one parameter (e.g., add a new city) without redoing the whole flow

## Requirements

### Must-Have (P0)

**Onboarding Flow**
- [ ] Ask target role title(s) — free text with examples
- [ ] Ask target locations — multiple cities + remote/hybrid/relocation preferences
- [ ] Ask company stage preference — seed/A, growth, flexible (multiple choice)
- [ ] Ask work style — pure design, design+eng, AI/NLP focus, etc. (multiple choice)
- [ ] Ask minimum compensation — salary floor, equity preference
- [ ] Ask minimum fit score threshold — default 6/10
- [ ] Ask for CV file path or portfolio/website URL
- [ ] Present "search brief" summary for confirmation before executing

**Profile Extraction**
- [ ] Scrape portfolio website via WebFetch — extract role, skills, experience, tools, domains, work history
- [ ] Read local CV/resume file via Read tool if file path provided
- [ ] Build structured candidate profile from extracted data
- [ ] Store profile as memory file (`~/.claude/skills/job-scout/profile.md`) for reuse

**Job Search — Web (always available)**
- [ ] Run 4-6 parallel WebSearch queries covering: role variants × location combinations
- [ ] Include time-scoped queries ("posted today", "last 24 hours", date strings)
- [ ] Fetch job details via WebFetch for top candidates to verify posting date, requirements, apply link
- [ ] Handle search failures gracefully — retry with alternate query terms

**Job Search — LinkedIn via Apify (optional)**
- [ ] Check if Apify MCP tools are available
- [ ] If available, use `worldunboxer/rapid-linkedin-scraper` with `job_post_time: "r86400"` (last 24 hours)
- [ ] Run one scraper call per location
- [ ] Extract: job_title, company_name, location, time_posted, salary_range, job_url, job_description
- [ ] If Apify not available, inform user and proceed with web search only

**Scoring**
- [ ] Score each job 0-10 against candidate profile
- [ ] Scoring dimensions: role fit (2pts), responsibilities match (2pts), skills overlap (2pts), experience/seniority alignment (2pts), domain relevance (1pt), location match (1pt)
- [ ] Strict scoring — founding designer with 15yr experience at AI startups should not score 8+ on a mid-level role at a non-tech company
- [ ] Filter out jobs below user's threshold score

**Output**
- [ ] Markdown table: # | Role | Company | Posted | Location | Source | Fit Score | Apply Link
- [ ] Sort by: 1) most recently posted, 2) highest fit score
- [ ] Limit to top 10 results
- [ ] Include 1-line score justification per role (collapsed or after table)
- [ ] Show total jobs found vs. jobs that passed the threshold

### Nice-to-Have (P1)

- [ ] Apify API key setup wizard — guide user through adding key to MCP config
- [ ] "Deep dive" mode — fetch full job descriptions and provide detailed match analysis
- [ ] Multiple role searches in one run (e.g., "founding designer" + "head of design")
- [ ] Exclude companies list (e.g., current employer)
- [ ] Save results to a markdown file for offline review

### Future Considerations (P2)

- [ ] Scheduled daily runs via CronCreate — automatic morning job digest
- [ ] Track previously seen jobs to show only new listings
- [ ] Integration with application tracking (Notion, Airtable)
- [ ] Cover letter generation per role
- [ ] Salary benchmarking against market data

## Scoring Algorithm Detail

```
Role Fit (0-2):
  2 = exact title match (e.g., "founding product designer" when seeking founding roles)
  1 = adjacent title (e.g., "senior product designer" or "design lead")
  0 = mismatched title (e.g., "graphic designer", "UI developer")

Responsibilities Match (0-2):
  2 = core responsibilities align (design systems, user research, 0-to-1, AI interfaces)
  1 = partial overlap
  0 = different discipline entirely

Skills Overlap (0-2):
  2 = 70%+ skills match (Figma, design systems, prototyping, AI tools, B2B SaaS)
  1 = 40-70% match
  0 = <40% match

Experience/Seniority (0-2):
  2 = seniority level matches (founding/staff/principal for senior candidates)
  1 = one level off (senior role for founding-level candidate)
  0 = major mismatch (junior role, intern, or VP/C-level)

Domain Relevance (0-1):
  1 = relevant domain (AI, SaaS, NLP, conversational, startups)
  0 = unrelated domain (fashion, physical retail, government)

Location Match (0-1):
  1 = matches preferred location or remote preference
  0 = location not in preference list and not remote
```

## Conversational UX Guidelines

**Tone**: Warm, direct, professional. Like a smart career advisor who respects your time. No corporate HR-speak. No excessive enthusiasm.

**Pacing**: 
- Ask 1-2 questions per turn, not a wall of questions
- Use AskUserQuestion with multiple-choice options where appropriate
- Always offer "Other" / free text option
- Show progress via TodoWrite checklist

**Search Brief**: Before executing, present a summary like:
```
Here's your search brief:
- Role: Founding Product Designer
- Locations: Berlin, London, Remote (US)
- Stage: Seed / Series A
- Focus: Pure product design
- Min comp: €170k
- Min fit score: 6/10
- Sources: Google + LinkedIn (Apify connected)

Ready to search? Or want to adjust anything?
```

**Results Presentation**: Lead with the table. Follow with brief justifications. End with "Want me to dig deeper into any of these, or run the search again with different parameters?"

## Technical Architecture

```
SKILL.md (entry point)
  ├── Phase 1: Onboarding (AskUserQuestion, Read)
  │   ├── Collect preferences
  │   ├── Scrape CV/portfolio (WebFetch, Read)
  │   └── Build + save profile
  ├── Phase 2: Search (WebSearch, Apify MCP)
  │   ├── Google web search (4-6 parallel queries)
  │   ├── LinkedIn scraper (1 per location, if Apify available)
  │   └── Fetch job details (WebFetch for top candidates)
  ├── Phase 3: Score + Rank
  │   ├── Match each job against profile
  │   ├── Apply scoring algorithm
  │   └── Filter + sort
  └── Phase 4: Output
      ├── Render table
      ├── Show justifications
      └── Offer to save profile/results
```

## Open Questions

- **Engineering**: Should the skill auto-detect if Apify MCP is connected, or always ask? → Recommend: auto-detect first, ask only if not found
- **Design**: Should score justifications be inline in the table or in a separate section? → Recommend: separate section after table to keep table scannable
- **Data**: What's the realistic hit rate for "last 24 hours" filtering via web search? → From session: ~30-50% of results are genuinely recent; WebFetch verification needed for top candidates

## Success Metrics

- **Activation**: User completes onboarding and gets results in first run
- **Relevance**: Average fit score of returned results > 6.5 (no junk padding)
- **Coverage**: Finds 5+ scoreable jobs per search across sources
- **Speed**: Full flow (onboarding → results) < 5 minutes
- **Reuse**: Returning users skip onboarding and get results in < 2 minutes
