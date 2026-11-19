# Technical Specification Document

**Project:** Financial Literacy & Stock Learning Platform
**Team members:** Jal Patel, Jay Natarajan
**Course deliverable:** Assignment #9, Technical Specification

This is how the system in the requirements document (version 1.0) will be built. It does not add a new design. Diagrams stay in the D0, D1, and D2 packets and are named below, not linked, because this file is submitted on its own.

---

## 1. Version and change log

| Version | Date | Change | Author |
| --- | --- | --- | --- |
| 0.1 | Nov 12, 2026 | D0 architecture and D1 planning engine. | Jal Patel |
| 0.2 | Nov 14, 2026 | D2 for the web client, gateway, learning service, stock service, database, and edge cases. | Jal Patel |
| 1.0 | Nov 18, 2026 | One specification traced to requirements document v1.0: interfaces, endpoint table, schema, stack, constraints, and the traceability matrix. | Jay Natarajan, Jal Patel |

---

## 2. Architecture summary

The app is a browser client and a layered backend. That picture is the D0 block diagram. The learner only talks to C1, the web client. C1 only talks to C2, the API gateway, over HTTPS. C2 checks the OpenID Connect token with E1 on every private request, then forwards the call down to one service: C3 (lessons, progress, tooltips), C4 (budget math and growth), or C5 (stocks and paper picks). C3 may call E3 to reword a tooltip. C5 may call E2 for a quote. C3, C4, and C5 do not touch tables directly. They call named operations on C6, PostgreSQL (D2 decision R-06, interface I9).

For version 1 we ship C2 through C5 as one Python container, C1 as static files, and one managed Postgres database. That fits a free tier and still leaves a written contract on each service if we split one out later (D0 decision D4).

Two flows matter.

1. Planning (D0 data-flow A). C1 checks the numbers, then POSTs them to C2 with the sign-in token (I2). C2 checks the token (I3) and calls C4 (I5). C4 does the decimal math, writes the plan and its rows in one transaction (I9), and returns the summary. If the save fails, the summary still comes back with persisted set to false.
2. Stock research (D0 data-flow B, and the D2 error-path figure). C1 asks C2 for a ticker (I2). C5 uses the saved snapshot if it is under 15 minutes old. Otherwise it calls E2 with a 3 second cutoff (I8), computes P/E and the red flags, and saves the snapshot. If E2 fails, C5 returns the last snapshot with stale set to true. If there is no snapshot, it returns 503.

---

## 3. Interfaces and data formats

### 3.1 Boundaries (I1 to I9)

