---
title: Saved Brain — Monetization / GTM Audit
project: saved-brain
type: audit
domain: monetization
created: 2026-06-09
tags: [saved-brain, audit, monetization, gtm]
---

# Monetization + GTM Audit — Saved Brain

Back to [[index]] · feeds [[build-prompt]]

**Verdict: checkout is real, but there is no product to sell.** Payment plumbing works end-to-end, yet zero license enforcement — every "Pro" feature is fully usable on free tier. Nothing gates revenue because nothing is gated. Combined with a serverless persistence bug that loses purchase records, the app currently cannot reliably monetize.

## P0 — Blocks revenue entirely

1. **No free-vs-paid gating anywhere.** Grep for `license|isPro|premium|requirePro|hasAccess` returns only pricing page, checkout, webhook, Sidebar "Upgrade to Pro" link. Item-create (`app/api/items/route.ts:75`), board-create (`app/api/boards/route.ts:39`), search, enrich, graph, export — no entitlement check. Advertised "50 items / 1 board" free limit + all 10 Pro features unenforced. User gets everything for £0. No reason to pay.

2. **`/api/license/verify` + `marketing/` referenced but don't exist.** `app/api/license/` is an empty dir; `marketing/` empty. No license verification endpoint — even client-side gating couldn't validate a purchase server-side.

3. **Purchase records written to ephemeral JSON that won't persist on Vercel.** Webhook (`app/api/webhooks/lemonsqueezy/route.ts:52`) records via `INSERT ... purchase_${email}` into `lib/db.ts` — a flat JSON file at `data/saved-brain.json` (despite SQLite-flavored API + "SQLite" marketing). Vercel serverless FS read-only/ephemeral — webhook writes vanish between invocations. Purchases lost. No LemonSqueezy **license-key** issuance/validation; `license_key_created` logged + ignored.

4. **"Local-first SQLite, data never leaves your machine" is architecturally incompatible with the Vercel SaaS being sold.** Pricing/README/marketing claim local SQLite + privacy, but deployed app is a single shared serverless instance with one JSON file. Either hosted multi-tenant (then no per-user isolation or auth — every visitor shares one library) or local download (then hosted checkout + email-keyed purchases make no sense). Must resolve before any legitimate sale.

## P1 — Major GTM gaps

5. **No authentication / no concept of a user.** No login, accounts, session. Purchases keyed by webhook email, but app can't know *who* the current visitor is — could never map payment to session even if persistence worked. Root blocker beneath #1, #3.

6. **Activation/retention hooks are stubs.** Weekly digest (`app/api/digest/route.ts`) only sends when manually POSTed with email; **no scheduler**. README claims "cron every 6 hours" but `vercel.json` has **no `crons` key**. `app/api/sync/cron/route.ts` is an explicit placeholder. So no automated digests, no automated sync — primary retention loop doesn't run.

7. **Zero referral / virality.** No referral/invite/affiliate code anywhere. Shareable public boards exist (`/b/[slug]`, OG images, clone/lineage) — the one organic-growth surface — but not tied to acquisition (no "made with Saved Brain" attribution, no signup prompt on shared boards). LemonSqueezy affiliate program unused.

8. **Checkout discount hardcoded on.** `app/api/checkout/route.ts:327` sets `discount_enabled: true` permanently — invites every buyer to hunt a code, erodes £49 price. No coupon strategy / launch-window logic.

## P2 — Positioning & launch readiness

9. **Pricing unanchored, comparison invisible.** £49 lifetime with struck-through "£9.99/mo equivalent" — no comparison table vs named competitors. Against Raindrop (free generous, $3/mo), mymind (~$8/mo), Readwise ($8/mo), Notion Web Clipper (free), the wedge is *cross-platform social-save ingestion + BYO-key AI + knowledge graph + lifetime price*. Lives only in `docs/MARKETING.md`, not on pricing page. "BYO LLM key = zero cost" story is strong + underused.

10. **"Join thousands" social proof fabricated** (pricing final CTA), no testimonials/logos/numbers. Refund/trust risk + credibility hit at launch.

11. **Launch collateral thorough but full of placeholders.** `docs/launch/` (Product Hunt, HN, Reddit, IH, Twitter, IG, press kit) + `docs/MARKETING.md` well-developed but contain `[your-domain.com]`, `[@yourhandle]`, `[your-email]`. No custom domain, no privacy/ToS (LAUNCH-CHECKLIST flags unchecked), no analytics (`PLAUSIBLE_DOMAIN` empty). Not launch-ready.

## Recommendations (priority order)
1. **Decide model first** (hosted SaaS vs local download). Hosted: add auth (Clerk/Auth.js), per-user data, real DB (Postgres/Turso). Local: sell LemonSqueezy license key, enforce offline, drop hosted email checkout.
2. **Build entitlement layer**: `getEntitlement(user)` in items POST (50-item cap), boards POST (1-board cap), enrich/semantic/graph/export. Implement `/api/license/verify`. Without this, fix nothing else.
3. **Make persistence real** so webhook purchases survive (hosted DB); issue + validate LemonSqueezy license keys instead of email-keyed settings rows.
4. **Turn on retention**: `crons` entry in `vercel.json` for weekly `/api/digest`; real sync replacing the stub. Capture email at onboarding (none today).
5. **Add virality loop**: enable LemonSqueezy affiliates; "powered by Saved Brain -> start your own" CTA on public board pages.
6. **Tighten offer page**: competitor comparison table; remove fabricated "thousands"; scope discount to launch window.
7. **Finish launch prereqs**: domain, privacy/ToS, analytics, fill placeholders.

Key files: `app/api/items/route.ts`, `app/api/boards/route.ts` (no gating); `lib/db.ts` (ephemeral) + `app/api/webhooks/lemonsqueezy/route.ts`; missing `/api/license/verify`; `app/api/digest/route.ts` + no `crons`; `app/api/sync/cron/route.ts`; `app/api/checkout/route.ts:327`.
