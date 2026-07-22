---
type: audit
project: jobfilter
date: 2026-06-11
---

# 04 Product UX Audit

Journey: visitor → understands → trusts → pricing → signup → trade/area → pays → leads → WhatsApp GOLD → returns → stays.

| Step | What happens now | What should happen | Leak / friction | Fastest fix |
|---|---|---|---|---|
| Land on homepage | Strong brutalist hero, "SCAN FREE — NO CARD NEEDED", live signal table | Same, but server-rendered + a 15-sec "how it works" strip | Slow first paint (client SPA); crawler sees little | SSR hero; 3-step strip: Scan → Score → WhatsApp |
| Understand offer | Good copy on home/pricing; weak on /my-link and /intake | Every page answers "how does this get me more work?" | Intake page confuses homeowners (per 06-02 audit) | Rewrite intake intro: "Tell [trade name] about your job — takes 60 seconds" |
| Trust | VS pages, guarantee, TrustCenter, methodology page | Add 1 real case study + sample WhatsApp screenshot | No real proof yet (no users) — biggest gap, only fixable with beta users | Founder beta → collect 3 quotes |
| Pricing | Clear £39/mo, objections handled | Annual option; "what happens after payment" section | Post-payment expectations unset | Add "Day 1: you get X" block |
| Signup | Supabase email flow | Same + capture trade/postcode immediately | Trade/area not captured at signup → product can't deliver relevant leads | 2-question step post-signup, save to profile |
| Pay | Checkout works (test mode) | Server-authenticated checkout | Security trust issue (F2) | Fix checkout route |
| Get leads | Scan on demand; fallback samples if sources empty | Leads waiting in dashboard within 24h of paying | Risk: user pays, sees fallback/sample-looking leads → instant churn | Concierge: founder manually curates first leads per user |
| WhatsApp GOLD | **Does not happen** (no Twilio config, no auto-trigger) | GOLD ≥80 → WhatsApp within minutes | THE core promise — currently broken | Twilio sandbox + auto-trigger (1 day) |
| Return to dashboard | localStorage leads, ROI tracker | Cross-device Supabase leads + outcome capture | Data loss per device kills habit | Persist leads server-side |
| Stay subscribed | Nothing proactive | Weekly "your week in leads" WhatsApp/email | No retention loop | n8n workflow 01 (daily digest) activation |

## Does the site clearly explain…
| Question | Answer | Where / gap |
|---|---|---|
| What JobFilter does | ✅ | Homepage |
| Who it's for | ✅ | Trade pages ×15+ |
| Where leads come from | ✅ | /methodology, source labels |
| What GOLD leads are | 🟡 | Explained on lead list, but threshold inconsistent in code undermines it |
| Why scoring matters | ✅ | "HOW IT'S SCORED" block |
| Why WhatsApp alerts matter | 🟡 | Mentioned; not demonstrated (no screenshot/sample) |
| What happens after payment | ❌ | Nowhere — top copy gap |
| Areas covered | 🟡 | City pages imply; no explicit coverage checker |
| How often leads arrive | ❌ | Never stated — set expectation ("daily scan, instant GOLD alerts") |
| Why beats manual searching | ✅ | VS pages, homepage |
| Why £39 worth it | ✅ | ROI framing ("one job pays for a year") |

## Trade-customer reality check
Trades won't tolerate: empty dashboards, sample-looking leads, learning curve. They will tolerate: WhatsApp-only, no dashboard at all, founder texting them manually. **Implication: WhatsApp-first concierge beta beats polishing the dashboard.** See [[08 Growth and Revenue Improvements]].
