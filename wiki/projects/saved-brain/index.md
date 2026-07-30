---
title: Saved Brain — Audit & Build Prompt
project: saved-brain
type: project-index
created: 2026-06-09
tags: [saved-brain, audit, nextjs, build-prompt]
---

# Saved Brain

Browser extension scrapes Instagram/Twitter saves → AI-enriched, searchable knowledge base. Boards, knowledge graph, search, LemonSqueezy license/checkout, digest emails. Next.js 14 app-router + Chrome extension.

- **Repo:** https://github.com/manazoid4/saved-brain
- **Local:** `C:\Users\manaz\saved-brain`
- **Deploy:** Vercel (preview behind deploy-protection)

## Audit — 2026-06-09 (4 parallel agents, read-only)

Three root failures cascade into everything:
1. **No auth / no user concept** — every API route fully public.
2. **Flat JSON file "DB"** (`lib/db.ts`) — does not persist on Vercel serverless; webhook purchases vanish.
3. **No entitlement gating** — all Pro features free; checkout works but sells nothing.

Plus stub features sold as real: semantic search == fulltext, embeddings dead code (random vectors), cron unwired, sync/graph subsystems don't exist.

## Docs
- [[build-prompt]] — phased production-hardening build prompt (the deliverable)
- [[audit-security]] — security findings (CRITICAL→LOW)
- [[audit-ux]] — UX / product gaps
- [[audit-monetization]] — monetization / GTM gaps
- [[audit-architecture]] — architecture / tech-debt

## Status
Audit only. Nothing implemented. Phases 1–4 non-negotiable before any public launch or sale.
