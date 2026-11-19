# Test and Validation Plan

**Project:** Financial Literacy & Stock Learning Platform
**Team members:** Jal Patel, Jay Natarajan
**Course deliverable:** Assignment #10, Test and Validation Plan

This plan says how we will prove the platform works before we call it done in CS 5002. Every test below traces to a Week 9 requirement ID and is written so either teammate can run it without asking the other what a step means. Jay Natarajan owns this plan and writes the cases for C2, C3, C6, and the lessons and stock screens. Jal Patel leads execution in task T14 and writes the cases for C4, C5, and the planner screens.

---

## 1. What is being tested this cycle

### In scope this cycle

We are testing every Must and Should requirement from the requirements document (v1.0) across all six components we build (C1 through C6).

| Component | Requirement IDs tested | What is covered |
| --- | --- | --- |
| C1 Web Client | REQ-04, REQ-06, REQ-07, REQ-08, NFR-01, NFR-02, NFR-05, NFR-06 | Planner form checks, double-submit lock, "Estimate for me", P/E calculator, stale-data banner, practice picks view, text chart versions, keyboard and screen-reader pass. |
| C2 API Gateway | NFR-03, NFR-04, CON-03, CON-04 | JWT verification on private routes, public access to lessons, uniform error JSON, 401 and 503 handling when E1 is down, request size limits, log hygiene. |
| C3 Learning Service | REQ-04, REQ-09, REQ-10, NFR-03, CON-02 | Public lesson delivery, P/E lesson content, idempotent lesson completion, easier-lesson link, AI tooltip guardrails and 4 second template fallback. |
| C4 Planning Engine | REQ-01, REQ-02, REQ-03, NFR-01, NFR-03 | Exact decimal budget math, emergency-fund targets, debt flag, zero or negative leftover handling, growth projections, atomic saves, persisted false fallback on DB write error. |
| C5 Stock Research Service | REQ-05, REQ-06, REQ-07, NFR-02, NFR-03, CON-02 | Ticker validation, 15-minute cache, P/E math with negative EPS, three red-flag rules, 3 second E2 cutoff and stale fallback, paper practice picks and duplicate 409 rejection. |
| C6 App Database | REQ-01, REQ-03, REQ-04, REQ-05, REQ-07, NFR-03, CON-03 | All-or-nothing plan transactions, unique constraints on user, lesson progress, and picks, one-time read retry, account-scoped queries. |

### Out of scope this cycle

| Excluded item | Category | Reason |
| --- | --- | --- |
| REQ-11 (GET /v1/plan/latest reload screen) | Deferred Could feature | Not needed for MVP; will only be tested if built after M6. |
| REQ-12 (5-year stock price chart) | Deferred Could feature | First item to drop if the screens run behind (Week 8 plan, risk R3). |
| Bank or brokerage links, real trades, selling picks, dividends, notifications, inflation or Monte Carlo math, native mobile apps | Won't list (Requirements v1.0, section 7) | Explicitly excluded from version 1.0 by D0 decisions D1 and D3 and D2 decision R-02. |
| E1 Identity Provider internals (login UI, password reset, MFA) | Third-party component | Vendor-run OpenID Connect service; we only test C2's handling of valid, expired, missing, and unreachable tokens. |
| E2 Stock Data API internal price accuracy and market feeds | Third-party component | Vendor-run quote API; we only test C5's parsing, caching, red flags, and failure fallbacks. |
| E3 AI Language Model API internals and model weights | Third-party component | Vendor-run LLM; we only test C3's 4 second timeout and output filter (length, numbers kept, no buy/sell advice). |
| High-scale load testing above 10 concurrent users | Known limitation | Classroom and pilot scope on free-tier hosting (CON-01); NFR-01 and NFR-02 only target up to 10 concurrent learners. |

---

## 2. Verification and validation strategy

We use two lenses across the requirements: verification asks whether we built the system right against our own specs, and validation asks whether we built the right system for early earners and our advisor.

