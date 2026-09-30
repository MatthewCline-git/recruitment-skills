---
name: job-prospecting
description: Find fresh open roles at companies a recruiting firm could win as clients, ranked by fit. Use when the user wants to prospect for hiring companies, find who is hiring for ops, RevOps, customer success, marketing, finance or executive-assistant roles, check a list of target companies for open roles, or build a weekly hiring-signal list. Every criterion is optional.
---

# Job prospecting

Turn "who is hiring for the roles we place?" into a ranked list of companies with an open role, each with a plain reason it's worth a call. A company with a fresh open role is a warm reason to reach out: "you posted this role, we just filled a similar one."

## Start here: ask for the parameters

When the skill is invoked **without any criteria**, do not search yet. Reply with exactly this greeting and list, then stop and wait. Do not run anything yet:

> Hey! I find fresh open roles at companies you could recruit for and rank them by fit. I support the parameters below. Please provide them. Everything is optional; skip anything that doesn't matter.
>
> - **Target companies** (names or websites). Leave this out and I'll search broadly instead.
> - **Roles you place** (e.g. RevOps, customer success, marketing, finance, executive assistant)
> - **Company size**
> - **Kind of company** (e.g. venture-backed startups, PE-backed, B2B software)
> - **Location**
> - **Posted within** (default: the past week)
> - **How many results** (default 25)
> - **One line on who you place and for whom**, so I can judge fit

Once the user answers, run the search with whatever they gave. If they invoked the skill with criteria already in their message, skip the list and run it straight away. If they give nothing usable, ask for just the roles, in one short question.

## Steps

1. **Restate the search in one line**, then go.

2. **Pick the mode.**
   - **Target list given** -> mode A.
   - **No list** -> mode B.

3. **Mode A, target companies.** Use the Apify Actor `eiv/company-jobs-scraper`. It reads each company's own careers board through the official job-board systems (Greenhouse, Lever, Ashby, SmartRecruiters, Workable, Recruitee, Personio, Breezy), so matches are exact per company.
   - Input: `companies` (website domains, e.g. `ramp.com`; if the user gave names, work out the domain and show the mapping), `titleKeywords` (the roles, plus close variants), `postedWithinDays` (default 7; 0 for no date filter, because some boards omit dates), `maxJobsPerCompany` (default 10), `includeDescriptions: false`.
   - **Say which companies returned nothing or use an unsupported careers page.** Do not present "no result" as "not hiring". Offer mode B for those companies.

4. **Mode B, broad search.** Use the Apify Actor `curious_coder/linkedin-jobs-scraper`, one search per role title the user named (and close variants).
   - Input: `keywords` (the title), `location` (default "United States"), `datePosted` (`pastWeek` by default; `past24Hours`, `pastMonth`, `anyTime` also valid), `limitPerSource` (default 100), `scrapeCompany: true`.
   - Searches are fuzzy. Re-check each result's title against the roles asked for, and drop the ones that don't match. Drop companies outside the requested size using the employee count on the result.
   - Remove duplicates (same company and title).

5. **Judge each company and role as a lead**, using only what the result shows: the title, seniority, the company's size and what it does, and the posting text. For each, decide:
   - does it look like the kind of company the user asked for?
   - is it a role a recruiter could plausibly be paid to fill (senior, first-of-its-kind, hard to fill in-house)?
   - score 1-10 for likelihood they'd pay a recruiter, and write one plain sentence naming the evidence (e.g. "first dedicated RevOps hire at a 67-person insurance-software company").
   - **Never state or guess funding stage, investors, amounts, revenue or growth.** You don't have that data. If the company's own text says it is venture- or PE-backed, you may quote that as what it says about itself.
   - Keep scores of 6 and above unless the user asks for more.

6. **Return the list**, best first, sized to the request (default 25):

   | # | Company | What they do | Open role | Location | Posted | Employees | Fit | Why | Link |
   |---|---|---|---|---|---|---|---|---|---|

   Then, in a few lines:
   - **How it narrowed**: postings found, how many matched the roles, how many made the list.
   - **Companies with nothing** (mode A): which ones and why.
   - **What I'd change**: one suggestion for the next run.

7. **Offer the next step** in one line: save it to a Google Sheet if that connector is available, or find the founder or hiring contact for the top companies. Do neither unasked.

## Rules

- Cost is small: about $0.001 per LinkedIn job result, and the company-board Actor charges only for companies it can read. Mention the estimate in one line only if the run would exceed about 500 results.
- Do not contact anyone. This skill finds and ranks; it doesn't send messages.
- Report only what the results show. If a field is missing, write "not stated".
- Keep the output plain: no model names, raw JSON or tool chatter in what the user sees.