| ID | From and to | Inputs | Outputs | Format | What happens on an error, and who handles it |
| --- | --- | --- | --- | --- | --- |
| I1 | Learner to C1 | Income, savings, expenses, debts, years, ticker, clicks | Lessons, charts plus a text version, growth table, tooltips, stock pages | Browser UI | C1 blocks empty, negative, or non-numeric fields and does not send (EC-01). Submit stays disabled while a request is out (EC-02). |
| I2 | C1 to C2 | Method, path, JSON body, bearer token | JSON data, or an error code and message, plus an HTTP status | JSON over HTTPS | 400 bad input, 401 not signed in, 413 too large, 502 or 503 if a service is down. C1 sends the learner to sign in on 401. On a network or 5xx error it shows a plain message and Retry, and keeps what they typed (EC-03, EC-04). |
| I3 | C2 to E1 | ID token | Claims: subject and expiry | JWT over HTTPS (OpenID Connect) | Bad or expired token: C2 returns 401. If E1 is down, people already signed in keep working and new sign-ins see "sign-in unavailable". Lessons stay readable (EC-05). |
| I4 | C2 to C3 | Lesson id, user id, or metric, value, and benchmark | Lesson, completion, or a tooltip with its source | JSON over internal HTTP | 404 unknown lesson. 422 bad numbers, which C2 relays as 400. Completing a lesson twice changes nothing (EC-06). A bad AI reply never leaves C3. The template is used instead (EC-07). |
| I5 | C2 to C4 | Plan request plus the verified user id | Plan summary | JSON over internal HTTP | 422 names the bad field. C2 relays that as 400 (EC-09). Leftover of 0 or less is still 200, with no projections and null months (EC-08). A failed save is still 200, with persisted false (EC-10). |
| I6 | C2 to C5 | Ticker, user id | Snapshot, or the list of picks | JSON over internal HTTP | 400 bad ticker, 404 unknown ticker, 409 duplicate pick (EC-11). If E2 fails, 200 with the saved snapshot and stale true, or 503 if there is no copy (EC-12). |
| I7 | C3 to E3 | Prompt, max tokens, API key | Completion text | JSON over HTTPS | 4 second cutoff, an HTTP error, or a reply that is too long, changes a number, or gives buy or sell advice: C3 throws the reply away and returns the template (EC-07). |
| I8 | C5 to E2 | Quote request for one symbol, API key | Vendor JSON: symbol, price, EPS, sector, history | JSON over HTTPS | 3 second cutoff, no retry, 429, 5xx, or junk JSON: C5 uses the saved snapshot and sets stale true (EC-12). |
| I9 | C3, C4, C5 to C6 | Named operations such as save plan, get lesson, save snapshot, save pick | Rows, or none | SQL over TLS | Multi-row writes are one transaction. A failed read is retried once. After that, C3 and C5 return 503 and C4 returns the summary with persisted false. A failed cache read counts as no cache (EC-10, EC-13). |

### 3.2 Endpoints (/v1)

The version is in the path. Adding an optional request field or a new response field is fine on /v1. Removing or renaming a field, changing a unit, tightening a range, or adding a required field means /v2, and /v1 stays up until the client has moved.

| Method and path | Owner | Sign-in | Request | Success | Errors |
| --- | --- | --- | --- | --- | --- |
| POST /v1/plan | C4 | Yes | monthlyIncome, monthlySavings, years, expenses, debts. C2 sets userId from the token. | 200, plan summary: planId, createdAt, totals, leftover, emergency-fund min and max, months to goal, debtFlag, savingsCapped, monthlyInvested, totalContributed, years, projections, persisted | 400, 401, 422, 500 |
| GET /v1/plan/latest | C4 | Yes | none | 200, newest plan summary and the inputs. This is the Could item, REQ-11. | 401, 404, 503 |
| GET /v1/lessons/{lessonId} | C3 | No | lesson id | 200, title, sections, examples, optional easier lesson id | 404, 503 |
| POST /v1/lessons/{lessonId}/complete | C3 | Yes | lesson id | 200, lesson id and completedAt. A second call does not add a row. | 401, 404, 503 |
| POST /v1/explain | C3 | No | metric, value, optional benchmark | 200, text and source ("ai" or "template") | 400, 422 |
| GET /v1/stocks/{ticker} | C5 | Yes | ticker, 1 to 5 letters | 200, snapshot: ticker, name, sector, price, eps, pe, industry average, flags, history, asOf, stale | 400, 401, 404, 503 |
| POST /v1/picks | C5 | Yes | ticker | 201, pick id, ticker, entry price, time | 400, 401, 404, 409, 503 |
| GET /v1/picks | C5 | Yes | none | 200, list of picks with entry price, current price, and percent change | 401, 503 |

