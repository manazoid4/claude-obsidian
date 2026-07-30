---
title: Saved Brain — Security Audit
project: saved-brain
type: audit
domain: security
created: 2026-06-09
tags: [saved-brain, audit, security]
---

# Security Audit — saved-brain (Next.js + browser extension)

Back to [[index]] · feeds [[build-prompt]]

The app has **no authentication or authorization layer of any kind**. No `middleware.ts`, no session, no user model (`owner_id` always `null` in `lib/db.ts`), no API key check on any route. Every API route is fully public. The "database" is a flat JSON file (`data/saved-brain.json`) loaded into a process-global singleton. Net: the entire deployment is a single shared, unauthenticated data store.

## CRITICAL

1. **All API routes lack authn/authz — full public read/write/delete.** `app/api/settings`, `items/**`, `ingest`, `boards/**`, `export/obsidian`, `digest`, `enrich`, `search`. None check identity. Anyone can read all items, delete any item (`items/[id]/route.ts:39`), wipe the whole DB (`lib/db.ts:427-438` — DELETE with no id deletes ALL items/enrichments/boards/edges), clone boards, trigger paid LLM calls. Fix: auth gate in `middleware.ts` on `/api/*` + per-user ownership checks.

2. **`GET /api/settings` leaks all secrets and PII unauthenticated.** `settings/route.ts:4-7` returns `getAllSettings()` incl. `llm_api_key` and every `purchase_<email>` record (webhook `lemonsqueezy/route.ts:49-60`) with customer emails, order numbers, license status. Any visitor harvests the API key + customer list. Fix: never return secret-class settings; gate; segregate secrets.

3. **API keys stored plaintext despite "encrypted at rest" claim.** `lib/settings.ts:9-16` writes `llm_api_key` straight to JSON; `lib/db.ts:403-408/141` persists unencrypted. `KEY_ENCRYPTION_SECRET` declared in `.env.example:3`, documented, health-checked — but **no encryption code exists**. Settings UI says "Stored locally, encrypted at rest" (`app/settings/page.tsx:168`) — false. Fix: AES-GCM with `KEY_ENCRYPTION_SECRET`, or remove the claim.

## HIGH

4. **No license verification — Pro trivially forgeable.** Purchases tracked only as `purchase_<email>` settings key. Since `POST /api/settings` is open and writes arbitrary keys (`settings/route.ts:9-14`), anyone POSTs `{"purchase_attacker@x.com":"{\"status\":\"active\"}"}` to mint a license. `jose` dependency unused — no signed licenses. Fix: verify entitlements server-side vs LemonSqueezy; never trust client-writable settings for authz.

5. **SSRF via user-controlled `ollama_url` / provider URLs.** `lib/ai/providers.ts:51-52`, `lib/ai/embed.ts:47-51` fetch `getSetting('ollama_url')` server-side. Open settings POST -> attacker sets `ollama_url` to `http://169.254.169.254/...` or internal host, triggers `/api/enrich`. Fix: allowlist scheme+host, block private/link-local, or pin Ollama config.

6. **Webhook signature non-constant-time compare.** `lemonsqueezy/route.ts:22` `signature !== digest` plain compare = HMAC timing side-channel; no length/format check. Fix: `crypto.timingSafeEqual` on equal-length buffers.

7. **Flat-file DB unsafe for serverless.** `lib/db.ts` holds whole DB in module global `_db`; `saveDB()` rewrites entire file per mutation (`:141`). Vercel filesystem ephemeral/per-instance: writes lost/inconsistent across invocations; concurrent writes race, no locking. Data-integrity + availability issue. Fix: real DB (`DATABASE_URL` defined, unused).

8. **No rate limiting on expensive/abusable endpoints.** `/api/enrich`, `/api/digest` spend money (LLM/Resend); `/api/ingest` accepts unbounded arrays (`ingest/route.ts:8`, no cap) writing to flat file. Unauth + unthrottled = cost-amplification / DoS. Fix: rate limit + size limits + auth.

## MEDIUM

9. **Email/HTML injection in digest emails.** `digest/route.ts:88-90` interpolates `item.url/title/summary/author` into HTML email unescaped. Scraped/attacker content becomes raw HTML/links in inbox; `item.url` into `href` unescaped (attribute breakout / `javascript:`). Fix: HTML-escape + validate URL scheme.

10. **Extension backend URL trust / messaging.** `background.ts:68` POSTs scraped data to user-set `config.backendUrl`, no scheme validation (could be `http://` cleartext). `onMessage` (`:21-40`) accepts `scrape-complete` from any content-script context, no origin check. API key in `chrome.storage` plaintext (`popup.js:9-25`). Enforce https; validate senders.

11. **`mode` param ignored in search.** `search/route.ts`: `semantic` and `fulltext` branches identical (`:14-37`) — "semantic search" silently does nothing. Correctness gap. (The fake SQL layer parses by uppercased substring match — fragile; `ON CONFLICT` detection at `:165` depends on substrings.)

12. **`next.config.mjs` images from any host** (`hostname:'**'`, `:8-9`); `PostCard.tsx:83` renders scraped `thumbnailUrl` into `<img>`. No stored-XSS (React escapes; no `dangerouslySetInnerHTML`), but open allowlist enables arbitrary outbound image requests / tracking-pixel injection. Fix: restrict `remotePatterns`.

## LOW

13. `/api/sync/cron` + `/api/digest` GET have no `CRON_SECRET` guard. Currently placeholder/no `crons` configured, but would be public trigger if wired. Add `Authorization`/`CRON_SECRET`.
14. Verbose error leakage — `err.message` to client (`enrich:90`, `digest:122`, `items:103`). Prefer generic + server log.
15. `X-XSS-Protection: 1; mode=block` (`vercel.json:25`) deprecated; prefer a CSP (none set anywhere).

## Notes
No `auth`, `oauth`, `license`, `github`, `parse-export` routes exist (were in brief, not in code). OAuth client IDs/secrets in `.env.example` (YouTube/Reddit) unused. `jose` is a dependency but **not imported anywhere** — no JWT issuance/verification — the core weakness behind #1/#4.

Key files: `lib/db.ts`, `lib/settings.ts`, `app/api/settings/route.ts`, `app/api/webhooks/lemonsqueezy/route.ts`, `lib/ai/providers.ts`, `lib/ai/embed.ts`, `app/api/digest/route.ts`, `extension/background.ts`.
