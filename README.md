# Recruiting skills for Claude

Two Claude skills for a recruiting firm. Invoke either one with no message and it lists the parameters it supports; answer in plain language and it runs.

| Skill | What it does | Upload |
|---|---|---|
| **job-prospecting** | Finds fresh open roles at companies you could recruit for (your target list, or a broad search) and ranks them by fit. | `job-prospecting.zip` |
| **candidate-search** | Searches LinkedIn for candidates and returns a scored shortlist. | `candidate-search.zip` |

Both run on the same Apify connection (see Set up), and neither contacts anyone.

## job-prospecting

Give it any of: target companies (names or websites), the roles you place, company size, kind of company, location, how recently posted, and how many results. Leave the company list out and it searches broadly. It returns a ranked list with a one-line reason for each and a link to the posting, and it tells you which target companies had no readable careers page rather than calling them "not hiring."

## candidate-search

## What it does

You give it a role and any of these (all optional except the role):

- years of experience
- location
- company size
- industry or type of company
- seniority
- past employers or schools
- softer criteria, such as "first 10 employees", "built it from zero", "B2B not D2C"
- must-haves and deal-breakers
- shortlist size (default 25) and pool size (default 100)

It runs the search, reads each candidate's full work history, scores them against a rubric built from your criteria, and returns a ranked shortlist. Each person gets a one-line reason backed by their profile. It also lists near misses and suggests how to improve the next run.

Criteria LinkedIn can filter on (titles, location, years, company size, seniority) go into the search. The rest are judged by Claude from each person's work history. It never claims the search enforced something it can't.

## Set up (Claude desktop app)

1. **Upload the skills.** Settings → Capabilities → Skills → upload `job-prospecting.zip` and `candidate-search.zip`.
2. **Connect Apify.** Settings → Connectors → Apify. It asks for an API token: get one at Apify Console → Settings → API & Integrations. Keep the "actors" tools enabled. Searches are billed to that Apify account.
3. **Run it.** Invoke the skill with no message and it lists the parameters it supports. Answer in plain language.

## Example answer

> Head of Revenue Operations (also Director or VP of RevOps). United States. 6-10 years of experience. Currently at a company of 50-500 people. B2B SaaS that sells to mid-market or enterprise, not D2C. Has built RevOps from scratch or was the first dedicated RevOps hire, and has managed a team. Deal-breaker: agency or consulting-only background. Shortlist of 25 from a pool of 100.

## Cost

Roughly $0.10 per 25 search results plus about $0.004 per full profile, so a 100-person pool is about $0.80.

## How it works

`candidate-search` calls the Apify Actor [`harvestapi/linkedin-profile-search`](https://apify.com/harvestapi/linkedin-profile-search). `job-prospecting` calls [`eiv/company-jobs-scraper`](https://apify.com/eiv/company-jobs-scraper) for a target list and [`curious_coder/linkedin-jobs-scraper`](https://apify.com/curious_coder/linkedin-jobs-scraper) for a broad search. No LinkedIn login or cookies are used, and nobody's personal LinkedIn account is involved. It doesn't contact anyone.

Each skill is one file: [`candidate-search/SKILL.md`](candidate-search/SKILL.md) and [`job-prospecting/SKILL.md`](job-prospecting/SKILL.md). Edit it to change how it asks, searches or scores.
