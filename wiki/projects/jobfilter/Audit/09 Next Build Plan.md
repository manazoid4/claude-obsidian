---
type: audit
project: jobfilter
date: 2026-06-11
---

# 09 Next Build Plan

## 1. Quick wins today (≤2h each)
| P | Task | Why | Files | Effort | Risk | Acceptance |
|---|---|---|---|---|---|---|
| 1 | Unify GOLD threshold: export `GOLD_THRESHOLD=80`, `SILVER_THRESHOLD=50` from one module; import in `leadEngine/scorer.ts`, `decisionScoring.ts`, `leadScoring.ts`, LeadListPage | GOLD must mean one thing | leadEngine/scorer.ts:239, server/services/decisionScoring.ts:60 | S | L | grep shows zero literal 80/90 thresholds outside constants file |
| 2 | Gate `/test` + `/dev-portal`: `notFound()` unless owner session | Public dev console | app/test/page.tsx, app/dev-portal/page.tsx | S | L | Anonymous visit → 404 |
| 3 | `npm audit fix` + retest build | 2 high CVEs | package.json | S | L | 0 high; build green |
| 4 | Gitignore + delete root debris (logs, tsbuildinfo, fix-*.cjs, zips) | Repo hygiene | root | S | L | git status clean of junk |
| 5 | Resolve branch state: commit/merge `fix/mobile-nav-rebuild` → main via PR | Work loss risk | git | S | M | main contains mobile nav; no dirty tree |

## 2. 1–2 day fixes (launch blockers)
| P | Task | Why | Files | Effort | Risk | Acceptance |
|---|---|---|---|---|---|---|
| 1 | **Restore intake pipeline**: read `username`, persist to `intake_submissions` via service client, trigger WhatsApp on GOLD (graceful no-op without env). Check git log of pre-merge branches for the lost 06-02 version first | Core promise | app/api/intake/score/route.ts | M | M | Test submission lands in Supabase + WhatsApp arrives (sandbox) |
| 2 | **Harden checkout**: derive user from Supabase auth session server-side; ignore client `priceId`; map tier→price strictly server-side | Payment integrity | app/api/stripe/checkout/route.ts | S | L | Forged body cannot change price/user |
| 3 | **Twilio setup** (founder): sandbox sender + `TWILIO_*` env in Vercel; test `/api/leads/whatsapp` | Delivery | .env / Vercel | S | L | Real WhatsApp received |
| 4 | **Run migrations + envs** (founder): 20260531 pair; Vercel vars from TODO.md; verify `/api/subscription/status` | Prod truth | supabase/migrations, Vercel | S | L | Status endpoint returns owner business tier |
| 5 | Rate limit scan/intake/waitlist routes (simple in-memory or Upstash) | Abuse/cost | app/api/leads/*, intake, waitlist | S | L | >N req/min → 429 |

## 3. 1-week MVP hardening
- Persist scanned/intake leads per user in Supabase `leads`; LeadList + Dashboard read server-first, localStorage fallback. Acceptance: leads visible on second device.
- Post-signup onboarding: trade + postcode districts saved to `profiles`. Acceptance: scan defaults to saved area.
- Auto-GOLD pipeline: daily scheduled scan (Vercel cron or n8n workflow 01/02) for each paid user's trade+area → score → persist → WhatsApp GOLD. Acceptance: founder gets daily digest + instant GOLD message without touching app.
- Stripe customer portal link on /account. Acceptance: self-serve cancel works in test mode.
- Webhook: return 500 on DB failure paths so Stripe retries.

## 4. Revenue (parallel, non-code)
Concierge beta + outreach per [[08 Growth and Revenue Improvements]].

## 5. Design improvements
GOLD card treatment, lead-card field set, empty states per [[05 Design System Audit]]. After blockers.

## 6. Automation improvements
Activate n8n 01 (daily digest), 02 (GOLD alert), 05 (dedup). Verify instance + webhook URLs first.

## 7. Post-launch
SSR migration of trade/city/vs pages with `generateMetadata`; delete Express/Vite layers; Sentry; smoke tests (scorer, webhook, checkout); conversion event tracking.