C2 returns errors in one shape:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "One or more input fields are invalid.",
    "details": [{ "field": "monthlyIncome", "problem": "must be between 0.01 and 1000000" }]
  }
}
```

The plan request and the 200 response are the D1 example (income 2500, savings 400, 30 years, expenses 1850). Leftover is 650.00, the goal is 5550.00 to 11100.00, months are 9 and 17, and the 7% stock-index end value is 487988.40. The full JSON is in D1 section 5. It is not copied here again.

### 3.3 Database (C6)

The tables are the D1 data model plus the D2 data model. Money columns are exact NUMERIC, not floats.

| Table | Used by | Primary key | Other keys |
| --- | --- | --- | --- |
| User | C2 | user_id | idp_subject, unique. This is the E1 subject id. |
| Plan | C4 | plan_id | user_id, foreign key to User. Index on (user_id, created_at newest first) for "latest plan". |
| ExpenseItem | C4 | expense_id | plan_id, foreign key to Plan. category 1 to 40 characters, amount in USD per month. |
| DebtItem | C4 | debt_id | plan_id, foreign key to Plan. type, balance, apr as a fraction from 0 to 1. |
| InvestmentForm | C4 | form_id | unique name, assumed annual return from 0 to 0.20, display order. |
| ProjectionResult | C4 | (plan_id, form_id) | Both parts are foreign keys. Stores the rate used and the end value, so an old plan does not change when we edit the assumed rates. |
| Lesson | C3 | lesson_id | topic, title, content, optional easier_lesson_id, optional form_id. |
| LessonProgress | C3 | (user_id, lesson_id) | Foreign keys to User and Lesson. completed_at. The key makes a second complete a no-op. |
| StockSnapshot | C5 | ticker | name, sector, price, eps, pe (empty when eps is 0 or less), industry average pe, flags, history, as_of. Keyed by ticker, not by user. |
| PracticePick | C5 | pick_id | user_id foreign key to User. Unique (user_id, ticker). entry_price, added_at. The ticker is not a foreign key, so a cache row can expire without deleting the pick. |

### 3.4 The two calculations and the two guardrails

Budget summary (C4, D1 algorithm 1). Sum the expenses, subtract from income, set the 3-month and 6-month goals, and if leftover is positive divide each goal by leftover and round to a whole month (halves up). Otherwise the months are null. The debt flag is "any balance above 0". Time grows with the number of expense rows plus debt rows, which is tiny next to the database write.

Growth (C4, D1 algorithm 2). Let i be the yearly return divided by 12, and n be 12 times the years. End value is monthly invested times ((1 + i) to the n, minus 1) divided by i. If the return is 0, end value is monthly invested times n, so we never divide by zero.

Red flags (C5, D2 decision R-05). Three plain rules: EPS is 0 or less, P/E is far above the industry average, or the price jumped a lot recently. The learner can see which rule fired. The exact cutoffs are still AQ-04.

Tooltips (C3, D0 decision D3). Write the template first. Optionally ask E3 to reword it inside 4 seconds. Keep the template if the reply is too long, changes a number, or gives buy or sell advice.

---

## 4. Technology stack

Versions below are the ones we plan to pin in requirements.txt and package.json when we start coding (D1 sections 4 and 6).

| Layer | Choice | Version and license | Why, tied to an ID |
| --- | --- | --- | --- |
| Frontend (C1) | React | 18.3.1, MIT | Static files on cheap hosting (CON-01). Accessibility tools for NFR-06 and CON-06. Jay has used it (CON-07). MIT fits CON-05. |
| Backend (C2 to C5) | Python 3, FastAPI, Uvicorn, Pydantic | Python 3.12.x (PSF), FastAPI 0.115.0 (MIT), Uvicorn 0.32.0 (BSD-3-Clause), Pydantic 2.9.2 (MIT) | The standard decimal type is the exact money math in REQ-01 and REQ-03. Pydantic is the 422 field errors. Both of us have used Python. One small container fits CON-01 and the 1 second budget in NFR-01. |
| Database access (I9) | SQLAlchemy 2 and psycopg 3 | SQLAlchemy 2.0.36 (MIT), psycopg 3.2.3 (LGPL, unmodified) | One transaction for a plan and its rows (REQ-03) and named operations instead of open table access (D2 R-06, NFR-03). Unmodified LGPL does not force us off MIT (CON-05). |
| Database (C6) | Managed PostgreSQL | 16.x, PostgreSQL License | Foreign keys, a real NUMERIC type, and transactions (REQ-01, REQ-03, REQ-07). Permissive license (CON-05). A managed free tier (CON-01). |
| Sign-in (E1) | A hosted OpenID Connect provider, not picked yet (task T5) | Vendor terms | No password table (NFR-04, CON-04). Has to fit CON-01. |
| Stock data (E2) | An outside quote API, not picked yet (task T4) | Vendor terms | Real price and EPS for REQ-05. The 15-minute cache keeps us inside a free quota (CON-01, REQ-06, NFR-02). |
| Tooltips (E3) | A general AI API, not picked yet (task T8). Leaning toward a cheaper model, maybe local later. | Vendor terms | Rewords a template (REQ-09). The app still works with the template if the vendor is down or starts costing money (NFR-03, CON-02). |
| Hosting | A managed container plus static files. Secrets in environment variables. | The host's terms | Nobody administers a server (CON-07). Free or student tier (CON-01). The process refuses to start if a secret is missing (NFR-04, CON-04). |

---

## 5. Known constraints

Each CON item from the requirements document, and what it makes us do.

| ID | It forces | It rules out |
| --- | --- | --- |
| CON-01 | One container for C2 to C5, static files for C1, managed Postgres, and a 15-minute stock cache in C6. | A separate deploy per service, a paid market-data feed, a cache server we would have to run ourselves. |
| CON-02 | Paper picks only, three fixed red-flag rules, and a filter that blocks buy or sell wording from E3. | Linking a bank or brokerage, placing a trade, letting the AI write the lesson or the advice. |
| CON-03 | Queries scoped by user id, a disclosure on the planner form before the first submit, logs with no income or tokens, and deletion of learner rows when the account or the project ends. | Extra personal fields, one learner reading another's plan, logging the raw request body. |
| CON-04 | E1 checked at C2, HTTPS and TLS, secrets only in the environment, startup fails if one is missing. | Our own password table, keys in the source tree, a committed .env file. |
| CON-05 | OSI permissive libraries, or LGPL only if we do not modify the library. Read the vendor terms before go-live. | A dependency that would force us to relicense the project. MongoDB's server license. |
| CON-06 | Plain wording, a text version of every chart, warnings that are not color-only, and a keyboard and screen-reader pass in task T14. | A chart with no table, a red flag that is only a red dot. |
| CON-07 | Half the requirements are Must, one container, the buffer days in Assignment 8 before each major milestone, and a named owner per task. | Kubernetes, a native phone app, and features that are not on the Must or Should lists. |

---

## 6. Traceability matrix

Status is Designed when D0, D1, or D2 already specifies it, and Planned when it is a Could or it waits on a vendor or on the Week 10 test writeup. Test ids do not exist yet. Assignment 10 will fill that column. Nothing here is orphaned: every row names a design element.

| Req ID | Description | Design Component (D0/D1/D2) | Status | Test Case |
| --- | --- | --- | --- | --- |
| REQ-01 | Budget summary and emergency-fund goal, exact decimals | C4, D1 algorithm 1, interface I5. C6 tables Plan, ExpenseItem, DebtItem | Designed | Week 10, not written yet |
| REQ-02 | Debt flag, debt lesson, leftover of 0 or less | C4, D1 algorithm 1, EC-08. C3 for the lesson. C6 table DebtItem | Designed | Week 10, not written yet |
| REQ-03 | Growth projections and one transaction for the save | C4, D1 algorithm 2, I5. C6 tables InvestmentForm and ProjectionResult, I9 | Designed | Week 10, not written yet |
| REQ-04 | Public lessons, P/E examples and calculator, complete twice is safe | C3, I4, EC-06. C1 calculator. C6 tables Lesson and LessonProgress | Designed | Week 10, not written yet |
| REQ-05 | Ticker lookup, 15-minute cache, P/E, three red flags | C5, I6, I8, EC-11. E2. C6 table StockSnapshot | Designed | Week 10, not written yet |
| REQ-06 | Stale copy and the "Data from yesterday" sentence | C5, D2 error-path figure, EC-12. C1 for the message. C6 table StockSnapshot | Designed | Week 10, not written yet |
| REQ-07 | Paper picks, entry price, percent change | C5, I6, EC-11. C1. C6 table PracticePick | Designed | Week 10, not written yet |
| REQ-08 | Field checks, no double submit, Estimate for me | C1, D2 web client behavior, EC-01, EC-02 | Designed | Week 10, not written yet |
| REQ-09 | Tooltips, 4 second cutoff, template fallback | C3, I4, I7, EC-07. E3 | Designed | Week 10, not written yet |
| REQ-10 | Easier-lesson link | C3, D2 learning service. C6 column Lesson.easier_lesson_id. C1 | Designed | Week 10, not written yet |
| REQ-11 | Load the latest plan (Could) | C4. C6 index on Plan (user_id, created_at). C1. GET /v1/plan/latest | Planned | Week 10, not written yet |
| REQ-12 | 5-year chart with a text summary (Could) | C1, C5, C6 column StockSnapshot.history | Planned | Week 10, not written yet |
| NFR-01 | Plan in 1 second for 95% of requests | C1, C2, C4, C6. D0 data-flow A. D1 algorithms 1 and 2 | Designed | Week 10, not written yet |
| NFR-02 | Stock page in 3 seconds, cache hit in 0.5 seconds | C1, C2, C5, C6, E2. D0 data-flow B. D2 error-path figure | Designed | Week 10, not written yet |
| NFR-03 | EC-03, EC-05, EC-07, EC-10, EC-12, EC-13 each have a defined result | C1, C2, C3, C4, C5, C6. D2 edge-case table | Designed | Week 10, not written yet |
| NFR-04 | E1 sign-in, TLS, secrets in the environment, logs without figures | C2, I3, EC-04. E1. D2 glue: authentication, configuration, logging | Designed | Week 10, not written yet |
| NFR-05 | 3 testers finish both use cases in 10 minutes | C1 and the full stack. Assignment 8 milestone M8 | Planned | Week 10, not written yet |
| NFR-06 | Text for charts, keyboard and screen reader | C1, D2 web client. Assignment 8 task T14 | Designed | Week 10, not written yet |
| CON-01 | Student-tier hosting and free API quotas | One container for C2 to C5 (D0 decision D4). Managed Postgres and the stock cache | Designed | Week 10, not written yet |
| CON-02 | Education only: no bank credentials, no trades, no advice | C3 tooltip checks (D0 decision D3). C5 paper picks (D0 decision D1). C6 schema | Designed | Week 10, not written yet |
| CON-03 | Account scoping, logs without figures, deletion | C2 token check. C6 foreign keys on user_id. D2 logging rules | Designed | Week 10, not written yet |
| CON-04 | Outside sign-in and secrets outside git | C2, I3. E1. D2 configuration glue | Designed | Week 10, not written yet |
| CON-05 | MIT project license and compatible libraries | Stack table in section 4. D1 sections 4 and 6 | Designed | Week 10, not written yet |
| CON-06 | Plain language, not color-only, keyboard and screen reader | C1. C3 lesson text. D0 constraints. D2 web client | Designed | Week 10, not written yet |
| CON-07 | Two people, about 219 hours, 14 weeks | One container (D0 decision D4). Assignment 8 task list and milestones | Designed | Week 10, not written yet |

Orphan check: none. Every REQ, NFR, and CON row names at least one design element.

---

## 7. Advisor review

Not held yet. It is set for Friday, November 20, 2026. Jal Patel and Jay Natarajan will both be there. We will bring this specification, the requirements document (v1.0), and the matrix above.

What we want to walk through:

1. The Must list is half of the requirements. Is that a realistic first release for CS 5002?
2. Open items AQ-03 through AQ-06: which sign-in host, which stock vendor, which AI API, and the actual red-flag cutoffs.
3. Confirm CON-01 through CON-07, consolidated from the Week 3 constraints and D0 through D2, still read the same way in the essay and in this specification.

No edits have come out of a review yet. After the meeting we will add the date, the advisor's name, what was said, and any ID or interface we changed, and we will log that as version 1.1 in section 1 of both documents.
