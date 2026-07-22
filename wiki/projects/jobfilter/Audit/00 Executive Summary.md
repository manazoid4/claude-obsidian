---
type: audit
project: jobfilter
date: 2026-06-11
auditor: claude-fable-5
---

# JobFilter — Executive Summary (2026-06-11)

## Verdict
Build is green, design system coherent, copy strong — but **the core product promise (GOLD leads → WhatsApp) does not work end-to-end**. The intake API regressed: leads are not persisted, not tied to a tradesperson, and never trigger WhatsApp. Payments are wired but checkout trusts client input. Not ready to take money honestly. **2–4 focused days from a sellable private beta.**

## Scores
| Area | /100 |
|---|---|
| Repo health | 55 |
| Product clarity | 70 |
| Design quality | 75 |
| Technical readiness | 55 |
| Payment readiness | 60 |
| Lead delivery readiness | 35 |
| **Launch readiness** | **50** |

## What's real
- `npx tsc --noEmit` ✅ zero errors. `npm run build` ✅ clean (verified 2026-06-11).
- ~100 routes: trade pages, city pages, VS-competitor pages, free tools, dashboard, pricing.
- Stripe webhook with proper signature verification (`app/api/stripe/webhook/route.ts`).
- WhatsApp send route, Twilio-backed, paid-gated (`app/api/leads/whatsapp/route.ts`) — code good, env vars missing.
- Lead engine with 10 fetchers (planning, EPC, contracts, Companies House, etc.) + fallback safety nets.
- Coherent Brutalist-Yellow design system.

## What's broken / fake
1. **Intake engine regression** — `app/api/intake/score/route.ts` discards `username`, persists nothing, hardcodes `whatsapp: { triggered: false }`. The 2026-06-02 audit recorded these as FIXED; current branch (`fix/mobile-nav-rebuild`) has none of it.
2. **No automated GOLD → WhatsApp alert** — only manual per-lead send, and Twilio env vars are not set anywhere.
3. **GOLD means two different things** — `leadEngine/scorer.ts:239` (≥90) vs `server/services/decisionScoring.ts:60` (≥80).
4. **Checkout trusts the client** — `app/api/stripe/checkout/route.ts` accepts `priceId`, `userId`, `email` from request body with no auth.
5. Lead/win data largely **localStorage-only** — vanishes per device, invisible to the business.
6. `/test` and `/dev-portal` public. No rate limiting on scan APIs.
7. Frankenstein debris: Vite config, Express server, root fix-scripts, legacy folders.

## Single most important next step
Fix the intake → persist → WhatsApp pipeline (restore the 06-02 fixes + set Twilio env vars), then onboard 3–5 real trades manually. See [[12 Final Recommendation]].

Files: [[01 Repo Map]] · [[02 Feature Map]] · [[03 Technical Audit]] · [[04 Product UX Audit]] · [[05 Design System Audit]] · [[06 Bugs Risks and Blockers]] · [[07 Launch Readiness]] · [[08 Growth and Revenue Improvements]] · [[09 Next Build Plan]] · [[10 Agent Build Prompts]] · [[11 Useful Vault Findings]] · [[12 Final Recommendation]]
