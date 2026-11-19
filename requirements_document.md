# Requirements Document

**Project:** Financial Literacy & Stock Learning Platform
**Team members:** Jal Patel, Jay Natarajan
**Course deliverable:** Assignment #9, Requirements Document

This is the list of what the system has to do, for whom, and what we are leaving out. It pulls together the Week 4 stories, the D0, D1, and D2 designs, and the Week 8 plan. The IDs below are the ones the Week 10 test plan will check. Diagrams stay in the D0, D1, and D2 packets. They are cited by name here, not linked, because this file is submitted on its own.

---

## 1. Version and change log


| Version | Date         | Change                                                                                                                                                       | Author                   |
| ------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------ |
| 0.1     | Sep 28, 2026 | First user stories, use cases, and acceptance criteria (Assignment 4).                                                                                       | Jal Patel                |
| 0.2     | Nov 14, 2026 | Added the proposed non-functional requirements (NFR-01 to NFR-04 in D2) and the edge cases from D0 through D2.                                               | Jal Patel, Jay Natarajan |
| 1.0     | Nov 18, 2026 | First full requirements document. Stable REQ, NFR, and CON IDs, MoSCoW priorities, and the constraints from the Week 3 essay carried through D0, D1, and D2. | Jay Natarajan, Jal Patel |


Requirement IDs are permanent. If we drop one later, we mark it Withdrawn in this log and leave the number unused. We do not renumber.

---



## 2. Purpose and scope

The product helps early earners (ages 18 to 30) learn budgeting, saving, and the main forms of investing, using their own numbers and real market data, before they risk real money.

**In this release:**

- A browser app (C1) that talks only to one gateway (C2). C2 routes to the learning service (C3), the planning engine (C4), and the stock research service (C5). All of them use one database (C6).
- A budget summary, an emergency-fund goal, a debt flag, and an illustrative growth comparison (C4).
- Short lessons, a P/E calculator, lesson progress, and tooltips that start from a fixed sentence and may be reworded by an outside AI service (C3, E3).
- Stock lookup with three plain red-flag rules, a saved copy when the vendor is down, and paper-only practice picks (C5, E2).
- Sign-in through an outside OpenID Connect provider (E1). We do not store passwords.

**Not in this release:** real bank or brokerage links, real trades, buy or sell advice, selling a practice pick, notifications, and a native phone app. The Won't list in section 7 is the full set.

---



## 3. Stakeholders and user goals

The person this is for is the learner, an early earner (18 to 30) using a browser. They want a budget and an emergency-fund goal from their own numbers, a plain explanation of how savings could grow, a P/E lesson they can try on a real stock, and a way to practice stock research without real money.

Jal Patel and Jay Natarajan are building it, about 219 hours, split about 50/50, from December 21, 2026 to March 28, 2027. The advisor reviews the design. The instructor grades the deliverables.

The Week 4 stories and checks this document traces to:


| Source     | What it asks for                                                                                                                |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------- |
| US-01      | Learn budgeting and emergency funds with my own income, expenses, and debts.                                                    |
| US-02      | Understand stocks, risk, and diversification in plain language.                                                                 |
| US-03      | Research real stocks and track practice picks.                                                                                  |
| US-04      | Avoid beginner mistakes, using red flags and guides.                                                                            |
| UC-01      | Learn P/E in a lesson and a calculator, then try it on a real stock. Offer an easier lesson if the learner is stuck.            |
| UC-02      | Build a monthly budget, with an "Estimate for me" option, and a debt lesson first when any debt has a balance.                  |
| UC-02-P    | Come back later and update a saved plan. Not required for the first release.                                                    |
| AC-UC01-01 | The P/E lesson has an explanation, three fixed examples (Apple, Coca-Cola, Tesla), and a working calculator.                    |
| AC-UC01-02 | If live data is down, show the last saved copy with the exact line "Data from yesterday. Try another stock or come back later." |
| AC-UC02-01 | Income 2500 and expenses 1850 gives leftover 650, an emergency-fund goal of 5550 to 11100, and 9 to 17 months.                  |
| AC-UC02-02 | Any debt with a balance above 0 sets the debt flag and opens the debt lesson first.                                             |


---



## 4. Glossary

These words mean the same thing here, in the technical specification, and later in the test plan.


