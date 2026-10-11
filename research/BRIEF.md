# BRIEF — varna-ui design system (decision brief, 2026-10-11)

Inputs: research/00–06 plus lead corrections of 2026-10-11 (Cal.com main repo is still public/MIT; coss ui is Cal.com's official DS on Base UI, AGPL-3.0; shadcn defaults to Base UI; npm versions re-verified; apca-w3 is under a "Limited W3 License", not MIT). Where a research file conflicts with those corrections, the corrections win.

## 1. Positioning

**varna-ui** is a small, accessibility-evidenced React design system on Base UI for product teams that need to *prove* accessibility rather than assert it. Every component ships with a published keyboard map, screen-reader notes, an axe/Playwright environment matrix (light, dark, high-contrast, forced-colors, reduced-motion, RTL, 320px reflow) and tokens that cannot be released below WCAG 2.2 AA because the token build fails first. It is not another shadcn: shadcn/ui and coss ui already own the "copy-paste Base UI + Tailwind, 50+ components" niche, and React Aria Components already owns "deepest a11y engine". akuru competes on *depth per component and evidence per release*: 12 components done to a definition-of-done that includes manual AT transcripts, a CI contrast gate reusing the ContrastMate engine, a first-party Figma kit bound to the same DTCG tokens, Sinhala/Tamil typography as a first-class script layer, and a real consumer app. Audience: hiring managers at Cal.com-type companies first, adopters who need an honest ACR second.

## 2. Decision table

| Decision | Recommendation | Why (1 line) | Rejected | Risk | Conf. |
|---|---|---|---|---|---|
| Primitive layer | `@base-ui/react` ^1.9, wrapped in a thin internal layer (`src/primitives/*`) | shadcn default + coss ui foundation = direct Cal.com relevance; monthly funded releases; has Combobox/Field/Toast | Radix (single-committer bus factor, #1343 touch-SR Select open since 2022); RAC (best a11y evidence but zero Cal.com alignment, heavier API); Ark (Vue story only) | Open iOS VoiceOver Menu bug #5794; Toast timing debate #4253; monthly minors force an upgrade habit | High |
| Styling / CSS | Plain CSS custom properties + CSS Modules compiled to one `styles.css`; `@layer varna.reset, varna.base, varna.components`; logical properties only; `data-*` + CSS vars as override API | Zero runtime, RSC-safe, framework-agnostic, forced-colors/`prefers-*` first-class | Tailwind-in-library (couples consumers; coss already does this); vanilla-extract (consumer plugin); CSS-in-JS (RSC) | Hashed class names hinder overrides → mitigated by documented `data-*` hooks; tsdown CSS is experimental | High |
| Tailwind bridge | `@varna-ui/tokens/tailwind.css` generated `@theme` block (tokens only, no component styles) | Cal.com/coss consumers are on Tailwind v4; cost is one Style Dictionary format | Full Tailwind preset with component classes | Drift if hand-maintained → must be generated | Med |
| Token format | DTCG 2025.10 JSON (`*.tokens.json`), OKLCH colour values with `hex` fallback, JSON-schema validated | First stable spec; tool support now real (SD5, Terrazzo, Figma import) | Tokens Studio JSON (plugin lock-in); Figma-native | Resolver module unsupported by SD → theme loop in build script | High |
| Token pipeline | Style Dictionary 5.6 → `tokens.css` (per-theme blocks + media wrappers), `tokens.ts`, `tokens.resolved.json`, `figma-variables.json`, `tailwind.css` | Mature, custom formats trivial; one build emits every target | Terrazzo 2.7 (smaller ecosystem; re-evaluate if SD theme loop hurts) | DTCG 2025.10 "not fully supported" note on SD docs → spike it | High |
| Source of truth | Git (DTCG) → Figma Variables are a pushed mirror via own plugin | Figma Variables REST API is Enterprise-only; git gives review, CI, public artefact | Figma-as-source; Tokens Studio two-way | Designers edit via PR/plugin only | High |
| Palette generation | Script: binary-search OKLCH lightness per hue (`culori`) to hit target WCAG ratios; 12 role-based steps (Radix-style) | Contrast guaranteed by construction, then verified; ~60 lines showcasing owner's domain | Leonardo (fine, but adds Adobe dep and hides the maths); hand-picked hex | Chroma clamping into sRGB needs care | Med |
| Themes / modes | v1: `light`, `dark`, `hc-light`, `hc-dark` (semantic sets), `forced-colors` (hand-written system-colour map), reduced-motion (duration tokens → 0.01ms), `:lang(si)`/`:lang(ta)` typography layer. Selection by `[data-theme]` with `prefers-*` defaults | Covers the WCAG 2.2 + beyond-WCAG story; every mode is a column in the contrast matrix | Density (defer to v1.1), brand multi-theme, `light-dark()` single-declaration (support check later) | 4 themes × every semantic colour token = must all be defined or build fails (that is the point) | High |
| Contrast gate | `tokens/a11y/pairs.json` manifest + coverage check (every `fg.*`/`border.*`/`focus.*` token must appear in a pair) → Vitest test over resolved themes; **WCAG 2.2 AA normative** (AAA in HC), **APCA Lc informational column only**; axe on every story as second net | Turns ContrastMate from a plugin into a release gate; fails CI, publishes matrix page | Terrazzo `a11y/min-contrast` alone (no non-text, alpha, theme coverage); axe alone (misses hover/focus) | apca-w3 "Limited W3 License" → see licensing row | High |
| Contrast engine | New MIT package `@varna-ui/contrast`: WCAG 2.x maths, alpha compositing, large-text rules, lifted from ContrastMate `src/contrast.ts` + tests. **APCA constants are NOT included**; CI computes APCA via `apca-w3` as a devDependency for the informational report only | Keeps the MIT package clean; ContrastMate and CI share one engine | Re-implementing APCA constants under MIT (current ContrastMate state; licence risk) | ContrastMate's existing APCA code needs the same treatment or a licence note | Med |
| Repo structure | pnpm 12 workspace: `packages/tokens`, `packages/react`, `packages/contrast`, `apps/docs` (Storybook), `apps/demo` (consumer app), `e2e/`, `decisions/` (ADRs); no Turborepo until pain | 3 published packages is the smallest "tokens are a product" shape | Single package; Turborepo/Nx day one (decoration) | Slightly more config for a solo dev | High |
| Build | tsdown 0.23 ESM-only, `unbundle` per-file output, `dts: true`, publint + attw (`esm-only`), CI grep for `"use client"`; fallback CSS build via `lightningcss-cli` | Rolldown successor to tsup; tsup is de facto unmaintained | tsup; Vite lib mode; dual ESM/CJS (hazard, attw complexity) | 0.x option churn; experimental CSS | Med |
| Toolchain pins | TS 6.0.x (typescript-eslint peers <6.1), ESLint 9 (jsx-a11y peers ≤9), Vite 8.3, Vitest 5.0, Storybook 10.6, React 19.3, Node 24 | Lead-verified peer constraints | TS 7, ESLint 10, Biome (weaker a11y rules) | New majors (pnpm 12/Vite 8/Vitest 5) untested together → M0 spike | High |
| React support | React 19 only (`peer ^19`), `ref` as prop, no `forwardRef` | Base UI targets 19; halves CI matrix for a solo maintainer; Cal.com is on 19 | `^18.2 \|\| ^19` with forwardRef | Smaller reach; revisit if adopters ask | Med |
| Docs | Storybook 10.6 on GitHub Pages is the single docs surface for v1: autodocs + MDX pages (Intro, Tokens + live contrast matrix, Accessibility statement, per-component a11y section, ADR index). `addon-designs` embeds Figma frames | One source of truth, zero extra site; prose needs are small at 12 components | Starlight/Fumadocs now (add post-v1 only if MDX proves limiting) | Storybook MDX is less pleasant for long prose | Med |
| Testing pyramid | Static (TS strict, ESLint jsx-a11y strict, knip, publint/attw) → unit (hooks, token build, contrast) → **story tests** (addon-vitest browser mode, `play` keyboard scripts, `a11y.test:'error'`) → Playwright env matrix (forced-colors, reduced-motion, contrast:more, RTL, 320px, 400% zoom, text-spacing injection, `@axe-core/playwright` on open states) → visual (`toHaveScreenshot`, Linux baselines) → Guidepup VO/NVDA nightly non-blocking → manual SR log in `ACCESSIBILITY.md` | Stories as single source of truth; evidence per environment is the product | Jest/jsdom for components; Chromatic as mandatory (optional free tier only); vitest-axe (stale) | Guidepup flake; Playwright `forcedColors` is Chromium-only | High |
| Release | Changesets 3 + npm Trusted Publishing (OIDC, provenance), `0.x` until v1.0, size-limit budgets, SHA-pinned actions, OpenSSF Scorecard, CodeQL, Renovate grouped | No long-lived token; provenance badge; looks like a real project | semantic-release; manual publish | Trusted-publisher bootstrap may need a first manual publish | High |
| Figma kit | Two libraries (Foundations: variables/type/icons; Components) + one Community file, CC BY 4.0. Variables: Primitives (hidden) → Semantic (modes Light/Dark/HC Light/HC Dark) → thin Component; `codeSyntax.WEB` = CSS var name; component properties = React props 1:1; Slots for children; CVS-style a11y annotation frames (credited) | Pro plan suffices; parity by convention + plugin-enforced tokens | Code Connect (Org/Enterprise only; do not commit untested `.figma.tsx`); Tokens Studio Pro | Figma modes cap 10 on Pro (fine for 4); native DTCG import is sRGB-only and unverified on Pro | High |
| ContrastMate integration | ContrastMate v2 consumes `@varna-ui/contrast`; new "Token Auditor" mode reads local variable collections, resolves per mode, runs the same `pairs.json`, shows matrix + nearest passing OKLCH fix, exports `contrast-report.json`; "Token Sync" mode imports `figma-variables.json` | Same maths at design time and build time from one token source; the connected portfolio story | Separate new plugin; REST sync (Enterprise) | Plugin is user-triggered (no headless CI on Pro) | High |
| Licensing | Code MIT; docs + Figma kit CC BY 4.0; **APCA reported only, never claimed**; no code copied from coss (AGPL) — study layering only | Honest, OSS-friendly, matches Community default | Dual licence; AGPL | apca-w3 licence terms; CVS kit attribution required | High |
| Name | `varna-ui` (Sinhala "colour"); npm `@varna-ui/react`, `@varna-ui/tokens`, `@varna-ui/contrast`; repo `SusithD/varna-ui` | Short, free on npm, ties to the script pillar | `pahan`, `sevana` (free fallbacks) | npm org `varna-ui` and GitHub org unverified → claim day 1 of M0 | Med |

### 2.1 Architecture snapshot (what the decisions add up to)

```
varna-ui/
  packages/contrast   @varna-ui/contrast   WCAG 2.x maths + alpha compositing (from ContrastMate); MIT; no APCA constants
  packages/tokens     @varna-ui/tokens     DTCG JSON -> SD5 -> tokens.css | tokens.ts | tokens.resolved.json | figma-variables.json | tailwind.css
  packages/react      @varna-ui/react      Base UI wrappers + CSS Modules -> dist/*.mjs + styles.css (ESM-only, React 19)
  apps/docs           Storybook 10.6    autodocs + MDX (tokens matrix, ACR, ADR index); GitHub Pages
  apps/demo           booking flow      built only from @varna-ui/react; axe + Playwright in CI; deployed
  e2e/                Playwright        forced-colors | reduced-motion | contrast:more | rtl | zoom-400 | text-spacing | sr/ (Guidepup)
  decisions/          ADR-NNN.md        one per row of the decision table that changes
  ACCESSIBILITY.md    conformance target, AT matrix, known issues, last manual audit date
```

Token tiers: primitive (`color.{hue}.{1..12}` OKLCH, `space.*`, `font.*`, `duration.*`) → semantic public API (`color.bg.*`, `color.fg.*`, `color.border.*`, `color.focus.ring`, `text.*`, `motion.*`) → thin component tier. Components may reference tiers 2–3 only (lint-enforced). CSS var prefix `--ak-`. Every semantic colour token must exist in all four themes or the build fails.

CI gates on `main` (required): `static` (tsc, eslint jsx-a11y strict, prettier, knip, changeset status) · `unit` (contrast, token build, hooks) · `build` (tsdown, publint, attw, `use client` grep, size-limit) · `story-tests` (addon-vitest, axe `error`) · `e2e` (Playwright matrix + axe on open states + demo app) · `contrast-gate` (pairs + coverage). Non-blocking: `visual`, `a11y-nightly` (Guidepup VO on macOS, NVDA on Windows), `scorecard`, `codeql`.

## 3. v1 scope

Build order (foundations → forms → overlays → composites). Cut line if behind: drop 12 then 11; never drop 5, 9, 10.

| # | Component | Base UI parts | Signature a11y/i18n proof |
|---|---|---|---|
| 0 | Tokens + 4 themes + forced-colors + script layer | — | CI contrast gate, published matrix, `:lang(si/ta)` type |
| 1 | Button, IconButton (`label` required), LinkButton | Button | 2px/3:1 focus ring, 24px floor/40px default target, forced-colors border |
| 2 | Field (Label/Description/Error) + Input + Textarea + PasswordInput | Field, Input | `aria-describedby` wiring, paste allowed (3.3.8), error announcement |
| 3 | Checkbox, RadioGroup, Switch | Checkbox, Radio, Switch | Group semantics, mixed state, system-colour glyphs |
| 4 | Dialog + AlertDialog | Dialog, AlertDialog | Focus return when trigger unmounts, `inert` background, 400% zoom scroll |
| 5 | Select | Select | iOS VO + TalkBack transcript, Sinhala/Tamil long labels, RTL |
| 6 | Popover + Tooltip | Popover, Tooltip | 1.4.13 dismissible/hoverable; touch fallback to Popover |
| 7 | Menu | Menu | APG menu button, RTL submenu arrows, #5794 workaround + regression test |
| 8 | Tabs | Tabs | Manual vs automatic activation documented; RTL arrows |
| 9 | Toast | Toast | Pre-mounted live region, no auto-dismiss for action/error, F6 hotkey, pause on hover/focus |
| 10 | Combobox | Combobox | Result-count live region, grouped options, 3-SR transcript |
| 11 | Accordion | Accordion | Heading level prop, `hidden="until-found"`, reduced-motion height |
| 12 | Badge, Avatar, Skeleton, Progress, VisuallyHidden, SkipLink | Avatar, Progress | Text alternatives, `aria-busy` once, static under reduced motion |
| S | Calendar + DatePicker (v1.1) | composed; `@internationalized/date` | `si-LK`/`ta-LK`/`ar` locales, week start, timezone label, APG grid keys |

**Definition of done (rendered on every docs page, enforced by PR template):**
- [ ] API: typed props, `className`/`render` composition, controlled + uncontrolled, no `any`, `'use client'` only where needed
- [ ] Semantic tokens only; renders correctly in light / dark / hc-light / hc-dark / forced-colors
- [ ] States: default, hover, focus-visible, active, disabled, pending, invalid, read-only, long Sinhala/Tamil + RTL text
- [ ] Keyboard map documented against APG; `play` tests for every key (incl. RTL variant)
- [ ] axe `error` on every story including open/invalid/loading states
- [ ] Playwright matrix green: forced-colors, reduced-motion, contrast:more, 320px reflow, 400% zoom, text-spacing injection
- [ ] Contrast: all pairs in `pairs.json`; ratios + APCA Lc shown on docs page
- [ ] Manual SR pass logged: NVDA+Firefox, VoiceOver macOS Safari, VoiceOver iOS (TalkBack for Select/Menu/Combobox) with date, versions, known limitations
- [ ] Docs page: usage, anatomy, do/don't, a11y section (template from 04 §5.1), consumer responsibilities
- [ ] Figma component with properties = props, bound to semantic variables; `addon-designs` link
- [ ] Changeset, size-limit budget, used in `apps/demo`

**Non-goals (v1):** Table/DataGrid, charts, rich text, theme builder, density mode, multi-brand, Vue port, Code Connect, icon set (recommend Lucide), >15 components, Chromatic as a requirement.

**House standards (stricter than AA, applied to every component):** focus ring 2px solid, 3:1 against adjacent colours, 2px offset (meets 2.4.13 AAA); pointer target 40px default, 24px floor with documented spacing exception; muted text 4.5:1 (no "disabled-looking" text that is not inert); body text 7:1 in `hc-*` themes; `rem` for all type and spacing; no `outline: none` without replacement; no `aria-errormessage` (use `aria-describedby`); live-region containers mounted before content.

**`apps/demo` spec (the consumer):** a three-step booking flow (pick event type → pick date/slot → attendee form with validation → confirmation toast) using only `@varna-ui/react`; runs in all four themes and RTL via a header switcher; `lang="si"`/`"ta"` strings on one screen; Playwright + axe on each step in CI; deployed alongside Storybook. It exists to surface API friction (filed as issues) and to be the screenshot in the case study.

**Accessibility statement + ACR:** `ACCESSIBILITY.md` states WCAG 2.2 AA target, self-assessed, lists AT/browser matrix with versions and dates, links upstream issues per known limitation, and carries the disclaimer that components are building blocks and product conformance depends on usage. From v0.2 a VPAT 2.5 (WCAG edition) ACR is kept in `a11y/acr/` and regenerated per minor release; page-level SCs marked "Applicable to implementers"; AAA marked "Not Evaluated" except where met (HC themes).

## 4. Differentiators ranked (signal ÷ effort)

| Rank | Differentiator | Effort | Proof artefact a hiring manager sees |
|---|---|---|---|
| 1 | Contrast-gated tokens via ContrastMate engine | M | Red CI run on a PR that lowers `fg.muted`; live contrast matrix page per theme; `@varna-ui/contrast` on npm used by both repos |
| 2 | Real consumer app (`apps/demo` booking flow) | M | Deployed demo built only from `@varna-ui/react`; issues filed from its friction; case study |
| 3 | Forced-colors + reduced-motion + HC themes | L–M | Screenshot grid per component: light / dark / hc / forced-colors; Playwright report |
| 4 | Per-component a11y evidence + versioned ACR | M (ongoing) | `ACCESSIBILITY.md` + VPAT 2.5 WCAG-edition ACR with upstream issue links; SR transcripts |
| 5 | Upstream PRs to coss ui / Base UI (a11y, forced-colors, contrast) | L each | 2–4 merged PR links in README "Upstream" section |
| 6 | Sinhala/Tamil script layer | M | Stories with real strings, measured line-height table, no-clip visual tests, `Intl` demos |
| 7 | Figma kit in token parity + ContrastMate Token Auditor | M | Community file; 60s screen recording of the auditor failing a pair in Figma and CI failing the same pair |
| 8 | Decision log (ADRs) + honest APCA framing | L | `decisions/` folder; "Why APCA is informational" ADR |
| 9 | AI-native layer (`AGENTS.md`, `llms.txt`, shadcn-compatible registry) | L | `npx shadcn add` from the varna-ui registry working in a fresh project |
| 10 | Locale-aware Calendar (v1.1) | H | Date picker story in `si-LK`, `ta-LK`, `ar`; timezone label |

## 5. Milestones (solo, evenings ≈ 10 h/week)

| Milestone | Weeks | Ships |
|---|---|---|
| **M0 Spike** | 1–2 | Repo skeleton, names claimed, all spikes in §6 passed or decisions revised; ADR-001..005 |
| **v0.1** | 3–8 | `@varna-ui/tokens` (4 themes, forced-colors, script layer, contrast gate, matrix page), `@varna-ui/contrast`, `@varna-ui/react` with Button, Field/Input/Textarea, Checkbox/Radio/Switch; Storybook on Pages; CI (static/unit/story/e2e/release); first OIDC publish; `apps/demo` skeleton consuming the package |
| **v0.2** | 9–13 | Dialog, Select, Popover/Tooltip; Playwright env matrix complete; first manual SR log; ACR skeleton |
| **v0.3** | 14–18 | Menu, Tabs, Toast; Guidepup nightly; demo booking flow functional; 1–2 upstream PRs |
| **v0.4** | 19–22 | Combobox, Accordion, small batch; size budgets; registry + `AGENTS.md`/`llms.txt` |
| **v1.0** | 23–28 | Figma Foundations + Components libraries, Community file; ContrastMate v2 Token Auditor/Sync; ACR v1; docs complete; case study; portfolio page |
| v1.1 | 29–34 | Calendar/DatePicker; density; Starlight site if needed |

Total to v1.0 ≈ 28 weeks (~7 months). If the Cal.com role is time-sensitive, v0.2 + demo + 1 upstream PR (week ~13) is the earliest credible "apply" point.

**Exit criteria per milestone (no exceptions):**

| Milestone | Must be true before tagging |
|---|---|
| M0 | Every spike in §6 has a written result in `decisions/`; any failed spike has its decision-table row revised here |
| v0.1 | `npm i @varna-ui/react @varna-ui/tokens` works in a fresh Vite app with provenance badges; contrast gate demonstrably fails on a bad PR (keep the PR link); 4 components meet full DoD incl. manual SR log |
| v0.2 | Dialog/Select pass iOS VO + TalkBack manual pass; ACR skeleton published; Playwright matrix is a required check |
| v0.3 | Demo booking flow completes end-to-end by keyboard only and with NVDA; one upstream PR merged or in review |
| v0.4 | All 12 components meet DoD; `npx shadcn add` from the varna-ui registry works; size budgets enforced |
| v1.0 | Figma Community file published with variables matching `tokens.resolved.json` (checked by the Token Auditor); ACR v1; README leads with proof links, not component count |

**M0 week plan (evenings):**
- Days 1–2: claim npm org/GitHub repo; pnpm workspace; TS 6.0 / ESLint 9 / Prettier; CI `static` job; ADR-001 (Base UI), ADR-002 (CSS strategy), ADR-003 (DTCG in git).
- Days 3–5: `@varna-ui/contrast` ported with tests; sample DTCG tokens; SD5 build for 4 themes; `pairs.json` + failing-pair test; ADR-004 (WCAG normative, APCA informational).
- Days 6–8: tsdown build of a throwaway Button with CSS Module; `use client` check; publint/attw; Storybook 10.6 + addon-vitest + axe error; first OIDC publish of `@varna-ui/contrast@0.0.1`.
- Days 9–12: raw Base UI Select + Menu on iOS VoiceOver/TalkBack; Playwright forced-colors project; Sinhala/Tamil line-height measurement story; ADR-005 (React 19-only).
- Days 13–14: write spike results; re-baseline §5 estimates.

**Upstream PR candidates (for rank-5 differentiator, pick 2–4):** coss ui forced-colors borders on Button/Input; coss ui focus-ring contrast in dark mode; Base UI Toast a11y guide (#4229 asks for one); Base UI #4184 Progress/Meter label announcement repro + fix; Base UI docs: note that WCAG 2.2 AA conformance requires WCAG 2 ratios, not APCA (their a11y page currently recommends APCA). Each PR links back to an akuru regression test.

## 6. Top risks + spikes to run first (M0)

| Risk | Spike | Acceptance criteria |
|---|---|---|
| tsdown 0.x CSS + `'use client'` stripping | Build Button with CSS Module via tsdown | `dist/styles.css` emitted; `grep -r "use client" dist` finds the directive; publint + attw esm-only pass; fallback lightningcss script committed if not |
| Storybook 10.6 + Vite 8 + Vitest 5 + pnpm 12 combo | Scaffold `apps/docs`, one story with `play` + `a11y.test:'error'` | `vitest --project=storybook --run` fails on an injected axe violation and passes when fixed |
| Style Dictionary 5.6 DTCG 2025.10 (oklch object values, theme loop) | Build 4 themes from sample tokens | `tokens.css` with `:root`, `[data-theme]`, media wrappers; `hex` fallback present; TS types generated |
| Contrast gate correctness | Port `contrast.ts` + 41 tests into `@varna-ui/contrast`; write `pairs.json` test | A deliberately failing pair fails CI; coverage check flags an unpaired `fg.*` token; APCA column from `apca-w3` devDep only |
| Base UI Select/Menu on touch screen readers | Build raw Base UI Select + Menu, test iOS VoiceOver + TalkBack | Written transcript; either passes or workaround + regression test + known-issue entry drafted |
| Forced-colors emulation fidelity | Playwright project `forcedColors:'active'` on Button/Checkbox | Focus ring and control borders visible in screenshot; one real Windows HCM check logged |
| Sinhala/Tamil rendering | Story with Noto Sans Sinhala/Tamil strings in Button/Input at line-height 1.5/1.7/1.75 | No clipping at 200% zoom; measured values written into tokens; font subsetting plan noted |
| npm Trusted Publishing bootstrap | Publish `@varna-ui/contrast@0.0.1` | Provenance badge on npm page; no `NPM_TOKEN` secret in repo |
| Name availability | Create npm org `varna-ui`, GitHub repo | Both owned; fallback `pahan` if either fails |
| Scope creep / burnout | Cut line in §3; ADR per added component | No component starts before the previous meets DoD |

**Secondary risks (track, no spike):** Base UI minor releases change behaviour monthly → Renovate grouped PR + story tests as the safety net; Guidepup flake → nightly, `continue-on-error`; Windows HCM differs from Chromium emulation → one real check per minor logged in `ACCESSIBILITY.md`; coss ui AGPL → never open its source while writing akuru component code, only its docs; portfolio reads as "AI-generated sameness" → ADRs record every trade-off, visual identity is set in M0 (not Inter + indigo + rounded-xl); EN 301 549 OJEU citation date unknown → docs say "intended to help adopters meet", never a legal claim.

## 7. Owner decisions (resolved 2026-10-11)

1. React 19 only.
2. ContrastMate v2 replaces its re-implemented APCA constants with the `apca-w3` dependency and a licence notice. APCA stays informational.
3. The consumer app is a booking-flow demo inside the repo (`apps/demo`).
4. Calendar and DatePicker ship in v1.1.
5. A generated Tailwind v4 `@theme` bridge ships from v0.1.
6. The name is **varna-ui** (වර්ණ, "colour" in Sinhala), with packages `@varna-ui/tokens`, `@varna-ui/react` and `@varna-ui/contrast`. The README and docs lead with the Sinhala meaning, because "varna" also has a caste meaning in Sanskrit and Hindi.

## 8. Verified-vs-unverified ledger

| Claim the plan depends on | Status | Source |
|---|---|---|
| Base UI 1.9.0, monthly releases, shadcn default, coss ui official Cal.com DS (AGPL-3.0) | Verified | lead corrections; 01 §1.1/§1.3; 06 §1 |
| calcom/cal.com still public + MIT | Verified (lead) | overrides 01 §0.5/§3 |
| Cal.com Senior Product Design Engineer role open, worldwide | Unverified (Remotive vs Built In conflict) | 06 §1 |
| npm versions for the stack (SD 5.6.1, Terrazzo 2.7.1, Storybook 10.6.1, Vitest 5.0.3, Vite 8.3.4, tsdown 0.23.1, React 19.3.0, pnpm 12.11.2, Changesets 3.0.3, @axe-core/playwright 4.13.0, Guidepup 0.35.0, size-limit 14.2.0, culori 4.0.2, Leonardo 1.1.0) | Verified | lead corrections; 03 §1 |
| typescript-eslint peers TS <6.1; jsx-a11y peers ESLint ≤9 | Verified | lead; 03 §1 |
| apca-w3 0.1.9 "Limited W3 License"; ContrastMate re-implements constants | Verified (lead) | 00 §open questions |
| ContrastMate `contrast.ts` 450 lines, 41 tests, MIT | Verified (repo read) | 00 |
| Radix #1343 open since 2022-04-26; Base UI #5794 open | Verified | lead; 04 §3 |
| DTCG 2025.10 stable; Color module supports oklch; Resolver module exists | Verified | 02 §1 |
| Style Dictionary fully supports DTCG 2025.10 object values and resolver | Unverified (docs say "not full support"; no native resolver) | 02 §2.1, §7 |
| tsdown preserves `'use client'`; option names (`unbundle`, `css`, `attw`) | Unverified | 03 §3.5, §9 |
| Storybook `a11y.test:'error'` fails CI; addon-vitest browser mode | Verified | 03 §4.2–4.3 |
| npm Trusted Publishing requirements (`repository.url` match, id-token) | Verified; package-must-exist-first unverified | 03 §6.2 |
| Playwright `forcedColors`/`contrast` emulation per browser | Unverified (Chromium assumed) | 03 §5.4 |
| Figma Variables REST API Enterprise-only; Plugin API reads/writes variables on Pro | Verified | 05 §1 |
| Figma Pro mode cap 10; native DTCG import on Pro; Slots on Pro Full seat | Unverified / Secondary | 05 §1, §6 |
| Code Connect needs Org/Enterprise | Verified | 05 §3 |
| Community files default CC BY 4.0 | Verified | 05 §2 |
| WCAG 2.2 SC list (2.4.11, 2.5.7, 2.5.8, 3.3.8); EN 301 549 v4.1.1 adopts 2.2 AA | Verified | 04 §1 |
| APCA removed from WCAG 3 draft; WCAG 3 still draft | Verified (secondary for Roselli) | 02 §4.2 |
| Sinhala/Tamil line-height 1.6–1.75 needed | Unverified (single forum test) — measure in M0 | 02 §5.2; 06 §4 |
| No design system documents Sinhala/Tamil; no first-party coss Figma kit | Unverified (absence of evidence) | 01 §2.2, §5 |
| Radix/shadcn/Base UI lack documented forced-colors support | Unverified (absence of evidence) | 01 §2.2 |
| axe catches ~57% of issue volume (Deque 2021) | Verified (vendor study) | 04 §2.3 |
| npm names `varna-ui`, `pahan`, `sevana` free; npm/GitHub org availability | Names verified; orgs unverified | lead; 06 §5 |
| Chromatic free tier 5,000 snapshots | Verified | 03 §4.4 |
| Cal.com role lists Base UI first and asks for a repo/side project | Verified | 06 §1 |
