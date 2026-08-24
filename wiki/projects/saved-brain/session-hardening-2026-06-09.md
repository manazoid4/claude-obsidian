---
title: Saved Brain — Production Hardening Session
project: saved-brain
type: session-log
created: 2026-06-09
tags: [saved-brain, session, hardening, shipped]
---

# Session: Production Hardening

Back to [[index]]

Commit: `1c0b15d` on `main` — https://github.com/manazoid4/saved-brain

## What shipped (67 files, 3282 insertions)

### Core architecture
- Clerk auth + `middleware.ts` gating all API routes with `owner_id`
- Dual-mode DB (`lib/db.ts`): Supabase client + SQLite fallback, `owner_id` on all tables
- Route groups: `app/(app)/` + `app/(marketing)/` with separate layouts

### Security
- Settings encrypted with AES-256-GCM (`lib/encryption.ts`)
- Secrets filtered from `GET /api/settings`; POST allowlisted
- SSRF protection on Ollama/provider URLs (`lib/ai/providers.ts`)
- Provider fetch 30s timeout (`AbortController`) on all providers
- Provider response envelope validation (safe optional chaining, typed errors)
- Webhook HMAC uses `timingSafeEqual`
- HTML-escaped + URL-validated digest email content
- Zod validation on all POST routes (`lib/schemas.ts`)
- Rate limiting 60 req/min (`lib/rate-limit.ts`)
- `CRON_SECRET` guard on `/api/sync/cron`

### Monetization
- Entitlement enforcement: 50-item cap, 1-board cap, Pro gates (`lib/entitlements.ts`)
- `/api/license/verify` endpoint
- Purchase records to DB via webhook; LemonSqueezy license keys issued
- Competitor comparison table on pricing page

### UX
- Landing page + marketing layout (no sidebar bleed)
- Item detail view (`/items/[id]`)
- Loading skeletons + relative timestamps
- Global `+ Add Content` CTA in Sidebar
- Semantic search wired in Library
- Add to board modal + `/api/boards/[slug]/items` route
- No-results vs no-items empty states
- Responsive graph canvas + force-sim settle
- Privacy + Terms pages
- Public board virality CTA: "Build your own brain" → sign-up
- Sign-in / sign-up pages (Clerk)

### Tooling
- ESLint config + real lint script; `swcMinify` re-enabled
- Vercel weekly digest cron; migrations + test scaffolding; DESIGN.md

## Still to do
- Chrome extension: connect-account / issued-token flow
- Vercel env vars configured in dashboard
- Supabase project created (DATABASE_URL still empty)
- Tests beyond scaffolding
- NEXT_PUBLIC_APP_URL + Plausible in production