**Verification (we run these against the spec):** Jal and Jay check REQ-01 through REQ-10, NFR-01 through NFR-04, and NFR-06 with unit tests, API tests, fault tests, timers, a log and git check, and a keyboard and screen-reader pass. Every C4 number is checked against the D1 examples. Every D2 failure path (EC-01 through EC-13) is triggered on purpose, with a fake timeout, a bad status code, or a dropped database, so a fallback is not something we only notice if we get lucky.

**Validation (someone outside the team runs these):** This covers NFR-05 and whether REQ-04, REQ-05, and REQ-08 are plain enough to use. At milestone M8 (task T14, Week 13 of the build), 3 people ages 18 to 30 who are not on the team each do UC-01 and UC-02 on staging, with no hints. We time them and note where they stall. The advisor sees the same build at M9 and checks that it still teaches, that the red flags say why, and that nothing tells the learner to buy or sell.

Because this is a web application with no custom hardware, sensors, or self-trained machine learning models, hardware tolerance trials and ML model accuracy benchmarks do not apply.

### Test pyramid

| Layer | Approximate count | Coverage focus |
| --- | --- | --- |
| Unit tests (pytest on the Python services) | about 35 | C4 budget and growth formulas, C5 P/E and red-flag rules, C3 tooltip filter, C1 field checks and the 50/30/20 estimate. |
| Integration tests (API calls against local Postgres) | about 25 | C2 routing and token checks, the /v1 endpoints, all-or-nothing saves, unique keys, the 15-minute cache, and fake E1, E2, and E3 failures (EC-04 through EC-13). |
| System tests (browser, some scripted and some by hand) | about 12 | UC-01 and UC-02 in the browser, the stale-data line, Retry, the NFR-01 and NFR-02 timers, and the NFR-06 keyboard and screen-reader pass. |
| Acceptance and pilot validation (outside testers + advisor) | 3 pilot users + 1 advisor demo | Unassisted completion of UC-01 and UC-02 under 10 minutes each (`NFR-05`), plain-language clarity, and final sign-off at M8 and M9. |

---

## 3. Success criteria

Every measurable requirement has a concrete threshold and a clear way to measure it below.

| Metric ID | Metric | Target threshold | How it is measured | Requirement ID |
| --- | --- | --- | --- | --- |
| M-01 | Budget math accuracy | Exact match on the UC-02 case: leftover 650.00, goal 5550.00 to 11100.00, months 9 and 17. Off by 0.01 fails. | Unit test on C4 | REQ-01 |
| M-02 | Growth projection accuracy | Exact match: 400 a month, 30 years, 7% ends at 487988.40. Off by 0.01 fails. | Unit test on C4 | REQ-03 |
| M-03 | Planner response time, warm container | At most 1.0 second at the 95th percentile, with at most 10 people at once | 20 timed POST /v1/plan calls on staging | NFR-01 |
| M-04 | Stock lookup time | At most 3.0 seconds at the 95th percentile for a live lookup. At most 0.5 seconds when the cache is under 15 minutes old | 10 cold and 10 warm GET /v1/stocks/{ticker} calls on staging | NFR-02 |
| M-05 | Vendor cutoffs | E2 stops at 3.0 seconds, plus or minus 0.2. E3 stops at 4.0 seconds, plus or minus 0.2 | A fake server that waits 6 seconds, and a timer in the test | REQ-06, REQ-09, NFR-02 |
| M-06 | Single-dependency failures | All 6 of EC-03, EC-05, EC-07, EC-10, EC-12, and EC-13 return the designed fallback. None of them return an unhandled 500 | Fault tests that check the status code and the body | NFR-03 |
| M-07 | Auth and secrets | Every private route returns 401 with no valid token. No passwords stored. No secrets in git. Logs have no income, tokens, or keys | Auth tests, a startup with a missing environment variable, and a search of the logs and git history | NFR-04, CON-03, CON-04 |
| M-08 | Pilot time | 3 of 3 outside testers finish UC-01 and UC-02 with no help, each in 10 minutes or less | A stopwatch and a notes sheet at the M8 pilot | NFR-05 |
| M-09 | Accessibility | Every chart has a text version. No warning is color only. Keyboard and a screen reader can get through all 4 screens | A keyboard-only pass (Tab, Enter, Esc) and VoiceOver or NVDA on the planner, lessons, stock search, and picks | NFR-06, CON-06 |

