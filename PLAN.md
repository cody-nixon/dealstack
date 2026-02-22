# PLAN.md — Technical Design
**Product:** DealStack  
**Tagline:** "Compare contractor quotes in minutes, not spreadsheet hours."

---

## Product Name & Branding

- **Name:** DealStack
- **Tagline:** "Every quote, side by side."
- **Subheadline:** "Upload contractor quotes in any format. DealStack reads them, normalizes them, and shows you a clean comparison — so you can decide with confidence."
- **Domain target:** dealstack.app

---

## Core User Stories (only the ones that matter)

### User: Homeowner getting renovation quotes

1. **AS A** homeowner who received 3 roofing quotes  
   **I WANT TO** upload all 3 and see them side by side in a normalized table  
   **SO THAT** I can make a fair comparison without building a spreadsheet  

2. **AS A** homeowner  
   **I WANT TO** add notes to each quote ("contractor was rude", "uses cheap shingles")  
   **SO THAT** I don't lose track of qualitative factors  

3. **AS A** homeowner  
   **I WANT TO** mark one quote as "selected" and see why it won  
   **SO THAT** I have a record of my decision for future reference  

4. **AS A** homeowner  
   **I WANT TO** share my comparison with my spouse or partner  
   **SO THAT** we can review together  

### User: Real estate investor / property manager

5. **AS A** property manager  
   **I WANT TO** have multiple active projects (bathroom, roof, HVAC)  
   **SO THAT** I manage all vendor quotes in one place  

6. **AS A** repeat user  
   **I WANT TO** see a history of past projects and who I picked  
   **SO THAT** I can reference past contractor relationships and pricing trends  

7. **AS A** property manager  
   **I WANT TO** export a comparison as a PDF to share with my client  
   **SO THAT** I can demonstrate due diligence  

---

## Tech Stack

### Frontend
- **React + Vite + TypeScript** — standard, fast
- **shadcn/ui** — component library (per LESSONS.md rule)
- **TailwindCSS** — styling
- **TanStack Query** — data fetching
- **Zustand** — client state for comparison table editing

### Backend
- **Node.js + Express + TypeScript**
- **Supabase** — PostgreSQL database + auth (email + Google OAuth)
- **Supabase Storage** — PDF/image file storage
- **Resend** — transactional email
- **Stripe** — billing (Free/Pro/Teams tiers)

### AI Layer
- **Anthropic Claude claude-sonnet-4-6** — PDF/text quote parsing + normalization
- **Supabase Edge Functions** — AI calls run serverside (keys never in browser)
- **PDF parsing**: pdfjs-dist for client-side text extraction before sending to Claude; raw text → structured JSON output via Claude structured output

### Infrastructure
- **Supabase** (managed Postgres + auth + storage + edge functions)
- **Vercel** — frontend hosting
- **Railway** — backend API hosting
- **Stripe webhooks** — billing events

---

## Architecture Overview

```
User Browser
    │
    ├── React/Vite SPA (Vercel CDN)
    │       │
    │       ├── Upload PDF/paste text
    │       ├── Edit comparison table
    │       └── View normalized quotes
    │
    ▼
Node.js/Express API (Railway)
    │
    ├── POST /api/quotes/parse
    │       └── Calls Claude with quote text
    │               └── Returns structured JSON (line items)
    │
    ├── POST /api/projects
    ├── GET  /api/projects/:id
    ├── POST /api/projects/:id/quotes
    ├── PATCH /api/projects/:id/quotes/:quoteId
    └── POST /api/projects/:id/export
    │
    ▼
Supabase (DB + Auth + Storage)
    ├── users (via Supabase auth)
    ├── projects table
    ├── quotes table (raw + normalized)
    └── Storage bucket: raw PDFs/images
    │
    ▼
Stripe (billing)
    └── Webhooks → backend → update users.plan_tier
```

---

## API Design

### Auth
All routes require `Authorization: Bearer <supabase_jwt>` header.

### Projects

**POST /api/projects**
```json
Request:
{
  "name": "Kitchen Remodel",
  "category": "kitchen_remodel",  // or "roofing", "hvac", "plumbing", "landscaping", "custom"
  "description": "Full kitchen gut and remodel, ~250sqft"
}

Response:
{
  "id": "uuid",
  "name": "Kitchen Remodel",
  "category": "kitchen_remodel",
  "description": "...",
  "status": "active",
  "created_at": "2026-02-22T...",
  "quotes": []
}
```

