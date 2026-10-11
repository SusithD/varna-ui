# 06 - Hiring Signals and Positioning (as of 2026-10-11)

All pages accessed 2026-10-11. Tags: [V] = read directly from the primary source this session; [S] = search-engine summary or aggregator, treat as secondhand; [U] = unverified / my inference.

## 1. Cal.com (the headline finding)

**Cal.com's design system is now coss ui, not @calcom/ui.**
- coss.com/ui is "the official design system of Cal.com". Repo `cosscom/coss`: 10.7k stars, 547 forks, 1,122 commits [V] https://github.com/cosscom/coss
- Stack of coss: Turborepo monorepo, Next.js, TypeScript, Bun, Biome, Tailwind CSS, **Base UI** primitives (not Radix); legacy "Origin UI" (Radix, shadcn-style) lives in `apps/origin`. `apps/ui` and `apps/origin` are MIT; default license AGPLv3 [V]
- Distribution is shadcn-registry style (copy source, install with the shadcn CLI). Docs index showed 55 components plus 510 "particles"; sources disagree on counts (28 / 55 / 362) [V]/[S] https://coss.com/ui
- Positioned as "built for developers and AI"; Cal.com is "progressively adopting" it (launched with Cal.com v5.8) [S] https://next.cal.com/blog/calcom-v5-8
- Base UI itself hit stable 1.0 in early 2026, 1.1 shortly after; 35 components [S] https://infoq.com/news/2026/02/baseui-v1-accessible/ , https://mui.com/blog/2026-and-beyond/ . Earlier coss warnings of "early development" may be stale.
- Legacy `@calcom/ui` (calcom/cal.com `packages/ui`, 48.9k-star repo; Next.js, tRPC, Prisma, Tailwind, Turborepo, Yarn): deps include Radix primitives (avatar, checkbox, dialog, popover, select, slider), cva 0.4, tailwind-merge, cmdk, sonner, react-hook-form, react-day-picker, tanstack table, lexical; **no Storybook** [V] https://raw.githubusercontent.com/calcom/cal.com/main/packages/ui/package.json . Note the versions are old, so this file may not reflect the live tree [U].

