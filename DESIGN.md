# DESIGN.md — Final Design Document
**Product:** DealStack  
**Date:** 2026-02-22  
**Designer:** Cody Nixon

---

## Executive Summary

**Problem:** When homeowners and small businesses collect contractor quotes for a project (roofing, HVAC, kitchen remodel, etc.), each quote arrives in a different format with different categories. There is no good way to compare them — people resort to building spreadsheets from scratch, or just guessing.

**Solution:** DealStack reads contractor quotes in any format (PDF upload, paste text), normalizes them to a standard set of categories using AI, and presents all quotes in a clean side-by-side comparison table. Missing items are flagged, warnings are surfaced, and the user can add qualitative notes before marking a winner.

**Why it wins:** Existing solutions (Angi, Thumbtack) help you *find* contractors but completely abandon you once you have quotes in hand. DealStack fills the 48-hour gap between "I got my quotes" and "I signed the contract."

---

## Target User Persona

### Primary: Homeowner Sarah

- **Who:** Homeowner, 35-55, owns a house or condo. Not very technical.
- **Trigger event:** Just got 3 roofing quotes that are completely different formats. Has no idea how to compare them fairly.
- **Pain:** Spent 90 minutes building a spreadsheet and still feels like she's comparing apples to oranges. Worried she's going to pick wrong.
- **Motivation:** Make a confident, defensible decision without spending all weekend on it.
- **Willingness to pay:** Free for single project; would pay $9/month while actively managing a renovation.

### Secondary: Real Estate Investor / Property Manager James

- **Who:** Manages 5-15 rental properties, or flips 2-4 homes per year.
- **Trigger event:** Gets 5-10 quotes per rehab project, has multiple projects running simultaneously.
- **Pain:** Comparing quotes for the kitchen AND the roof AND the HVAC all at the same time. Quotes live in email, Dropbox, phone photos.
- **Motivation:** Move faster, catch the contractor who left out permits, export a comparison to show his partner.
- **Willingness to pay:** $29/month Teams plan — saves hours per project.

### Tertiary: Small Business Owner

- **Who:** Business owner getting quotes for office remodel, parking lot repaving, commercial HVAC.
- **Same core pain, higher dollar values** — more motivation to be thorough.
- **Willingness to pay:** Pro or Teams tier.

---

## Complete Technical Spec

### Tech Stack

| Layer | Tech | Rationale |
|-------|------|-----------|
| Frontend | React + Vite + TypeScript | Fast, modern, type-safe |
| UI Components | shadcn/ui + TailwindCSS | Pre-built, consistent |
| State | Zustand + TanStack Query | Client state + server sync |
| Backend | Node.js + Express + TypeScript | Standard, Railway-deployable |
| Database | Supabase PostgreSQL | Managed, integrated with auth |
| Auth | Supabase Auth | Email/password + Google OAuth |
| Storage | Supabase Storage | PDF storage, signed URLs |
| AI | Anthropic Claude claude-sonnet-4-6 | Quote parsing + normalization |
| PDF parsing | pdfjs-dist (client-side) | Text extraction before AI |
| Email | Resend | Transactional (verify, reset, notify) |
| Billing | Stripe | Subscriptions + webhooks |
| Frontend hosting | Vercel | CDN, fast deploys |
| Backend hosting | Railway | Node.js, auto-deploy from git |

### Data Model Summary

**Tables:**
- `user_profiles` — extends Supabase auth users; plan_tier, stripe_customer_id
- `projects` — name, category, status, selected_quote_id
- `quotes` — contractor_name, raw_text, raw_file_url, total_price, line_items (JSONB), warnings (JSONB), parse_status, confidence, notes, is_selected
- `project_shares` — share_token, expires_at (30 days)

**Key design choices:**
- Line items stored as JSONB in quotes table (not normalized) — quote structure is too variable for a relational schema
- RLS enforces per-user isolation; share token route is the only public read path

