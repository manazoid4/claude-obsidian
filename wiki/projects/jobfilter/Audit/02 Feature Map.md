---
type: audit
project: jobfilter
date: 2026-06-11
---

# 02 Feature Map

Status legend: ✅ Complete · 🟡 Partial · 🎭 Mocked · 🔴 Broken · ⬜ Missing
Risk: L/M/H/C · Effort: S/M/L

| # | Feature | Status | Files | What works / what doesn't | Risk | Next action | Effort |
|---|---|---|---|---|---|---|---|
| 1 | Homepage | ✅ | `app/page.tsx` → `src/pages/HomePage.tsx` | Strong hero, proof points, signal table, city list. Client-rendered (SEO weak) | M | Server-render hero + meta | M |
| 2 | Signup | 🟡 | `app/signup`, `src/lib/supabase/*` | Supabase auth works locally; email confirm flow untested in prod | H | E2E test on prod domain | S |
| 3 | Login/auth | 🟡 | `app/login`, `AuthProvider.tsx`, `ProtectedRoute.tsx` | Works; protection is client-side only — API routes vary | H | Server-side guards on sensitive routes | M |
| 4 | Onboarding | 🟡 | signup → pricing → dashboard | No guided trade/postcode capture after payment; metadata fields exist in checkout but optional | H | 2-step post-signup: trade + postcode | M |
| 5 | Postcode/area selection | 🟡 | `FindJobsPage`, `territories` | Scan accepts postcode; no saved "my areas" cluster per account | M | Persist user areas to profile | M |
| 6 | Trade selection | 🟡 | scan params, trade pages | Selectable per scan; not persisted per account | M | Persist to profile | S |
| 7 | Pricing page | ✅ | `src/pages/PricingPage.tsx` | £39/mo founder tier, objection copy, guarantee. No annual option | L | Add annual (~£390) | S |
| 8 | Stripe checkout | 🟡 | `app/api/stripe/checkout/route.ts` | Creates session; ⚠️ trusts client `priceId`/`userId`/`email`, no server auth | **C** | Derive user from session server-side; whitelist price IDs | S |
| 9 | Stripe webhooks | ✅ | `app/api/stripe/webhook/route.ts` | Signature verified; subscription+profile upsert; owner protection. Needs `STRIPE_WEBHOOK_SECRET` in Vercel + endpoint registered | M | Founder: register webhook, set env | S |
| 10 | Subscription state | 🟡 | `useSubscription.ts`, `subscriptions` table | Logic exists; depends on migrations being run in prod Supabase | H | Verify prod schema; run 20260531 migrations | S |
| 11 | Dashboard | 🟡 | `DashboardPage.tsx`, ROITracker | Renders, ROI stats, win summary — but data localStorage-only | M | Read from Supabase leads | M |
| 12 | Lead list | 🟡 | `LeadListPage.tsx` | Tabs by tier (fixed 06-02); localStorage source | M | Server-backed lead list | M |
| 13 | Lead detail | ✅ | `LeadDetailPage.tsx` | Score, Why-this-lead, WhatsApp deep-link, ICS calendar | L | — | — |
| 14 | Lead score display | ✅ | `ScoreBadge.tsx` | Good visual treatment | L | — | — |
| 15 | GOLD lead logic | 🔴 | `leadEngine/scorer.ts:239` (≥90) vs `decisionScoring.ts:60` (≥80) | Two engines, two thresholds — GOLD inconsistent across product | **C** | Single threshold constant, one source of truth | S |
| 16 | WhatsApp alerts | 🔴 | `app/api/leads/whatsapp/route.ts` (manual, good code); `server/services/sms.ts` (auto, orphaned); intake route hardcodes `triggered:false` | No Twilio env vars → 503. No automatic GOLD alert anywhere live | **C** | Set Twilio vars + wire auto-trigger on GOLD | M |
| 17 | Email alerts | 🟡 | `app/api/alerts/route.ts`, `20260531_lead_alerts.sql`, Resend key present | Skeleton built; migration possibly not run; no cron sender | H | Run migration + n8n/cron digest sender | M |
| 18 | n8n automation | 🎭 | `n8n-workflows/*.json` ×16 | Workflows authored; activation state unknown; webhook URLs env-dependent | M | Verify n8n instance live, activate 01+02 | M |
| 19 | Supabase schema | 🟡 | 11 migrations | Comprehensive on paper; prod state unverified; RLS not audited | H | Schema diff vs prod; RLS review | M |
| 20 | Lead ingestion | 🟡 | `leadEngine/fetchers/*` ×10 | Contracts/FTS/Planning live no-key; CH/EPC need keys (absent); fallback injects 7 sample jobs | M | Add CH + EPC keys | S |
| 21 | Lead scoring | 🟡 | `scorer.ts`, `leadScoring.ts`, `decisionScoring.ts` | Works but fragmented (see #15) | H | Consolidate | M |
| 22 | Admin/internal | 🟡 | `/test`, `/dev-portal`, `adminGuard.ts` | ⚠️ Both pages public by URL | H | Auth-gate or remove from prod | S |
| 23 | Cancellation flow | 🟡 | webhook handles `subscription.deleted`; `customerPortal.ts` Express-only | No user-facing cancel button on Next path | M | Stripe customer portal link on /account | S |
| 24 | Refund/trial | ⬜ | pricing copy mentions guarantee | No mechanism; manual via Stripe dashboard is fine for now | L | Document manual process | S |
| 25 | SEO | 🟡 | `app/sitemap.ts`, trade/city/vs pages | Sitemap good; pages client-rendered so content invisible to crawlers; no per-page `generateMetadata` | H | Server-render SEO pages + metadata | L |
| 26 | Analytics | 🟡 | `@vercel/analytics` installed | Basic pageviews only; no conversion events | M | Add event tracking on CTA/checkout | S |
| 27 | Mobile UX | ✅ | TopNav hamburger, 44px targets | Fixed in recent sprints (current branch is mobile-nav rebuild) | L | Verify branch merge | S |
| 28 | Error states | ✅ | ErrorBoundary, scan fallbacks | Good — never blank | L | — | — |
| 29 | Empty states | ✅ | LeadList/Dashboard isEmpty | Handled | L | — | — |
| 30 | Loading states | ✅ | Skeleton.tsx | Handled | L | — | — |
| 31 | Security/privacy | 🟡 | privacy/terms pages exist | No rate limiting on Next API routes; checkout trust issue; npm audit 2 high | H | See [[03 Technical Audit]] | M |
| 32 | Deployment | 🟡 | vercel.json, Vercel auto-deploy | Builds clean; Vercel env vars incomplete (TODO.md list); branch state messy | H | Merge branch, set env, deploy checklist | S |

## Acceptance criteria for "core loop works"
1. New user signs up → picks trade + postcode → pays £39 → profile shows `plan: founding`.
2. Scan or intake produces lead scored by **one** engine; ≥80 (chosen single threshold) = GOLD.
3. GOLD lead → WhatsApp message arrives on user's phone within 5 min, no manual step.
4. Lead visible in dashboard from any device (Supabase, not localStorage).
5. Cancel via Stripe portal → plan downgrades on webhook.
