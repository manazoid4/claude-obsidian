---
title: Saved Brain — Architecture / Tech-Debt Audit
project: saved-brain
type: audit
domain: architecture
created: 2026-06-09
tags: [saved-brain, audit, architecture, tech-debt]
---

# Architecture & Tech-Debt Audit — Saved Brain

Back to [[index]] · feeds [[build-prompt]]

Next.js 14 app-router, flat-JSON "database", multi-provider AI, deployed to Vercel (`iad1`).

## Verdict
Functional demo/local-first prototype that **cannot run correctly on Vercel as architected**. Data layer assumes a writable, persistent, single-process filesystem — none hold on serverless. Several headline features (semantic search, sync, embeddings) are stubs. Pre-production.

## P0 — Will break or lose data in production

**1. Flat JSON file as DB on serverless (dominant risk).** `lib/db.ts` reads/writes `data/saved-brain.json` via `fs.writeFileSync`, whole DB cached in module-global `_db`.
- Ephemeral FS: Vercel lambda FS read-only except `/tmp`; `/tmp` wiped per-invocation, not shared across instances. Every write (ingest/enrich/settings/boards/webhook purchases) rejected (EROFS) or silently lost on recycle. Bundled `data/saved-brain.json` at `process.cwd()` is read-only -> writes throw.
- No concurrency control: `saveDB()` serializes entire object + overwrites. Two concurrent requests = lost-update / corruption. Module-global cache => instances diverge.
- Full-file rewrite per mutation: O(n) write of entire dataset per insert. Won't scale past a few MB.
- `transaction()` is fake: runs fn + `saveDB()`, no atomicity/rollback.

*Fix:* managed DB before any deploy. Schema already relational -> Vercel Postgres / Neon / Supabase (pgvector) or Turso/libSQL. Delete the shim, don't port.

**2. The DB layer is a SQL-string-sniffing emulator.** `getDb()` parses SQL by `upper.includes('SAVED_ITEMS')`, `includes('COUNT')`, regex `LIMIT (\d+)` to route to hand-written JS handlers. Brittle:
- Any query whose text doesn't match expected substrings silently returns `[]`/`undefined`/`{changes:0}`. Failures invisible — no error, just empty results.
- WHERE/LIMIT/OFFSET/ORDER BY partially reimplemented by regex; param positions guessed (`numericParams.filter(typeof === 'number')`). Easy silent mis-bind.
- `handleInsertSavedItem` dedups on `url` in one branch but stores by `id` — inconsistent identity.
Largest source of latent correctness bugs; must go with the DB migration.

**3. Semantic search doesn't exist; embeddings dead code.** `app/api/search/route.ts` has identical `LIKE` for `semantic` and fulltext (comment: "vector search needs sqlite-vss setup"). Nothing reads the `embeddings` table; `embedText` never invoked from any route (grep confirms `embed.ts` unreferenced). `embedFallback` returns **random vectors**. Embedding/vector-search entirely non-functional.

**4. Vercel cron not wired; sync engine is placeholder.** `app/api/sync/cron/route.ts` returns static "ready". `vercel.json` has **no `crons` array** — placeholder never invoked on schedule. No `lib/sync/`, `lib/sources/`, `lib/graph/` dirs at all (assumed in brief; don't exist). Sync, source connectors, graph builder unimplemented; `app/api/graph/route.ts` + enrich graph edges have no producer.

## P1 — Security & correctness

**5. No auth on any mutating endpoint except webhook.** Grep shows only `webhooks/lemonsqueezy` checks a signature.
- `POST /api/settings` writes arbitrary keys/values incl. `llm_api_key`, `ollama_url`. Anyone overwrites stored key, or reads it back if any path echoes settings.
- `POST /api/enrich` + `/api/ingest` unauthenticated, trigger paid LLM calls -> financial DoS.
- Cron route has no `CRON_SECRET` check; publicly triggerable once scheduled.

**6. Secrets plaintext in the JSON "DB".** `llm_api_key` persisted via `setSetting` into `saved-brain.json`. `health-check.ts` references `KEY_ENCRYPTION_SECRET` as required (encryption intended), but `settings.ts` stores raw. Keys sit plaintext in a file that may be committed.

**7. SSRF via configurable `ollama_url`.** `callOllama`/`embedOllama` fetch `${ollama_url}/api/...` where `ollama_url` is attacker-settable through open settings endpoint. With #5, server-side request forgery.

**8. Unvalidated LLM/provider response shapes.** Every provider call indexes blindly: `data.content[0].text`, `data.candidates[0].content.parts[0].text`, `data.choices[0].message.content`. On error/rate-limit body these throw `TypeError`. `enrichItem` guards JSON.parse; provider layer doesn't guard the envelope.

## P2 — Type safety, error handling, tooling

9. **"Lint" is type-check only.** `"lint": "tsc --noEmit"`; eslint/eslint-config-next installed but no script runs them. No actual linting in CI.
10. **Type safety undermined at DB boundary.** `strict: true` set, but every DB read is `as Record<string, unknown>` then cast field-by-field. `SavedItemRow` vs camelCase `SavedItem` mapped by hand per route, no shared mapper — drift-prone. `params as [string,...]` casts assume positional correctness, no runtime check.
11. **Zod installed but request bodies unvalidated.** `await request.json()` consumed directly in ingest/search/settings/enrich. `zod@3.23.8` unused.
12. **`swcMinify: false`** in `next.config.mjs` — disables minification, larger/slower bundles, no stated reason.
13. **`images.remotePatterns` hostname `'**'`** — Next image optimizer proxies any https host. Image-proxy abuse / SSRF-lite.
14. **Inconsistent error handling.** `ingest` swallows per-item errors into `skipped` counter (silent data loss, no log); search/settings have no try/catch (500 + stack on malformed JSON); enrich is the only structured catch.
15. **Zero automated tests.** No `*.test.ts`/`*.spec.ts` anywhere. Only `scripts/health-check.ts` (tsc + next build + route-file count) — no behavioral coverage. The fragile SQL-emulator is exactly what needs tests.

## Remediation priority
1. Replace JSON store with managed Postgres (+pgvector) or libSQL/Turso; delete `getDb()` emulator. Resolves #1, #2; unblocks real semantic search (#3).
2. Auth/secret protection: gate all mutating routes; `CRON_SECRET` on cron; move `llm_api_key` to env/encrypted; allowlist `POST /api/settings` keys (#5, #6, #7).
3. Implement or remove stubs (sync, graph, semantic search) so surface matches reality; wire `vercel.json` crons only once handler does something.
4. Harden provider layer: validate envelopes, timeouts/retries (#8).
5. Zod at every boundary; real eslint script; re-enable swcMinify; tighten `remotePatterns` (#9–#13).
6. Tests, starting with new data layer + provider abstraction (#15).

Notable files: `lib/db.ts` (#1,#2); `app/api/search/route.ts` (#3); `lib/ai/embed.ts` (#3); `app/api/sync/cron/route.ts` + `vercel.json` (#4); `app/api/settings/route.ts` + `lib/settings.ts` (#5,#6,#7); `lib/ai/providers.ts` (#8).

Note: `lib/sources/`, `lib/sync/`, `lib/graph/` and a real data storage layer **do not exist** — those subsystems unimplemented, itself a finding.