**Open roles** (https://cal.com/jobs [V]): only two listed - Senior Product Design Engineer ($100k-$150k, remote) and IC4 Senior Full-Stack Engineer ($132k-$150k + equity). There is no "UI engineer" role.

**Senior Product Design Engineer, full text** (https://remotive.com/remote/jobs/design/senior-product-design-engineer-5997860 [V], salary same regardless of location, "Worldwide"; a Built In listing says US-only, so confirm on cal.com) :
- Owns core surfaces (booking flows, scheduling primitives, team/enterprise); builds prototypes and ships production primitives; maintains consistency across tokens, spacing, states, dark mode, motion; **owns accessibility: keyboard, focus management, semantics, contrast**; documents decisions in writing; improves coss ui.
- Must have: product designer who moved into design engineering; HTML/CSS/TypeScript/React; headless primitives (Base UI, Radix); built and maintained a design system; WCAG/ARIA; daily AI tool use (Claude Code, Cursor, v0); async written communication.
- Bonus: open-source contributions, side projects, scheduling/timezone expertise, locale-aware APIs such as `Intl.DateTimeFormat`.
- Apply with "a portfolio of work you've shipped, such as a repo, component library, or side project, and explain what you cared about and why."

**Fit read.** Susith matches an unusually large share of the must-haves (designer who codes, Figma + TS, a11y, contrast tooling). Gaps to close: seniority ("Senior", though the posting lets people apply with a repo), Base UI experience, and a visible maintained design system. The bonus list (locale-aware, timezone) maps onto the Sinhala/Tamil idea.

**Strategic implication.** Do not build a generic shadcn-style kit. The same company already publishes one, open-source, on Base UI, with 10k stars. The strongest play is (a) build on Base UI so the stack is directly comparable, (b) land a few merged PRs in `cosscom/coss` (a11y fixes, contrast, forced-colors, i18n/script issues) before applying, (c) be able to say "here is what I'd do differently and why" about a real trade-off. This is [U] as a recommendation but follows from the sources above.

## 2. What other companies ask for

| Company / role | Source | Key asks | Portfolio signal |
|---|---|---|---|
| Linear, Design Engineer (Web & Brand); posted 6+ months ago, may be closed | https://jobs.accel.com/companies/linear/jobs/34559943-design-engineer-web-brand [V] | taste, React + TS, end-to-end sites, motion/interaction in code, Figma to web, agentic coding tools. Regions: NA/Europe only | "portfolio or Twitter/X profile showing range of technical skills, creative use of code, prototyping" |
| Raycast, Design Engineer | search summary [S]; the raycast.com/jobs URL 404'd for me | web design system contribution, open-source maintenance, prefers writing a bit more code over third-party deps; EUR 100-135k; CET +/-3h | not stated |
| Supabase, Design Engineer | [S] job boards | Figma-to-code, production React, Next.js/Vite/Tailwind, developer-tool audience; remote-first | explicit: "portfolio of personal projects or side experiments that demonstrate curiosity and craft"; AI prototyping |
| Resend, Design Engineer | [S] | Figma, React, Tailwind, "refined taste"; Americas/Europe | not found |
| Clerk, Design Engineer (marketing) | [S] | 5+ yrs marketing sites, animation, Core Web Vitals; US/EU; closed | not found |
| Vercel, Design Engineer | [S] aggregators | Senior/Staff, 5+ yrs, Next.js, polish, a11y, performance, SaaS marketing surfaces | not found |
| Perplexity, Design Systems Lead | https://jobs.accel.com/companies/perplexity-2/jobs/85911993-design-systems-lead [V] | 50/50 designer-engineer, semantic tokens, scalable component APIs, WCAG, documentation-driven adoption; nice-to-have: Storybook, governance/contribution models, multi-platform tokens; owns the GitHub repo incl. versioning, releases, changelogs | none stated |
| Generic design-systems engineer postings (Zoom, Instacart, Haufe, others) | [S] https://www.themuse.com/jobs/zoom/senior-design-systems-engineer etc. | WCAG/ARIA/keyboard; Storybook; "semantic and primitive tokens, theming, reliable distribution"; a11y enforced by linters/automated checks; AI-assisted workflows | rarely required |

Not found / not verified: current Shopify (Polaris), Atlassian, GitHub (Primer), Stripe design-system or design-engineer postings; my search for these returned irrelevant hits. Re-search before applying.

**Cross-posting synthesis** (mixture of [V] and [S]; small sample):
1. Always: React + TypeScript, Figma fluency, WCAG/ARIA, HTML/CSS depth.
2. Frequently: semantic/primitive/component token layers and theming; Storybook (usually preferred, not required); versioning/changelog/release discipline; Tailwind and Next.js; written communication/docs.
3. New in 2026: explicit expectation of daily AI-tool use (Cal.com, Linear, Supabase, Perplexity). Show this in the repo (CLAUDE.md/AGENTS.md, llms.txt, a registry or MCP for the system); coss's "built for developers and AI" tagline confirms the trend.
4. Portfolio is explicit only at Cal.com ("a repo, component library, or side project"), Linear, Supabase. For others, the repo + live docs are the portfolio by default.
5. Geography: many roles are region-limited (Linear NA/EU, Raycast CET+/-3, Resend Americas/Europe, Clerk US/EU). Cal.com says worldwide; Supabase is 55+ countries. Prioritize those two for Sri Lanka, and plan on async-overlap proof (written docs, PR discipline).

## 3. What separates a real design-system project from a tutorial clone

Caveat: I found no first-hand posts by Nathan Curtis, Brad Frost, Jina Anne, or Vercel/Linear design engineers on "portfolio design system projects"; searches returned secondhand material. Treat below as practitioner consensus, not attribution to those people.

What reads as credible (sources: https://www.portfoliobox.com/magazine/how-to-present-a-design-system-portfolio-without , https://intodesignsystems.substack.com/p/how-to-build-a-strong-design-systems , https://adobe.design/ideas/ask-adobe-design-how-can-applicants-stand-out-when-applying-to-design-jobs [all S]):
- Lead with the problem and the outcome, not the component count.
- Show the system **in use** next to a shipped product screen; don't just link Figma/Storybook.
- Document failures, revisions, and "what I refused to include and why".
- State your role precisely; fewer, stronger projects.
- Start small: one component, one problem, one story.

Failure modes (https://rangle.io/blog/design-systems-adoption-pitfalls-and-recommendations , https://www.viget.com/articles/design-systems-avoiding-common-pitfalls/ , https://envylabs.com/insights/starting-design-system-dont-overthink-it [S]):
- Building too much too early; dozens of unused abstract components.
- Building in isolation with no consuming team; big-bang launch with no feedback loop.
- Governance imposed before anyone felt the pain.
- Stat quoted secondhand: Zeroheight 2026 report, 91% trust vs 38% widely adopted [S, check original]. Curtis "zero percent success for federated systems" is also secondhand [S, check original].

**Red flags to avoid (my synthesis, [U]):**
1. 40+ components cloned from shadcn/Radix with a recolor. Cap scope: ~12-15 components done to a very high standard beats 50 shallow ones.
2. No consumer. A design system with no real app is the clearest "tutorial" tell.
3. Storybook that is only auto-generated prop tables; no usage guidance, no do/don't, no a11y notes per component.
4. Claims without evidence ("WCAG AA compliant") and no automated proof (axe in CI, contrast assertions, keyboard tests, screen-reader test notes).
5. No versioning, no changelog, no CONTRIBUTING, no issues/roadmap, one giant initial commit.
6. Figma and code that disagree (token drift).
7. AI-generated sameness with no decisions recorded; keep a `decisions/` (ADR) folder.
8. Default Inter + indigo + rounded-xl look. The Cal.com posting asks for "design-led"; have a point of view visually.

## 4. Differentiation ideas, evaluated

Scale: Credibility (will reviewers believe/respect it), Effort, Signal (for Cal.com-type roles).

| # | Idea | Credibility | Effort | Signal | Notes |
|---|---|---|---|---|---|
| A | **Contrast-guaranteed tokens** (ContrastMate tie-in): token build step computes WCAG 2.x and APCA for every fg/bg semantic pair; build fails if below threshold; ship the contrast matrix in docs; optional Figma variables export | Very high; Susith already owns an OSS contrast tool | Medium (build script + docs UI) | **Highest**; directly hits "contrast" in Cal.com's a11y bullet and "semantic tokens" everywhere | Prior art exists (Radix Colors, Leonardo, Material HCT), so frame as "tokens that cannot ship failing" with CI proof, not "new color science". Ship as a standalone package so it is useful outside the DS. |
| B | **Forced-colors / prefers-contrast theme** | High; rare in OSS React kits; Mozilla Acorn and FAST document the pattern (https://acorn.firefox.com/latest/desktop/design-tokens/how-design-tokens-work/overview-YYp6MVjt [S]) but not React-first | Low-medium if designed in from token layer (system colors `CanvasText`, `ButtonText`, `Highlight`; real borders/outlines under `forced-colors: active`) | High, a11y-specialist signal; cheap to demo with a Playwright `forcedColors: 'active'` screenshot suite | Do early: retrofitting is expensive. Pair with A: contrast matrix has a "high" column. |
| C | **Sinhala/Tamil script support** | High and unique to Susith; matches "locale-aware" bonus line at Cal.com | Medium | Medium-high as differentiator; narrow audience but memorable | Facts [S]: tall scripts clip at line-height 1.1, ~1.6 min suggested by a SuttaCentral forum test; fonts Noto Sans Sinhala/Tamil (@fontsource), Yaldevi (Sinhala+Tamil+Latin sharing vertical metrics, https://github.com/mooniak/yaldevi-font). Do: `:lang()` typography tokens, per-script line-height/letter-spacing, truncation and wrap tests, Intl number/date demos, RTL-ready logical properties. Avoid letter-spacing on Indic scripts and `text-transform` assumptions [U]. Keep it as a documented "multi-script" pillar, not the whole identity. |
| D | **Real consumer app** (migrate portfolio susith.theqexle.site and/or the PickMe prototype to the system) | Very high; fixes the #1 red flag | Medium | **Highest alongside A** | Do it from week 2, not at the end. Record friction as GitHub issues and write the case study from them. |
| E | **Documentation quality** (docs site with live examples, a11y notes per component, decision records, migration/changelog) | High | Medium, ongoing | High: Cal.com asks to "document decisions in writing"; Perplexity: "driving adoption through documentation" | Use Storybook or Fumadocs/Next MDX; both fine, Storybook is a recurring keyword so ship at least stories. |
| F | **Base UI foundation + coss-compatible registry** (shadcn registry JSON so people can `npx shadcn add`) | High | Low-medium | High for Cal.com specifically | Shows you've read their stack. Do not copy coss code (AGPL root, MIT only for ui/origin apps; check per-file). |
| G | **Contribute upstream to coss / Base UI** | Highest (third-party-verified) | Low per PR | Very high; Cal.com bonus lists OSS contributions | Do 2-4 small merged PRs: a11y bugs, forced-colors fixes, contrast issues. |
| H | **AI-native layer** (llms.txt, AGENTS.md, MCP/registry so Claude Code/Cursor generates on-system UI) | Medium-high; trending | Low | Medium-high given "daily AI tools" asks | Cheap; do after core is solid. |
| I | Figma library with variables in parity with code tokens | High for designer-engineer | Medium | Medium; expected, not differentiating | Publish to Community; link token-sync. Susith's Figma plugin experience helps. |
| J | Annotation-driven a11y test suite (axe + Playwright keyboard + SR notes), publicly displayed per component | High | Medium | High | Pairs with A/B; screenshot table "light / dark / forced-colors / si / ta". |

**Suggested positioning line [U]:** "An accessibility-first design system on Base UI whose tokens cannot ship below WCAG/APCA thresholds, that works in forced-colors mode and in Sinhala/Tamil, and is used by a real app." Keep the scope tight: ~12-15 components, 4 variants of proof per component (light, dark, forced-colors, si/ta).

**Recommended priority:** D + A + B first (build into the foundation), E ongoing, G in parallel, C as a pillar, F/H/I polish.

## 5. Naming

npm registry checks run 2026-10-11 via `curl https://registry.npmjs.org/<name>` (404 = unclaimed). GitHub: I checked users/orgs and `SusithD/<name>`; a taken *user* name does not block a repo under SusithD. Not checked: domains, trademarks, npm "too similar" name-squatting rules (npm blocks names that normalize to existing ones, e.g. punctuation variants), existing GitHub repos with the same name elsewhere. Search the web before committing.

| Candidate | npm 404? | GitHub SusithD/<name> | Meaning / rationale |
|---|---|---|---|
| **akuru** | yes (and akuru-ui, akuru-ds) | free | Sinhala "letters/characters" (akuru). Signals the multi-script pillar; short, pronounceable. Caveat: "Dives Akuru" is a Maldivian script (Noto font exists), no conflict as an npm name. |
| **pahan** | yes (pahan-ui, pahan-ds) | free | Sinhala "lamp/light" (pahana). Light = contrast/legibility. Caveat: "pahan" is also a common Indian surname; fine. |
| **nila-ui** | yes (bare `nila` is TAKEN) | free | Sinhala/Tamil "blue / moonlight". Short; use `nila-ui` or scoped. |
| **sandhi-ui** | yes (bare `sandhi` is TAKEN) | free | Sanskrit/Sinhala "joining" (also a linguistic term). Fits tokens joining design and code. |
| **kontrast-ui** | yes (bare `kontrast` TAKEN) | free | Direct ContrastMate lineage; descriptive for the main differentiator. Less distinctive. |
| **varna-ui** | yes | free | "Color" and "letter/script" in Sanskrit; double meaning covers color + script. |
| **aluth** | yes | free | Sinhala "new". Plain, short; weaker meaning. |
| **sahana** | yes | free | Sinhala "relief/support"; also a surname. Generic. |
| **sevana** | yes | free | Sinhala "shade" (sevanella); suits color/contrast. Reasonably short. |
| **mihira** | yes | free | Sanskrit "sun". Pleasant, but less relevant. |
| lumena / wattu / ruwan | yes | free | Fallbacks; weaker story. |

Taken on npm (avoid): prismatic, lumen, tessera, arcus, tokn, vellum, lumora, halide, polaris-ui, sandhi, nila, varna, lipi, chroma-ui, kontrast, kontra, lumo-ui, tessera-ui, pol-ui. `contrastly` is free on npm but too close to ContrastMate/Contrastly-style tools [U].

**Top 3 confirmed available on npm (unscoped):** `akuru`, `pahan`, `sevana` (also `akuru-ui`, `pahan-ui`, `sandhi-ui`, `varna-ui`, `nila-ui`, `kontrast-ui`). Scoped publishing (`@susithd/...`) also avoids collisions; I could not verify org availability (npmjs.com returned 403 to curl).

**My pick [U]:** `akuru` (monorepo packages: `akuru`, `@akuru/tokens`, `@akuru/react`, `@akuru/contrast`). Short, memorable, ties to the Sinhala/Tamil script pillar and to legibility.

## Open questions / unverified
- Whether Cal.com's Senior Product Design Engineer role is truly worldwide (Remotive) or US-only (Built In).
- Raycast job page (404 for me), Vercel/Clerk/Resend/Supabase postings only through aggregators; current status unknown.
- Whether Cal.com's role is open to non-senior candidates; the application asks for shipped work, so a strong repo plus merged PRs may carry it.
- Primary writings from Curtis/Frost/Anne on this question were not retrieved.
- Licensing: check coss repo per-file licenses before borrowing any code.

## Sources (all accessed 2026-10-11)
- https://cal.com/jobs
- https://remotive.com/remote/jobs/design/senior-product-design-engineer-5997860
- https://github.com/cosscom/coss , https://coss.com/ui , https://next.cal.com/blog/calcom-v5-8
- https://github.com/calcom/cal.com , https://raw.githubusercontent.com/calcom/cal.com/main/packages/ui/package.json
- https://jobs.accel.com/companies/linear/jobs/34559943-design-engineer-web-brand
- https://jobs.accel.com/companies/perplexity-2/jobs/85911993-design-systems-lead
- https://infoq.com/news/2026/02/baseui-v1-accessible/ , https://mui.com/blog/2026-and-beyond/
- https://www.themuse.com/jobs/zoom/senior-design-systems-engineer
- https://www.portfoliobox.com/magazine/how-to-present-a-design-system-portfolio-without
- https://intodesignsystems.substack.com/p/how-to-build-a-strong-design-systems
- https://adobe.design/ideas/ask-adobe-design-how-can-applicants-stand-out-when-applying-to-design-jobs
- https://rangle.io/blog/design-systems-adoption-pitfalls-and-recommendations
- https://www.viget.com/articles/design-systems-avoiding-common-pitfalls/
- https://acorn.firefox.com/latest/desktop/design-tokens/how-design-tokens-work/overview-YYp6MVjt
- https://github.com/mooniak/yaldevi-font , https://discourse.suttacentral.net/t/testing-for-line-height-with-tall-scripts/6196
- https://registry.npmjs.org/<name> (name checks)
