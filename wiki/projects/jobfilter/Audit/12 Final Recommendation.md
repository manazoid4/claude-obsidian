---
type: audit
project: jobfilter
date: 2026-06-11
---

# 12 Final Recommendation

## 1. What JobFilter currently is
A well-designed, type-clean Next.js app with ~100 pages, a genuine multi-source UK lead engine (planning/contracts/EPC/Companies House), Stripe wiring, and a distinctive brutalist brand — whose **delivery layer (the actual product) doesn't run end-to-end**.

## 2. Actually working
Build/tsc green. Scan with live public sources + fallbacks. Scoring (fragmented but functional). Lead detail with WhatsApp deep-links + calendar. Pricing + checkout session creation. Webhook with signature verification. Free tools. Mobile nav (current branch). Design system.

## 3. Fake / mocked / incomplete
Intake → nothing persisted, no tradesperson link, WhatsApp hardcoded off. No automated GOLD alerts (no Twilio config anywhere). Dashboard/ROI data = localStorage theatre. n8n workflows authored, activation unverified. Fallback sample leads can masquerade as real. Prod Supabase/Vercel setup incomplete.

## 4. Ready to sell?
**No — but close.** Honest gap: 2–4 focused days of fixes + founder's manual setup (Twilio, Stripe dashboard, migrations, Vercel envs). Concierge-selling is possible THIS WEEK without code.

## 5. Build next (in order)
1. Run **Prompt 1** (launch blockers) from [[10 Agent Build Prompts]].
2. Founder completes [[STICKY-TODO]] (Twilio, Stripe, migrations, envs).
3. Prompt 5 (WhatsApp auto-delivery) + Prompt 3 (onboarding capture).
4. Prompt 6 (server-backed leads).

## 6. Ignore for now
SSR/SEO migration (post-revenue), Express/Vite cleanup beyond deletion, Vantage/Codex sub-brands, more free tools/cities/VS pages, annual billing, referral schemes.

## 7. Best path to first paying users
Concierge beta: this week, founder manually runs scans for 3–5 known trades, sends best leads via personal WhatsApp daily, asks for £39 once they see one good lead. Product catches up behind the promise. Code exists to support this TODAY (FindJobs scan + manual WhatsApp route once Twilio set).

## 8. Recommended next agent prompt
**Prompt 1 — Fix launch blockers** ([[10 Agent Build Prompts]]). It unblocks everything else and starts with recovering the lost intake fixes from git history.

## 9. 7-day execution plan
| Day | Action |
|---|---|
| 1 | Run Prompt 1 (blockers). Founder: Twilio sandbox + Supabase service key + run 20260531 migrations |
| 2 | Prompt 5 (auto WhatsApp). Founder: Stripe product + webhook + Vercel envs. E2E test: intake → WhatsApp |
| 3 | Merge to main via PR, deploy, verify prod. Generate sample lead PDF from real Manchester scan |
| 4 | Outreach: 30 DMs in trade Facebook groups + personal contacts with PDF. Start concierge sends |
| 5 | Prompt 3 (onboarding capture) + Prompt 6 start (server-backed leads) |
| 6 | Onboard first 3 beta trades properly (account, trade, area). Daily digest live |
| 7 | Ask for money: founder offer £39 locked. Target: 1–3 paying. Collect quotes for case study |