**GET /api/projects**
```json
Response:
{
  "projects": [
    {
      "id": "uuid",
      "name": "Kitchen Remodel",
      "category": "kitchen_remodel",
      "status": "active",
      "quote_count": 3,
      "created_at": "..."
    }
  ]
}
```

**GET /api/projects/:id**
```json
Response:
{
  "id": "uuid",
  "name": "Kitchen Remodel",
  "quotes": [
    {
      "id": "uuid",
      "contractor_name": "Bob's Remodeling",
      "total_price": 28500,
      "currency": "USD",
      "is_selected": false,
      "notes": "Seemed very professional",
      "line_items": [
        {"category": "labor", "label": "Demo & Prep", "amount": 2000, "notes": "3 days"},
        {"category": "materials", "label": "Cabinets (IKEA SEKTION)", "amount": 4200, "notes": ""},
        {"category": "materials", "label": "Countertops (quartz)", "amount": 3800, "notes": "Includes install"},
        {"category": "labor", "label": "Installation", "amount": 8500, "notes": ""},
        {"category": "permits", "label": "Building permit", "amount": 450, "notes": "City of Toronto"},
        {"category": "other", "label": "Disposal", "amount": 800, "notes": "Dumpster"},
        {"category": "other", "label": "Contingency", "amount": 1500, "notes": "10% buffer"},
        {"category": "markup", "label": "Contractor markup", "amount": 7250, "notes": "Estimated ~25%"}
      ],
      "raw_text": "...",
      "created_at": "..."
    }
  ],
  "comparison": {
    "categories": ["labor", "materials", "permits", "disposal", "markup"],
    "warnings": [
      "Quote from ABC Contractors does not include permits — add ~$400-800",
      "Quote from Mike's Kitchen doesn't itemize disposal costs"
    ]
  }
}
```

**POST /api/projects/:id/quotes**
```json
Request:
{
  "contractor_name": "Bob's Remodeling",
  "input_type": "text" | "pdf_url",
  "content": "...raw quote text or pre-uploaded PDF URL..."
}

Response:
{
  "id": "uuid",
  "status": "parsing",  // async parse starts
  "contractor_name": "Bob's Remodeling"
}

// Webhook/polling: quote.status → "parsed" with line_items populated
```

**PATCH /api/projects/:id/quotes/:quoteId**
```json
Request:
{
  "contractor_name": "Bob's Remodeling LLC",
  "notes": "Uses 3M certified installers",
  "is_selected": true,
  "line_items": [...]  // allow manual override/edit of AI parse
}
```

**POST /api/quotes/parse** (internal, called by worker)
```json
Request:
{
  "project_category": "kitchen_remodel",
  "raw_text": "...full quote text...",
  "quote_id": "uuid"
}

Response (structured JSON from Claude):
{
  "contractor_name": "Bob's Remodeling",
  "total_price": 28500,
  "currency": "USD",
  "confidence": 0.91,
  "line_items": [...],
  "warnings": [
    "No permit costs listed — typical permits for this scope are $300-600",
    "No disposal mentioned — ask about dumpster"
  ],
  "missing_categories": ["permits", "disposal"]
}
```

**POST /api/projects/:id/export**
```json
Request:
{ "format": "pdf" }

Response:
{ "download_url": "https://...", "expires_at": "..." }
// Pro+ only
```

### Billing

**POST /api/billing/create-checkout**  
**POST /api/billing/portal**  
**POST /api/webhooks/stripe** (signature verified)

---

## Data Model

