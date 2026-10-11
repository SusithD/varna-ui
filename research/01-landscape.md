# Landscape research: an open-source React design system as a portfolio piece

Research date / "access date" for every URL and number below: **2026-10-11**.
Method: live npm registry queries, GitHub API, official docs/changelogs fetched directly, plus web search. Anything I could not confirm from a primary source is tagged **[UNVERIFIED]**. Claims taken from SEO/vendor blogs are tagged **[BLOG CLAIM]**. Statements from my own background knowledge (not re-checked this session) are tagged **[BACKGROUND]**.

---

## 0. Executive summary (read this first)

1. **Foundation: use Base UI (`@base-ui/react`), not Radix.** In the last 12 months the market moved. shadcn/ui made Base UI the default for new projects (3 Jul 2026), and **Cal.com's own design system, coss ui, is built on Base UI** and their open design-engineer job lists Base UI first among required primitives. Base UI is v1.9.0 (9 Oct 2026), ~monthly minor releases, full-time MUI staff, and ~16.8M weekly npm downloads (same order as the `radix-ui` meta-package at ~16.8M).
2. **Radix is not dead, but it is a weaker bet than a year ago.** It is actively committed to (latest commits 9 Oct 2026), but ~90 of the last 100 commits are from one person, and the `radix-ui` meta-package went ~10 months with no stable release (1.4.3 on 13 Aug 2025 to 1.5.0 on 6 Jun 2026). Radix Themes is quiet (v3.3.0, Jan 2026; last push Apr 2026).
3. **React Aria Components is the a11y/i18n benchmark** (screen-reader matrix on 6 pairs, 30+ locales, RTL, 13 calendar systems) and is the strongest foundation if your pitch is *"accessibility-first and i18n-first"*. The trade-off is API ergonomics, bundle/complexity, and zero alignment with Cal.com. Recommendation: **Base UI as the layer; study React Aria's tests/docs and borrow `@internationalized/date` for the calendar stretch component.**
4. **Do not build "another shadcn".** The copy-paste Base-UI-plus-Tailwind niche is occupied by shadcn/ui (125k stars) and by Cal.com's coss ui (10.7k stars, ~50 components). A portfolio DS wins by being *narrower and deeper*: a11y evidence, contrast-guaranteed tokens, forced-colors, motion-safe, RTL and Sinhala/Tamil, and your own Figma kit + ContrastMate tie-in.
5. **Cal.com context changed in 2026.** Cal.com closed-sourced its main repo in April 2026; `@calcom/ui` is a private, unpublished workspace package (not a learning target any more). The learning target is **coss ui** (AGPL-3.0, so *read and learn, do not copy code* into an MIT library).
6. **Scope: 12 core components + 1 signature (Calendar/Date picker), fully "done" beats 30 half-done.** See section 4.

---

## 1. Headless / primitive libraries

### 1.1 Hard numbers (queried 2026-10-11)

| Library | npm package | Latest (date) | Weekly downloads (3-9 Oct 2026) | GitHub stars / last push | Notes |
|---|---|---|---|---|---|
| Radix Primitives | `radix-ui` (meta) / `@radix-ui/react-*` | 1.7.0 (5 Oct 2026) | 16.75M (`radix-ui`); 74.2M (`@radix-ui/react-dialog`, legacy per-package installs) | 19.4k / 9 Oct 2026 | Maintained by WorkOS (README). 129 open issues / 134 open PRs (WebFetch of repo page) |
| Radix Themes | `@radix-ui/themes` | 3.3.0 (31 Jan 2026) | n/a | 8.8k / 11 Apr 2026 | Slow: no release in ~8 months |
| Base UI | `@base-ui/react` | 1.9.0 (9 Oct 2026) | 16.81M | 11.1k / 11 Oct 2026 | v1.0.0 on 11 Dec 2025; 37 components in sidebar; 478 open issues |
| React Aria Components | `react-aria-components` | 1.22.1 (9 Oct 2026) | 4.86M | 15.9k (monorepo) / 9 Oct 2026 | Adobe; also powers Spectrum 2 (`@react-spectrum/s2` 1.8.1) |
| Headless UI | `@headlessui/react` | 2.2.10 (13 Apr 2026) | 7.48M | 28.8k / 13 Apr 2026 | Tailwind Labs; no push for ~6 months, small component set |
| Ariakit | `@ariakit/react` | 0.4.41 (5 Oct 2026) | 1.32M | 8.6k / 11 Oct 2026 | Still 0.x; single-maintainer-led [BACKGROUND] |
| Ark UI | `@ark-ui/react` | 5.39.3 (6 Oct 2026) | 0.81M | 5.4k / 10 Oct 2026 | Zag.js state machines; multi-framework (React/Vue/Solid/Svelte); powers Chakra v3 and Park UI |

