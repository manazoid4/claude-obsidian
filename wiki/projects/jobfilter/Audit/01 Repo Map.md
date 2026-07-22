---
type: audit
project: jobfilter
date: 2026-06-11
---

# 01 Repo Map

**Path:** `C:\Users\manaz\Desktop\jobfilter\jobfilterv1`
**Branch at audit:** `fix/mobile-nav-rebuild` (uncommitted changes present)
**Stack:** Next.js 16 (App Router) + React 19 + Tailwind 4 + Supabase + Stripe + Resend. npm. Deployed to Vercel (`vercel.json`). Domain: jobfilter.uk.

## Folders

| Folder | Purpose | Status |
|---|---|---|
| `app/` | Next.js App Router — ~100 routes (home, pricing, dashboard, leads, trade/*, construction-leads/*, vs/*, free tools, intake/[username], api/*) | LIVE — but most pages are `'use client'` wrappers re-exporting `src/pages/*` |
| `src/pages/` | Actual page components (~60 files) | LIVE (rendered via app/ wrappers) |
| `src/components/` | UI library: LeadCard, ScoreBadge, TopNav, TrustBadges, ROITracker, CheckoutButton, etc. | LIVE, good quality |
| `src/lib/` | supabase clients, stripe helpers, leadStore/winStore/chaseStore (localStorage), types | LIVE — heavy localStorage reliance |
| `leadEngine/` | Scan pipeline: 10 fetchers (planning, EPC, contracts, FTS, Companies House, land registry, charity, forestry, directory), normaliser, scorer, contactPath, opportunityAtoms | LIVE core IP |
| `server/` | **Legacy Express app** (`server/app.ts`) + routes/services. Some services (decisionScoring, sms, leadPersistence) imported by Next routes; Express server itself orphaned | MIXED — partially dead, partially load-bearing |
| `pages/api/[[...path]].ts` | Catch-all proxy to Express | DEAD-ISH — anti-pattern leftover |
| `api/index.ts` | Vercel serverless entry leftover | DEAD-ISH |
| `supabase/` | schema.sql + 11 migrations (profiles, leads, subscriptions, lead_alerts, scan counter, owner access) | Code ready; unclear which migrations actually ran in prod |
| `n8n-workflows/` | 16 workflow JSONs (daily digest, GOLD alert, dedup, stripe→vault, etc.) + push/activate scripts | BUILT, activation state unknown |
| `scripts/` | n8n sync, daily/weekly/monthly agent runners (PowerShell) | Internal ops |
| `agents/`, `agent_prompts/`, `codex-output/` | Past agent prompts + audit outputs (codex audits 2026-06-06) | Docs/history |
| `Obsidian_Memory/`, `claude-notes/`, `memoryraw_claude/` | Embedded notes vaults | Should live in claude-obsidian vault, not repo |
| `legacy/`, `stitch-extract/`, `FinalDesignJobFilter/` | Old design exports + zips | DEAD weight (2MB+ zips committed) |
| `docs/`, `data/`, `tools/` | Misc docs, data, instagram-memory python tool | Mixed |

## Root debris (clean-up candidates)
`fix-*.cjs` ×8, `migrate-*.js`, `replace-router.cjs`, `use-client.cjs` (one-shot codemods), `index.html` + `vite.config.ts` + `index.mjs` (dead Vite era), `server.ts` (old monolith), `*.log` ×5, `FinalDesignJobFilter.zip`, `JobFilterStich.zip`, `tsconfig.tsbuildinfo` (should be gitignored).

## Key files
- `app/api/stripe/webhook/route.ts` — signature-verified webhook → subscriptions + profiles upsert. Owner-account protection.
- `app/api/stripe/checkout/route.ts` — checkout session creation (⚠️ trusts client body).
- `app/api/intake/score/route.ts` — intake scoring (⚠️ REGRESSED — no persistence/username/WhatsApp).
- `app/api/leads/whatsapp/route.ts` — Twilio WhatsApp send, paid-gated (env vars missing).
- `leadEngine/scan.ts` + `scorer.ts` — main scan + scoring (GOLD ≥90 here).
- `server/services/decisionScoring.ts` — intake scoring (GOLD ≥80 — mismatch).
- `server/lib/ownerAccess.ts` — owner allowlist (manazoid4@gmail.com).
- `supabase/migrations/20260531_*.sql` — latest migrations, founder may not have run them.

## Duplicated logic
- **Two scoring engines**: `leadEngine/scorer.ts` vs `server/services/decisionScoring.ts` + `server/services/leadScoring.ts`.
- **Two API layers**: Express `server/routes/*` vs Next `app/api/*` (intake exists in both, behaviour differs).
- **Two Supabase client sets**: `src/lib/supabase.ts` vs `src/lib/supabase/{client,server,auth-server}.ts`.
- **Page wrappers**: every `app/x/page.tsx` is 5 lines re-exporting `src/pages/XPage.tsx` — double the surface.

## Secrets check
✅ Only `.env.example` tracked in git. `.env.local` holds Stripe/Supabase/Resend keys locally (not committed). Missing keys: Twilio (WhatsApp), `COMPANIES_HOUSE_API_KEY`, `EPC_API_KEY`.

## Missing obvious files
- No tests at all (zero test files, no test runner in package.json).
- `lint` script is just `tsc --noEmit` — no ESLint config.
- No CI checks beyond Firebase-era workflows in `.github/`.
- package.json name is still `"react-example"`.
