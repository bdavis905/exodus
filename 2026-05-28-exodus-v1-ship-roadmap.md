# Exodus V1 Ship Roadmap — Brad's Implementation Backlog

> Living doc. Source: 2026-05-27 team meeting + Luke's 2026-05-28 follow-up Loom.
> Working materials in `Thursday May 28th fixes/`. **Ship review: Thu 2026-06-04.**
> Status keys: `[ ]` todo · `[~]` in progress · `[x]` done.
> Last updated: 2026-05-28 (late) — **Tier A shipped** (#114/#115). **B3/Apify SHIPPED**
> (#117). **Primer pipeline unblocked:** #116 (CLI submission accepted) + #118 (async build
> escapes Vercel's 300s timeout, Luke's A.28) — submit+build now work end-to-end; robust
> parser (C2) still open. Also exodus **2026.5.2800** shipped (#119, skills as `SKILL.md`
> dirs). **Phase 0** (dev/prod separation) is underway ahead of the customer release.

## Context

The 2026-05-27 team meeting + Luke's 2026-05-28 follow-up Loom defined a ~1-week
sprint to ship Exodus V1. Two framing decisions drive everything: **CLI-first
(dashboard = visual layer)** and **remove relentlessly, hide-don't-delete**. Luke
made one executive reversal in the Loom: **"Run from Ad" stays** (one ad at a
time, with template + realistic-enhancer folded in); **"Run from References" and
the standalone Template tab are killed.**

This roadmap turns the meeting/Loom into the implementation items **Brad owns in
this codebase**, sorted simplest → hardest, with Luke's Friday-2026-05-29 critical
path flagged. It is the canonical, evolving doc Luke asked Brad to own (H6).

**Scope correction (verified against the code, not the meeting):** several items
are already partly built. This roadmap is re-scoped to the *real* gap and opens
with short verify-first spikes so we don't rebuild what exists. Key corrections:
- Primers are **already 2-track** (`unawareProblemAware` + `solutionProductAware`)
  plus a `hookBank` + `headlineBank` — the "1 → 4" task is really "formalize the
  two banks into real primers + UI + parser."
- **No 4 awareness DB tables exist** — already one pool; the "merge" is likely a
  stale-install artifact. Verify, probably nothing to do.
- The **"no hooks" empty-label bug is not in the current Genesis parser** —
  reproduce before fixing.
- **Pipeline step visualization largely exists** (`pipeline-flow.tsx` et al.).
- **Swipe-mining UI is already built** at `/admin/labs/swipes` — surface it, don't build it.
- **Apify** has a per-user encrypted key table already — ~1-hour add.

**Execution model:** one item → one worktree in `.worktrees/<name>/` → one PR
(per `docs/WORKTREE-PROCESS.md`). Knock out Tier A as a batch; B and C one at a time.

---

## PHASE 0 — Dev/Prod separation

**Decided 2026-05-28** ([[project-dev-prod-separation-required]]): Exodus releases
to **paying customers** ~week of 2026-06-01, so it's moving onto its own production
setup, separate from shared dev infra.
Today everything (Vercel prod, every preview, local dev, the CLI, Trigger.dev, the
Genesis VPS) runs on the single Convex dev deployment `dev:good-cod-360` + a single
Clerk **dev** instance. Why: Clerk dev caps ~100 users / non-prod auth; one
shared DB = dev work + previews read-write real customer rows with no staging.
**Brad's calls:** custom **branded domain**; **migrate** dev data → prod.

This is the spine of the sprint. Tier A/B continue in parallel; heavy Tier C
(Instagram OAuth, 4-primer rework) may slip past launch.

**Clerk → production instance** *(long pole — start first; DNS propagation)*
- [ ] Pick/secure the branded domain; confirm DNS access. **(Brad)**
- [ ] Create the Clerk prod instance + add Clerk's CNAME records; wait for verify. **(Brad)**
- [ ] Mirror config dev→prod: `clerk config pull` (dev) → `clerk config put` (prod) — providers, sessions, the `convex` JWT template. Get `pk_live`/`sk_live`.
- [ ] Add prod Google OAuth redirect URIs (Drive connect breaks otherwise).
- [ ] Decide: migrate team users vs. fresh customer signups.

**Convex → production deployment**
- [ ] `npx convex deploy` to stand up the prod deployment (schema + functions).
- [ ] Migrate data: `npx convex export` (good-cod-360) → `npx convex import` (prod) — brands / primers / bots / workspaces / swipeAds. Verify counts.
- [ ] Set Convex **prod** env vars (Genesis key, OpenRouter, Trigger, Drive, **`CLERK_JWT_ISSUER_DOMAIN` = prod Clerk issuer**). `convex/auth.config.ts` is already env-driven — no code change, just set this var on prod.

**Cutover (DNS verified + prod populated) — keep Preview on dev as staging**
- [ ] Vercel **Production** scope → `pk_live`/`sk_live`, prod issuer, prod Convex URL + deployment.
- [ ] Trigger.dev prod environment so prod runs write to prod Convex; re-point the Genesis VPS callback URL.
- [ ] Point the published CLI at prod Clerk + prod Convex.
- [ ] Smoke-test the full loop on the prod domain (login → brand → Genesis run → Google Doc) before announcing.

**Status (2026-05-28):** auth.config confirmed env-driven (no code change). Dev
data export validated as the migration source. Awaiting domain + Clerk prod
instance (Brad) before standing up/populating prod.

---

## 🔴 BLOCKING / VERIFY-FIRST (do before building — mostly fast)

- [ ] **V0. Fix Luke's CLI install** *(critical path, tonight — not code).* Send
  Luke the latest version; have him fresh-install in a new folder (user-scoped
  keys). Debug his Slack output offline. Everything he tests is suspect until this lands.
- [x] **V1. "No hooks" empty-label bug — LOCATED 2026-05-28.** The override is decided
  by the **Genesis server bot `primer-prompt-builder`** (gas.copycoders.ai), not repo
  code. Repo side just transcribes it: `src/app/api/primer/build/route.ts:47-54` calls
  the bot, then `extractPrimerSection(primer,"HOOK")`
  (`src/lib/primer/extract-section.ts:18-43`) slices whatever the bot emitted — empty
  HOOK section → returns `""`; `exodus/commands/primer.ts:187` surfaces the degrade.
  `extractHooks` in `strip-bot-junk.ts` is confirmed **off this path** (parses bot
  output, not pasted input). **Fix per the no-server-prompt-edit rule = normalize the
  pasted submission on INPUT** (strip blank `Hooks:` labels so the bot doesn't read
  "blank = skip") before the bot call. → scopes **C2**.
- [x] **V2. Awareness "merge" is a NO-OP — CONFIRMED 2026-05-28 (high confidence).**
  `workspaces.primerByAwareness` is already 2-track (`unawareProblemAware` +
  `solutionProductAware`, `convex/schema.ts:1010-1013`). No per-awareness tables exist
  (70 tables, none awareness-keyed). The 4→2 collapse is handled by three code mapping
  layers (`scout/src/types/brand-foundation.ts:47-52`, `convex/workspaces.ts:19-24`,
  `convex/workspaces.ts:692-698`). **Nothing to migrate** — don't build a phantom migration.
- [x] **V3. Genesis writing flow — VERIFIED 2026-05-28; visual produced.** Full
  step-by-step + Luke reconciliation table at
  `docs/superpowers/specs/2026-05-28-genesis-writing-flow-for-luke.md`. Skeleton matches
  Luke's spec but diverges in 3 real ways: (1) **one** shared brand primer, not separate
  Hook+Body primers; (2) **two** hook bots in parallel (`ad-hook-bot-1` Mario voice +
  `in-feed-vsl-bot`), ~40 hooks merged; (3) body copy = **3 tracks / 6 variants**
  (Mario×brand, In-Feed×brand, Top-Ads), headlines written by whichever bot wrote that
  variant (Call 3, same convo). Plus seed-capture / hook-selection / QA / doc-write
  steps. Models are resolved server-side (bot slug is the only selector in-repo). Open
  questions for Luke captured in the doc → confirm, then surface in pipelines tab (C4).

---

## ✅ TIER A — SHIPPED 2026-05-28 (PRs #114 + #115, live on prod)

All verified on the preview via the Codex QA loop, then merged + deployed.

- [x] **A1. Remove Template + Scraper tabs from top nav.** `max-shell.ts`.
- [x] **A2. Gate "+ Run from References".** `runs/page.tsx` — gated on admin
  **mode** (not just role) as of #115, so admin-mode-off = "preview as customer".
- [x] **A3. Kill the PIPELINES…SHARED secondary nav row.** Removed
  `CreativeSuiteSubNav` from `creative-suite/layout.tsx`; deleted orphaned `sub-nav.tsx`.
- [x] **A4. Drop the "Google" library sub-tab.** `creative-suite/library/page.tsx`.
- [x] **A5. Copy Editor — "← Runs" back link.** `CopyEditorShell.tsx`.
- [x] **A6. Default GPT Image 2, drop Nano Banana Pro.** `creative-suite/template/page.tsx`.
- [x] **A7. Image-card cleanup** — Source Blocks + Add-as-Scene/Object removed;
  image + Download only. `RunDetail.tsx`.
- [x] **A8. One "Show Ad Copy" per ad** (grouped run page). `RunDetail.tsx` +
  `creative-suite/runs/[id]/page.tsx`.
- [x] **A9. Exodus logo → `/onboarding`** (new minimal page). `dashboard-shell.tsx`.

**Also shipped in this batch (emergent):**
- [x] Home `/` redirects to `/runs`; retired feed UI deleted (`page.tsx`,
  `run-card/`, `feed-filters`, `day-group`, `empty-state`). *(Covers part of B9.)*
- [x] `useAdminMode` / `useShowAllBrands` cross-instance sync (admin tabs update live).
- [x] Ad Account tab admin-gated (`adminOnly: true`).
- [x] `feed.listForUser` bounded per-table reads — fixes the 16MB overflow that 500'd
  the home page (deployed to `good-cod-360`).

---

## 🟡 TIER B — Medium (≈half-day to a day each)

- [ ] **B1. Run-from-Ad modal rework** *(critical path).* Remove the **Reference**
  engine row; add a **Template** engine; fold in realistic-enhancer modes
  (auto/manual/hybrid, shown only for template engine); make it **one ad at a
  time**; fold wildcard/scraper as options.
  `src/app/(dashboard)/creative-suite/_components/NewRunModal.tsx` (engine rows
  ~L332–378, kickoff handler ~L166–240).
- [ ] **B2. Runs feed — add "copy" filter + finalize per-ad grouping** *(critical path).*
  Variations (copy-derived/reptile/template) roll up under one ad card.
  `src/app/(dashboard)/runs/page.tsx` (`ENGINE_OPTIONS`, grouping ~L83–108) + `RunCard.tsx`.
- [x] **B3. Apify API key field in Settings + server wiring — SHIPPED 2026-05-28 (PR #117, squash d640765f).**
  Scope was bigger than the "~1hr" estimate: the creative-suite scraper had NO per-user
  key path (all global `APIFY_API_TOKEN`). Wired **fully** per Brad's call — a customer's
  own Apify key bills their account. `apify` added to the vault (`convex/schema.ts` union +
  `userKeys.ts` ProviderArg + `/api/settings/keys` allowlist) + a Settings card with the
  ~$0.50/run note. Threading mirrors pixar `studentKeys`: encrypted key snapshotted onto
  `creativeSuiteScraperRuns.apifyKeyEncrypted` at run-create (`http.ts` `snapshotScraperApifyKey`);
  scout worker loads the row + decrypts (`scout/.../scrapers/keys.ts`), env fallback in
  `fetchGoogleImages`. Only `scraperRunId` rides the Trigger payload. **Deployed:** good-cod-360
  (convex) + scout/Trigger v20260529.3 + prod Vercel from d640765f. Codex QA plan staged
  (`exodus-qa/v1/2026-05-28-pr117-qa-test-plan.md`) but Brad green-lit the ship pre-QA.
- [ ] **B4. Surface swipe-mining into the Brand Profiles tab** *(critical path).*
  Move/mirror `/admin/labs/swipes/_components/*` into
  `src/app/(dashboard)/brand-profiles/`. Show organic accounts + scraped ads +
  metadata. Add missing fields (impressions / est. impressions / date-off) to
  `swipeAds` if ScrapeCreators provides them.
- [ ] **B5. Primer view/edit UI in Settings** *(critical path).* Show **examples
  only** (add/remove) + an "always do / never mention" steering field — never the
  raw prompt. Reads/writes `workspaces.primerByAwareness` + `hookBank`/`headlineBank`.
  New tab alongside existing `settings/_components/*-tab.tsx`.
- [ ] **B6. Brand-info editing area in Settings** — product mockups (front/back/
  versions), founder info, label — routed to the templates that need it. Builds on
  `brand-profiles/page.tsx` data; needs a per-user/per-brand edit surface in Settings.
- [ ] **B7. Disable memes in the CLI for V1** (dashboard memes stay). Backend
  `src/app/api/meme/*` + the CLI/skill side.
- [ ] **B8. Library "use as reference" action** for Reddit/Pinterest items → type
  change → lands in Generated. `creative-suite/library/page.tsx` + underlying queries.
- [ ] **B9. Remove stray legacy "old dashboard" pages** Luke kept hitting; ensure
  their content shows up in Runs.
- [ ] **B10. Delete the standalone Template page** once B1 folds it into Run-from-Ad.
  `src/app/(dashboard)/creative-suite/template/*` (do after B1 lands).

---

## 🟠 TIER C — Design-heavy / cross-owner (multi-day; needs strategy)

- [ ] **C1. 4-primer architecture.** Formalize `hookBank` + `headlineBank` into real
  Hook + Headline primers (keep the 2 body primers); consider **separate bots per
  primer**. Depends on **Elise's 4 primers + parser**. Touches `scout/src/primers/*`,
  `convex/bots.ts`, `convex/seedGenesisPipeline.ts`.
- [~] **C2. Primer docs — submission + build SHIPPED (#116, #118); parser robustness remaining**
  *(critical path).* **Shipped:** (a) PR #116 — CLI-submission blocker (Finding #13:
  `foundation/set` `ALLOWED_FIELDS` only whitelisted legacy 4-track names, so 2-track CLI
  fields 400'd before reaching Convex) + clearer Genesis gate; (b) PR #118 — `exodus primer`
  build moved to async Trigger.dev enqueue→poll (new `primerBuilds` table + `convex/primer.ts`),
  escaping Vercel's 300s timeout (Luke's A.28 hard blocker). Primers now submit and build
  end-to-end. **Remaining:** robust parser — raw, unstructured ads → hooks/headlines/body +
  awareness; **empty-label override fix (per V1)** by normalizing the pasted submission on
  input before the server bot call. Possibly two bots (parse fields → parse awareness). Shared with Elise.
- [ ] **C3. Meta headline export ingestion** (net-new). Upload + parse a Meta report
  → headline primer/bank. No route exists today; `convex/topAdsPrimer.ts` is the
  closest analog (dynamic, not an upload).
- [ ] **C4. Genesis pipeline step view** — verify the existing pipeline-flow
  components (`settings/_components/pipeline-flow.tsx`, `pipeline-run-trace-panel.tsx`)
  render the Genesis flow correctly; wire it in. *May drop to Tier B once V3 confirms
  how much is already working.*
- [ ] **C5. Organic Instagram per-user OAuth scrape** *(critical path; hardest).* The
  "last pipeline piece." Today it's one global Browserbase account
  (`scout/src/trigger/scout-organic-scan.ts`). Model the per-user flow on the Google
  Drive OAuth (`src/app/api/auth/google/*` + `google-drive-tab.tsx`): new
  `/api/auth/instagram/*` routes, IG token fields on `users`, a Settings tab.
  **Risk: Meta app review / IG permissions** — scope the auth approach before building.
- [ ] **C6. Wild-source rebuild** (Reddit / Pinterest / Google via Apify). Reddit +
  Google (Apify) work; **Pinterest is deferred** (SC returns no resolvable URLs).
  **Blocked on Max** handing over his working approach.
  `scout/src/creative-suite/scrapers/*` (`run-scraper.ts`, `sc-reddit.ts`,
  `apify-google-images.ts`).

---

## ⚪ V2 — Deferred (documented, not this sprint)

Dashboard generation beyond Run-from-Ad · image remix / edit-in-place ·
add-as-scene/object mixing · scraper visual flow · memes via CLI · global
reference library · extra primer types (short-form / video / story / most-aware) ·
object + scene libraries with metadata · AI-cartoon meme thumbnails (undecided).

---

## Cross-owner dependencies (gate Brad's items)

- **Elise** → 4 primers + parser (building 2026-05-28) → gates C1/C2.
- **Max** → working wild-source/scraper approach → gates C6; meme AI-vs-classic UI.
- **Fernando** → template UI simplification → informs B1.

---

## Verification

- **Per item:** typecheck before trusting any deploy. Frontend/Convex:
  `npx tsc --noEmit -p convex` and the app build. Scout/Trigger:
  `cd scout && npx tsc --noEmit` (esbuild ships TS errors silently).
- **UI items (A1–A9, B1/B2/B4/B5/B6/B8):** verify in the live dashboard manually
  (no Playwright in this repo) — confirm tabs/buttons/cards render and gate correctly.
- **Pipeline items (V3, C1–C4):** run a Genesis writing pass end-to-end via the CLI
  and confirm the Google Doc output + the step view in Settings.
- **B3/C5/C6:** confirm the new key/OAuth round-trips and a scrape/IG pull populates
  the DB; check Trigger.dev run logs.
- **Deploy:** after merging each branch, run the Convex + Trigger.dev deploys as
  part of finishing (don't hand Luke a "you deploy" message).

## Cadence (absolute dates)

Fri 2026-05-29 12pm ET (Brad + Elise) · Mon 2026-06-01 4pm ET ·
Tue 2026-06-02 3pm ET (TBD) · Wed 2026-06-03 2:30pm ET · **Thu 2026-06-04 ship review.**