Sources: npm registry + `api.npmjs.org/downloads`, `api.github.com/repos/...` (all accessed 2026-10-11); https://base-ui.com/react/overview/releases ; https://github.com/radix-ui/primitives . Caveat: Radix's per-package downloads are inflated by transitive installs via older shadcn/ui; the `radix-ui` and `@base-ui/react` rows are the fairest comparison.

### 1.2 Radix Primitives: honest maintenance assessment

Evidence it is alive:
- Commits on 9 Oct 2026 (Slider, Select, Accordion migrating to a new internal collection implementation; GitHub API).
- `radix-ui` 1.5.0 (6 Jun 2026) to 1.7.0 (5 Oct 2026) with many release candidates in between.
- Official release notes for Jun-Jul 2026 list real a11y/React 19 fixes: Dialog no longer leaves broken ARIA references when title/description are absent; Toast no longer removes unfocused toasts on Escape; Slider keyboard stepping snaps to valid steps; Slot/Presence loops fixed on React 19; `use client` directives added; subpath entry points + tree-shaking (https://www.radix-ui.com/primitives/docs/overview/releases).
- shadcn shipped a "Unified Radix UI Package" entry in Feb 2026 and states Radix "stays fully supported" (https://ui.shadcn.com/docs/changelog).

Evidence for caution:
- **Bus factor**: of the 100 most recent commits (GitHub API, window since 1 Jan 2026), 90 are by one author (Chance Strickland, a WorkOS engineer per a Nov 2025 podcast transcript). Source concentration is a risk, not proof of decline.
- **Release gap**: no stable `radix-ui` release between 1.4.3 (13 Aug 2025) and 1.5.0 (6 Jun 2026), by npm publish times. Only RCs in between.
- Radix Themes quiet (above); the Radix blog's only post is Themes 3.0 (Mar 2024).
- Third-party narrative that "Radix slowed after the WorkOS acquisition" exists (https://dev.to/mashuktamim/is-your-shadcn-ui-project-at-risk-a-deep-dive-into-radixs-future) **[BLOG CLAIM, opinion; the commit/release data above is what I'd rely on]**.
- Known functional gaps people cite: no native Combobox/Autocomplete primitive (Select has no search), Radix Select "native"-style limits **[BLOG CLAIM: https://www.pkgpulse.com/guides/shadcn-ui-vs-base-ui-vs-radix-components-2026]**.
- Known a11y issues: I did not find a curated, reliable list. GitHub issue search returned nothing useful via API. **[UNVERIFIED]** - treat "Radix has known a11y bugs" as unproven; do your own VoiceOver/NVDA pass.

### 1.3 Base UI: v1 status

- v1.0.0 on **11 Dec 2025**, "35 unstyled UI components"; now 37 listed; releases 1.1 (15 Jan) to 1.9 (9 Oct 2026), i.e. roughly monthly (https://base-ui.com/react/overview/releases).
- Package renamed `@base-ui-components/react` to `@base-ui/react` at v1 (https://infoq.com/news/2026/02/baseui-v1-accessible/).
- MUI says Base UI is "a major area of focus" in 2026 and the long-term direction for foundational unstyled components; MUI Base is deprecated (https://mui.com/blog/2026-and-beyond/).
- a11y: components follow WAI-ARIA APG; v1.6.0 (Jun 2026) aligned Accordion keyboard nav with APG; iOS VoiceOver voice-control fix in beta.7. Its a11y guide **explicitly recommends APCA for contrast "unless stricter standards apply"** (https://base-ui.com/react/overview/accessibility) - a direct hook for your ContrastMate background.
- Has a "Direction Provider" utility (listed in nav; details not fetched) **[UNVERIFIED depth of RTL coverage]**.
- Built-in Combobox/Autocomplete (full ARIA combobox pattern per [BLOG CLAIM]), Field/Fieldset/Form with automatic label association, Toast, Menubar, PreviewCard, etc.
- Style hooks: `data-*` attributes, `render` prop for composition (replaces Radix `asChild`), `Positioner/Popup` split for overlays.
- Migration cost from Radix is real (asChild to render, nullable Select values, behavior differences such as manual tab activation) **[BLOG CLAIM]**; irrelevant to you since you start fresh.

### 1.4 React Aria / React Aria Components (Adobe)

- Most rigorous documented a11y QA: tested on **VoiceOver (macOS Safari/Chrome), JAWS + NVDA (Windows Firefox/Chrome), VoiceOver iOS, TalkBack Android Chrome**; normalizes press/hover/focus across input modalities (https://react-aria.adobe.com/quality).
- i18n: translated built-in strings in **30+ locales**, RTL mirroring and interactions, locale-aware formatting/collation, `@internationalized/date` with **13 calendar systems** (https://react-spectrum.adobe.com/react-aria/internationalization.html). Sinhala/Tamil built-in strings: **[UNVERIFIED - not in the docs I found; test the locale list]**.
- Now a first-class base in shadcn (`init --base aria`, July 2026) and powers HeroUI v3 (https://ui.shadcn.com/docs/changelog/2026-07-react-aria).
- Costs: bigger runtime, `Collection`/`Key`-based API with its own mental model, strong opinions (e.g., `onPress` vs `onClick`), RSC/SSR string-bundle handling needs a build plugin.

### 1.5 Others (short)

- **Headless UI**: great for Tailwind Plus users, but ~10 components, no push since Apr 2026; not a foundation for a 12-component system.
- **Ariakit**: excellent Combobox/Composite/Select, composable `render`-prop API (influenced Base UI's `render`), but still 0.x and smaller community.
- **Ark UI / Park UI**: broad component count (~45+ with date/color pickers, tree view etc.), framework-agnostic via Zag state machines; a11y good but fewer community eyeballs; Park UI is Panda-CSS-based (preset 0.43.1). Choose only if multi-framework (Vue!) is a goal. **Note**: your Vue/Nuxt background makes Ark UI interesting for a later Vue port, but it hurts the Cal.com alignment.
- **Bits UI / Melt UI** (Svelte): out of scope (non-React).

### 1.6 Comparison

| Criterion | Radix | Base UI | React Aria Components | Ariakit | Headless UI | Ark UI |
|---|---|---|---|---|---|---|
| a11y rigor (documented) | Good, APG-based; no public AT matrix | Good, APG-based, "tested across screen readers" (vague) | **Best**: public 6-pair AT matrix, long track record | Very good on Combobox/Composite | Good, small surface | Good (state machines), less scrutiny |
| i18n / RTL | RTL `dir` props; no locale layer | `DirectionProvider`; no locale layer **[depth UNVERIFIED]** | **Best**: 30+ locales, 13 calendars | RTL ok [BACKGROUND] | Minimal | Locale support in date components via Zag |
| API style | Compound parts, `asChild` | Compound parts, `render` prop, `Positioner/Popup` | Compound RAC parts + render props/`data-*` states, Collections | Store + render props | Compound | Compound + `asChild`, machine-driven |
| Composability | Good | Good; closest successor to Radix | Very good but heavier | Excellent | Limited | Good |
| Component breadth | ~30 (no Combobox) | 37 incl. Combobox, Autocomplete, Menubar, Toast | ~50 incl. Calendar, DatePicker, Table, GridList, Tree | ~25 | ~10 | ~45+ |
| Styling | Unstyled | Unstyled | Unstyled (`data-*`, className fns) | Unstyled | Unstyled | Unstyled |
| Maintenance (Oct 2026) | Active, single main committer, 10-mo release gap | **Very active, funded, monthly** | **Very active, Adobe** | Active | Slow | Active |
| Ecosystem pull | shadcn legacy base | **shadcn default + Cal.com coss ui** | shadcn option, HeroUI v3, Spectrum 2 | niche | Tailwind Plus | Chakra v3, Park UI |

### 1.7 Verdict - which primitive layer?

**Base UI is the best foundation for *this* project in 2026.** Reasoning:
1. **Career fit**: Cal.com's design-engineer listing names Base UI first and says the team builds on coss ui, which is Base UI-based; the application asks for "a repo, component library, or side project" (https://remotive.com/remote/jobs/design/senior-product-design-engineer-5997860, accessed 2026-10-11; posting date unclear, ~2 weeks old at capture; **[UNVERIFIED whether still open]**). Building on the same primitive shows instantly relevant fluency.
2. **Momentum + funding**: monthly releases, full-time MUI team, shadcn default.
3. **Completeness**: it already has the awkward components (Combobox, Toast, Field/Form) that Radix lacks or is still migrating.
4. **Honest caveat on a11y**: React Aria is *stronger on evidence* (published AT matrix, i18n depth). Base UI's a11y is good but less documented. That is a gap *you* can fill: run your own VoiceOver/NVDA/axe/forced-colors matrix on top of Base UI and publish it per component. That is a better story than "I used the best library".
5. **Radix** remains defensible but "I chose Radix in late 2026" now needs a justification; "I chose Base UI like Cal.com and shadcn" does not.

Alternative if the pitch is purely a11y/i18n: React Aria Components. Do not mix layers in v1 (except borrowing `@internationalized/date` for the Calendar stretch).

---

## 2. Styled systems and copy-paste kits

### 2.1 What exists (Oct 2026)

| System | Foundation | Styling | Distribution | State (verified) | Notes |
|---|---|---|---|---|---|
| **shadcn/ui** | Radix **or Base UI or React Aria** (choice at init) | Tailwind v4 (Feb 2025), CSS variables, `cn` util (Sep 2026) | CLI + registry (`shadcn` CLI v4 Mar 2026), private GitHub registries (Aug 2026), presets/`apply`, 8 styles (Vega, Nova, Maia, Lyra, Mira, Luma, Rhea, Sera), RTL support (Jan 2026), `@shadcn/helpers`; "Base UI as default" on 3 Jul 2026 | 125k stars; pushed 10 Oct 2026 | Changelog: https://ui.shadcn.com/docs/changelog |
| **coss ui (Cal.com)** | Base UI | Tailwind | Copy-paste via shadcn CLI-compatible registry + MCP server; layers: Primitives / Particles / Atoms | 10.7k stars; AGPL-3.0; ~50 components | https://coss.com/ui/docs ; repo `cosscom/coss` |
| **Mantine 9** | Own (not Radix) | CSS Modules, CSS variables, style props | npm package | 9.7.2 (11 Oct 2026); requires React 19.2+ [BLOG CLAIM] | Very broad (100+ components, hooks, charts, dates, new `@mantine/schedule`) |
| **Chakra UI v3** | Ark UI / Zag | Panda CSS | npm + snippets | 3.37.0 (28 Aug 2026) | v3 rewrite moved off Emotion |
| **Park UI** | Ark UI | Panda CSS (or Tailwind) | copy-paste | preset 0.43.1 | Chakra v3 flavored, shadcn-like DX, multi-framework |
| **MUI (Material UI)** | Own + Base UI direction | Emotion / pigment | npm | active; new components built on Base UI | Material look; heavy |
| **HeroUI v3** (ex-NextUI) | React Aria | Tailwind v4 | npm | 3.2.6 | ~71 components [BLOG CLAIM, vendor] |
| **Catalyst (Tailwind Plus)** | Headless UI | Tailwind | Paid, copy source | **[UNVERIFIED: could not retrieve current status]** | Commercial; not a competitor you can match on polish |

### 2.2 Where the gaps are (ranked by credibility for a solo author)

| Gap | Evidence of gap | Can a solo DS credibly fill it? | How |
|---|---|---|---|
| **Contrast-guaranteed tokens (WCAG 2.2 AA enforced + APCA reported)** | Salesforce SLDS 2 and Atlassian guarantee contrast at token level (SLDS docs; IT Brief Feb 2026); Radix Colors guarantees APCA Lc 60/90 for steps 11/12 on step 2 but gives **no WCAG ratio** (https://www.radix-ui.com/colors/docs/palette-composition/understanding-the-scale). shadcn/coss ship colors without published guarantees [BACKGROUND, not exhaustively checked] | **Yes - your strongest angle.** You wrote ContrastMate | Token build step that *fails CI* if any text/surface pair < 4.5:1 (WCAG 2.2) and reports APCA Lc; publish a generated contrast matrix page per theme |
| **APCA awareness, done responsibly** | WCAG 3.0 Working Draft (3 Mar 2026): contrast algorithm "yet to be determined"; APCA removed from draft in 2023 per Adrian Roselli (https://adrianroselli.com/2026/04/wcag3-contrast-as-of-april-2026.html ; https://www.w3.org/WAI/news/2026-03-03/wcag3). Bridge-PCA 0.1.6 is an unofficial beta | Yes, if framed correctly | **Normative gate = WCAG 2.2 AA. APCA = informational second column.** Do not claim "APCA compliant". Explaining that nuance is itself a hiring signal |
| **Forced-colors / Windows High Contrast themes** | General guidance shows shadcn-style transparent-border buttons vanish in forced colors; FAST/Fluent had dedicated support; I found **no** primary-source evidence that Radix/shadcn/Base UI/coss document forced-colors behavior (**absence of evidence, [UNVERIFIED]**) | **Yes** | Playwright `forcedColors: 'active'` + `colorScheme` + `reducedMotion` emulation tests and screenshots per component; system colors (`ButtonText`, `Highlight`, `CanvasText`); document `forced-color-adjust` decisions |
| **Motion-safe defaults** | React Aria's guide lists "respect motion preferences" as a manual check | Yes | All transitions behind `@media (prefers-reduced-motion: no-preference)`; token `--motion-duration-*` collapse to 0; test in CI |
| **RTL + logical properties** | shadcn added RTL support Jan 2026; React Aria supports RTL; most smaller kits still use physical `left/right` [BACKGROUND] | Yes | Logical CSS only (`inline-start`, `padding-inline`); Storybook `dir` toolbar; Playwright RTL snapshots |
| **Sinhala / Tamil (Indic script) support** | Searches found **no** design system documenting Sinhala or Tamil; React Aria's 30+ locale list not confirmed to include them **[UNVERIFIED]** | **Yes, and it's unique to you** (Sri Lanka) | Font stacks (Noto Sans Sinhala / Noto Sans Tamil), taller `line-height` and no clipping of stacked marks, no uppercase transform, `lang` attribute on regions, `Intl` (`si-LK`, `ta-LK`) in date/number formats, truncation that respects grapheme clusters (`Intl.Segmenter`), a11y labels translated. Be explicit about what you tested [BACKGROUND for the typographic claims; verify with real rendering] |
| **Per-component a11y documentation** | GOV.UK and Primer do guidance; React Aria has a quality page; copy-paste kits mostly don't show AT results per component | **Yes** | Each doc page: keyboard map, ARIA pattern link (APG), name/role/value, screen-reader transcript (VoiceOver + NVDA), known limitations, axe result badge |
| Breadth of components | Mantine/HeroUI/MUI/shadcn dominate | **No - don't compete** | |
| Theming engine / visual builder | shadcn/create, presets | No | |
| Multi-framework | Ark/Park | Not v1 (maybe Vue port as v2 story) | |

### 2.3 Strategic implication

Position: **"a small, accessibility-evidenced Base UI system: every component ships with a published a11y proof, contrast-gated tokens, forced-colors/reduced-motion/RTL/Indic-script tested"**. Not "a nicer-looking shadcn". Do **not** copy coss ui code or visual identity (AGPL-3.0; also looks derivative). Study its layering ideas (Primitives, Particles) instead.

Distribution recommendation: publish an **npm package with precompiled CSS** (consumable without Tailwind) *and* a small shadcn-compatible **registry** for copy-paste. Tokens as CSS custom properties (+ DTCG JSON source). Rationale: npm publication was in your brief and proves packaging skills; the registry shows you understand the 2026 distribution norm. If time is tight, ship npm first, registry second.

---

## 3. Notable open-source design systems: what to learn

Registry data from npm (accessed 2026-10-11): `@primer/react` 38.40.1 (7 Oct 2026, MIT, 3.9k stars), `@carbon/react` 1.118.0 (7 Oct 2026), `@shopify/polaris` 13.9.5 (10 Aug 2026), `@twilio-paste/core` 21.5.0 (22 Jul 2026), `govuk-frontend` 6.5.1, `@react-spectrum/s2` 1.8.1, `@mantine/core` 9.7.2. Descriptions of internals beyond the cited pages are **[BACKGROUND]** unless linked.

| System | Tokens | a11y practices | Docs site | Testing | Exemplary | Caveat |
|---|---|---|---|---|---|---|
| **GitHub Primer** (primer.style) | Tokens (`@primer/primitives`) with light/dark/colorblind/high-contrast themes [BACKGROUND] | Public a11y hub: foundations, Scenarios Framework, Annotation Toolkit, designer/engineer/mobile checklists, per-component guidance, motion, focus mgmt (https://primer.style/accessibility) | Excellent, a11y baked into every component page | Storybook, axe-based unit tests, visual regression [BACKGROUND] | **Best model for the a11y docs structure and colorblind/high-contrast themes** | Big team |
| **Shopify Polaris** | Tokens/CSS vars | Strong a11y guidance | polaris-react.shopify.com (legacy) | n/a | Content/UX guidelines | **Shifted to Polaris web components (`s-*`) from CDN; React package locked to React 18 per a Shopify community thread (undated); no official end-of-life notice found.** Treat as a cautionary tale about platform shifts (https://infoq.com/news/2026/10/shopify-web-components) |
| **Atlassian (ADS)** | `@atlaskit/tokens`, `--ds-*` CSS vars, `token()` helper; color tokens adjusted for contrast in light/dark (IT Brief 3 Feb 2026) | Dedicated a11y team in the DS for ~2 years; Figma annotation library (internal) | atlassian.design | internal | **Token-level contrast fixes inherited by components** | Source is not an open community project **[UNVERIFIED licensing]**; Figma libs for employees only |
| **GOV.UK Design System** | Sass/CSS | Gold standard for *evidence*: per-component research notes, assistive-tech testing, published accessibility strategy and statement (https://design-system.service.gov.uk/accessibility/ confirms strategy/statement pages; AT matrix details **[UNVERIFIED this session]**) | Component pages = guidance + research + code + "how it's built" | Tested with real AT and users [BACKGROUND] | **Per-component "Research on this component" + progressive enhancement** | Not React |
| **Adobe Spectrum 2 / React Aria** | Style macros (`@react-spectrum/s2`), design tokens | AT matrix (VoiceOver, JAWS, NVDA, TalkBack); i18n/RTL | react-aria.adobe.com | Cross-browser/AT QA; documents automated-tool false positives (https://react-aria.adobe.com/quality) | **Interaction normalization + i18n** | Heavy |
| **IBM Carbon** | Tokens/themes (WCAG 2.1 AA aim), IBM Accessibility Checklist; Equal Access Toolkit (https://carbondesignsystem.com/guidelines/accessibility/overview/) | Checklist-driven, component-level a11y tabs [BACKGROUND] | carbondesignsystem.com | Automated + manual [BACKGROUND] | **Process rigor (checklist per component)** | Enterprise scale |
| **Radix Themes** | CSS vars + Radix Colors (12-step, APCA-guaranteed 11/12) | Inherits Primitives | radix-ui.com/themes | n/a | **Color scale design: the 12-step semantic scale is the model for your tokens** | Slow release cadence (v3.3.0 Jan 2026) |
| **Twilio Paste** | Design tokens, theme packages | Strong a11y statement, a11y-annotated docs | paste.twilio.design | Jest + axe + Chromatic [BACKGROUND] | **Docs with a11y per component; monorepo of tiny packages** | Maintained (v21.5.0, Jul 2026) but 2025-26 roadmap **[UNVERIFIED]** |
| **Salesforce SLDS 2** | "Global styling hooks" `--slds-g-*` (CSS custom properties), Cosmos theme; component hooks not supported yet | Global color hooks align to WCAG 2.1 contrast; warns that overriding generated dark-mode colors voids guaranteed contrast (https://www.salesforce.com/blog/what-is-slds-2/) | lightningdesignsystem.com | n/a | **Contrast guarantee as a contract tied to hooks** | Incomplete coverage (Visualforce, Experience Cloud) |
| **Wise (Neptune)** | Theming package `@wise/components-theming` 1.10.3 | **[UNVERIFIED]** | **[UNVERIFIED]** | **[UNVERIFIED]** | Known for strong content/UX principles [BACKGROUND] | I could not verify package names/details; skip unless you research further |
| **Cal.com `@calcom/ui`** | n/a | n/a | n/a | n/a | n/a | **Private package (`"private": true`, version 0.0.0) in `calcom/cal.diy`, ~50 component entrypoints, Radix-era. Main repo closed-sourced April 2026 (https://cal.com/blog/cal-com-goes-closed-source-why).** `packages/coss-ui` also present in cal.diy. Use **coss ui** as the reference |

Cross-cutting patterns worth copying:
- Tokens are **tiered** (primitive to semantic to component); components consume only semantic tokens (Atlassian, Primer, SLDS).
- **Contrast is guaranteed at the token layer** and *tested in CI*, not left to consumers.
- Docs have **an a11y section per component** with keyboard map and ARIA pattern (Primer, GOV.UK, React Aria).
- **Evidence beats claims**: AT matrices and research notes (GOV.UK, React Aria).

---

## 4. Portfolio angle

### 4.1 What impresses vs red flags

Search found little design-system-specific hiring evidence; general guidance (hiring-manager guides, Boot.dev) says: 2-3 deep projects over many shallow ones; at least one *library*; tests, clean code and READMEs; measurable results; broken links and "template with swapped colors" are fast rejections ([setproduct](https://www.setproduct.com/blog/hiring-frontlines-to-craft-a-portfolio), [boot.dev](https://www.boot.dev/blog/jobs/libraries-and-packages-in-coding-portfolio), [uxplanet](https://uxplanet.org/good-ux-portfolios-never-get-interviews-6ea28fb2f9b3); mostly generic). The design-system-specific rows below are **my judgment [BACKGROUND]**.

| Impressive | Red flag |
|---|---|
| 12 components, each with docs, tests, a11y notes, Figma parity | 40 components, half undocumented / no tests |
| Published npm package with semver, changelog, provenance, working install in a CodeSandbox/StackBlitz | "Coming soon" components, empty Storybook stories, TODOs in the README |
| CI badge: axe + Playwright (forced-colors, RTL, reduced-motion) + visual regression | Axe passes only because stories are trivially simple (the classic "axe green != accessible" trap) |
| Real AT evidence: VoiceOver/NVDA notes per component, with honest known limitations | Blanket "WCAG AAA compliant" claims |
| Decision log / ADRs (why Base UI, why WCAG-gated tokens, why APCA informational) | A visual copy of shadcn/coss (AGPL risk and derivative look) |
| One realistic demo app (e.g., a mini booking flow) built only from the system | Gallery only; no usage in a product context |
| Figma kit bound to the same tokens (variables), with component props matching code | Figma and code diverge; Figma kit unmaintained |
| Narrow scope stated up front ("v1 = 12 components + roadmap") | Endless roadmap, many open `wip` branches |
| Lists what it does *not* do | Scope creep: theme builder, charts, data grid |

Note on seniority: the Cal.com listing is "Senior Product Design Engineer" ($100k-150k + equity, worldwide remote; same source). Your product-design + engineering + a11y + open-source profile fits their "product designer who has moved substantially into design engineering" description, but you should expect a senior bar. The portfolio piece must read as *shipped*, not exploratory. Their stated stack/skills: HTML, CSS, TypeScript, React, headless primitives (Base UI/Radix), WCAG/ARIA, motion, dark mode; bonus for open-source, scheduling/timezone expertise, `Intl.DateTimeFormat` (https://remotive.com/remote/jobs/design/senior-product-design-engineer-5997860). **This makes a locale-aware Calendar/Date picker the single most on-target "signature" component.**

### 4.2 Recommended v1 scope (12 core + 1 signature), in build order

Order rationale: foundations first (tokens + Button prove the pipeline), then form controls (highest a11y value), then overlays (hardest focus management), then composite widgets.

| # | Component | Base UI part | Why it's in | Key a11y/i18n showcase |
|---|---|---|---|---|
| 0 | **Tokens + themes** (not a component) | n/a | Everything depends on it | Light, dark, high-contrast, forced-colors; CI contrast gate; DTCG to CSS vars (+ Figma variables) |
| 1 | Button (+ IconButton, Link-button) | Button | Proves tokens, focus ring, pending/disabled semantics | Visible focus (WCAG 2.4.11/2.4.13 area), 24px target (2.5.8), forced-colors border |
| 2 | Field (Label, Description, Error) + Input + Textarea | Field, Input | Foundation for all forms | Label association, `aria-describedby`, error announcement, logical padding |
| 3 | Checkbox + Radio Group + Switch | Checkbox, Radio, Switch | Cheap, high-impact, easy to get subtly wrong | Group semantics, indeterminate state, forced-colors indicators |
| 4 | Select | Select | Hard; users compare it | Typeahead, RTL, long labels incl. Sinhala/Tamil |
| 5 | Dialog (+ AlertDialog) | Dialog, AlertDialog | The classic focus-trap test | Focus return, scroll lock, Escape, reduced motion |
| 6 | Popover + Tooltip | Popover, Tooltip | Positioning, touch behavior | Tooltip is supplemental only (WCAG 1.4.13) |
| 7 | Menu (Dropdown) | Menu | Roving focus, typeahead | APG menu button pattern, RTL arrow keys |
| 8 | Tabs | Tabs | Keyboard model, automatic vs manual activation | APG tabs; RTL arrow reversal |
| 9 | Toast | Toast | Live regions, a common failure | `role=status` vs alert, pause on hover/focus, 2.2.1 timing |
| 10 | Combobox | Combobox | The "gap Radix leaves" | ARIA 1.2 combobox pattern; screen-reader transcript |
| 11 | Accordion | Accordion | Quick win, APG alignment | Heading structure, `hidden=until-found` |
| 12 | Badge / Avatar / Skeleton / Progress (one batch, tiny) | Avatar, Progress | Fill-out; low risk | Text alternatives, `prefers-reduced-motion` |
| S | **Calendar + DatePicker** (stretch, v1.1 if time-boxed) | Compose (Base UI has no date picker; use `@internationalized/date` or `Intl`) | Cal.com scheduling relevance | Locale-aware (`si-LK`, `ta-LK`, `ar`), RTL, week start, timezone display, grid keyboard model (APG date picker dialog) |

Cut line: if behind schedule, drop 12 and 11; **never** drop 5, 9, 10 (they carry the credibility). Explicit non-goals for v1: Table/DataGrid, charts, rich text, theme builder, Vue port.

### 4.3 Definition of "done" per component

A component is done only when all boxes are checked (put this checklist in the repo and render it on each docs page):

1. **API**: documented props, forwarded refs, `className`/`render` composition, controlled and uncontrolled, TypeScript types exported, no `any`.
2. **Tokens only**: no hard-coded colors/spacing/durations; works in light, dark, high-contrast themes.
3. **States**: default, hover, focus-visible, active, disabled, loading/pending, invalid, read-only, and empty/overflow (long text, Sinhala/Tamil strings).
4. **Keyboard**: full keyboard map documented against the APG pattern; tested with `userEvent` in Vitest/Storybook interaction tests.
5. **Automated a11y**: `@storybook/addon-a11y` + axe-core (v4.14.0; Storybook 10.6.1 as of today) on **every story including open/error states**, and Playwright + `@axe-core/playwright` on built Storybook for overlays. Automated tools catch only a subset of issues (React Aria's own docs note false positives and manual checks), so this is necessary, not sufficient.
6. **Manual AT pass**: VoiceOver (macOS Safari) + NVDA (Windows Firefox/Chrome) minimum, with a short written transcript and known limitations in the docs. Optional: TalkBack/iOS VoiceOver.
7. **Environment matrix** in CI (Playwright): `forcedColors: 'active'`, `colorScheme: dark`, `reducedMotion: 'reduce'`, `dir="rtl"`, 200% zoom/320px reflow screenshot.
8. **Contrast**: all text/graphics pairs pass WCAG 2.2 AA (4.5:1 / 3:1 for UI components); APCA Lc reported in docs.
9. **WCAG 2.2 checks relevant to the component**: focus not obscured (2.4.11), target size (2.5.8), dragging alternatives (2.5.7), consistent help etc. where applicable.
10. **Docs page**: usage, anatomy, do/don't, a11y section (keyboard table, ARIA, SR notes), RTL + Indic-script example.
11. **Figma parity**: matching component set in the Figma kit with variants = props, bound to the same variables.
12. **Release hygiene**: Changesets entry, semver, changelog, bundle-size budget checked (size-limit or similar), tree-shakeable exports, published with npm provenance.

### 4.4 Technical stack recommendation (brief)

- React 19, TypeScript, `@base-ui/react` 1.x pinned with caret and a weekly Renovate/Dependabot (Base UI ships monthly, so build the upgrade habit and show it).
- Tokens: DTCG JSON to Style Dictionary to CSS custom properties (+ Figma variables sync). CI step: contrast gate (WCAG 2.2) + APCA report; reuse your ContrastMate logic.
- Styling: Tailwind v4 (4.3.3 today) using `@theme` tokens, mirroring the Cal.com/coss stack; ship precompiled CSS in the npm package. (Trade-off: Tailwind-in-library is awkward for consumers, mitigated by precompiled CSS; plain CSS with `@layer` is the more portable alternative. Choose Tailwind if the Cal.com signal matters more.)
- Storybook 10 + `addon-a11y` + `addon-vitest`; Playwright for the environment matrix and visual regression; Changesets for releases.
- Docs: Storybook as the component reference plus a small docs site (per-component a11y pages, token/contrast matrix, decision log).

---

## 5. Open questions / things I could not verify

- Whether the Cal.com Senior Product Design Engineer role is still open (posted ~2 weeks before capture; check Cal.com's careers page/Breezy link).
- Radix Primitives' concrete open a11y bugs (no reliable list found); do hands-on testing before making any claim about them.
- Base UI `DirectionProvider` depth; whether React Aria's 30+ locales include Sinhala/Tamil; any design system's Sinhala/Tamil documentation (none found).
- Catalyst/Tailwind Plus current status; Wise's design system internals/packages; Atlassian licensing; Twilio Paste's 2026 roadmap.
- Whether the Cal.com team's coss ui has published Figma assets (not found).
- Exact download figures are point-in-time weekly numbers; Radix per-package figures include transitive installs.

## 6. Sources (all accessed 2026-10-11 unless noted)

- npm registry + downloads API (`registry.npmjs.org`, `api.npmjs.org/downloads/point/last-week/...`); GitHub REST API for repo stats, commits, releases.
- Base UI releases: https://base-ui.com/react/overview/releases ; a11y: https://base-ui.com/react/overview/accessibility ; MUI: https://mui.com/blog/2026-and-beyond/ ; InfoQ: https://infoq.com/news/2026/02/baseui-v1-accessible/
- Radix: https://github.com/radix-ui/primitives ; https://www.radix-ui.com/primitives/docs/overview/releases ; https://www.radix-ui.com/blog ; Colors: https://www.radix-ui.com/colors/docs/palette-composition/understanding-the-scale
- shadcn: https://ui.shadcn.com/docs/changelog ; https://ui.shadcn.com/docs/changelog/2026-07-react-aria ; https://www.shadcnblocks.com/blog/introducing-base-ui-and-component-styles (5 Feb 2026)
- Comparison blogs [BLOG CLAIM]: https://www.pkgpulse.com/guides/shadcn-ui-vs-base-ui-vs-radix-components-2026 ; https://dev.to/mashuktamim/is-your-shadcn-ui-project-at-risk-a-deep-dive-into-radixs-future
- React Aria: https://react-aria.adobe.com/quality ; https://react-spectrum.adobe.com/react-aria/internationalization.html ; https://react-spectrum.adobe.com/releases
- Cal.com: https://remotive.com/remote/jobs/design/senior-product-design-engineer-5997860 ; https://coss.com/ui/docs ; https://github.com/cosscom/coss (AGPL-3.0) ; https://github.com/calcom/cal.diy ; https://cal.com/blog/cal-com-goes-closed-source-why ; https://noqta.tn/en/blog/cal-com-closed-source-ai-open-source-reckoning-2026
- WCAG 3 / APCA: https://www.w3.org/WAI/news/2026-03-03/wcag3 ; https://adrianroselli.com/2026/04/wcag3-contrast-as-of-april-2026.html ; https://www.npmjs.com/package/bridge-pca
- Design systems: https://primer.style/accessibility ; https://carbondesignsystem.com/guidelines/accessibility/overview/ ; https://design-system.service.gov.uk/accessibility/ ; https://www.salesforce.com/blog/what-is-slds-2/ ; https://itbrief.co.uk/story/atlassian-embeds-accessibility-in-design-system-work ; https://infoq.com/news/2026/10/shopify-web-components
- Forced colors guidance: https://modern-css.com/high-contrast-support-without-broken-ui/ ; https://fast.design/docs/design-systems/high-contrast
- Portfolio guidance (generic): https://www.setproduct.com/blog/hiring-frontlines-to-craft-a-portfolio ; https://www.boot.dev/blog/jobs/libraries-and-packages-in-coding-portfolio
