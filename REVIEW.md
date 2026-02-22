# REVIEW.md — Multi-Role Review
**Product:** DealStack  
**Reviewer:** Cody Nixon (wearing multiple hats)

---

## Product Review

**Core value prop in one sentence:**  
"Paste or upload your contractor quotes; DealStack normalizes them all to the same categories so you can compare apples to apples in 5 minutes."

**Does this solve the actual problem?**  
Yes — the core pain is quote format chaos. Three quotes in three different formats → one normalized table. That's the job.

**Concerns:**

1. **The cold start problem.** If a user only has 1 quote, there's nothing to compare. The UI must handle this gracefully — "Add your first quote to start comparing. Add a second to see the magic." Frame it as a staged reveal.

2. **"AI normalization" trust.** If Claude maps "framing lumber + labor" incorrectly, the user might make a bad decision. We need:
   - Visual confidence scores per line item
   - Clear "edit this" affordances — every line item must be user-editable
   - "Show me the original text" toggle so they can verify

3. **Category selection is critical.** The normalization categories differ wildly by project type. Roofing has very different line items than landscaping. We need **per-category normalization templates** baked into the Claude prompt.

4. **The sharing feature needs a clear design.** "Share with my spouse" — does the spouse need an account? No. Read-only view via link is the right call.

5. **Missing the "why did I choose X" story.** After marking a quote as selected, prompt: "In 1-2 sentences, why did you choose this contractor?" Save it. Show it in history. This is what makes history useful.

**Revised scope addition:** Add per-line-item confidence badges + edit mode. This is not optional — it's safety-critical for the product.

---

## QA Review

**Most likely things to break:**

1. **PDF parsing edge cases** — Scanned PDFs (images inside PDF) are unreadable by pdfjs-dist. Need fallback: "We couldn't read this PDF. Please paste the text manually." OCR is a v2 feature (too complex for now).

2. **Very short quotes** — "Job will cost $8,500" with zero line items. Claude should handle this and flag "This quote has no breakdown — we estimated typical line items for your project type. Verify with contractor."

3. **Non-USD currencies.** Field exists. Make sure the comparison table shows currency per quote and warns if quotes are in different currencies.

4. **Long parsing times.** Claude API calls with large PDFs can take 10-30 seconds. Must show meaningful loading state. Use optimistic UI — show the quote card immediately as "Parsing..." with skeleton line items, fill in when done.

5. **Parse failure path.** Claude might fail or return malformed JSON. Must catch this gracefully: show "Parse failed — you can enter line items manually" with a manual entry form.

6. **Sharing expired links.** Share tokens expire in 30 days. Must show a clean "This comparison link has expired" page, not a 404.

7. **Free tier limit enforcement.** When a free user tries to add a 4th quote, the CTA must be an upgrade prompt — NOT just a silent failure.

8. **Auth edge cases:**
   - Email already registered → try to sign in with Google → should link accounts (Supabase handles this, but test it)
   - Password reset flow must work before launch
   - Session expiry → redirect to login, restore deep link after

**Key edge cases to test:**
- Empty project (no quotes)
- 1-quote project (no comparison yet)
- All quotes at same price (comparison still useful for breakdown)
- Quote with negative line items (credit/discount) — handle gracefully
- PDF with multiple pages
- Very long contractor name — truncate in table header

---

## Engineering Review

**Is this the simplest architecture?**

Yes and no. Using Supabase for both auth + DB + storage is the right call — it eliminates a lot of boilerplate. However:

1. **Async parsing needs careful design.** Two options:
   - **Option A (simpler):** Synchronous — POST quote → API calls Claude → waits → returns parsed. Max 20-30 seconds. Show loading state. Simple but slow-feeling.
   - **Option B (better UX):** Async — POST quote → API saves pending quote → returns quote ID → client polls GET /quotes/:id every 2s → when status = parsed, reveal line items. More complex but feels snappier.
   
   **Recommendation:** Start with Option A (synchronous), add streaming response via SSE if needed. Do not over-engineer with queues for V1.

2. **PDF extraction strategy:**
   - Client-side: use pdfjs-dist to extract text from uploaded PDFs in the browser before uploading
   - If pdfjs-dist fails (scanned PDF), upload raw file to Supabase Storage, then flag as "manual entry needed"
   - No server-side PDF parsing library needed (avoids binary deps on Railway)

3. **Claude prompt design is the core engineering challenge.** The system prompt must:
   - Know which project category is being parsed (roofing vs. kitchen vs. HVAC)
   - Have examples of each category's typical line items
   - Return structured JSON (use `response_format` or explicit JSON instruction + Zod validation)
   - Flag missing categories explicitly
   - Handle ambiguous/incomplete quotes gracefully

