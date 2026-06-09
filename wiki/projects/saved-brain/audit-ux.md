---
title: Saved Brain — UX / Product Audit
project: saved-brain
type: audit
domain: ux
created: 2026-06-09
tags: [saved-brain, audit, ux, product]
---

# Saved Brain — UX / Product Audit

Back to [[index]] · feeds [[build-prompt]]

Clean, well-built Next.js single-user "second brain." Consistent visual design (yellow/ink theme, lucide icons, good empty states). Gaps are mostly flow continuity, the extension auth story, and first-run value-clarity.

## P0 — Critical (block a polished first run)

1. **Sidebar bleeds onto marketing/landing.** `app/layout.tsx` renders `<Sidebar>` on every route incl. `/`, `/onboarding`, `/pricing`. Logged-out visitor sees app chrome ("Library/Boards/Graph/Upload/Settings/Upgrade to Pro") before doing anything; landing hero+footer squeezed into `lg:ml-64`. Fix: route groups `(marketing)` vs `(app)` with separate layouts, or conditional Sidebar by pathname.

2. **Onboarding is a dead end on completion paths.** Step 3 import options are `<a href>` to `/upload` — clicking navigates away *before* `handleSaveSettings` runs, so API key/provider from steps 1–2 silently discarded unless "Finish Setup" clicked. Fix: save settings first, then route.

3. **`onboarding_complete` written but never enforced.** Set in onboarding, read nowhere. No gate from `/library` -> `/onboarding`, no redirect away once done. New users on `/library` (active nav target) see empty state, no provider configured, no nudge.

4. **Privacy & Terms links 404.** Footer `/privacy` + `/terms` don't exist. Pricing makes legal/refund claims ("30-day money-back guarantee") with no terms. Fix: add stub pages.

## P1 — High (core loop + extension friction)

5. **Extension auth/install friction = weakest seam.** README + onboarding say "build from /extension" — no installable extension; user must clone, `npm run extension:build`, load unpacked. Popup asks for raw "Backend URL" + optional API key, but web app has no auth/session/API-key issuance. So "Chrome extension auto-sync" (paid Pro feature) has no real connect-account path. Biggest marketing-vs-product credibility gap.

6. **Semantic toggle does nothing in Library.** `SearchBar` flips fulltext<->semantic, but `library/page.tsx` `fetchItems` forwards only `search/source/tag` to `/api/items` — `mode` dropped. Users toggle Semantic, get identical results, lose trust. Wire mode through or hide until Pro.

7. **Search facets never appear.** `SearchBar` only renders source/tag/entity/category dropdowns when arrays non-empty, but Library calls `<SearchBar>` with no `sources/tags/entities` props. Filters panel opens to just two date inputs. Fix: derive facet lists from loaded items, pass in.

8. **No "add to board" path anywhere.** `PostCard` supports `onAddToBoard`, but neither Library nor Board pages pass it. Empty board state links to `/library`, but Library can't add to a board. Boards effectively un-fillable through UI — core-loop dead end.

9. **Enrichment has no visible state/progress.** "Enrich All" fires `/api/enrich {all:true}`, flashes "saved" — no progress/count/async feedback for a long LLM job. PostCards show no "not yet enriched" state; unenriched item renders empty summary. The "enrich" half of save->enrich->find is invisible.

## P2 — Medium (polish, a11y, IA)

10. **Raw timestamps shown.** `PostCard` renders `savedAt`/`createdAt` as ISO string. Need relative/formatted ("3 days ago").
11. **Graph canvas not responsive/accessible.** Fixed `width=900 height=600`, no resize (overflow on mobile/large), click-only `<canvas>`, no keyboard/text alt. Re-runs full force sim every frame forever — perf/battery cost.
12. **Mobile: hamburger overlaps page H1.** Fixed hamburger at `left-4 top-4`; page headers start same top-left, no top padding -> overlap. Add `pl-12 lg:pl-0` / top offset.
13. **A11y gaps:** color-only active states on toggles; active-filter is bare "!" no `aria-label`; API-key password inputs no show/hide; search input no `<label>`; canvas graph inaccessible; low contrast `text-muted` on `bg-panel`.
14. **Marketing claims outrun product.** "Join thousands," "£9.99/mo equivalent" strikethrough, "Encrypted and stored locally"; "shareable public boards" + clone lineage imply multi-tenant server the no-auth architecture doesn't support.
15. **No global "save/add" affordance.** Upload buried in nav; no persistent "+ Add" quick-capture in header. For a frictionless-saving pitch, primary action is 2 clicks from any page.

## Missing for a polished product
- **Auth/account layer** — boards sharing, extension sync, Pro entitlement all presume accounts that don't exist. Central gap.
- **Item detail view** — no route to open a single item; can only click out to original URL. No full summary, entities, tag edit, board assign.
- **Onboarding->empty-library bridge** — no sample data, no guided first import, no first-enriched celebration.
- **Extension store listing / one-click install** — dev-only today.
- **Loading skeletons** vs full-page spinners.
- **Search empty-results state** — Library says "No items yet / Upload" even when a *search* returns nothing.

## Top 5 quick wins
1. Split marketing vs app layouts (route groups). (#1)
2. Onboarding tiles save settings before navigating. (#2)
3. Pass facet arrays + thread `mode` into Library SearchBar/fetch. (#6, #7)
4. Add stub `/privacy` + `/terms`. (#4)
5. Relative `savedAt` dates + distinct "no search results" state. (#10)

Key files: `app/layout.tsx`, `app/onboarding/page.tsx`, `app/library/page.tsx`, `components/SearchBar.tsx`, `components/PostCard.tsx`, `app/graph/page.tsx`, `extension/README.md` + `app/api/*`.