| Term                  | Meaning                                                                                                                                                      |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Learner               | The person using the site. Target age 18 to 30. Lessons can be read without an account. Plans, progress, and picks belong to a signed-in learner.            |
| Plan                  | One saved run of the planner: income, expenses, debts, savings target, years, and the computed summary.                                                      |
| Leftover              | Monthly income minus total monthly expenses, in USD. It can be zero or negative.                                                                             |
| Emergency-fund target | 3 months of expenses (emergencyFundMin) to 6 months (emergencyFundMax).                                                                                      |
| Debt flag             | True when any debt row has a balance above 0. A balance of 0 does not count.                                                                                 |
| Investment form       | A kind of saving or investing (high-yield savings, bond fund, stock index fund) with a fixed assumed yearly return. Not a prediction.                        |
| Growth projection     | What a fixed monthly amount grows to over a number of years at that assumed return. Nominal only. No inflation, taxes, or fees.                              |
| P/E                   | Price divided by earnings per share. Left blank when earnings are zero or negative.                                                                          |
| Stock snapshot        | The saved copy of a quote: price, EPS, P/E, industry average P/E, red flags, history, and the time it was fetched.                                           |
| Stale                 | A snapshot older than 15 minutes, shown when the stock vendor fails.                                                                                         |
| Red flag              | One of three fixed warnings: no profit, P/E far above the industry, or a large recent price jump. The numeric cutoffs are still open (AQ-04).                |
| Practice pick         | A paper pick. We store the price when the learner picked it and show the percent change. No real trade.                                                      |
| Tooltip               | A short explanation. We write a template sentence first. E3 may reword it only if the reply stays short, keeps the numbers, and gives no buy or sell advice. |
| ID token              | The sign-in token from E1. C2 checks it on every private request. We store the subject id (`sub`), not a password.                                           |


---



## 5. Functional requirements


| ID     | The system shall                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | Priority | Source                                 | How we will check it                                                       |
| ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | -------------------------------------- | -------------------------------------------------------------------------- |
| REQ-01 | Compute total expenses, leftover, an emergency-fund goal of 3 times and 6 times total expenses, and the whole months of leftover needed to reach each end (halves round up). Use exact decimal math to 2 places. Accept monthly income from 0.01 to 1,000,000, 0 to 50 expenses (each 0 to 1,000,000), and 0 to 20 debts (each balance 0 to 10,000,000). The UC-02 example shall come out to leftover 650.00, goal 5550.00 to 11100.00, and 9 to 17 months.                                    | Must     | US-01, UC-02, AC-UC02-01               | Unit test against the UC-02 example                                        |
| REQ-02 | Set the debt flag when any debt balance is above 0, and point the learner at the debt lesson first. A balance of 0 does not count. If leftover is 0 or less, months-to-goal shall be null, projections shall be an empty list, the plan shall still be saved, and the budgeting lesson shall be suggested.                                                                                                                                                                                     | Must     | US-01, UC-02, AC-UC02-02               | Unit test: one debt above 0, one debt at 0, leftover of 0                  |
| REQ-03 | Project growth for each seeded investment form over 1 to 60 years. Monthly amount invested is the smaller of monthly savings and leftover. If savings was larger, set savingsCapped. If the yearly return is 0, end value is monthly invested times the number of months. Otherwise use the standard end-of-month compound formula. 400 a month for 30 years at 7% shall equal 487988.40. Save the plan, its expense rows, its debt rows, and the projection rows in one database transaction. | Must     | US-02, UC-02                           | Unit test against 487988.40, plus a failed-save test                       |
| REQ-04 | Serve the lessons (budgeting, debt, investing basics, P/E, common mistakes) without sign-in. The P/E lesson shall include the explanation, the Apple, Coca-Cola, and Tesla examples, and a calculator. Marking a lesson complete twice for the same learner shall not create a second row or an error.                                                                                                                                                                                         | Must     | US-01, US-02, US-04, UC-01, AC-UC01-01 | Integration test, including a second complete call                         |
| REQ-05 | Accept a ticker of 1 to 5 letters (400 if not, 404 if the vendor does not know it). Use the saved snapshot if it is under 15 minutes old, otherwise ask E2. Compute P/E, and leave P/E empty when EPS is 0 or less. Apply the three red-flag rules and return the snapshot with the fetch time and stale set to false.                                                                                                                                                                         | Must     | US-02, US-03, US-04, UC-01             | Integration test with a known ticker and a bad ticker                      |
| REQ-06 | If E2 times out (over 3 seconds), returns 429 or 5xx, or returns junk JSON, show the last saved snapshot of any age with stale set to true and the exact sentence "Data from yesterday. Try another stock or come back later." If there is no saved snapshot, return 503 and say to try another stock.                                                                                                                                                                                         | Must     | US-03, UC-01, AC-UC01-02               | Fault test: vendor down with a cache, and vendor down with an empty cache  |
| REQ-07 | Let a signed-in learner save a paper pick once per ticker, store the price at that moment, reject a duplicate with 409, and show percent change against the latest price. Do not place a real trade.                                                                                                                                                                                                                                                                                           | Must     | US-03                                  | Integration test, including a second save of the same ticker               |
| REQ-08 | Check planner and stock-search fields in the browser before sending. Block empty, non-numeric, and out-of-range values, and disable Submit while a request is in flight. "Estimate for me" shall fill a 50/30/20 budget from the income the learner typed.                                                                                                                                                                                                                                     | Should   | US-01, UC-02                           | UI check against EC-01 and EC-02                                           |
| REQ-09 | Build a tooltip from a fixed template sentence. E3 may reword it, with a 4 second cutoff. If E3 is slow, down, too long, changes a number, or gives buy or sell advice, return the template and do not show an error.                                                                                                                                                                                                                                                                          | Should   | US-02, US-04                           | Unit test of each rejected AI reply                                        |
| REQ-10 | Include an optional easier-lesson id on a lesson so a learner stuck on P/E can open a simpler lesson.                                                                                                                                                                                                                                                                                                                                                                                          | Should   | UC-01                                  | Integration test on the P/E lesson payload                                 |
| REQ-11 | Let a signed-in learner load their newest saved plan back into the planner form.                                                                                                                                                                                                                                                                                                                                                                                                               | Could    | UC-02-P                                | Integration test once the screen exists                                    |
| REQ-12 | Show a 5-year price chart on the stock page, with a text summary next to it.                                                                                                                                                                                                                                                                                                                                                                                                                   | Could    | US-03                                  | UI check. Drop this first if the screens run long (Assignment 8, risk R3). |