---

## 4. Test cases

Every Must requirement (REQ-01 through REQ-07, NFR-03, NFR-04) has at least 3 cases, and each of those groups includes a bad input or a failure from D2. The Should items (REQ-08 through REQ-10, NFR-01, NFR-02, NFR-05, NFR-06) are at the end. Status starts as Not Run. We run these in CS 5002, after coding starts.

Expense and debt rows use the D1 shape. Where a step says rent 1200, that means `{ "category": "rent", "amount": 1200 }`. A debt is `{ "type", "balance", "apr" }`. Calls go through C2 unless a step says otherwise.

### REQ-01: Budget summary and emergency-fund goal

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-001 | Happy path: UC-02 reference budget calculation (`REQ-01`, `AC-UC02-01`) | 1. Send `POST /v1/plan` with valid JWT, `monthlyIncome: 2500`, `expenses: [{rent: 1200}, {food: 300}, {transport: 200}, {entertainment: 150}]`, `debts: []`, `monthlySavings: 400`, `years: 30`. 2. Inspect HTTP status and JSON fields. | HTTP 200. `totalExpenses` is `1850.00`, `leftover` is `650.00`, `emergencyFundMin` is `5550.00`, `emergencyFundMax` is `11100.00`, `monthsToGoalMin` is `9`, `monthsToGoalMax` is `17`, `persisted` is `true`. | Not Run |
| TC-002 | Boundary: zero expenses and duplicate categories (`REQ-01`) | 1. Send `POST /v1/plan` with `monthlyIncome: 2000`, `expenses: []`. 2. Send a second request with `expenses: [{food: 100.10}, {food: 50.25}]`. | First call returns HTTP 200 with `totalExpenses: 0.00`, `leftover: 2000.00`, targets `0.00`, months `0`. Second call sums duplicates cleanly to `totalExpenses: 150.35` with exact 2-decimal precision (no float noise). | Not Run |
| TC-003 | Failure: negative or out-of-range income and expenses (`REQ-01`, `EC-09`) | 1. Send `POST /v1/plan` with `monthlyIncome: -100`. 2. Send `POST /v1/plan` with `monthlyIncome: 2500` and 51 expense items. 3. Query `Plan` table in C6. | C4 returns 422 (relayed by C2 as 400 `VALIDATION_ERROR`) naming the bad field in `details`. 0 new rows are written to `Plan`. | Not Run |

### REQ-02: Debt flag and zero or negative leftover

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-004 | Happy path: active debt sets flag and suggests debt lesson (`REQ-02`, `AC-UC02-02`) | 1. Send `POST /v1/plan` with `monthlyIncome: 2500`, `expenses: [{rent: 1200}]`, `debts: [{type: "credit card", balance: 1500, apr: 0.22}]`. 2. Check response and UI suggestion. | HTTP 200. `debtFlag` is `true` and the suggested lesson points to the debt-payoff lesson first. | Not Run |
| TC-005 | Boundary: debt with $0 balance is ignored (`REQ-02`) | 1. Send `POST /v1/plan` with `monthlyIncome: 2500`, `expenses: [{rent: 1200}]`, `debts: [{type: "car loan", balance: 0, apr: 0.05}]`. | HTTP 200. `debtFlag` is `false` and the debt-payoff lesson is not forced first. | Not Run |
| TC-006 | Edge case: leftover is $0 or negative (`REQ-02`, `EC-08`) | 1. Send `POST /v1/plan` with `monthlyIncome: 2000`, `expenses: [{rent: 2100}]`, `monthlySavings: 200`, `years: 10`. 2. Query `Plan` table in C6. | HTTP 200. `leftover` is `-100.00`, `monthsToGoalMin` and `monthsToGoalMax` are `null`, `projections` is `[]`, `persisted` is `true`, plan row exists in C6, and C1 suggests the budgeting lesson. | Not Run |

