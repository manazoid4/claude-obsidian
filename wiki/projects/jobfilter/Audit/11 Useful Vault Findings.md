---
type: audit
project: jobfilter
date: 2026-06-11
---

# 11 Useful Vault Findings

Searched: `wiki/projects/jobfilter/`, `wiki/concepts/JobFilter *`, repo-embedded docs (`PROJECT_STATUS.md`, `TODO.md`, `AUDIT-REPORT.md`, `JOBFILTER-APPLE-LEVEL-AUDIT-REPORT.md`, `agents/codex-*-2026-06-06.md`, `COMPETITOR_STRATEGY_PLAYBOOK.md`).

## Most valuable finds
1. **[[2026-06-02-2325-full-audit|2026-06-02 full audit]]** — best prior work. Root-caused the intake engine (4 compounding issues) and claims fixes applied. **Critical: those fixes are NOT in the current branch** — recover via `git log --all -- app/api/intake/score/route.ts`. Its 8 build prompts (auth-gate /test, phone validation, account-tied username, intake_submissions DDL, Stripe setup, BRONZE tier, paid-gating intake) are still valid — superseded versions folded into [[10 Agent Build Prompts]].
2. **[[STICKY-TODO]]** — founder-manual action list (Twilio env, Supabase tables, Stripe product, service-role key). Still accurate; nothing on it appears done. This is THE unblock list.
3. **Repo `TODO.md` (2026-05-31)** — Vercel env var list + migration instructions. Matches STICKY-TODO; confirms prod setup never completed.
4. **Apple-level audit (2026-05-24)** — Frankenstein architecture diagnosis still ~80% true (Express/Vite corpses remain; pages now App-Router-wrapped but still client-only). SEO warning remains fully valid.
5. **BuildScout competitive analysis + COMPETITOR_STRATEGY_PLAYBOOK** — positioning gold: signals-before-marketplace angle, VS-page strategy (already shipped ×6).
6. **Concept notes** ([[JobFilter Hub]], [[JobFilter Product Features]], [[JobFilter Onboarding Stages]], [[JobFilter Design System]], [[Intake Engine]], [[Vantage]], [[Vicinity]], [[Codex]]) — product vocabulary and feature intent; design system note matches what's implemented.

## Ideas worth reviving
- Onboarding stages concept (vault) — maps directly to the missing trade/postcode capture.
- Area exclusivity / territory scarcity (Vicinity + territories page) — strongest premium-pricing idea, currently only cosmetic.
- 2026-04 conversion phrases audit ("NO CONTRACTS", "REAL LEADS" etc.) — partially shipped; re-check coverage on current homepage.

## Ideas to ignore for now
- Vantage/Codex sub-brands — three sub-products before one paying user = diffusion.
- More free tool packs (acm/nasc/ozev/swmp pages) — already over-built; no more.
- Firebase-era anything (old workflows, applet config) — dead platform, Vercel is current.

## Contradictions found
- 06-02 audit says intake fixed → code says not fixed (branch/merge loss). Trust git, not notes.
- PROJECT_STATUS.md claims "all working" for an older nav/demo structure that no longer matches current routes — historical only.
- GOLD defined as 80+ in marketing/vault, 90 in `leadEngine/scorer.ts`.

## Strongest positioning line in vault
"Find building work before your competitors" + "JobFilter sells control over better work" (repo CLAUDE.md). Keep as spine.

## Best next prompt found in vault
06-02 audit Prompt 3 (tie intake username to account) — folded into [[10 Agent Build Prompts]] Prompt 1/5.
