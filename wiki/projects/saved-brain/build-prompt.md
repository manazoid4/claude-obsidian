---
title: Saved Brain — Production Hardening Build Prompt
project: saved-brain
type: build-prompt
created: 2026-06-09
tags: [saved-brain, build-prompt, hardening]
---

# BUILD PROMPT — Saved Brain: Production Hardening

Repo: C:\Users\manaz\saved-brain (Next.js 14 app-router + Chrome extension)
Source of truth: 4-agent audit (security / UX / monetization / architecture), 2026-06-09.
See [[audit-security]], [[audit-ux]], [[audit-monetization]], [[audit-architecture]].

Single-line problem: payment plumbing works, but there is no real product behind it —
no auth, no persistent DB, no entitlement enforcement, and several headline
features are stubs.

## GATE 0 — Decide the model FIRST (blocks everything)
Current code is self-contradictory: marketing says "local-first SQLite, data never
leaves your machine," but it's deployed as ONE shared Vercel instance with ONE JSON
file = every visitor shares one library, no isolation.

Pick ONE:
  A) HOSTED MULTI-TENANT SaaS  -> add real auth + per-user DB + entitlements.
  B) LOCAL DOWNLOAD app        -> sell a LemonSqueezy license KEY, enforce offline,
                                  drop hosted email-keyed checkout.
Everything below assumes (A). If (B), the DB/auth phases shrink but license-key
issuance + offline validation replace them.

## PHASE 1 — Data layer (P0, do before anything else)
- Replace flat JSON store (lib/db.ts) with managed Postgres + pgvector
  (Neon/Supabase/Vercel Postgres) or Turso/libSQL. DATABASE_URL already declared, unused.
- DELETE the SQL-string-sniffing emulator in lib/db.ts (routes queries by
  `upper.includes('SAVED_ITEMS')` — silently returns [] on any unmatched query).
  Do not port it.
- Add `owner_id` to every table; it is currently always null.
- Replace fake `transaction()` (no atomicity) with real DB transactions.
- Add a single shared row->domain mapper (kill per-route `as Record<string,unknown>` casts).

## PHASE 2 — Auth + per-user isolation (P0)
- Add auth (Clerk or Auth.js). No login/session/user exists today.
- Add middleware.ts matching /api/* — gate every mutating route.
- Scope all reads/writes by owner_id.
- Capture email at onboarding (none captured today) so purchases can map to a user.
- Extension: replace raw "Backend URL + optional API key" popup with a real
  connect-account / issued-token flow (extension/background.ts:68, popup.js).

## PHASE 3 — Entitlements / monetization (P0 — without this nothing sells)
- Build getEntitlement(user). Enforce in:
    app/api/items/route.ts   -> 50-item free cap
    app/api/boards/route.ts  -> 1-board free cap
    enrich / search(semantic) / graph / export -> Pro-gate
  (grep confirms NO license/isPro/requirePro check exists anywhere except pricing UI.)
- Implement the missing /api/license/verify (dir is empty).
- Issue + validate LemonSqueezy LICENSE KEYS server-side. Stop trusting
  client-writable `purchase_<email>` settings rows — POST /api/settings is open,
  so anyone can mint themselves Pro today.
- Move purchase records into the real DB (webhook writes currently lost on Vercel).

## PHASE 4 — Security hardening (P0/P1)
- CRITICAL: GET /api/settings returns ALL settings incl. llm_api_key + every
  customer email/purchase. Never expose secret-class settings over API.
- CRITICAL: actually encrypt secrets with KEY_ENCRYPTION_SECRET (AES-GCM).
  Code claims "encrypted at rest" (settings UI), but stores plaintext. Either
  encrypt or delete the claim.
- Lock POST /api/settings to an allowlist of non-secret keys.
- SSRF: validate/allowlist ollama_url + provider URLs (block private/link-local/
  169.254.169.254) — lib/ai/providers.ts:51, lib/ai/embed.ts:47.
- Webhook: use crypto.timingSafeEqual, not `!==` (webhooks/lemonsqueezy:22).
- Add rate limiting + request-size caps on /api/enrich, /api/digest, /api/ingest
  (unauth + unbounded = cost-amplification DoS).
- HTML-escape all digest-email interpolation + restrict href to http/https
  (app/api/digest:88) — scraped content is attacker-controlled.
- Add CRON_SECRET guard to /api/sync/cron before wiring it.
- Tighten next.config images.remotePatterns from `'**'` to known CDNs.
- Add Zod validation at every route boundary (zod installed, unused).

## PHASE 5 — Make stub features real OR remove them (P1)
Marketed-but-fake (pick: implement or cut from pricing/marketing):
- Semantic search: identical to fulltext today; embeddings table never read;
  embedFallback returns RANDOM vectors. Wire pgvector -> real cosine search.
- Vercel cron: NO `crons` key in vercel.json despite README "every 6 hours."
  Add crons entry to drive /api/digest weekly; replace /api/sync/cron stub.
- Sync engine / source connectors / graph builder: lib/sync, lib/sources,
  lib/graph DO NOT EXIST. Either build or stop advertising auto-sync.

## PHASE 6 — UX / first-run polish (P1/P2)
- Split marketing vs app layouts (route groups) — Sidebar currently bleeds onto
  landing/onboarding/pricing for logged-out visitors (app/layout.tsx).
- Onboarding import tiles are <a href> that navigate BEFORE settings save ->
  config silently lost unless "Finish Setup" clicked (app/onboarding/page.tsx).
- Enforce onboarding_complete (written, read nowhere). Gate /library -> /onboarding.
- Library drops `mode` and never passes facet arrays -> Semantic toggle does
  nothing, Filters panel is empty (app/library/page.tsx, components/SearchBar.tsx).
- Add "add to board" path (PostCard.onAddToBoard orphaned) — boards un-fillable today.
- Add item detail view (no route to open a single item).
- Add /privacy + /terms (footer 404s; pricing makes refund claims with no terms).
- Format raw ISO timestamps -> relative dates (PostCard).
- Distinct "no search results" empty state (currently says "library empty").
- Graph canvas: responsive sizing + stop infinite force-sim + a11y alternative.
- Remove fabricated "Join thousands" social proof until real.

## PHASE 7 — GTM / launch prereqs (P2)
- Competitor comparison table on pricing (Raindrop/mymind/Readwise/Notion);
  lead with the real wedge: cross-platform social-save ingest + BYO-key AI (zero
  marginal cost) + knowledge graph + lifetime price.
- Scope checkout discount to a launch window (discount_enabled hardcoded true,
  checkout/route.ts:327).
- Enable LemonSqueezy affiliates; add "powered by Saved Brain" CTA on public /b/[slug]
  boards (only organic-growth surface).
- Fill launch-kit placeholders ([your-domain], [@handle], [email]); set custom
  domain, PLAUSIBLE_DOMAIN analytics, privacy/ToS before posting.

## PHASE 8 — Quality baseline (P2)
- Add real eslint script (lint is `tsc --noEmit` only; eslint installed, never run).
- Re-enable swcMinify (currently false).
- Harden provider layer: validate response envelopes (blind data.content[0].text
  throws on error bodies), add timeouts/retries — lib/ai/providers.ts.
- Add tests — zero exist. Start with new data layer + provider abstraction +
  entitlement checks.

## Recommended order
GATE 0 -> Phase 1 -> 2 -> 3 -> 4 -> (5 implement-or-cut) -> 6 -> 7 -> 8.
Phases 1–4 are non-negotiable before any public launch or sale.
