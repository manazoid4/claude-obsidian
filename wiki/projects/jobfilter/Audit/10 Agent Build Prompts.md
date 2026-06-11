---
type: audit
project: jobfilter
date: 2026-06-11
---

# 10 Agent Build Prompts

Copy/paste-ready. All prompts share these RULES:
> Repo: `C:\Users\manaz\Desktop\jobfilter\jobfilterv1`. Read before editing. Surgical changes only — no refactors beyond scope. Run `npx tsc --noEmit` and `npm run build` before reporting done. Never commit secrets. Work on a branch, open a PR — never push to main. Save a session summary to `C:\Users\manaz\claude-obsidian\wiki\projects\jobfilter\` (sessions or changelog folder) and update `STICKY-TODO.md` with any new founder-manual actions. Reference audit: `wiki/projects/jobfilter/Audit/`.

---

## Prompt 1 — Fix launch blockers
**Context:** Audit 2026-06-11 found 5 critical blockers (see Audit/06). **Goal:** all critical blockers closed. **Inspect:** `app/api/intake/score/route.ts`, `app/api/stripe/checkout/route.ts`, `leadEngine/scorer.ts`, `server/services/decisionScoring.ts`, `app/test/page.tsx`, `app/dev-portal/page.tsx`. **Tasks:** (1) Restore intake pipeline: capture `username`, persist to Supabase `intake_submissions` via service client (graceful no-op if table/env missing), trigger WhatsApp on GOLD — first check git history (`git log --all -- app/api/intake/score/route.ts`) for the lost 2026-06-02 version. (2) Harden checkout: user from Supabase auth session, server-side tier→price map only. (3) Single `GOLD_THRESHOLD=80` constant imported everywhere. (4) Gate /test + /dev-portal. (5) `npm audit fix`. **Acceptance:** tsc+build green; forged checkout body rejected; test intake row in Supabase; grep finds no stray thresholds.

## Prompt 2 — Redesign landing page (design system)
**Context:** Brutalist-Yellow system stays; tighten per Audit/05. **Goal:** landing converts trades. **Inspect:** `src/pages/HomePage.tsx`, `src/index.css`, `TopNav.tsx`. **Tasks:** section order = offer bar → hero+postcode scan → 3-step how-it-works → live signal table → redacted GOLD sample card → trades grid → pricing → guarantee → FAQ; add "what happens after payment" block; CTAs renamed to outcome form ("GET LOCAL LEADS"); apply the 10 Impeccable Design Rules. **Acceptance:** mobile-first check at 375px; one yellow CTA per viewport; build green.

## Prompt 3 — Harden Stripe, Supabase, auth, onboarding
**Context:** Audit/03 F2 + onboarding gap. **Goal:** safe payments + trade/postcode captured. **Inspect:** `app/api/stripe/*`, `src/lib/supabase/*`, `app/signup`, `supabase/migrations/`. **Tasks:** checkout auth (if not done by Prompt 1); webhook returns 500 on DB failures; post-signup step saving `trade` + `postcode_districts` to profiles (migration if needed); Stripe customer portal link on /account; rate limiting on scan/intake/waitlist. **Acceptance:** new user flow captures trade+area; cancel works in test mode; 429 on hammering.

## Prompt 4 — Lead scoring + GOLD logic
**Context:** 3 scoring modules disagree (Audit/03 F3). **Goal:** one engine, explainable scores. **Inspect:** `leadEngine/scorer.ts`, `server/services/{decisionScoring,leadScoring}.ts`, `LeadListPage.tsx`. **Tasks:** consolidate to `leadEngine/scoring.ts` with shared thresholds; every score returns `reasons: string[]` (top 3 signals); tier taxonomy decision: GOLD ≥80, SILVER ≥50, BRONZE ≥30, BIN <30; update types + UI. **Acceptance:** same lead scores identically via scan and intake; lead card shows reason line.

## Prompt 5 — WhatsApp-first lead delivery
**Context:** headline feature; manual route exists (`app/api/leads/whatsapp/route.ts`), auto missing. **Goal:** GOLD → WhatsApp ≤5 min, no manual step. **Inspect:** that route, `server/services/sms.ts`, `n8n-workflows/02-ready-signal-alert.json`, `leadEngine/scan.ts`. **Tasks:** on GOLD persist, enqueue WhatsApp to user's number from profile; dedupe via `delivery_events`; daily 7am digest (Vercel cron `app/api/cron/daily-scan/route.ts` with `CRON_SECRET`); document Twilio env setup in STICKY-TODO. **Acceptance:** test GOLD lead → sandbox WhatsApp received; duplicate lead → no second message.

## Prompt 6 — Dashboard + lead cards
**Context:** localStorage-only data (Audit/03 F5). **Goal:** server-backed, trade-focused dashboard. **Inspect:** `DashboardPage.tsx`, `LeadListPage.tsx`, `LeadCard.tsx`, `src/lib/*Store.ts`, `supabase/migrations/20260523_*`. **Tasks:** persist leads/outcomes to Supabase keyed by user; read server-first with localStorage fallback; lead card = score, work type, area, est. value, urgency, source badge, freshness, WhatsApp button; GOLD visual treatment per Audit/05. **Acceptance:** leads visible from second browser; outcomes survive device switch.

## Prompt 7 — SEO + trade landing pages
**Context:** ~30 SEO pages client-rendered = invisible (Audit/02 #25). **Goal:** crawlable money pages. **Inspect:** `app/trade/*/page.tsx`, `app/construction-leads/*`, `src/pages/TradePage.tsx`, `app/sitemap.ts`. **Tasks:** convert trade + city + vs pages to server components with `generateMetadata` (title/desc/JSON-LD LocalBusiness); interactive bits stay client islands; start with plumbers/electricians/builders ×Manchester/Birmingham/London. **Acceptance:** `curl` of page HTML contains real copy + meta; build green.

## Prompt 8 — Analytics + conversion tracking
**Context:** pageviews only. **Goal:** measurable funnel. **Inspect:** `@vercel/analytics` usage in `app/layout.tsx`, CTA components. **Tasks:** track events — scan_started, scan_results, signup, checkout_started, checkout_completed, whatsapp_clicked; thin `track()` helper; document funnel queries. **Acceptance:** events visible in Vercel Analytics within a test session.

## Prompt 9 — Private beta prep (5 trades)
**Context:** Stage 2 of Audit/07. **Goal:** 5 trades onboarded in one week. **Tasks:** beta checklist doc; onboarding script (WhatsApp message templates); seed each user's trade+area; daily concierge lead-send routine using FindJobs scans; feedback form (3 questions); collect testimonial quotes. Mostly ops — write docs to vault `wiki/projects/jobfilter/beta/`. **Acceptance:** checklist + templates exist; first user onboarded dry-run.

## Prompt 10 — Founder launch polish
**Context:** Stage 3 of Audit/07. **Goal:** take real money confidently. **Tasks:** Stripe live-mode checklist; real payment end-to-end test (£39, then refund); guarantee process doc; cancel flow verified; error monitoring (Sentry free) on api routes; status page or uptime ping on scan pipeline; final copy pass on pricing. **Acceptance:** one real transaction processed + refunded; Sentry captures a thrown test error.
