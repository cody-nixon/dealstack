# REPORT.md — Daily Design Session Summary
**Date:** 2026-02-22  
**Agent:** Cody Nixon  
**Session:** Daily Problem Hunter & App Designer

---

## The Problem

**Contractor and vendor quote comparison chaos.**

Every homeowner, real estate investor, and small business that hires contractors for home or commercial work faces the same problem: quotes arrive in completely different formats (PDFs, emails, handwritten photos, Google Docs) with different categories, different assumptions, and different inclusions. There is no standard way to compare them.

**Sources:**
- Market Clarity analysis of 21 underserved digital niches (October 2025) — Construction payment/coordination gap documented at 53,000+ general contractors
- Reddit r/SomebodyMakeThis: "What is a repetitive, ugly data-entry task your business hasn't solved?" (Feb 20, 2026) — Comments validated contractor quote normalization as a recurring pain
- Personal signal: Angi/HomeAdvisor help find contractors but have zero post-quote tools
- HN article: "My first experience with an AI-ed call centre" (27 pts, 13 comments, Feb 21) — corroborated frustration with generic AI that doesn't understand context (same problem in a different domain)

---

## Why I Chose This

**Reasoning:**

1. **Universal pain.** 66M US homeowners, millions of SMBs. Anyone who has ever renovated anything has felt this pain.

2. **Nobody owns this problem.** Angi, Thumbtack, and HomeAdvisor monetize contractor lead-gen — they have NO incentive to help homeowners compare quotes objectively. Procore and Buildertrend are enterprise construction management at $500+/month. Nothing exists for the homeowner/small investor segment.

3. **AI makes something previously impossible trivially possible.** Before LLMs, parsing contractor quotes into normalized categories required manual data entry or expensive NLP systems. Now a single Claude call (with a good system prompt) can do it reliably in seconds. This is a genuine AI moat.

4. **The bookmark test passes.** Every new project means a new batch of quotes. Property managers come back weekly. It's not a one-time tool.

5. **Warnings are the killer feature.** "This quote doesn't include permits" — surfacing what non-experts would miss is genuinely valuable and worth sharing.

6. **Completely distinct from past builds.** Nothing in PAST_BUILDS.md touches this problem space.

---

## Design Summary

**Product:** DealStack — "Every contractor quote, side by side."

**Core flow:**
1. User signs up (email or Google OAuth)
2. Creates a project (e.g., "New Roof — Front Street")
3. Adds quotes by pasting text or uploading a PDF
4. Claude reads the quote and normalizes it into standard categories (labor, materials, permits, disposal, markup)
5. After 2+ quotes, the comparison table shows all quotes side by side with highlighted minimums and warning flags for missing items
6. User adds notes, marks a winner, can share a read-only link

**Key technical decisions:**
- React + Vite + shadcn/ui frontend (Vercel)
- Node.js + Express + TypeScript backend (Railway)
- Supabase for DB + auth + storage + RLS
- Claude claude-sonnet-4-6 with structured JSON output for quote parsing
- pdfjs-dist client-side for PDF text extraction before sending to Claude
- Synchronous Claude calls (no queue) for V1 simplicity
- Stripe for Free/Pro ($9/month)/Teams ($29/month) tiers

**Architecture:** Standard 3-tier web app with Supabase handling most infrastructure complexity. Backend thin — primarily a proxy for Claude calls and Stripe webhooks.

**Wireframes:** 12 screens including landing, auth, dashboard, project view (0/1/3+ quotes), add quote modal, parse flow, comparison table, share view, upgrade prompt, settings, manual entry.

---

## GitHub Repo

**https://github.com/cody-nixon/dealstack**

Files committed:
- `RESEARCH.md` — 20+ candidate problems from Twitter, Reddit, HN, Market Clarity
- `DECISION.md` — Scoring matrix, deep reasoning for winning choice
- `PLAN.md` — Full tech stack, API design, data model, auth flow, tier limits
- `REVIEW.md` — Multi-role review (Product, QA, Engineering, Design, Security)
- `WIREFRAMES.md` — 12 screens with all states, mobile layout notes
- `DESIGN.md` — Final comprehensive design document

---

## Key Insight

The AI moat here isn't "this uses AI" — it's **normalization**. The real problem isn't that quotes are hard to *read*; it's that they're hard to *compare* because they use different vocabulary and structure. Claude can translate "Bob says '$6,800 for GAF shingles plus tear-off labor'" and "Mike says '$7,200 materials' into a common table. That translation is the product.

The warnings ("this quote doesn't include permits") are what make someone tell their friend about it. That's the viral loop.

---

*Session complete.*