### API Design Summary

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| /api/projects | POST | JWT | Create project |
| /api/projects | GET | JWT | List user's projects |
| /api/projects/:id | GET | JWT | Get project + all quotes |
| /api/projects/:id/quotes | POST | JWT | Add quote (triggers parse) |
| /api/projects/:id/quotes/:qid | PATCH | JWT | Edit quote/notes/select |
| /api/projects/:id/quotes/:qid | DELETE | JWT | Remove quote |
| /api/projects/:id/export | POST | JWT+Pro | Export PDF |
| /api/quotes/parse | POST | Internal | Claude call (internal) |
| /api/share/:token | GET | None | Read-only shared view |
| /api/billing/checkout | POST | JWT | Stripe checkout |
| /api/billing/portal | POST | JWT | Stripe portal |
| /api/webhooks/stripe | POST | Stripe sig | Handle billing events |

### Claude Prompt Strategy

**System prompt includes:**
1. Project category context (roofing, HVAC, kitchen, etc.) with typical line items for that category
2. Standard output schema (Zod-validated JSON)
3. Instructions to flag missing common items for the category
4. Instructions to estimate confidence per line item (0-1 scale)
5. Instructions to note what's ambiguous or unclear

**Key prompt structure:**
```
You are an expert construction cost analyst helping homeowners compare contractor quotes.

Project type: {category}
Typical line items for this project type: {category_template}

The user has shared this contractor quote. Extract and normalize it.
Return JSON matching this schema: {schema}

Rules:
- If a common line item for {category} is missing, add it to warnings[]
- Rate your confidence per line item (0.0-1.0)
- If the quote is too vague (e.g., "total $8,500") mark confidence as 0.4 and note it
- Do not invent dollar amounts — mark missing items with amount: null

<quote>
{raw_quote_text}
</quote>
```

---

## UI/UX Summary

### 5-Second Test (first-time user)
The landing page hero shows:
1. **Headline:** "Every contractor quote, side by side."
2. **Before/after visual:** Three chaotic quote documents → one clean comparison table
3. **CTA:** "Compare My Quotes — It's Free"

A non-technical person must understand what the product does in 5 seconds. No jargon ("AI-powered", "LLM") on the landing — just the outcome.

### The Core UX Flow
1. Sign up (email or Google, 30 seconds)
2. Create project (name + category, 15 seconds)
3. Add quote (paste or upload, 30 seconds)
4. Watch it parse (5-15 seconds)
5. Add 2 more quotes
6. See comparison table with warnings
7. Add notes, mark winner

**Total first-session time: 5-8 minutes.** If it takes more than 10 minutes to see value, the user won't come back.

### Key UX Principles Applied
1. **Progressive reveal:** Don't show the full table until there's something to compare. Lead the user through: 0 quotes → 1 quote → 2 quotes (magic moment) → comparison
2. **Trust through transparency:** Every parsed line item shows confidence level + "Show original text" option. Users can catch errors.
3. **Editability:** AI is a starting point, not gospel. Every line item is editable. Manual entry is always available as fallback.
4. **Warnings are the real value:** "Mike's quote doesn't include permits" — this is what saves people money. These warnings are visually prominent.
5. **Share makes it viral:** "I'm comparing roofing quotes — here's what I found." Users share with spouses/partners who don't have accounts. Shared views promote DealStack at the bottom.

---

## Implementation Roadmap

### Week 1 — Foundation
- [ ] Supabase project setup (DB schema, RLS policies, auth config)
- [ ] Google OAuth setup (Supabase + Google Cloud Console)
- [ ] Railway backend scaffold (Express + TypeScript + auth middleware)
- [ ] Vercel frontend scaffold (React + Vite + shadcn/ui)
- [ ] Basic API: create project, list projects, get project
- [ ] Landing page HTML/CSS (static, no functionality)

### Week 2 — Core Feature (Quote Parsing)
- [ ] Claude integration: quote parse endpoint
- [ ] Category templates (10 project types) in Claude system prompt
- [ ] pdfjs-dist integration for client-side PDF text extraction
- [ ] Add quote flow: paste text path (PDF in week 3)
- [ ] Comparison table component (React, Zustand state)
- [ ] Confidence badges + hover tooltips
- [ ] Warning display component
- [ ] Line item edit mode

