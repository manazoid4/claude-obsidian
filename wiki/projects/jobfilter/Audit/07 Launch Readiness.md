---
type: audit
project: jobfilter
date: 2026-06-11
---

# 07 Launch Readiness

| Score | /100 | Why | +10 points | +25 points |
|---|---|---|---|---|
| Repo health | 55 | Builds green, tsc clean; but dead stacks, zero tests, debris, branch chaos | Delete Vite/Express corpses + root debris | Consolidate scoring + API layers, add smoke tests |
| Product clarity | 70 | Home/pricing/VS copy strong; intake + post-payment expectations weak | "After payment" block + intake rewrite | Real case study + sample WhatsApp screenshot |
| Design quality | 75 | Coherent Brutalist-Yellow, mobile fixed | GOLD visual treatment + lead card upgrade | SSR landing performance |
| Technical readiness | 55 | See [[03 Technical Audit]] | Rate limiting + checkout fix + /test gate | Supabase persistence + monitoring |
| Payment readiness | 60 | Webhook solid; checkout trust flaw; Stripe dashboard/Vercel setup incomplete | Fix checkout route + register webhook + envs | Customer portal + annual plan + payment tested live |
| Lead delivery readiness | 35 | Scan works w/ fallbacks; intake regressed; WhatsApp not configured; no auto GOLD alert | Restore intake fixes + Twilio sandbox | Full auto pipeline: scan→score→persist→WhatsApp + daily digest |
| **Launch readiness** | **50** | Core loop broken at delivery step | Fix blockers in [[06 Bugs Risks and Blockers]] | Stage 2 beta complete |

## What MUST be fixed before taking payments
1. Checkout route server-side auth + price whitelist (F2).
2. Intake persistence + WhatsApp trigger (F1) — or honestly sell "daily scan + manual alerts" instead.
3. Stripe live keys, webhook registered, migrations run, `/api/subscription/status` verified.
4. Cancel path (Stripe portal link).
5. GOLD threshold unified — the thing being sold must be defined.

## Launch stages

### Stage 1 — Internal working demo (target: +2 days)
- [ ] Intake → Supabase → WhatsApp (founder's phone, Twilio sandbox) works end-to-end
- [ ] One scoring engine, GOLD=80
- [ ] Checkout in Stripe test mode upgrades profile via webhook
- [ ] /test, /dev-portal gated

### Stage 2 — Private beta, 3–5 trades (target: +1 week)
- [ ] Onboard manually (WhatsApp group / personal contact)
- [ ] Trade + postcode saved per account; leads delivered daily (concierge-curated OK)
- [ ] Rate limiting live; npm audit clean
- [ ] Collect feedback + 3 testimonial quotes

### Stage 3 — Paid founder users (£39/mo)
- [ ] Stripe live mode, real payment tested, portal cancel works
- [ ] Auto GOLD WhatsApp within 5 min of detection
- [ ] Leads cross-device (Supabase)
- [ ] Money-back guarantee process documented

### Stage 4 — Public launch
- [ ] SEO pages server-rendered + metadata
- [ ] Conversion analytics funnel
- [ ] Case study live; sample lead PDF
- [ ] Monitoring (Sentry) + uptime alert on scan pipeline