### REQ-03: Growth projections and atomic plan saving

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-007 | Happy path: reference growth projection and atomic save (REQ-03) | 1. Seed InvestmentForm with high-yield savings at 4%, a bond fund at 5%, and a stock index fund at 7%. 2. Send POST /v1/plan with monthlyIncome 2500, the four expenses from TC-001 (total 1850), monthlySavings 400, years 30. 3. Read Plan, ExpenseItem, and ProjectionResult in C6. | HTTP 200. totalContributed is 144000.00. End values are 277619.76, 332903.45, and 487988.40. savingsCapped is false. The rows in C6 share one plan_id. | Not Run |
| TC-008 | Boundary: savings capped at leftover, and a 0% return (REQ-03) | 1. Seed an InvestmentForm with annual return 0. 2. Send POST /v1/plan with monthlyIncome 1000, one expense of 700 (leftover 300), monthlySavings 500, years 10. | HTTP 200. savingsCapped is true. monthlyInvested is 300.00. The 0% form ends at 36000.00 (300 times 120 months), with no divide-by-zero error. | Not Run |
| TC-009 | Failure: database write fails mid-transaction (`REQ-03`, `EC-10`) | 1. Inject a C6 write error during `save_plan` after `Plan` insert and before `ProjectionResult` insert. 2. Send valid `POST /v1/plan`. 3. Query `Plan` and `ExpenseItem` tables. | HTTP 200. Full summary and projections are still returned with `planId: null` and `persisted: false`. Transaction rolls back so 0 partial or orphan rows remain in C6. | Not Run |

### REQ-04: Public lessons, P/E lesson, and idempotent progress

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-010 | Happy path: P/E lesson loads publicly with examples and calculator (`REQ-04`, `AC-UC01-01`) | 1. Open the P/E lesson without signing in (no `Authorization` header to `GET /v1/lessons/pe-ratio`). 2. Enter price `150` and EPS `6` in the lesson P/E calculator. | HTTP 200. Page displays the plain-language explanation, the three fixed examples (Apple, Coca-Cola, Tesla), and the calculator outputs `25.00`. | Not Run |
| TC-011 | Edge case: marking a lesson complete twice is idempotent (`REQ-04`, `EC-06`) | 1. Call `POST /v1/lessons/pe-ratio/complete` with valid JWT. 2. Call the exact same endpoint a second time. 3. Count rows in `LessonProgress` for `(user_id, "pe-ratio")`. | Both calls return HTTP 200 with the same `completedAt` timestamp. Exactly 1 row exists in `LessonProgress`. | Not Run |
| TC-012 | Failure: unknown lesson ID (`REQ-04`) | 1. Call `GET /v1/lessons/does-not-exist`. 2. Call `POST /v1/lessons/does-not-exist/complete` with valid JWT. | Both calls return HTTP 404 with uniform C2 error JSON and write 0 rows to `LessonProgress`. | Not Run |

### REQ-05: Stock ticker lookup, 15-minute cache, P/E, and red flags

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-013 | Happy path: fresh lookup populates cache and second call hits cache (`REQ-05`) | 1. Clear `StockSnapshot` for `MSFT`. 2. Call `GET /v1/stocks/msft` with valid JWT. 3. Call `GET /v1/stocks/MSFT` again 10 seconds later while counting outbound E2 requests. | Both calls return HTTP 200 with upper-cased `MSFT`, computed `pe`, `flags`, `asOf`, and `stale: false`. E2 is called exactly once; the second call reads from C6 cache. | Not Run |
| TC-014 | Edge case: negative EPS leaves P/E empty and triggers no-profit flag (`REQ-05`) | 1. Mock E2 to return `price: 40.00`, `eps: -2.50`, `sector: "Tech"`. 2. Call `GET /v1/stocks/LOSS` with valid JWT. | HTTP 200. `pe` is `null` (not negative or infinity) and `flags` includes the plain-language "no profit" red flag. | Not Run |
| TC-015 | Failure: invalid ticker format and unknown ticker (`REQ-05`, `EC-11`) | 1. Call `GET /v1/stocks/123456` with valid JWT. 2. Mock E2 unknown symbol and call `GET /v1/stocks/ZZZZZ`. | First call returns HTTP 400 `INVALID_TICKER` without calling E2. Second call returns HTTP 404 `UNKNOWN_TICKER` and C1 shows "No such stock, try another". | Not Run |