---



## 6. Non-functional requirements


| ID     | Area          | The system shall                                                                                                                                                                                                                                                                                                                                                                    | Priority | Source                                                      | How we will check it                                       |
| ------ | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ----------------------------------------------------------- | ---------------------------------------------------------- |
| NFR-01 | Performance   | Return a plan and show the summary within 1 second for 95% of requests, with 10 or fewer learners at once.                                                                                                                                                                                                                                                                          | Should   | D0 timing notes, D2 NFR-01                                  | Timer on POST /v1/plan                                     |
| NFR-02 | Performance   | Show a stock page within 3 seconds for 95% of lookups, cut the E2 call off at 3 seconds, and answer from a cache younger than 15 minutes within 0.5 seconds.                                                                                                                                                                                                                        | Should   | D0 timing notes, D2 NFR-01                                  | Timer on GET /v1/stocks/{ticker}, including a warm cache   |
| NFR-03 | Reliability   | Handle EC-03, EC-05, EC-07, EC-10, EC-12, and EC-13 without an unhandled 500. Lessons stay readable if sign-in is down. A failed stock vendor serves the stale copy. A failed AI call serves the template. A failed plan save still returns the summary with persisted set to false. A failed read is retried once.                                                                 | Must     | D2 NFR-02                                                   | One fault test per edge-case id                            |
| NFR-04 | Security      | Sign learners in only through E1. Store no passwords and no bank or brokerage credentials. Check the ID token at C2 on every private request, and return 401 if it is missing or expired. Use HTTPS for the browser and TLS for the database. Read secrets from environment variables and refuse to start if one is missing. Never log income figures, sign-in tokens, or API keys. | Must     | D2 NFR-03, D2 glue (authentication, configuration, logging) | Auth test, a missing-secret startup test, and a log review |
| NFR-05 | Usability     | Have 3 outside testers, ages 18 to 30, each finish UC-01 and UC-02 with no help, in 10 minutes or less per use case.                                                                                                                                                                                                                                                                | Should   | Assignment 8 milestone M8, D2 NFR-04                        | Pilot session notes                                        |
| NFR-06 | Accessibility | Give every chart a text version, not rely on color alone for a warning, and pass a keyboard-only and screen-reader pass of the planner, lessons, stock search, and practice picks.                                                                                                                                                                                                  | Should   | D0 constraints, D2 NFR-04, Assignment 8 task T14            | Manual pass, recorded in the test notes                    |


---



## 7. MoSCoW summary

There are 18 requirements (REQ-01 through REQ-12, NFR-01 through NFR-06). Nine are Must, which is half. That is as high as we want for two people over 14 weeks. Latency, the AI tooltip, and the accessibility pass are Should, because the lessons and the math still work if those slip.

**Must**

- REQ-01. Budget summary and emergency-fund goal, exact to the UC-02 example.
- REQ-02. Debt flag, debt lesson first, and leftover of 0 or less.
- REQ-03. Growth comparison and an all-or-nothing plan save.
- REQ-04. Public lessons, the P/E lesson, and completing a lesson twice is safe.
- REQ-05. Stock lookup, 15-minute cache, P/E, and three red flags.
- REQ-06. Stale copy and the exact "Data from yesterday" sentence.
- REQ-07. Paper picks with entry price and percent change.
- NFR-03. The failure cases in D2 still return a defined result.
- NFR-04. Outside sign-in, no passwords, TLS, secrets outside the repo, logs without figures.

