# DECISION.md — Problem Selection
**Chosen Problem:** Contractor & Vendor Quote Comparison  
**Product Name:** DealStack  
**Tagline:** "Compare contractor quotes in minutes, not spreadsheet hours."

---

## Scoring Matrix (1-10 each)

| Criterion | Estimate Comparison | AI Agent Manager | Home Inventory | Meeting Bridge | Freelance Invoice |
|-----------|--------------------:|----------------:|---------------:|---------------:|------------------:|
| **Urgency** (burning right now?) | 9 | 8 | 4 | 6 | 7 |
| **Impact** (# people affected) | 10 | 6 | 9 | 8 | 8 |
| **Novelty** (no good solution?) | 9 | 7 | 8 | 5 | 6 |
| **Feasibility** (buildable as web app?) | 9 | 6 | 8 | 7 | 9 |
| **Monetization potential** | 9 | 8 | 6 | 7 | 7 |
| **Bookmark test** (come back tomorrow?) | 10 | 7 | 5 | 4 | 7 |
| **Uniqueness from past builds** | 10 | 10 | 10 | 9 | 10 |
| **TOTAL** | **66** | **52** | **50** | **46** | **54** |

---

## Why Estimate Comparison Wins

### The Real Problem

When a homeowner or small business owner needs work done, they get 2-5 quotes from different contractors. Every single quote arrives in a different format:

- Contractor A sends a 3-page PDF with labor/materials broken out by trade
- Contractor B sends an email: "For the job, I'm thinking $8,500 all in"
- Contractor C sends a Google Doc with a table but uses different category names
- Contractor D sends a photo of a hand-written page

The homeowner now has to: read 4 different documents, mentally normalize the categories, figure out what's included vs. excluded in each, guess at the margin differences, and decide — all while not really knowing what's standard pricing.

**Current solution: Build a spreadsheet from scratch.** Or just go with your gut.

This is maddening. It happens millions of times per week.

### Why Existing Solutions Fail

- **Angi / HomeAdvisor / Thumbtack**: Find contractors for you. Zero help once you have quotes in hand.
- **Google Sheets**: No structure, no templates, requires the user to do all the normalization work.
- **BuildZoom**: Shows contractor license/review data, not quote comparison.
- **Procore / PlanGrid**: Enterprise construction management, $500+/month, designed for GCs not homeowners.
- **Nothing**: The actual status quo for 90% of people getting home service quotes.

### The Insight That Makes This Win

The innovation isn't just "a comparison spreadsheet" — it's **intelligent normalization**. When someone uploads or pastes a quote, AI reads it and maps it to a standard set of line items (labor, materials, permits, disposal, markup). Suddenly all quotes are in the same language, and you can compare column by column.

This is the "aha" moment: **translate contractor-speak into your language**.

### Who Has This Problem

1. **Homeowners getting renovation/repair quotes** — roof replacement, kitchen remodel, HVAC replacement, bathroom renovation, landscaping. Every homeowner eventually. In the US, ~66M homeowners, and estimates suggest 20M+ major home projects per year.

2. **Small business owners hiring contractors** — office remodel, parking lot repaving, electrical work. Same problem, higher dollar values.

3. **Real estate investors / flippers** — get 5-10 quotes per rehab project. Extremely painful. High willingness to pay.

4. **Property managers** — constant stream of maintenance quotes. Repeat users. Best B2B angle.

### Bookmark Test: Would They Return?

YES. Every time you start a new project, you have a new batch of quotes. Users come back for every bathroom, every roof, every HVAC replacement. Property managers come back every week.

### Monetization Clarity

- **Free tier**: Up to 3 quotes per project, 1 active project
- **Pro ($9/month or $79/year)**: Unlimited projects, PDF/email parsing, AI normalization, export to PDF
- **Teams ($29/month)**: Property managers, investors — multiple projects, contractor rolodex, notes/history

Revenue from real estate investors and property managers alone is a solid business.

---

## What We're NOT Building

- Not a contractor marketplace (that's Angi)
- Not a project management tool (that's Buildertrend)
- Not a full procurement platform (that's Coupa)

We are building: the layer BETWEEN "getting quotes" and "making a decision."

---

## Confidence Level

**High.** This is a universal pain that affects 10s of millions of people, existing solutions are genuinely inadequate, and the AI-powered normalization angle provides a clear technical moat. The scope is tightly bounded. The value is immediately obvious.

---

*Decision made. Proceeding to Phase 3: Technical Planning.*