### REQ-06: Stale stock fallback and exact "Data from yesterday" message

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-016 | Failure: E2 times out over 3 s with existing cache (`REQ-06`, `AC-UC01-02`, `EC-12`) | 1. Seed `StockSnapshot` for `AAPL` with `as_of` set to 24 hours ago. 2. Configure mock E2 to delay 6 seconds. 3. Load `AAPL` in C1 (calling `GET /v1/stocks/AAPL`). | Request finishes in under 3.5 s with HTTP 200, cached `AAPL` data, and `stale: true`. C1 displays the exact text: "Data from yesterday. Try another stock or come back later." | Not Run |
| TC-017 | Failure: E2 returns HTTP 429 or junk JSON with existing cache (`REQ-06`, `EC-12`) | 1. Seed expired `StockSnapshot` for `KO` (2 hours old). 2. Mock E2 to return HTTP 429, then malformed JSON `{bad`. 3. Call `GET /v1/stocks/KO` for each. | Both calls return HTTP 200 with the cached `KO` snapshot and `stale: true`. | Not Run |
| TC-018 | Failure: E2 down and no cached snapshot exists (`REQ-06`, `EC-12`) | 1. Ensure `StockSnapshot` has no row for `NVDA`. 2. Mock E2 to return HTTP 503. 3. Search `NVDA` in C1. | Backend returns HTTP 503 `NO_DATA`. C1 displays "Data is unavailable, try another stock". | Not Run |

### REQ-07: Paper-only practice picks

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-019 | Happy path: add pick and track percentage change (`REQ-07`) | 1. Seed `MSFT` at price `400.00`. 2. Call `POST /v1/picks` with `{ticker: "MSFT"}` and valid JWT. 3. Update cached `MSFT` price to `440.00`. 4. Call `GET /v1/picks`. | `POST` returns HTTP 201 with `entryPrice: 400.00`. `GET` returns HTTP 200 showing `MSFT` with `entryPrice: 400.00`, `currentPrice: 440.00`, and `changePct: 10.00`. | Not Run |
| TC-020 | Edge case: duplicate practice pick rejected (`REQ-07`, `EC-11`) | 1. Call `POST /v1/picks` with `{ticker: "MSFT"}` for a user who already picked `MSFT`. 2. Count `PracticePick` rows for `(user_id, "MSFT")`. | Returns HTTP 409 Conflict with plain error message. Exactly 1 row remains in `PracticePick`. | Not Run |
| TC-021 | Edge case: expired cache row does not delete pick (`REQ-07`) | 1. Save a practice pick for `AAPL`. 2. Delete the `AAPL` row from `StockSnapshot`. 3. Query `PracticePick` for that user. | Deletion of `StockSnapshot` succeeds without foreign-key failure and the user's `PracticePick` row remains intact. | Not Run |

### NFR-03: Reliability and graceful degradation

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-022 | Failure: E1 is down, and the API itself is down (NFR-03, EC-05, EC-03) | 1. Block C2 from reaching E1. 2. Open GET /v1/lessons/budgeting with no token. 3. Try a new sign-in from C1. 4. Stop the API, type a plan in C1, and submit. | The public lesson is HTTP 200. New sign-in is HTTP 503 and C1 says "sign-in unavailable". With the API stopped, C1 says something went wrong, offers Retry, and keeps the typed values. No unhandled 500. | Not Run |
| TC-023 | Failure: E3 AI tooltip service times out or gives bad advice (`NFR-03`, `REQ-09`, `EC-07`) | 1. Mock E3 to delay 6 seconds and call `POST /v1/explain` with `{metric: "pe", value: 35, benchmark: 20}`. 2. Mock E3 to reply `"You should buy this stock right now!"` and repeat. 3. Mock E3 to change `35` to `15` and repeat. | All three calls return HTTP 200 with `source: "template"` and the exact template sentence. Timeout finishes in under 4.5 s. Learner sees no error and no buy/sell wording. | Not Run |
| TC-024 | Failure: C6 read failure triggers one retry (`NFR-03`, `EC-13`) | 1. Configure C6 test double to fail the first `get_lesson` read and succeed on the second. 2. Call `GET /v1/lessons/budgeting`. 3. Configure C6 to fail both reads and repeat. | First test succeeds with HTTP 200 after 1 automatic retry. Second test returns HTTP 503 with a plain "try later" message and 0 unhandled 500s. | Not Run |

