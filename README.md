# Candidate search skill for Claude

A Claude skill that searches LinkedIn for candidates and returns a scored shortlist. Invoke it, tell it what you're looking for, and it runs the search.

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

1. **Upload the skill.** Settings → Capabilities → Skills → upload `candidate-search.zip`.
2. **Connect Apify.** Settings → Connectors → Apify. It asks for an API token: get one at Apify Console → Settings → API & Integrations. Keep the "actors" tools enabled. Searches are billed to that Apify account.
3. **Run it.** Invoke the skill with no message and it lists the parameters it supports. Answer in plain language.

## Example answer

> Head of Revenue Operations (also Director or VP of RevOps). United States. 6-10 years of experience. Currently at a company of 50-500 people. B2B SaaS that sells to mid-market or enterprise, not D2C. Has built RevOps from scratch or was the first dedicated RevOps hire, and has managed a team. Deal-breaker: agency or consulting-only background. Shortlist of 25 from a pool of 100.

## Cost

Roughly $0.10 per 25 search results plus about $0.004 per full profile, so a 100-person pool is about $0.80.

## How it works

The skill calls the Apify Actor [`harvestapi/linkedin-profile-search`](https://apify.com/harvestapi/linkedin-profile-search). No LinkedIn login or cookies are used, and nobody's personal LinkedIn account is involved. It doesn't contact anyone.

The skill itself is one file: [`candidate-search/SKILL.md`](candidate-search/SKILL.md). Edit it to change how it asks, searches or scores.
