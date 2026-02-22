# RESEARCH.md — Problem Discovery
**Date:** 2026-02-22  
**Researcher:** Cody Nixon  
**Sources:** Twitter/X, Reddit (r/SomebodyMakeThis, r/SaaS, r/Entrepreneur), Hacker News Ask HN, Market Clarity analysis, dev.to

---

## Research Signals Collected

### Signal Set 1: Twitter/X Searches

**"someone should build" / "wish there was a tool"**
- "Wish there was a tool for removing the manual QA part [of code reviews]" — @vzeroaman (Feb 21, 2026)
- General frustration with manual review workflows in software
- No clear app-shaped gap here; many tools exist (Playwright, etc.)

**"frustrated with [my] workflow"**  
- @sujee (Feb 22, 2026): "I'm frustrated with my current workflow [for handling screenshots in AI coding]. Starting a small project for personal use."
- This reflects a broader pattern: AI coding workflows are still improvised and personal

**HN Ask Questions (active today)**  
- "Watching an elderly relative trying to use the modern web" — 48 pts — UX for seniors is still broken
- "How do you motivate your humans to stop AI-washing their emails?" — 29 pts — AI content authenticity problem
- "My first experience with an 'AI'-ed call centre" — 27 pts — AI customer service failing users

**Builder.io blog (3 days ago): "AI Agent Orchestration is Broken"**
- Multi-agent development workflows are fragmented; no good UI for managing parallel AI sessions
- Developers use tmux + named sessions manually — no structure, high cognitive load

**dev.to article (14 hours ago): "Why Multitasking With AI Coding Agents Breaks Down"**
- Running Claude Code + Codex + Gemini CLI in parallel is chaotic
- No tool exists to give each agent its own isolated branch + workspace + visual pane
- Author built a rough desktop app "Parallel Code" but it's early and desktop-only

---

### Signal Set 2: Reddit r/SomebodyMakeThis (Hot Posts)

1. **"What is a repetitive, ugly data-entry task in your business that generic software hasn't fixed yet?"** — 2 days ago — developer asking for market research. Comments reveal: PDF invoice extraction, contractor payment tracking, manual copy-paste between apps.

2. **"Tool that shows inflation price gouging"** — 3 days ago — compare what things should cost vs what they actually cost. Interesting but requires complex data pipeline.

3. **"An app that reminds you about subscriptions, renewals, and deadlines before you forget"** — 2 days ago — 13 upvotes, 11 comments. But this is very similar to past build CancelScore (subscription management).

4. **"A website with a Tinder like set up, that matches influencers to people who want their product promoted"** — 1 day ago — creator economy matchmaking. Crowded space (AspireIQ, Grin, etc.)

5. **"Bad habit mountain" app** — 9 upvotes, 16 comments — gamified habit breaking. Crowded (Habitica, etc.)

---

### Signal Set 3: Market Clarity Analysis — Underserved Niches

1. **Subcontractor Payment & Relationship Management for Construction** — 53,000+ general contractors use spreadsheets; 30-90 day payment visibility gap. Big market but B2B sales cycles are long.

2. **Multi-Location Franchise Operations** — 806k franchise units need better brand compliance tooling. Good market, but very B2B.

3. **Catering & Event Operations** — 300K+ restaurants doing catering manually. Interesting niche.

4. **SMB Compliance Tracking** — GDPR/HIPAA/PCI-DSS in spreadsheets. Legal moat, hard to validate.

5. **Professional Services Capacity Planning** — spreadsheet-based, 100k+ firms affected. Crowded (Harvest, TeamGantt).

---

### Signal Set 4: Deeper Themes Emerging

**Theme A: "I have 3+ quotes and no way to compare them intelligently"**
- When homeowners get roofing estimates, plumbing quotes, or renovation bids from multiple contractors, they receive PDFs, emails, paper quotes — all in different formats with different line items. They can't easily compare apples to apples.
- No good tool exists. Google Sheets is the de facto solution.
- This is also true for: small businesses comparing vendor proposals, freelancers comparing tool subscriptions, procurement teams comparing supplier RFQs.

**Theme B: "AI coding multi-agent workflow chaos"**  
- Developers using 2-3 AI agents simultaneously have no orchestration layer
- Cognitive overhead is high; existing tools (tmux) don't address workflow structure
- Desktop-only solution in early alpha (Parallel Code). No web-based solution.

**Theme C: "Elderly/non-technical users getting destroyed by modern web UX"**  
- HN thread got 48 points. The modern web assumes young, tech-literate users.
- No tool addresses this at the browser/web level that's both effective and simple.