### NFR-04: Security, authentication, config, and logging

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-025 | Failure: missing or expired JWT on private endpoints (`NFR-04`, `EC-04`) | 1. Call `POST /v1/plan`, `GET /v1/stocks/MSFT`, `POST /v1/picks`, and `POST /v1/lessons/pe-ratio/complete` with no `Authorization` header. 2. Repeat with an expired JWT. | Every call returns HTTP 401. C1 redirects the learner to sign in. | Not Run |
| TC-026 | Failure: missing required secret refuses startup (`NFR-04`, `CON-04`) | 1. Unset `STOCK_API_KEY` (and then `DATABASE_URL`) in the runtime environment. 2. Start the FastAPI app container. | Process exits immediately with a non-zero exit code and logs which config key is missing, refusing to serve traffic. | Not Run |
| TC-027 | Audit: zero sensitive data in logs and git history (`NFR-04`, `CON-03`, `CON-04`) | 1. Run TC-001 (`monthlyIncome: 2500`) and TC-013 with a known test JWT and API key. 2. Search application logs for `2500`, the JWT string, and the API key. 3. Scan git history for `.env` or secret patterns. | Logs show only timestamp, route, HTTP status, and duration. 0 matches for income figures, tokens, or keys in logs or git history. | Not Run |

### Additional coverage for Should requirements

| ID | Description | Steps | Expected Result | Status |
| --- | --- | --- | --- | --- |
| TC-028 | Browser form checks, double-submit lock, and "Estimate for me" (`REQ-08`, `EC-01`, `EC-02`) | 1. Type `"abc"` in monthly income and click Submit. 2. Enter `3000` and click "Estimate for me". 3. Throttle network to Slow 3G and double-click Submit. | Step 1 blocks submit with inline message and sends 0 requests. Step 2 fills 50/30/20 expenses totaling `$3,000` (`1500` needs, `900` wants, `600` savings). Step 3 disables Submit while waiting and sends only 1 `POST /v1/plan`. | Not Run |
| TC-029 | Easier-lesson link navigation (`REQ-10`) | 1. Request `GET /v1/lessons/pe-ratio`. 2. Click the easier-lesson link in C1. | Payload includes a valid `easier_lesson_id`. Clicking it opens the simpler prerequisite lesson with HTTP 200. | Not Run |
| TC-030 | Planner latency under load (`NFR-01`) | 1. Warm the staging container. 2. Run 20 `POST /v1/plan` requests across 10 concurrent virtual users. 3. Compute the 95th percentile response time. | 100% return HTTP 200 and p95 latency is at most 1.0 s. | Not Run |
| TC-031 | Stock lookup latency, live and cached (`NFR-02`) | 1. Warm staging. 2. Time 10 uncached `GET /v1/stocks/{ticker}` calls and 10 cached calls (under 15 min old). | Live lookup p95 is at most 3.0 s; warm cache hit p95 is at most 0.5 s. | Not Run |
| TC-032 | Pilot usability session (`NFR-05`) | 1. Give 3 outside testers (ages 18 to 30) the staging URL and written prompts for UC-01 and UC-02. 2. Time each use case without giving hints. | 3 of 3 testers finish both UC-01 and UC-02 without help, each in at most 10 minutes. | Not Run |
| TC-033 | Keyboard, screen-reader, and text next to charts (NFR-06, CON-06) | 1. Unplug the mouse. Move through the planner, lessons, stock search, and practice picks with Tab, Shift+Tab, Enter, and Esc only. 2. Repeat with VoiceOver on macOS or NVDA on Windows. | Every button, field, and link is reachable from the keyboard. Every chart has a text version. A red flag is a word, not only a color. | Not Run |

---

## 5. Test environment

To keep results repeatable, every test runs against one of three named environments.