**Should**

- REQ-08. Browser-side checks, no double submit, and "Estimate for me".
- REQ-09. Tooltips, 4 second AI cutoff, template if the AI reply is bad.
- REQ-10. Easier-lesson link.
- NFR-01. Plan result in 1 second (95th percentile).
- NFR-02. Stock page in 3 seconds, cache hit in 0.5 seconds.
- NFR-05. 3 testers, both use cases, 10 minutes each, no help.
- NFR-06. Text for every chart, keyboard and screen reader.

**Could**

- REQ-11. Load the latest plan back into the form (UC-02-P).
- REQ-12. 5-year price chart. Drop this first if we are behind (risk R3).

**Won't (this release)**

These stay out. They are not a backlog we quietly build.

- No bank or brokerage linking, and no real-money investing (D0 decision D1).
- No selling a practice pick, and no dividends, taxes, or full portfolio accounting (D2, US-03 gap).
- No AI-written lessons and no buy or sell advice (D0 decision D3).
- No email, SMS, or push notifications (D2 glue: notifications are none).
- No inflation, fees, changing contributions, or random simulations in the growth math (D1 algorithm 2, D2 decision R-02).
- No native iOS or Android app. The client is a browser page (D0 architecture).

---



## 8. Inherited constraints

These are consolidated from the Week 3 constraints and from D0, D1, and D2. The essay uses this same list, so the two stay in agreement (AQ-07).


| ID     | Area              | Constraint                                                                                                                                                                                                                 |
| ------ | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CON-01 | Budget            | Stay on a student budget: one small container, managed PostgreSQL, and free or student tiers for the stock and AI APIs.                                                                                                    |
| CON-02 | Legal and ethical | Education only. No bank credentials, no real trades, no personalized buy or sell advice.                                                                                                                                   |
| CON-03 | Privacy           | Store only the figures the learner types and the sign-in subject id. Scope rows to that account. Keep them out of logs. Delete them when the account is deleted (within 30 days of the request) and when the project ends. |
| CON-04 | Security          | Sign-in is E1 only. No password table. API keys and the database password live in environment variables, not in git.                                                                                                       |
| CON-05 | Licensing         | The project is MIT. Libraries must be compatible: PSF, MIT, BSD-3-Clause, the PostgreSQL license, or unmodified LGPL (psycopg). MongoDB's server license is out.                                                           |
| CON-06 | Accessibility     | Plain language for someone with no finance background. Do not use color as the only signal. Keyboard and screen reader have to work.                                                                                       |
| CON-07 | Team and time     | Two people, about 219 hours, about a 50/50 split, December 21, 2026 through March 28, 2027.                                                                                                                                |


---



## 9. Assumptions and open questions


| ID    | Kind       | What we are assuming, or still need to decide                                                                                                                                                                 | Owner         | Date         |
| ----- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- | ------------ |
| AQ-01 | Assumption | Coding starts Monday, December 21, 2026. The plan ends March 28, 2027, with March 29 to 31 held in reserve (Assignment 8).                                                                                    | Both          | Dec 21, 2026 |
| AQ-02 | Assumption | The 1 second and 3 second targets are measured after the container is warm. We will time a cold start in the Week 11 deploy and compare it to those budgets.                                                  | Jay Natarajan | Mar 5, 2027  |
| AQ-03 | Open       | Which stock vendor (E2) gives price, EPS, sector, and history on a free tier, and what its terms allow. Task T4, risk R2.                                                                                     | Jal Patel     | Jan 8, 2027  |
| AQ-04 | Open       | Where the industry-average P/E comes from, and the exact cutoffs for "P/E far above the industry" and "a big recent price jump" (D2 open questions).                                                          | Jal Patel     | Jan 15, 2027 |
| AQ-05 | Open       | Which OpenID Connect host (E1) and which managed Postgres host, including free-tier limits. Token lifetime is proposed at 1 hour and is not confirmed until the provider is picked (D2 glue, authentication). | Jay Natarajan | Jan 8, 2027  |
| AQ-06 | Open       | Which AI API (E3) rewords tooltips, and whether its terms allow sending one metric and one benchmark (D0 decision D3, task T8).                                                                               | Jay Natarajan | Jan 25, 2027 |
| AQ-07 | Assumption | CON-01 through CON-07 are the Week 3 constraints, carried forward through D0, D1, and D2. If the Nov 20 advisor review changes one, we update the list and the essay together.                                | Jay Natarajan | Nov 20, 2026 |


