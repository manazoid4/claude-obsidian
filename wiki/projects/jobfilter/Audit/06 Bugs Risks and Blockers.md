---
type: audit
project: jobfilter
date: 2026-06-11
---

# 06 Bugs, Risks and Blockers

## 🔴 Critical launch blockers

| Issue | Evidence | Files | Impact | Fix | Effort | Owner |
|---|---|---|---|---|---|---|
| Intake regression: no username/persistence/WhatsApp | Route returns `whatsapp:{triggered:false}` hardcoded; no Supabase call | `app/api/intake/score/route.ts` | Core promise broken; leads orphaned | Restore 06-02 fixes (check git history of other branches) or re-implement | M | backend |
| No automated GOLD→WhatsApp pipeline | Twilio env vars absent; only manual send route | `app/api/leads/whatsapp/route.ts`, `.env.local` | The headline feature doesn't exist for users | Twilio setup + trigger on GOLD persist | M | backend+automation |
| Checkout trusts client `priceId`/`userId`/`email` | Lines 22–28 use body values, no auth | `app/api/stripe/checkout/route.ts` | Plan escalation; payments attributed wrongly | Server-side session user + price whitelist | S | backend |
| GOLD threshold split (90 vs 80) | `scorer.ts:239` vs `decisionScoring.ts:60` | leadEngine + server services | Same lead GOLD in one view, SILVER in another — trust killer | Single exported `GOLD_THRESHOLD=80` | S | backend |
| Prod env/migrations unverified | TODO.md lists unset Vercel vars + unrun migrations | `supabase/migrations/20260531_*`, Vercel | Subscriptions/alerts silently broken in prod | Run migrations, set env, verify `/api/subscription/status` | S | DevOps (founder) |

## 🟠 High priority

| Issue | Evidence | Files | Impact | Fix | Effort | Owner |
|---|---|---|---|---|---|---|
| No rate limiting on Next APIs | Express middleware orphaned | all `app/api/*` | Quota burn, abuse, cost | Vercel/Upstash ratelimit wrapper | S | backend |
| `/test` + `/dev-portal` public | In sitemap, no guard | `app/test/page.tsx`, `app/dev-portal/page.tsx` | Dev console exposed | Auth-gate or `notFound()` in prod | S | frontend |
| Leads/wins localStorage-only | leadStore/winStore/chaseStore | `src/lib/*Store.ts` | Cross-device loss; zero business visibility | Supabase persistence (tables exist) | M | backend |
| SEO pages client-rendered | `'use client'` wrappers on trade/city pages | `app/trade/*`, `app/construction-leads/*` | Organic strategy dead on arrival | Server-render + `generateMetadata` | L | frontend |
| npm audit 2 high | `npm audit --omit=dev` | package.json | Known CVEs in prod deps | `npm audit fix`, review | S | DevOps |
| Branch chaos | `fix/mobile-nav-rebuild` dirty, unmerged; intake fixes possibly lost in merge `aa0b952` | git | Work loss recurring | Merge discipline; PR workflow | S | DevOps |
| Webhook returns 200 on handler errors | catch block comment "still return 200" | `app/api/stripe/webhook/route.ts:59-62` | Failed upgrades never retried by Stripe | Return 500 on retryable DB errors | S | backend |

## 🟡 Medium

| Issue | Evidence | Files | Impact | Fix | Effort | Owner |
|---|---|---|---|---|---|---|
| Intake username random per device | localStorage `trader-abc123` | `src/pages/MyLinkPage.tsx` | Intake links unstable | Derive from account | M | backend |
| No phone validation on intake | step 4 accepts blank | `src/pages/IntakePage.tsx` | Weak leads | Soft inline warning | S | frontend |
| No conversion events | only pageviews | `@vercel/analytics` | Can't measure funnel | Track scan/signup/checkout events | S | growth |
| No cancel button | portal route Express-only | `server/routes/customerPortal.ts` | Churn friction → refunds/disputes | Stripe portal link on /account | S | backend |
| No "what happens after payment" copy | pricing page | `src/pages/PricingPage.tsx` | Purchase anxiety | Add Day-1 expectations block | S | copy |
| Express/Vite/router corpses | server/app.ts, vite.config.ts, index.html, pages/api proxy | root | Confusion, accidental edits to dead code | Delete in cleanup PR | M | backend |

## 🟢 Low
- Root debris: fix-*.cjs ×8, logs, zips, tsbuildinfo tracked — delete/gitignore.
- package.json name `"react-example"`.
- `LeadDecision.tier` type includes `BIN` but no `BRONZE` — decide tier taxonomy.
- Embedded Obsidian_Memory vault in repo — migrate to claude-obsidian and remove.
- Signals page lacks end-of-page upgrade CTA.
