---
type: audit
project: jobfilter
date: 2026-06-11
---

# 05 Design System Audit

See also vault concept note: [[JobFilter Design System]].

## Current state: Brutalist-Yellow — KEEP IT
White/black/yellow (#facc15), `border-2`, hard shadows `shadow-[3px_3px_0_var(--line)]`, ALL-CAPS micro-labels, `jf-button`/`jf-box`/`ops-panel` classes. Per 2026-06-02 audit: applied consistently, high coherence. This is already distinctive, trade-appropriate, not generic AI SaaS. **Do not redesign. Tighten.**

| Element | Verdict | Note |
|---|---|---|
| Typography | 🟡 | Bold caps work; body text sometimes dense — increase line-height on long copy |
| Spacing | ✅ | `page-shell` consistent |
| Palette | ✅ | Yellow/black/white + intentional `--orange`/`--navy` accents |
| Buttons | ✅ | Strong; keep 44px min targets |
| Cards | ✅ | LeadCard good; add urgency + source + "why matched" consistently |
| Forms | 🟡 | Intake form fine; phone field needs inline validation message |
| Tables | ✅ | Signal table strong homepage proof |
| Badges | ✅ | ScoreBadge solid |
| Mobile | ✅ | Current branch literally rebuilds mobile nav — verify + merge |
| Hierarchy | ✅ | Hero → proof → CTA logical |
| GOLD treatment | 🟡 | GOLD needs to feel scarcer/heavier — see below |

## Improvements (not redesign)

- **GOLD badge**: black card, yellow thick border, yellow "★ GOLD" tag, subtle pulse on arrival. GOLD cards get 2px heavier border + hard shadow than SILVER. Never more than ~1 in 5 leads GOLD or it cheapens.
- **Lead score component**: score number XL (text-4xl), tier word under it, one-line reason ("EPC expired + planning approved 12 days ago"). Reason line is the trust-builder.
- **Lead card fields**: score, work type, area (outward postcode), est. value, urgency, source badge, freshness, WhatsApp button. Nothing else.
- **Empty state**: "No GOLD yet today. We scan every morning at 7am. You'll get WhatsApp the second one lands." — turns absence into proof of vigilance.
- **Landing section order**: Offer bar (founder price) → Hero + postcode scan → 3-step how-it-works → live signal table → GOLD sample card (redacted) → trades grid → pricing → guarantee → FAQ.
- **Copy swaps**: "Get started" → "GET LOCAL LEADS". "Insights" → "jobs". "Platform" → "radar". Keep "Find building work before your competitors" as the spine.

## Impeccable Design Rules for JobFilter
1. Every page must answer: "How does this get me more work?"
2. GOLD must visually feel valuable — heavier border, black/yellow inversion, scarcity.
3. Never show a dashboard before showing value (a lead).
4. No vague AI language. Replace "insights/intelligence" with jobs, leads, areas, alerts.
5. CTAs name the outcome: "GET LOCAL LEADS", "SEE JOBS IN [POSTCODE]".
6. Every lead shows its source and age. Trades smell fake data instantly.
7. One yellow CTA per viewport. Yellow = action, nothing else.
8. Numbers over adjectives: "£4,200 avg job value" not "high-value leads".
9. Mobile thumb-first: primary action bottom-reachable, 44px min.
10. Trade language, no cosplay: "jobs" not "opportunities"; never fake slang.
