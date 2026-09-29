# User Stories and Use Cases

**Team Members**: Jal Patel
**Project**: Financial Literacy & Stock Learning Platform

---

## Stakeholder Map

| Stakeholder | Category | Need |
|-------------|----------|------|
| Early earner (18–30): first job, post-college, building independence | **Primary** | Learn personal finance + stock investing to build real wealth and handle money independently |
| Person seeking financial literacy (any age) | **Secondary** | Accessible education on budgeting, saving, investing without overwhelming jargon |
| Data/systems engineer | **Hidden** | API reliability, data accuracy, system uptime, compliance with financial data handling |

---

## User Stories

### US-01 (Primary): Build Financial Foundation with First Income
As a **person with a first real job or fresh out of college**,  
I want to **learn budgeting, saving, and emergency funds with money I actually have**,  
so that **I can manage my paycheck wisely instead of living paycheck to paycheck**.

---

### US-02 (Primary): Learn Stock Investing Without Confusion
As a **young professional who never learned this in school**,  
I want to **understand stocks, risk, and diversification in simple terms with real examples**,  
so that **I can grow my money over time, not just save it**.

---

### US-03 (Primary): Practice Real Stock Research Before Real Money
As a **early career person wanting to invest but scared to lose money**,  
I want to **research actual stocks, practice buying/selling, and see how my picks perform**,  
so that **I know what I'm doing before risking my hard-earned paycheck**.

---

### US-04 (Primary): Avoid Losing Money to Beginner Mistakes
As a **young person with new financial responsibility**,  
I want to **understand when NOT to buy a stock and what makes a decision dumb**,  
so that **I build real wealth instead of getting burned chasing trends or hype**.

---

## Use Cases

### UC-01: Learn a Stock Metric and Apply It to Real Research
**Related Story:** US-02, US-03

**Primary Actor:** Early earner learning to invest  
**Secondary Actors:** Stock data API, AI tooltip service

**Preconditions:**
- User has opened the app
- Stock market data available

**Main Success Flow:**

1. User explores "Stock Investing" section and clicks "P/E Ratio—What It Means"
2. App shows 2-min explanation: "P/E = Price ÷ Earnings Per Share. Simple: is the stock expensive or cheap?"
3. App displays 3 real companies side-by-side:
   - Apple: Stock price $150, earnings $6/share = P/E of 25x
   - Coca-Cola: Stock price $60, earnings $3.50/share = P/E of 17x
   - Tesla: Stock price $250, earnings $1/share = P/E of 250x
4. App explains: "Apple (25x) = middle. Coca-Cola (17x) = cheap. Tesla (250x) = expensive or high growth expected"
5. User clicks "Try the calculator": enters any stock price and earnings, sees P/E auto-calculate
6. User wants to apply this: clicks "Research Real Stocks"
7. User searches "MSFT" (Microsoft)
8. App shows: Company info, 5-year chart, Key metrics including P/E (28x)
9. User hovers "?" on P/E → tooltip: "Microsoft P/E 28x. Tech industry average: 22x. This stock costs more than average."
10. User notes: "Microsoft is pricier than the average tech company. Why? Maybe growth potential."
11. App marks lesson complete + shows "Applied to Microsoft"

**Alternate Flow:**
- User doesn't understand P/E → app offers "Start with simpler metric (dividend)" or "See more examples"
- User picks examples, continues

**Exception Flow:**
- Stock data for a company is unavailable → app shows "Data from yesterday. Try another stock or come back later"
- User picks different stock, continues

**Postcondition:**
- User understands one real stock metric
- User has applied it to real company data
- User ready to learn next metric or research more stocks
- User can research stocks anytime without prerequisites

---

### UC-02: Learn Personal Finance and Build a Budget
**Related Story:** US-01

**Primary Actor:** Early earner (first job or post-college)  
**Secondary Actors:** Finance tracking, budgeting calculator

**Preconditions:**
- User has opened the app and wants to manage their money

**Main Success Flow:**

1. User opens app and selects "Personal Finance Basics"
2. App shows simple lesson: "Income - Expenses = What You Have Left"
3. User enters their monthly income (paycheck amount)
4. App asks: "Where does your money go each month?"
5. User enters categories: rent ($1,200), food ($300), transport ($200), entertainment ($150)
6. App calculates: "You spend $1,850/month. Income: $2,500. Left over: $650/month"
7. App explains: "That $650 is your opportunity—save it or invest it"
8. App shows breakdown visually: pie chart of spending
9. User learns: "Emergency fund = 3-6 months of expenses. You need $5,550-$11,100 saved"
10. App suggests: "At $650/month, you'd hit emergency fund in 9-17 months"
11. User marks lesson complete and can explore next topic

**Alternate Flow:**
- User says "I don't know all my expenses" → app offers "Track for one week, come back" or "Estimate based on common patterns"
- User picks estimation, app shows typical budget for their income level

**Exception Flow:**
- User has debt (credit card, student loans) → app flags: "You have debt. We should talk about this before investing"
- App pivots to debt payoff strategy lesson instead

**Postcondition:**
- User understands their personal cash flow
- User knows what emergency fund looks like
- User ready to learn about saving vs. investing
- User can return anytime to update budget

---


## Acceptance Criteria

**AC-UC01-01 (Main Flow):**  
Given user selects "P/E Ratio" lesson,  
When lesson loads,  
Then app displays 2-min explanation + 3 real companies (Apple 25x, Coca-Cola 17x, Tesla 250x) + calculator.

**AC-UC01-02 (Exception Flow):**  
Given stock data is unavailable,  
When user searches a stock,  
Then app shows "Data from yesterday. Try another stock or come back later".

---

**AC-UC02-01 (Main Flow):**  
Given user opens "Personal Finance Basics",  
When entering income and expenses,  
Then app calculates remaining money and displays emergency fund goal (3-6 months of expenses).

**AC-UC02-02 (Exception Flow):**  
Given user has debt,  
When app detects debt input,  
Then app flags "You have debt" and offers debt payoff lesson before investing guidance.