| Aspect | Development (local laptops) | Staging (hosted test deploy) | Pilot (live user validation) |
| --- | --- | --- | --- |
| Purpose | Unit tests, local integration tests, fault injection | Full system tests, timing checks (`NFR-01`, `NFR-02`), accessibility pass | Outside tester sessions (`NFR-05`) and advisor demo (M9) |
| Hardware and OS | Team laptops (Windows 11 / macOS 14+), 8 GB+ RAM | Free-tier managed Linux container (512 MB+ RAM) + managed Postgres 16.x | Testers' own laptops or phones on standard campus or home Wi-Fi |
| Browsers | Chrome (latest stable) and Firefox (latest stable) | Chrome, Firefox, and Safari (latest stable) | Chrome, Firefox, or Safari on desktop and mobile viewport (375 px wide) |
| Backend stack | Python 3.12.x, FastAPI 0.115.0, Uvicorn 0.32.0, Pydantic 2.9.2, SQLAlchemy 2.0.36, psycopg 3.2.3 | Same pinned versions from `requirements.txt` | Same pinned release tag as staging |
| Frontend stack | React 18.3.1 (Vite dev server) | Static production bundle served over HTTPS | Static production bundle served over HTTPS |
| External services | Mocked E1, E2, and E3 (deterministic JSON and fault toggles) | Real E1 test tenant, real E2 free-tier key, real E3 key (plus fault toggle for EC-12) | Real E1, E2, and E3 with production-like config |
| Seeded data | Local Postgres 16 container seeded with 3 `InvestmentForm` rows (4%, 5%, 7%), 5 lessons, 2 test users, and 3 sample `StockSnapshot` rows (`AAPL`, `MSFT`, `KO`) | Managed Postgres seeded with the same lessons and investment forms, plus 2 test accounts | Clean managed Postgres with seeded lessons and investment forms; pilot accounts wiped after M9 |
| Network conditions | Localhost (0 ms delay); mocked 6 s vendor delays for timeouts | Standard broadband + Chrome DevTools "Slow 3G" throttle for `EC-02` and `EC-03` | Normal campus or home internet (no artificial throttling) |

---

## 6. Pass and fail criteria

### Per-test criteria

- **Pass:** Every step finishes cleanly and the observed output matches the exact numeric, status-code, or text threshold in the `Expected Result` column (and its linked metric in section 3).
- **Fail:** Any step throws an unhandled exception or HTTP 500, returns a number off by even $0.01 on exact decimal math, misses a latency cutoff, drops a required fallback message, or leaks sensitive data into logs.
- **Blocked:** A prerequisite environment or vendor key is unavailable so the test cannot be executed; blocked tests do not count as passes.

### Overall plan criteria

The test plan as a whole **passes** only when all of the following hold at M8:

1. Every Must case (TC-001 through TC-027) passes, and no Critical or High defect is still open.
2. **All metrics M-01 through M-07 meet their target thresholds** from section 3.
3. TC-032 and TC-033 both pass (3 of 3 pilots finish both use cases in 10 minutes or less, and the keyboard and screen-reader pass has no screen that cannot be used that way). At least 3 of TC-028 through TC-031 also pass.
4. Logs and git history contain no income figures, tokens, API keys, or passwords (M-07).

The plan **fails** if any Must test case fails, if any Critical or High defect stays open past the M8 protected block (Mar 18 to 19), if any single-dependency fault causes an unhandled HTTP 500 (`M-06`), or if fewer than 3 of 3 pilot testers complete both main use cases unassisted (`M-08`).

---

## 7. Test schedule

Each test layer is tied to our Week 8 milestones (`M1` through `M9`). Core math and data integrity are tested first, followed by external integrations, full browser flows, and outside validation.

