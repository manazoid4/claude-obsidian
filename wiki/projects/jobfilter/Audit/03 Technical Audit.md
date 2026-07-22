---
type: audit
project: jobfilter
date: 2026-06-11
---

# 03 Technical Audit

## Checks run (2026-06-11)
| Command | Result |
|---|---|
| `npx tsc --noEmit` | ✅ exit 0, zero errors |
| `npm run build` | ✅ exit 0, clean |
| `npm audit --omit=dev` | ⚠️ 2 high, 2 moderate, 0 critical |
| Secrets in git | ✅ only `.env.example` tracked |
| Tests | ❌ none exist |

## Scorecard

| Category | /10 | Notes | Priority fix |
|---|---|---|---|
| Architecture clarity | 4 | App Router shell over client SPA pages; Express+Vite corpses; dual API layers | Delete dead layers, pick App Router only |
| Type safety | 8 | tsc clean, decent types in leadEngine | Remove `as any` in webhook period_end |
| Error handling | 7 | try/catch broad, fallbacks everywhere; webhook swallows handler errors with 200 | Log to durable store |
| API design | 5 | Mixed conventions; intake exists twice with different behaviour | Single source per endpoint |
| DB schema | 7 | 11 thoughtful migrations; prod drift unknown | Verify prod = migrations |
| RLS/policies | 4 | Service-role used server-side (fine) but RLS rules not reviewed; client anon key usage broad | RLS audit pass |
| Auth protection | 5 | WhatsApp route checks auth properly; checkout doesn't; `/test`,`/dev-portal` public | Gate checkout + dev pages |
| Stripe security | 5 | Webhook sig ✅. Checkout: client-supplied `priceId` accepted (plan/price mismatch attack), client-supplied `userId`/`email` unverified | Whitelist prices, derive user from session |
| Webhook verification | 9 | Proper `constructEvent` with secret | — |
| Env handling | 7 | Graceful degradation when keys missing; but silent feature-off can look like "working" | Startup config report |
| Input validation | 6 | Intake sanitizes; postcode regex ok; WhatsApp phone regex ok; scan params loose | Zod on scan |
| Rate limiting | 2 | Express middleware exists but Express is dead; Next routes unprotected | Upstash/Vercel ratelimit on scan + intake + waitlist |
| Secrets exposure | 8 | .env.local untracked; old Firebase key issue (2026-04) — confirm rotation | Confirm Firebase key rotated |
| Hardcoded values | 6 | Owner email hardcoded in webhook (also env — duplicated); fallback leads in code | Centralise |
| Duplication | 3 | 3 scoring modules, 2 API layers, 2 supabase client sets, wrapper pages ×60 | Consolidation sprint |
| Unused deps | 5 | `express`, `vite`, `@vitejs/plugin-react`, `alpinejs`, `react-router-dom`, `@tailwindcss/vite`, `ai` likely unused on Next path | Depcheck + prune |
| Tests | 0 | None | Smoke tests for scorer + webhook + checkout |
| Logging/monitoring | 3 | console.* only; no Sentry; webhook errors invisible in prod | Add Sentry free tier |
| Performance | 5 | Client-render everything = slow FCP; lead scan APIs 5–10s upstream | SSR + streaming |
| Deployment risk | 5 | Branch `fix/mobile-nav-rebuild` unmerged + dirty; Vercel envs incomplete | Merge + env checklist |

**Overall technical readiness: 55/100**

## Critical findings detail

### F1 — Intake regression (CRITICAL)
`app/api/intake/score/route.ts` (current branch): no `username` read, no Supabase persistence, `whatsapp: { triggered: false, provider: 'none' }` hardcoded. The 2026-06-02 vault audit recorded these exact issues as "FIXED THIS RUN". Either fixes landed on another branch and were lost in the `aa0b952` merge, or were reverted. **Recover via git history or re-implement.**

### F2 — Checkout privilege escalation (CRITICAL before real payments)
`app/api/stripe/checkout/route.ts:22` — `body.priceId` used verbatim if present. A user can POST `tier: "business"` + a cheaper `priceId`; webhook then sets `plan` from metadata tier. Also `userId`/`email` are client-claimed (lines 24–25). Fix: ignore client priceId, resolve strictly from server map; get user from Supabase auth session.

### F3 — Scoring split (HIGH)
GOLD ≥90 (`leadEngine/scorer.ts:239`) vs ≥80 (`decisionScoring.ts:60`). Marketing says 80+. Pick 80, export single `GOLD_THRESHOLD` constant, import everywhere.

### F4 — No rate limiting (HIGH)
All Next API routes open. `server/middleware/rateLimit.ts` exists but belongs to dead Express app. Scan endpoints fan out to slow upstream APIs — cheap DoS / quota burn.

### F5 — localStorage as primary datastore (HIGH)
`leadStore`, `winStore`, `chaseStore` — leads/wins per-device only. Business has zero visibility into user outcomes; ROI tracker lies across devices.

### F6 — npm audit (MEDIUM)
2 high (prod deps). Run `npm audit fix`, review remainder.