**Theme D: "Meeting action items never tracked anywhere"**  
- Action items from meetings live in notes apps, email, Notion — disconnected from task managers
- Several AI tools attacking this (Otter.ai, Fireflies) but the POST-meeting follow-through gap remains

**Theme E: "Manual QA for AI-generated apps"**  
- As vibe-coded apps proliferate, someone needs to test them. But traditional QA tools assume hand-written code structure.
- This is related to VibeAudit (already built) but focuses on functional QA rather than security.

---

## Candidate Problem List (20 total, cross-referenced against PAST_BUILDS.md)

| # | Problem | Who Has It | Size | Existing Solutions | Gap | Status |
|---|---------|-----------|------|-------------------|-----|--------|
| 1 | Compare contractor/vendor quotes side-by-side | Homeowners, SMBs, procurement | Massive (every property owner) | Google Sheets | No structured tool; different formats | ✅ Novel |
| 2 | Multi-agent AI coding session manager | Developers | Growing fast | tmux only | No web/structured UI | ✅ Novel |
| 3 | Inflation / price gouging tracker for groceries | Everyone | Massive | BLS CPI data scattered | No consumer-friendly interface | ✅ Novel |
| 4 | Elderly-friendly web browser / simplifier | Seniors, caregivers | Huge (aging population) | None | Too hard to build well | Complex |
| 5 | Meeting action item → task manager bridge | Office workers | Enormous | Otter.ai, Fireflies | Post-meeting follow-through gap | ✅ Novel |
| 6 | Subcontractor payment tracking for GCs | General contractors | 53k+ GCs | Lien tracking tools only | Mobile-first, integrated | ✅ Novel |
| 7 | Subscription/renewal deadline reminder | Everyone | Massive | Phone calendar | CancelScore-adjacent | ❌ Too similar to past |
| 8 | Influencer-brand matching | Small brands | Large | AspireIQ, Grin | Too crowded | Skip |
| 9 | SMB GDPR/HIPAA compliance tracker | SMBs | Millions | Drata, Vanta (expensive) | Price gap for small co | ✅ Novel |
| 10 | Freelancer project scope creep detector | Freelancers | Millions | None | Clear gap | ✅ Novel |
| 11 | AI image prompt → consistent style guide | Designers, content teams | Growing | Midjourney style ref | Hard to execute consistently | Complex |
| 12 | ER wait time tracker (HN trending) | Patients, hospitals | Public health | None | Novel, but limited web app use | Niche |
| 13 | Local business review aggregator (not Yelp) | Local biz owners | Enormous | Yelp, Google | Crowded | Skip |
| 14 | Home inventory + warranty tracker | Homeowners | ~130M US homeowners | None that stick | Good gap! | ✅ Novel |
| 15 | RFQ (Request for Quote) tracker for SMBs | Procurement teams | Millions of SMBs | Spreadsheets | Clear gap | Similar to #1 |
| 16 | AI call center quality monitor | Call centers | Enterprise | Chorus.ai | Too enterprise-y | Skip |
| 17 | Handyman job ticketing for solopreneurs | Solo tradespeople | Millions | JobBer (expensive) | Price gap | Interesting |
| 18 | Freelance invoice reminder + late fee calculator | Freelancers | Millions | FreshBooks (complex) | Simple version needed | ✅ Novel |
| 19 | Vibe-coded app functional QA runner | Vibe coders | Growing | VibeAudit (security only) | Functional test gap | Adjacent to past build |
| 20 | Contractor estimate comparison (home) | 66M US homeowners | Enormous | Angi, HomeAdvisor | They find contractors but don't help compare quotes | ✅ Clear gap |

---

## Cross-Reference Against PAST_BUILDS.md

✅ CLEAR to build: #1/#20, #2, #3, #5, #6, #9, #10, #14, #17, #18  
❌ Too similar to past build: #7 (like CancelScore/PriceShift), #19 (like VibeAudit)  
⚠️ Too crowded / too complex: #4, #8, #11, #12, #13, #15, #16  

---

## Top 5 Candidates for Deep Scoring

1. **Contractor Estimate Comparison** (#1/#20) — homeowners + SMBs need to compare 3+ quotes in different formats
2. **AI Multi-Agent Session Manager** (#2) — developers running parallel Claude Code/Codex sessions need orchestration
3. **Home Inventory + Warranty Tracker** (#14) — 130M US homeowners have no good way to track appliances/warranties
4. **Meeting Action Item Bridge** (#5) — meetings generate action items that die in notes apps
5. **Freelance Invoice Reminder + Late Fee** (#18) — freelancers get stiffed and have no structured follow-up tool

---

*Research saved. Moving to Phase 2: Scoring.*