```sql
-- Users (managed by Supabase Auth, extended here)
CREATE TABLE user_profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  email TEXT NOT NULL,
  plan_tier TEXT NOT NULL DEFAULT 'free', -- 'free', 'pro', 'teams'
  stripe_customer_id TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Projects
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  category TEXT NOT NULL, -- 'roofing', 'hvac', 'kitchen_remodel', 'plumbing', 'landscaping', 'custom'
  description TEXT,
  status TEXT NOT NULL DEFAULT 'active', -- 'active', 'decided', 'archived'
  selected_quote_id UUID, -- FK to quotes, set when decision made
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_projects_user_id ON projects(user_id);

-- Quotes
CREATE TABLE quotes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  contractor_name TEXT NOT NULL,
  raw_text TEXT,         -- original text/extracted PDF text
  raw_file_url TEXT,     -- Supabase Storage URL for original PDF
  total_price NUMERIC(12,2),
  currency TEXT DEFAULT 'USD',
  confidence NUMERIC(3,2), -- Claude's confidence score (0.00-1.00)
  parse_status TEXT NOT NULL DEFAULT 'pending', -- 'pending', 'parsing', 'parsed', 'failed'
  line_items JSONB DEFAULT '[]',   -- [{category, label, amount, notes}]
  warnings JSONB DEFAULT '[]',     -- AI-generated missing item warnings
  notes TEXT,            -- user's qualitative notes on this contractor
  is_selected BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_quotes_project_id ON quotes(project_id);

-- Project shares (for sharing comparison link with spouse etc.)
CREATE TABLE project_shares (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  share_token TEXT NOT NULL UNIQUE DEFAULT encode(gen_random_bytes(24), 'hex'),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  expires_at TIMESTAMPTZ DEFAULT (NOW() + INTERVAL '30 days')
);
```

### RLS Policies
- Users can only read/write their own projects and quotes
- Shared projects readable by anyone with the share token (read-only)

---

## Auth Flow

### Email + Password
1. User enters email + password → `supabase.auth.signUp()`
2. Email verification sent via Resend
3. After verify → redirect to app → `user_profiles` row created via DB trigger

### Google OAuth
1. Click "Sign in with Google" → `supabase.auth.signInWithOAuth({ provider: 'google' })`
2. Supabase handles the OAuth handshake
3. On success → `user_profiles` row upserted via trigger (handles existing email edge case)

### Edge Case: Google email matches existing email account
- Supabase handles this natively — it links the OAuth identity to the existing account

---

## UI Flow (Step by Step)

### New User Journey
1. **Landing page** → headline, demo GIF/video, "Start free" CTA
2. **Sign up** → email+password OR "Continue with Google"
3. **Empty state dashboard** → "You have no projects yet. Create your first one."
4. **Create Project modal** → name the project (e.g., "New Roof"), pick category, add optional description
5. **Project view** — empty — shows "Add your first quote"
6. **Add Quote flow** → modal → type contractor name → paste text OR upload PDF
7. **Parsing state** → skeleton loader, "Reading your quote..." (5-15 seconds)
8. **Quote card appears** with parsed line items
9. **Repeat for 2-3 more quotes**
10. **Comparison table** populates — all quotes normalized side by side
11. **Review warnings** — "Quote from ABC Roofing doesn't include permits"
12. **Add notes** to each quote
13. **Mark one as Selected** → decision recorded
14. **Share link** → copy URL to send to spouse/partner

### Returning User Journey
- Dashboard → see all active projects + quote counts
- Click project → straight to comparison view

---

## What We're NOT Building (Scope Cuts)

- ❌ Contractor marketplace / finding contractors (that's Angi)
- ❌ Project management / scheduling / milestone tracking
- ❌ Contractor-side portal (contractors don't use this tool)
- ❌ Automated email parsing (future feature — for now, paste or upload)
- ❌ Price fairness benchmarking (future — "is $8,500 for a kitchen reasonable?")
- ❌ Mobile apps (web-only for V1)
- ❌ Team collaboration beyond share links (that's v2)

---

## Tier Limits

| Feature | Free | Pro ($9/mo) | Teams ($29/mo) |
|---------|------|-------------|----------------|
| Projects | 1 | Unlimited | Unlimited |
| Quotes per project | 3 | Unlimited | Unlimited |
| PDF upload | ❌ | ✅ | ✅ |
| AI normalization | Basic | Full | Full |
| Share link | ✅ | ✅ | ✅ |
| PDF export | ❌ | ✅ | ✅ |
| Project history | Last 30 days | All | All |
| Team members | 1 | 1 | 5 |
| Priority support | ❌ | ❌ | ✅ |

---

*Technical plan complete. Proceeding to Phase 4: Multi-Role Review.*