### Week 3 — Polish + Auth
- [ ] PDF upload to Supabase Storage + signed URLs
- [ ] PDF parsing flow with pdfjs-dist → failure fallback
- [ ] Manual entry form
- [ ] "Show original text" toggle
- [ ] Note-taking per quote
- [ ] Select winner flow + decision reason prompt
- [ ] Share link generation + read-only share view
- [ ] Password reset flow
- [ ] Email verification via Resend

### Week 4 — Billing + Launch Prep
- [ ] Stripe integration: checkout, webhooks, customer portal
- [ ] Free tier enforcement (1 project, 3 quotes, paste-only)
- [ ] Pro tier feature gates (PDF upload, PDF export, unlimited)
- [ ] PDF export (Puppeteer on Railway)
- [ ] Rate limiting on parse endpoint
- [ ] Error tracking (Sentry)
- [ ] Mobile responsive polish
- [ ] Landing page final copy + social proof

---

## Estimated Effort (Experienced Developer)

| Phase | Effort |
|-------|--------|
| Foundation + DB setup | 2-3 days |
| Claude quote parsing (core) | 2-3 days |
| Comparison table UI | 2-3 days |
| Auth (email + Google OAuth) | 1 day |
| PDF upload + parsing flow | 1-2 days |
| Share links | 0.5 day |
| Stripe billing | 1-2 days |
| Polish + mobile | 1-2 days |
| **Total** | **~3 weeks** |

---

## Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Claude misparses complex quotes | Medium | High | Confidence scores + edit mode + original text view |
| Scanned PDF (image-only) fails parsing | High | Low | Graceful fallback to manual entry |
| Users don't add enough quotes to see value | Medium | Medium | Onboarding nudge: "Add 2+ quotes for comparison" |
| Contractors ask DealStack to remove their quotes | Low | Low | User-owned data — contractor has no rights to user's copy of their quote |
| Stripe billing complexity | Low | Medium | Use Stripe-hosted checkout (no PCI scope) + customer portal |
| GDPR/CCPA compliance | Medium | Medium | Clear privacy policy, right-to-delete in settings, minimal data collection |
| Competitive response from Angi | Low | High | We're solving a different phase (post-quote), not competing for lead gen |

---

## Success Metrics

### 30 Days After Launch
- 200+ registered users
- 500+ projects created
- 60%+ of users who create a project add 2+ quotes (comparison value achieved)
- 5%+ free-to-paid conversion rate
- <2% day-7 churn for Pro users

### 90 Days After Launch
- 10+ Pro users ($90+ MRR)
- NPS > 40 (users who feel genuine relief)
- 3+ organic word-of-mouth referrals per week (share links clicked)
- Average session duration > 8 minutes (users engaged with comparison)

### The Real Signal
**"Did someone use this tool and feel less anxious about their contractor decision?"**  
That's the metric that matters. If yes — the product works.

---

## Why This Design Is Different From Existing Solutions

1. **Not a lead-gen marketplace.** Angi/Thumbtack monetize contractor referrals. We monetize homeowner outcomes. This makes our interests aligned — we want them to find the *right* contractor, not the highest-paying one.

2. **AI that shows its work.** Every AI-parsed line item has a confidence level and a "show original" fallback. This isn't magic-box AI — it's transparent and editable. Users trust tools that let them verify.

3. **Warnings as the killer feature.** "This quote doesn't include permits" — that's the insight that makes people share the product. The warnings surface what non-experts would miss.

4. **Designed for anxiety.** A homeowner comparing contractor quotes is stressed. The UX is calm, the copy is plain, and the data is organized without being overwhelming. We're not building for construction professionals — we're building for nervous homeowners.

5. **Simple enough to use once, compelling enough to come back.** If you do one roof and one kitchen remodel in a year, you return for both. For property managers, it's weekly.

---

*Design complete. Repo: https://github.com/cody-nixon/dealstack*