4. **The comparison table state management** — React state for a dynamic table where columns are quotes and rows are categories, each cell potentially editable — use Zustand, not just useState. Avoid prop drilling hell.

5. **Don't over-normalize.** The categories should be loose — "labor", "materials", "permits", "disposal", "markup", "other". Don't try to get too granular (sub-labor categories, specific material types). That's where complexity explodes and correctness drops.

**Unnecessary complexity to avoid:**
- Real-time collaboration (Supabase Realtime) — not needed for V1
- Background job queue (BullMQ, Redis) — synchronous Claude calls are fine for V1
- Server-side PDF rendering for export — use a simple HTML-to-PDF approach (Puppeteer or html-pdf-node on Railway)

---

## Design Review

**First-time user lands on the page — 5-second test:**

The value prop must be visually obvious INSTANTLY. Options:

1. **Show a before/after:** Left side = three messy quote documents. Right side = clean normalized comparison table. No words needed.
2. **One-line headline:** "Every contractor quote, side by side." Not "AI-powered quote comparison" — that's too jargon-y. "Every contractor quote, side by side."
3. **CTA clarity:** "Compare my quotes free" — not "Get started" or "Sign up." Tell them what they're about to do.

**The comparison table UX is the product.** This is where design must be exceptional:
- Columns = contractors. Rows = categories.
- Each cell shows: dollar amount (large), optional note (small, gray below)
- Row totals + grand total row (sticky at bottom)
- Highlight lowest price in each row with a subtle green background
- Highlight warnings (missing items) with amber/yellow
- The "selected" quote column gets a subtle blue border

**Empty states must tell a story:**
- 0 quotes: "Add your first quote to get started. Upload a PDF or paste text." Big upload zone.
- 1 quote: "Add one more quote to see your comparison." Half-full table with gray placeholder column.
- 2+ quotes: Full comparison table.

**Mobile consideration:**
- The comparison table is hard on mobile when there are 3+ columns. Solution: on mobile, show one quote at a time with swipe navigation ("Quote 1 of 3"), with a summary row showing relative position.
- Or: horizontal scroll with sticky first column (category labels).
- Recommend: horizontal scroll + sticky categories for simplicity.

---

## Security Review

**Attack surface:**

1. **File uploads** — PDFs from untrusted users:
   - Validate file type server-side (not just MIME from client — check magic bytes)
   - Size limit: 10MB per file
   - Store in Supabase private bucket (not public URLs — generate signed URLs for access)
   - Never execute uploaded files

2. **Claude API key** — never exposed to client. All Claude calls go through backend API.

3. **Stripe webhooks** — verify signature with `stripe.webhooks.constructEvent()` before processing

4. **RLS policies** — Supabase RLS must prevent cross-user data access. Test: attempt to GET /api/projects/{other_user_id} should return 403.

5. **Share tokens** — read-only. No mutation possible via share token route. Enforce at DB level.

6. **Rate limiting** — the parse endpoint calls Claude (costs money per call). Rate limit per user: 20 parses/hour on free, 100/hour on Pro. Use Redis or in-memory counter for V1.

7. **Input sanitization** — raw quote text goes into Claude prompt. Ensure it's escaped properly to avoid prompt injection (use a system-prompt boundary that separates user content clearly).

8. **API authentication** — all routes check Supabase JWT validity server-side. No routes are publicly accessible except share token view (read-only) and Stripe webhooks.

---

## Plan Revisions Based on Review

After this review, the following changes are made to the plan:

1. ✅ **Add confidence badges to line items** — per-item confidence score (high/medium/low) from Claude
2. ✅ **Add inline edit mode** — users can edit any line item after AI parse
3. ✅ **Add "Show original text" toggle** per quote — so users can verify parsing
4. ✅ **Graceful PDF failure path** — if pdfjs-dist can't extract text, show "Paste text manually"
5. ✅ **Add "Why did you choose this?" prompt** when marking a quote as selected
6. ✅ **Start with synchronous Claude calls** (not async queue) — simpler, good enough for V1
7. ✅ **Highlight minimum per category** in comparison table — quick visual cue
8. ✅ **Mobile horizontal scroll** for comparison table with sticky category column
9. ✅ **Rate limit parse endpoint** — 20/hr free, 100/hr Pro
10. ✅ **File validation** — magic bytes check, 10MB limit, private Supabase Storage

---

*Review complete. No blocking issues. Proceeding to Phase 5: Wireframes.*