| Milestone (Week 8 plan) | Date | Test activity and layer | Owner | Evidence produced |
| --- | --- | --- | --- | --- |
| M1: Database and calculator work (T1, T2) | Jan 3, 2027 (W2) | Unit and local DB tests for C4 and C6 (`TC-001` to `TC-009`, metrics M-01, M-02). | Jal Patel (runs), Jay Natarajan (reviews) | `pytest` output log committed to repo; UC-02 numbers verified. |
| M2: Gateway works; stock source chosen (T3, T4) | Jan 10, 2027 (W3) | C2 routing and error-format integration tests; verify E2 sample payloads. | Jay Natarajan (C2), Jal Patel (E2) | Integration test log + E2 vendor evaluation note committed. |
| M3: Sign-in and stock lookup work (T5, T6) | Jan 24, 2027 (W5) | Integration and fault tests for E1 auth and C5 stock lookup + stale fallback (`TC-013` to `TC-018`, `TC-025`, `TC-026`). | Jay Natarajan (E1/C2), Jal Patel (C5/E2) | Passed `pytest` fault-injection report before protected block B1. |
| M4: Backend complete (T7, T8) | Feb 7, 2027 (W7) | Full backend API integration suite covering picks, lessons, and AI tooltip fallbacks (`TC-010` to `TC-012`, `TC-019` to `TC-024`). | Jal Patel (writes script), Jay Natarajan (reviews) | Automated API script output showing all `/v1` routes green before B2. |
| M5: Lessons work end to end (T9, T11) | Feb 14, 2027 (W8) | Browser system check of P/E lesson, 3 examples, calculator, and easier-lesson link (`TC-010`, `TC-029`). | Jal Patel | Signed UI checklist against `AC-UC01-01`. |
| M6: Feature complete (T10, T12) | Feb 28, 2027 (W10) | Full browser end-to-end walkthrough of UC-01, UC-02, and client checks (`TC-028`, all four user stories). | Jay Natarajan (leads), Jal Patel (checks) | Completed E2E checklist + screenshots before protected block B3. |
| M7: Deployed (T13) | Mar 7, 2027 (W11) | Smoke test on public staging URL from a second computer; warm latency checks (`TC-030`, `TC-031`, metrics M-03, M-04) and cold-start timing (`AQ-02`). | Jay Natarajan | Latency CSV/log from staging + live URL smoke-test note. |
| M8: Tested and validated (T14) | Mar 21, 2027 (W13) | Full regression run (`TC-001` to `TC-031`), security/log/git audit (`TC-027`), accessibility pass (`TC-033`), and 3-user pilot sessions (`TC-032`). | Jal Patel (leads), Jay Natarajan (reviews) | Filled test results table, accessibility checklist, audit log, and 3 pilot timing sheets before B4. |
| M9: Final demo and documentation (T15) | Mar 28, 2027 (W14) | Re-test all fixes from M8, live demo to advisor and instructor, finalize report. | Both members | Zero-failure regression log, advisor sign-off, and final report committed. |

---

## 8. Definition of done

### Feature-level definition of done

A feature or task (`T1` through `T15`) is done only when every box below is checked:

- [ ] Code is merged to `main` with no secrets or `.env` files in the commit (`NFR-04`).
- [ ] All unit and integration tests for that feature's `REQ` and `EC` IDs pass locally and in staging.
- [ ] The other teammate has reviewed the pull request and confirmed it matches the D1/D2 contract.
- [ ] Any error path shows a plain-language message in C1 (no raw stack traces or unhandled HTTP 500s).
- [ ] Charts or status indicators added by the feature include text alternatives (`NFR-06`).
- [ ] No Critical or High bugs remain open against the feature.

### Project-level definition of done

The Financial Literacy & Stock Learning Platform is ready for M9 final delivery only when every box below is checked:

- [ ] **Tests:** Every Must case (TC-001 through TC-027) passes, and section 6 is met.
- [ ] **Validation:** 3 of 3 outside testers finished UC-01 and UC-02 with no help, in 10 minutes or less (TC-032). The keyboard and screen-reader pass is done (TC-033).
- [ ] **Security:** A log review and a git history search find no income figures, tokens, or keys (TC-027).
- [ ] **Defects:** No Critical or High defect is still open. Anything Medium or Low that we are leaving is written down with the demo notes.
- [ ] **Documentation:** This plan, the requirements document, and the technical specification (matrix filled with TC-001 through TC-033) are committed, and this file is linked from README section 3 and the appendix.
- [ ] **Sign-off:** Both Jal Patel and Jay Natarajan have signed off on the final regression run, and the live app has been demonstrated to the advisor and instructor.