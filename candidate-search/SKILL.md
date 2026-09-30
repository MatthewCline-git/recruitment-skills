---
name: candidate-search
description: Build a scored candidate shortlist from LinkedIn for a role. Use when the user wants to find, source or shortlist candidates and gives a job title plus any of: years of experience, location, company size, industry or company type, seniority, past employers, or softer criteria like "first employee", "zero-to-one", "B2B not D2C". Every criterion except the job title is optional.
---

# Candidate search

Turn a role and whatever criteria the user gives into a shortlist of the best-fitting people on LinkedIn, each with a score and a plain reason. The user should never have to go back and forth between LinkedIn and Claude.

## Inputs

Only the **role** is required. Everything else is optional, and leaving it out means "don't filter on this."

| Criterion | Examples |
|---|---|
| Role / job titles | "Head of RevOps", "Director of Marketing" |
| Years of experience | "6-10 years", "at least 5" |
| Location | "New York", "United States", "remote US" |
| Company size (their current employer) | "50-500 people", "startup" |
| Industry / type of company | "B2B SaaS", "vertical SaaS", "fintech" |
| Seniority | "director", "VP", "manager" |
| Past employers / schools | "worked at Stripe or Ramp", "Wharton" |
| Softer criteria | "first 10 employees", "built it from zero", "B2B not D2C", "sold to mid-market" |
| Must-haves and deal-breakers | "must have managed a team", "no agency-only backgrounds" |
| Shortlist size (default 25) and pool size (default 100) | |

## Start here: ask for the parameters

When the skill is invoked **without any search criteria**, do not search yet. Reply with exactly this greeting and list, then stop and wait. Do not run anything yet:

> Hey! I run LinkedIn candidate searches and give you a scored shortlist. I support the parameters below. Please provide them. Only the role is required; skip anything that doesn't matter.
>
> - **Role / job title** (required)
> - **Years of experience**
> - **Location**
> - **Company size** (their current employer)
> - **Industry or type of company** (e.g. B2B SaaS, vertical SaaS)
> - **Seniority**
> - **Past employers or schools**
> - **Softer criteria** (e.g. first 10 employees, built it from zero, B2B not D2C)
> - **Must-haves and deal-breakers**
> - **Shortlist size** (default 25) and **pool size** (default 100)

Once the user answers, run the search with whatever they gave. If they invoked the skill with criteria already in their message, skip the list and run it straight away. If the role is missing from their answer, ask for just that, in one short question.

## Steps

1. **Restate the search in one line** so the user can see what you understood, then go.

2. **Split the criteria into two kinds.**
   - *Searchable* criteria LinkedIn can filter on: titles, location, years of experience, company size, seniority, function, past employers, schools. These go into the search.
   - *Judgement* criteria LinkedIn cannot filter on: "zero-to-one", "B2B vs D2C", "vertical SaaS", "managed a team", "first employee". These go into the scoring in step 5. Never drop them and never pretend the search enforced them.

3. **Run the search** with the Apify tool `harvestapi/linkedin-profile-search` (Apify connector; no LinkedIn login or cookies are involved, and nobody's personal LinkedIn account is used). Use `profileScraperMode: "Full"` so each result includes work history. Set `maxItems` to the pool size (default 100).

   | Criterion | Input field |
   |---|---|
   | Titles | `currentJobTitles` (add close variants), `pastJobTitles` for "has done X before" |
   | Location | `locations` (use full names, "United Kingdom" not "UK") |
   | Years of experience | `yearsOfExperienceIds`: 1 = under 1, 2 = 1-2, 3 = 3-5, 4 = 6-10, 5 = over 10 |
   | Company size | `companyHeadcount`: A self-employed, B 1-10, C 11-50, D 51-200, E 201-500, F 501-1,000, G 1,001-5,000, H 5,001-10,000, I 10,001+ |
   | Seniority | `seniorityLevelIds`: 120 senior, 200 entry manager, 210 experienced manager, 220 director, 300 VP, 310 CXO, 320 owner/partner |
   | Function | `functionIds`: 1 accounting, 2 administrative, 4 business development, 6 consulting, 10 finance, 12 HR, 15 marketing, 18 operations, 19 product, 25 sales, 26 customer success and support |
   | Past employers / schools | `pastCompanies` (LinkedIn company URLs), `schools` |
   | Free text | `searchQuery` |

   Do not use `industryIds` unless you are sure of the code. Put industry in `searchQuery` or judge it in step 5.

   **Don't over-filter.** If fewer than about 15 people come back, loosen the least important filter (usually years of experience, then company size), run once more, and tell the user exactly what you loosened and why.

4. **Write the rubric** before reading any profile, in a few lines: the hard requirements (fail = out), then the preferences with rough weights. Show it to the user. This is what makes scores comparable across people.

5. **Score every profile** against the rubric by reading the full profile: each role, company, dates, and description. For judgement criteria, infer from the companies and roles and say what the inference rests on. For example, "joined when the company was ~20 people" needs evidence; "B2B" can be inferred from what the employer sells.
   - Score 0-100. Anyone failing a hard requirement is out of the shortlist.
   - Use only what the profile shows. If something isn't stated, write "not stated". Never fill a gap with a guess presented as fact.
   - Label inferences as inferred.

6. **Return the shortlist**, best first, sized to the requested shortlist (default 25):

   | # | Name | Now | Location | Years | Score | Why | LinkedIn |
   |---|---|---|---|---|---|---|---|

   "Why" is one plain sentence naming the specific evidence (for example "Built RevOps from scratch at a 40-person B2B SaaS company, then led a team of 6 at a 400-person one").

   Then, in a few lines:
   - **Near misses**: up to 3 people just below the line and the one thing each lacked.
   - **How the pool narrowed**: pool size, how many failed a hard requirement, how many made the list.
   - **What I'd change**: one suggestion to improve the next run, such as a filter that was too tight or a title variant worth adding.

7. **Offer the next step** in one line: save the shortlist to a Google Sheet or draft the outreach message, if the relevant connector is available. Do not do either unasked.

## Rules

- Cost is about $0.10 per 25 search results plus about $0.004 per full profile, so a 100-person pool is roughly $0.80. Mention the estimate in one line only if the pool is over 200.
- Never search by or surface protected characteristics (age, gender, race, religion, and similar). Judge on experience.
- Do not contact anyone. This skill finds and ranks; it doesn't send messages.
- Keep the output plain: no model names, raw JSON or tool chatter in what the user sees.
