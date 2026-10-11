# 03 - Tooling, Testing and Release for an A11y-First React Design System

Research date: 2026-10-11. Solo maintainer, portfolio-grade, published to npm.

Verification legend: [V-npm] = version read live from the npm registry on 2026-10-11 (`npm view`); [V-doc] = read from official docs page fetched 2026-10-11; [S] = secondary/blog source; [U] = unverified, inferred, or from memory - check before relying.

---------------------------------------------------------------------

## 0. TL;DR recommendation

- Single published package inside a small pnpm workspace (NOT Turborepo/Nx at the start). Packages: `@scope/react` (components + CSS), `@scope/tokens` (W3C DTCG JSON -> CSS vars), plus `apps/docs` (Storybook) private. Add icons later only if you ship icons.
- Build with tsdown (tsup is effectively unmaintained-in-practice; see 2.1), ESM-only, plain CSS with CSS custom properties + CSS Modules compiled to one `styles.css`, `'use client'` preserved per component.
- Storybook 10.6 + addon-a11y + addon-vitest (browser mode, Playwright Chromium) with `parameters.a11y.test = 'error'` globally: one story = one render test + one interaction test + one axe test, failing CI.
- Extra tests: Vitest + Testing Library for hooks/logic, Playwright for forced-colors / reduced-motion / zoom / keyboard-APG flows against the built Storybook, @axe-core/playwright for page-level and contrast checks, Guidepup as an optional "wow" layer on a nightly job.
- Release: Changesets + npm Trusted Publishing (OIDC) with automatic provenance, publint + attw + size-limit gates, Storybook on GitHub Pages (Chromatic free tier optional for visual diffs).

---------------------------------------------------------------------

## 1. Recommended stack table

All versions verified live against npm registry 2026-10-11 [V-npm] unless noted.

| Area | Tool | Version (latest on 2026-10-11) | Why | Alternatives considered |
|---|---|---|---|---|
| Package manager | pnpm | 12.11.2 | Strict node_modules, workspaces, `catalog:` for shared versions | npm workspaces (fine, less polished), Bun |
| Runtime | Node | 24.x LTS for CI (local engines: 22.19+/24); Node 26.11.1 is current | Storybook 10 needs `^22.19 / ^24.5 / >=26`; tsdown needs `^22.18 / ^24.11`; Vitest 5 needs `^22.12 / ^24 / >=26` | Node 22 (still OK) |
| Language | TypeScript | **6.0.3 for the toolchain** (7.0.2 is `latest`) | typescript-eslint 8.71.1 peer is `typescript >=4.8.4 <6.1.0`, so TS 7 is not yet lint-compatible. tsdown peer allows 5/6/7. Pin 6.0.x; trial 7 in a branch | TS 5.9 |
| Build | tsdown | 0.23.1 (0.x, pre-1.0, published 2026-10-10) | Rolldown-based successor to tsup, fast d.ts, native `exports` generation, `publint`/`attw` integration hooks, CSS support (experimental) via `@tsdown/css` | tsup 8.5.1 (works, esbuild, no longer the direction of travel), Vite library mode (8.3.4), Rollup, bunchee 7.0.1, unbuild |
| Dev server / Storybook builder | Vite | 8.3.4 | Storybook react-vite peer `^5..^8` | Webpack (no) |
| Vite React plugin | @vitejs/plugin-react | 6.1.2 (peer vite ^8) | Matches Vite 8 | @vitejs/plugin-react-swc |
| Components | React | 19.3.0 (dev); peer `^18.2 \|\| ^19` | Wide adoption; test against both in CI matrix | React 19 only (cleaner, narrower reach) |
| Tokens | style-dictionary | 5.6.1 | DTCG-aware, CSS vars output, mature | Tokens Studio sd-transforms 2.0.3 (only if you design in Figma Tokens), Terrazzo, hand-written CSS (viable at small scale) |
| Styling | Plain CSS custom properties + CSS Modules (`.module.css`) | - | Zero runtime, RSC-safe, themable by consumers, works with every bundler, forced-colors/`prefers-*` media queries are first-class | vanilla-extract 1.21.3 (type-safe, but needs consumer bundler plugin unless prebuilt), Tailwind 4.3.3 (consumer coupling), CSS-in-JS (RSC problems) |
| Docs / workbench | Storybook | 10.6.1 (`react-vite`) | Industry-standard, has a11y + Vitest + docs addons first-party | Ladle 5.1.1 (no a11y/test story), Histoire, Docusaurus-only |
| Storybook addons | @storybook/addon-a11y, addon-vitest, addon-docs | all 10.6.1 | See section 3 | @storybook/test-runner (deprecated path, per [S]) |
| Unit/component test runner | Vitest | 5.0.3 (addon-vitest peer allows ^3/^4/^5) | Same runner as Storybook tests | Jest (no) |
| Vitest browser provider | @vitest/browser-playwright | 5.0.3 | Real Chromium; Storybook default | jsdom for pure logic only |
| Browser automation | Playwright / @playwright/test | 1.64.0 | forced-colors, reduced-motion, zoom, screenshots, keyboard | Cypress |
| a11y engine | axe-core | 4.14.0 | Used by addon-a11y; rule set | Pa11y (HTML_CodeSniffer), Lighthouse |
| a11y in Playwright | @axe-core/playwright | 4.13.0 | Page-level / full-flow checks | - |
| a11y in unit tests | vitest-axe | 0.1.0, last published 2025-01 [V-npm] | Simple matcher, but low maintenance; prefer axe in Storybook/Playwright. Optional | jest-axe 11.0.0 (works with Vitest via adapter) |
| Testing Library | @testing-library/react 16.3.3, user-event 14.6.7 | - | Query by role = a11y by default | - |
| Screen reader automation | @guidepup/guidepup 0.35.0 + @guidepup/playwright 0.19.1 (updated 2026-08) | - | Real VoiceOver (macOS) / NVDA (Windows) runs; virtual SR for unit tests. Needs macOS/Windows runners | Manual SR testing log (always required anyway), Assistiv Labs (paid) |
| Visual regression | Chromatic 18.12.0 (free tier: 5,000 snapshots/mo, no card [V-doc]; OSS program by contact) | - | Storybook-native, turbosnap | Playwright `toHaveScreenshot` (free, self-hosted baselines), Lost Pixel 3.22.0 |
| Versioning | @changesets/cli | 3.0.3 (+ changelog-github 1.0.1) | Works single/multi-package, PR-based | semantic-release (commit-driven, harder in monorepo), release-please |
| Publish auth | npm Trusted Publishing (OIDC) | needs npm CLI >= 11.5.1, Node >= 22.14 [V-doc] | No long-lived token; provenance automatic | `NPM_TOKEN` + `--provenance` (legacy) |
| Bundle size | size-limit + @size-limit/preset-small-lib | 14.2.0 | Per-entry budgets in CI | bundlephobia (post-hoc), bundlewatch |
| Package lint | publint 0.3.25, @arethetypeswrong/cli 0.18.5 | - | Catches bad `exports`/types | - |
| API docs | Storybook autodocs (react-docgen, built in) first; typedoc 0.28.20 only if you need a reference site | - | Props tables from TS types without extra deps | react-docgen-typescript 2.4.0 (only if built-in docgen is inadequate), api-extractor 7.59.4 |
| Docs site (later) | Storybook-only now; Starlight 0.42.6 or Fumadocs 16.17.1 if you need prose/guides | - | See 5.4 | Nextra 4.6.1, VitePress 1.6.4 |
| Lint | ESLint | **9.x (not 10)**: eslint-plugin-jsx-a11y 6.10.2 peer is `eslint ^3..^9` [V-npm] (v10 peer not listed on that package; the package was last published 2024-10) | jsx-a11y is the key plugin; stay on ESLint 9 until it declares v10 support (eslint 10.12.0 is latest) | Biome 2.5.15 (fast, has some a11y rules but fewer than jsx-a11y) |
| Lint plugins | typescript-eslint 8.71.1, eslint-plugin-react-hooks 7.1.1, eslint-plugin-storybook 10.6.1, eslint-plugin-jsx-a11y 6.10.2 | - | - | @eslint-react/eslint-plugin 5.26.0 |
| Format | Prettier 3.9.10 | - | Stable, universal | Biome formatter (single tool option) |
| Dead code | knip 6.41.0 | - | Catches unused exports/deps; looks professional | - |
| Hooks | husky 9.1.7 + lint-staged 17.6.0 | - | Optional; CI is source of truth | lefthook |

Notes on volatility: tsdown is 0.x with experimental CSS; Vitest is at 5.x; TypeScript 7 (native) just became `latest`. Pin exact versions via pnpm `catalog:` and use Renovate/Dependabot grouped updates.

---------------------------------------------------------------------

## 2. Repo structure

### 2.1 Single package vs monorepo

Recommendation: a minimal pnpm workspace, no Turborepo/Nx until there is measurable pain.

Why a workspace at all (instead of a flat single package):
- Storybook/docs app must consume the library like a user would (workspace link), and the published tarball must not contain Storybook deps.
- Tokens as a separate package is a credible signal ("tokens are a product"), enables non-React consumers, and lets you version them independently.
- Changesets handles this with zero extra config.

Why not Turborepo/Nx yet: with 2-3 packages, `pnpm -r --filter` plus CI job parallelism gives you the same thing. Turbo 2.11.7 is easy to add later (one `turbo.json`) when remote caching matters. Adding it on day one is portfolio decoration, not value. Nx: heavier, skip. [opinion]

Package split (smallest professional version):
1. `packages/tokens` -> `@scope/tokens` (DTCG JSON source, Style Dictionary build, outputs `tokens.css`, `tokens.json`, `tokens.d.ts`). Published.
2. `packages/react` -> `@scope/react` (components, hooks, `styles.css`). Published. Depends on tokens as a regular dependency or peer; simplest: bundle token CSS into `styles.css` AND export `@scope/tokens` separately.
3. `apps/docs` -> private Storybook (and later optional prose site).
4. NOT yet: `icons` (only if you create an icon set; otherwise recommend lucide/Phosphor and document), `figma-plugin` (separate repo or later; it has a different release cadence and toolchain), `eslint-plugin` (stretch goal; a consumer-side jsx-a11y config doc is enough).

If you want even smaller: one published package `@scope/react` with tokens as `src/tokens` and exported CSS subpath. Still use a workspace for `apps/docs`. This is the minimum I'd call "professional".

### 2.2 Example tree

```
design-system/
  .changeset/
    config.json
    README.md
  .github/
    workflows/
      ci.yml
      release.yml
      storybook-deploy.yml
      a11y-nightly.yml          # Guidepup / macOS+Windows, scheduled
      scorecard.yml
      codeql.yml
    ISSUE_TEMPLATE/ (bug.yml, feature.yml, a11y-report.yml)
    PULL_REQUEST_TEMPLATE.md
    dependabot.yml  (or renovate.json)
    CODEOWNERS
  apps/
    docs/                        # private Storybook
      .storybook/
        main.ts
        preview.ts               # a11y: { test: 'error' }, themes, forced-colors decorator
        vitest.setup.ts
      stories/ (MDX: Introduction, Accessibility, Tokens, Contributing)
      package.json
  packages/
    tokens/
      src/ (color.json, space.json, ... DTCG)
      style-dictionary.config.mjs
      package.json
    react/
      src/
        components/
          Button/
            Button.tsx
            Button.module.css
            Button.stories.tsx   # includes play() tests + a11y
            Button.test.tsx      # logic tests only if needed
            index.ts
        hooks/
        styles/ (reset.css, base.css)
        index.ts                 # named exports only
      tsdown.config.ts
      package.json
      .size-limit.json
  e2e/
    forced-colors.spec.ts
    keyboard-apg.spec.ts
    zoom-reflow.spec.ts
    sr/ (guidepup specs)
    playwright.config.ts
  research/
  eslint.config.js
  vitest.config.ts               # projects: unit (jsdom/node), storybook (browser)
  knip.json
  pnpm-workspace.yaml            # packages + catalog:
  package.json
  tsconfig.base.json
  README.md  CONTRIBUTING.md  CODE_OF_CONDUCT.md  SECURITY.md  LICENSE
  ACCESSIBILITY.md  (conformance statement + known issues + SR matrix)
  CHANGELOG (generated per package)
```

---------------------------------------------------------------------

## 3. Build and packaging

### 3.1 Bundler choice

- tsdown 0.23.1: Rolldown-based, positioned by its own docs as tsup's successor with a `tsdown-migrate` command; claims ~2x faster than tsup and up to 8x faster d.ts generation (project's own benchmark, treat as directional) [S: https://tsdown.dev/advanced/benchmark, accessed 2026-10-11]. Requires Node `^22.18 || ^24.11 || >=26` (build time only) [V-npm].
- tsup 8.5.1: still works; last registry modification of the package 2025-11 [V-npm]. Use only if you already have a tsup setup.
- Vite library mode: good when you need Vite plugins (e.g., CSS modules + `vite-plugin-dts` 5.1.2), but d.ts bundling is a separate plugin and you get one big config. Reasonable alternative.
- Rollup: maximum control, most boilerplate. bunchee 7.0.1: zero-config but exports-field-driven and smaller community.
- Pick: tsdown, with unbundled or per-entry output so `'use client'` directives survive.

Caveat: tsdown CSS support is experimental ("API and behavior may change") and requires `@tsdown/css` (0.23.1) + `lightningcss` (1.33.0) [V-doc https://tsdown.dev/options/css, accessed 2026-10-11]. By default all CSS merges into one `style.css` (renamable via `css.fileName`); `css.splitting: true` emits per-chunk CSS; `.module.css` supported with scoped names. Fallback plan if it misbehaves: build CSS separately with `lightningcss-cli` (or PostCSS) into `dist/styles.css` and keep tsdown for JS/d.ts only. That fallback is boring and robust.

### 3.2 ESM-only vs dual

- Recommend ESM-only for 2026: React 19 ecosystem, Storybook 10 is itself ESM-only [S: https://storybook.js.org/blog/storybook-10/], Node 22.12+/24 support `require(esm)` unflagged [U on exact version boundaries for require(esm); Storybook's blog says it needs Node 20.16+/22.19+/24+]. Dual output doubles the "dual package hazard" and attw complexity.
- Dual (ESM + CJS) only if you want maximum compatibility with legacy Jest/CJS consumers. If you do dual, attw (`--profile node16`) is mandatory.
- Run attw with `--profile esm-only` for ESM-only.

### 3.3 package.json shape (ESM-only)

```json
{
  "name": "@scope/react",
  "version": "0.0.0",
  "type": "module",
  "sideEffects": ["**/*.css"],
  "files": ["dist", "README.md", "LICENSE"],
  "exports": {
    ".": { "types": "./dist/index.d.mts", "default": "./dist/index.mjs" },
    "./styles.css": "./dist/styles.css",
    "./package.json": "./package.json"
  },
  "peerDependencies": { "react": "^18.2.0 || ^19.0.0", "react-dom": "^18.2.0 || ^19.0.0" },
  "repository": { "type": "git", "url": "git+https://github.com/<owner>/<repo>.git", "directory": "packages/react" },
  "publishConfig": { "access": "public", "provenance": true }
}
```

Points:
- `repository.url` must exactly match the GitHub repo or trusted publishing fails [V-doc https://docs.npmjs.com/trusted-publishers, accessed 2026-10-11].
- `sideEffects`: mark only CSS (or `false` if CSS is imported only by the consumer via `styles.css` and not from JS). If components import their own CSS Modules from JS, keep `"**/*.css"` so bundlers do not drop styles.
- Per-component subpath exports (`./button`) are optional; with ESM + `sideEffects` and per-module output, the barrel is tree-shakable. Verify with size-limit by importing a single component.
- Keep `react/jsx-runtime` external; `react` and `react-dom` as peers, never bundled.

### 3.4 CSS delivery options

| Option | Pros | Cons | Verdict |
|---|---|---|---|
| Plain CSS vars + one `styles.css` consumer imports | Simplest; no consumer tooling; themeable; RSC-safe; forced-colors in CSS | Consumer must import CSS once; no per-component CSS tree-shaking | Default |
| CSS Modules compiled into `styles.css` | Scoped class names, no collisions, authoring ergonomics | Hashed names hinder consumer overrides (expose `data-*` attributes and CSS vars as the override API); tsdown CSS experimental | Recommended authoring format |
| vanilla-extract 1.21.3 | Type-safe tokens, zero runtime | Consumer needs bundler integration unless prebuilt CSS shipped; extra complexity | Skip |
| Tailwind v4 plugin/preset | Familiar to consumers | Couples library to Tailwind; a11y-first DS should be framework-agnostic in CSS | Skip, but document how to use with Tailwind v4 (`@layer`) |
| CSS-in-JS runtime (emotion, styled-components) | - | RSC incompatibility, runtime cost | No |

Use `@layer` (e.g., `@layer ds.reset, ds.base, ds.components;`) so consumers can override without specificity wars. [opinion, widely used pattern]

### 3.5 RSC and 'use client'

- Components using state/effects/refs-with-handlers need `'use client'`. Bundlers often strip module-level directives; tsdown/Rolldown preserve them when each file stays a separate module or via per-entry output. Verify by grepping `dist` for `"use client"` in CI [U: confirm tsdown preservation behavior on your config; this is the single biggest packaging gotcha]. Alternative: add the directive in a post-build step.
- Keep purely presentational components (Text, Stack, VisuallyHidden) server-compatible (no directive) to prove RSC support. Test with a tiny Next.js app in `e2e/` or `examples/` (App Router) on CI, at least on release.
- Use React 19 `ref` as prop (no `forwardRef`) only if you drop React 18; with peer `^18 || ^19`, keep `forwardRef` or conditionally support both. Decide up front.

### 3.6 Declarations

tsdown `dts: true` bundles `.d.mts` (uses oxc/isolatedDeclarations when enabled, else TS). Enable `isolatedDeclarations` in tsconfig for the fast path. Because TS 7 is just out and typescript-eslint doesn't support it yet, keep tsdown's `typescript` at 6.0.x. Always run attw on the packed tarball.

### 3.7 Sample tsdown config

```ts
import { defineConfig } from 'tsdown';

export default defineConfig({
  entry: ['src/index.ts'],
  format: 'esm',
  platform: 'neutral',
  dts: true,
  unbundle: true,            // keeps per-file modules; helps 'use client' and tree-shaking [U: option name, check docs]
  deps: { neverBundle: ['react', 'react-dom', 'react/jsx-runtime'] }, // per tsdown skill docs [S]
  css: { fileName: 'styles.css' },
  publint: true,
  attw: { profile: 'esm-only' },
});
```
Option names move between 0.x releases; confirm against https://tsdown.dev before copying [U].

---------------------------------------------------------------------

## 4. Storybook

### 4.1 Version and features

- Current: Storybook 10.6.1 (latest dist-tag); 11.0.0-alpha.5 on `next`; 9.1.20 maintenance [V-npm].
- 10.x: ESM-only, ~29% lighter install, module automocking built with the Vitest team, CSF Factories, requires Node 20.16+/22.19+/24+ [S: https://storybook.js.org/blog/storybook-10/, accessed 2026-10-11]. Registry shows engines `^22.19 || ^24.5 || >=26` [V-npm].
- Storybook 9 introduced the Vitest-based component testing; `@storybook/test-runner` is the older Playwright-runner path, superseded by `@storybook/addon-vitest` [S, moderate confidence].
- Autodocs: via `@storybook/addon-docs` 10.6.1 (`tags: ['autodocs']`), MDX docs supported. Props tables come from built-in react-docgen/TS inference.

### 4.2 Failing CI on a11y violations (verified)

From official docs [V-doc https://storybook.js.org/docs/writing-tests/accessibility-testing, accessed 2026-10-11]:
- `parameters.a11y.test`: `'off'` | `'todo'` (warn in UI only) | `'error'` (fail in UI and CLI/CI). There is NO default; violations do not fail unless set to `'error'`.
- Other parameters: `context` (default `'body'`), `config` (axe.configure; the `region` rule is disabled by default), `options` (axe.run, e.g., `runOnly` tags), `globals.a11y.manual`.
- Recommended workflow: set `'error'` project-wide in `.storybook/preview.ts`, mark known-failing stories `'todo'`, fix, remove.

```ts
// .storybook/preview.ts
import type { Preview } from '@storybook/react-vite';
const preview: Preview = {
  parameters: {
    a11y: {
      test: 'error',
      options: { runOnly: ['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'wcag22aa', 'best-practice'] },
    },
  },
};
export default preview;
```
Colour contrast in axe runs against real browser rendering (needs browser mode, not jsdom).

### 4.3 Vitest addon and browser mode

[V-doc https://storybook.js.org/docs/writing-tests/integrations/vitest-addon, accessed 2026-10-11]:
- Requires a Vite-based framework (`@storybook/react-vite`), Vitest >= 3 (peer supports 3/4/5), browser mode with Playwright Chromium is the default/recommended.
- Config: `storybookTest` plugin from `@storybook/addon-vitest/vitest-plugin` inside a Vitest project; on Vitest 4+ use `projects`; `setupFiles: ['./.storybook/vitest.setup.ts']`.
- CLI: `"test-storybook": "vitest --project=storybook"`; add `--run` in CI.
- If you see "Failed to fetch dynamically imported module" / "Cannot connect to iframe", disable isolation or shard with `--shard`.
- Coverage from stories supported; visual tests are a separate Visual Tests (Chromatic) addon.
- Interaction tests: `play` functions with `storybook/test` (`userEvent`, `expect`, `within`); they run in the Vitest browser and in the Interactions panel.

### 4.4 Deploying

- GitHub Pages (free, static): `storybook build` -> `actions/upload-pages-artifact` + `actions/deploy-pages`. Recommended default: no third party, URL goes in README and npm `homepage`.
- Chromatic: free tier 5,000 snapshots/month, unlimited projects/users, no card [V-doc https://www.chromatic.com/pricing]; "Building in the open? ... Contact us to see if you're eligible" for OSS terms (specifics not published). Gives hosted Storybook + PR visual review + UI Tests/Review. Optional, run only on PRs touching `packages/react`.
- Vercel/Netlify: only for per-PR previews; Chromatic does this already.

---------------------------------------------------------------------

## 5. Testing strategy

### 5.1 Credible DS test pyramid (solo-friendly)

Bottom to top, with the idea that story files are the single source of truth:

1. Static (seconds): TypeScript strict, ESLint + jsx-a11y + storybook plugin, publint/attw, knip. Lints catch missing labels, invalid ARIA in JSX.
2. Unit (ms): pure logic - hooks (`useControllableState`, roving tabindex, typeahead), token build, ID generation, focus helpers. Vitest in Node/jsdom with Testing Library `renderHook`. Keep small.
3. Story-driven component tests (core layer, ~70% of value): every component state is a story; Storybook Vitest addon runs in real Chromium: render smoke + `play` function interactions (keyboard-only: Tab, Arrow, Enter, Space, Escape) + axe check (`a11y.test: 'error'`). One story per state per variant, including dark/forced-colors decorator variants.
4. E2E / browser-specific (Playwright against built Storybook static): things that need browser emulation or screenshots: `forced-colors`, `prefers-reduced-motion`, `prefers-contrast`, 200%/400% zoom reflow, RTL, focus-visible ring presence, real-browser focus trap behavior in Firefox/WebKit (cross-browser projects), `@axe-core/playwright` on composed pages.
5. Visual regression: Chromatic (easy) or Playwright `toHaveScreenshot` on a focused set (Linux-only baselines to avoid OS drift).
6. Assistive tech automation (thin, scheduled): Guidepup virtual screen reader in unit tests; real VoiceOver (macOS runner) / NVDA (Windows runner) smoke checks for 5-8 canonical patterns, nightly not per-PR.
7. Manual: SR matrix (NVDA+Firefox/Chrome, JAWS if available, VoiceOver+Safari iOS/macOS, TalkBack) and keyboard audit recorded in `ACCESSIBILITY.md` per release. Automated axe catches roughly a third to half of WCAG issues [S/commonly cited; not verified here]; the manual log is what makes the claim credible.

### 5.2 Vitest browser mode vs jsdom

Use browser mode (Playwright provider) for anything rendering components: jsdom has no layout, so no colour contrast, no real focus/`:focus-visible`, no forced-colors, no `inert`/popover/dialog parity. Keep jsdom (or plain Node) only for hook/logic tests where speed matters. Both in one Vitest config via `projects`.

### 5.3 Per-APG-pattern keyboard tests

Make a table in docs: pattern, APG keys, test file. Implement each as a `play` function or Playwright spec asserting focus location and ARIA state after each key. Examples: Button (Enter/Space), Dialog (focus trap, Esc, return focus), Tabs (Arrow, Home/End, automatic vs manual activation), Menu button (Arrow open, typeahead, Esc), Combobox (aria-activedescendant, Arrow/Esc/Enter), Disclosure/Accordion, Checkbox/Radio group (roving tabindex), Switch, Tooltip (Esc, hover/focus), Listbox. Source: https://www.w3.org/WAI/ARIA/apg/patterns/ [not re-fetched; stable reference].

### 5.4 Forced-colors in Playwright

Playwright supports `page.emulateMedia({ forcedColors: 'active', colorScheme: 'dark', reducedMotion: 'reduce', contrast: 'more' })` (Chromium; WebKit/Firefox support varies) [U: verify per-browser support for `forcedColors` and `contrast`]. Pattern: snapshot or assert computed styles (outline/border not `transparent`/`none` for focus and state indicators, SVG icons use `currentColor`/`forced-color-adjust`). Add a project in `playwright.config.ts` with `use: { forcedColors: 'active' }`. Windows High Contrast real-world differs from emulation, so note in conformance statement.

### 5.5 Guidepup

Packages: @guidepup/guidepup 0.35.0 and @guidepup/playwright 0.19.1 (latest publish 2026-08-16 [V-npm], so alive). Drives VoiceOver on macOS and NVDA on Windows; includes a virtual screen reader for jsdom-style tests; GitHub Action `guidepup/setup-action` prepares runners [S: README via https://cdn.jsdelivr.net/npm/@guidepup/guidepup@0.24.1/README.md and dev.to write-up; old version, confirm current README]. Constraints: needs macOS runner for VoiceOver and Windows runner for NVDA; assert on spoken phrase log (no audio); runs are slow/flaky-prone, so keep them nightly and non-blocking at first. Great portfolio differentiator.

### 5.6 axe tooling placement

- In Storybook (every story): addon-a11y with `'error'`.
- `@axe-core/playwright` 4.13.0: whole-page scenarios (docs pages, composed forms), tags `wcag22aa`, plus state-specific scans (dialog open, menu open, error state) that stories can't easily cover.
- vitest-axe 0.1.0: optional; stale since 2025-01 [V-npm], so I'd skip it, since Storybook already runs axe in browser.
- Treat axe pass as necessary-not-sufficient; state that in README.

---------------------------------------------------------------------

## 6. Release

### 6.1 Changesets + semver

- `@changesets/cli` 3.0.3 (last modified 2026-09-14) [V-npm]. Config: `"access": "public"`, `"baseBranch": "main"`, `changelog: ["@changesets/changelog-github", { "repo": "owner/repo" }]`, `"updateInternalDependencies": "patch"`.
- Start at `0.x` until API stabilises; define in docs what is breaking for a DS: removed/renamed props, changed ARIA semantics or keyboard behavior, token renames/removals, visual changes that break layouts, dropping browser/React versions. Treat a11y regressions as bugs (patch), a11y improvements that change DOM as minor with notes.
- CI check: `changeset status --since=origin/main` on PRs (or the changeset-bot GitHub App).

### 6.2 npm Trusted Publishing (OIDC) and provenance

Verified from npm docs [V-doc https://docs.npmjs.com/trusted-publishers, accessed 2026-10-11]:
- Requires npm CLI >= 11.5.1 and Node >= 22.14; workflow needs `permissions: id-token: write`; supports GitHub Actions (GitHub-hosted runners only), GitLab CI, CircleCI.
- Provenance is generated automatically for public repo + public package (not for private repos).
- Up to 10 trusted publishers per package; configs cannot be edited (delete and recreate); npm does not validate on save, errors show only at publish time; `repository.url` must exactly match; a new configuration must complete a first successful publish within 2 days or it expires. Dist-tag management via trusted publishing needs npm >= 11.21.0 / 12.2.0 and the "Allow npm dist-tag" option.
- Not stated in the page: whether a package must already exist before configuring. Practical workaround [U]: first publish `0.0.1` manually (or with a short-lived granular token), then configure the trusted publisher, then delete the token.
- Changesets: the `changesets/action` README recommends, when using trusted publishing, wiring the individual sub-actions to tighten publish permissions, and lists `id-token: write` as a needed permission [V: README fetched via curl 2026-10-11, lines 20 and 28]. Simplest pattern: the action opens/updates the "Version Packages" PR; on merge a `publish` script runs `pnpm changeset publish` using the OIDC token (no `NPM_TOKEN`). The Node 24 image ships npm 11.x, so no `npm i -g npm@latest` step is needed; on Node 22 add it [S: community reports].

Example release workflow:

```yaml
name: release
on:
  push:
    branches: [main]
concurrency: ${{ github.workflow }}-${{ github.ref }}
permissions:
  contents: write
  pull-requests: write
  id-token: write
jobs:
  release:
    if: github.repository == 'owner/repo'
    runs-on: ubuntu-latest
    environment: npm-release      # add required reviewer = yourself for a manual gate
    steps:
      - uses: actions/checkout@<sha>
      - uses: pnpm/action-setup@<sha>
      - uses: actions/setup-node@<sha>
        with: { node-version: 24, cache: pnpm, registry-url: 'https://registry.npmjs.org' }
      - run: pnpm install --frozen-lockfile
      - run: pnpm build && pnpm test:ci
      - uses: changesets/action@<sha>
        with:
          version: pnpm changeset version
          publish: pnpm changeset publish
          title: 'chore: version packages'
          commit: 'chore: version packages'
```
Pin actions by commit SHA (OpenSSF Scorecard rewards this). Environment name must match what you put in the npm trusted-publisher config if you set one.

### 6.3 Size and package checks

- size-limit 14.2.0 + preset-small-lib: budgets on `dist/index.mjs` entry and single-component import (`import { Button }`), gzip/brotli; fail PR on exceed. Add `size-limit` GitHub Action comment (andresz1/size-limit-action) [U: check it is maintained].
- `publint --strict` and `attw --pack --profile esm-only` on every PR; `npm pack --dry-run` listing check.
- `knip` for unused exports.

### 6.4 API docs and docs site

- API reference for props: Storybook autodocs is enough and keeps one source of truth. TypeDoc 0.28.20 only for non-component API (hooks, utils, tokens types). react-docgen-typescript 2.4.0 is legacy relative to built-in docgen; use only if prop descriptions are not picked up.
- Docs site options (if you need long-form guides, a11y principles, design rationale):
  - Storybook-only (MDX): lowest maintenance, deploy to GH Pages. Recommended for v0.
  - Starlight 0.42.6 (Astro): best a11y defaults, search (Pagefind), embed Storybook iframes or link; very low effort. Best "next step."
  - Fumadocs 16.17.1: Next.js-based, powerful, more moving parts. Nextra 4.6.1: similar. VitePress 1.6.4: simple, Vue-first tooling but fine for markdown.
  - Pick Starlight when you outgrow Storybook MDX.

---------------------------------------------------------------------

## 7. Quality signals checklist

- ESLint flat config (`eslint.config.js`), ESLint 9.x with typescript-eslint 8.71.1, jsx-a11y 6.10.2 `flatConfigs.strict`, react-hooks 7.1.1, storybook plugin 10.6.1. Hold at ESLint 9 until jsx-a11y declares ESLint 10 [V-npm peers]. Biome 2.5.15 is faster but its a11y rule set is smaller; if you choose Biome, you lose jsx-a11y parity, so ESLint + Prettier is the safer a11y-first pick [opinion].
- Prettier 3.9.10 (`prettier --check` in CI).
- publint / attw / knip / size-limit (above).
- Coverage: Storybook Vitest coverage -> Codecov (free for public repos) with a badge; do not chase 100%, state what's covered by story tests. [U: Codecov OSS terms; verify]
- OpenSSF Scorecard: `ossf/scorecard-action` workflow + badge; items it checks: pinned actions by SHA, least-privilege `permissions`, branch protection, `SECURITY.md`, Dependabot/Renovate, signed releases/provenance, CI tests, fuzz/SAST (CodeQL). [S/memory; verify checks list on scorecard.dev]
- npm provenance badge appears automatically on the package page after OIDC publish.
- Community files: README (install, usage, a11y promise, browser/AT support), CONTRIBUTING.md (setup, a11y checklist for new components, how to add a changeset), CODE_OF_CONDUCT.md (Contributor Covenant 2.1/3.0 [U: version]), SECURITY.md, issue templates incl. an "Accessibility issue" form, PR template with a11y checklist, `ACCESSIBILITY.md` (conformance target WCAG 2.2 AA, tested AT/browser matrix, known limitations, last manual audit date), CODEOWNERS, `LICENSE` (MIT).
- Branch protection on `main`: required checks, linear history, no force push; since you are solo, still require PRs + green CI to look professional and protect yourself.
- Dependabot/Renovate with grouped updates and `pnpm` catalog.

---------------------------------------------------------------------

## 8. CI pipeline outline (GitHub Actions)

`ci.yml` (on PR and push to main; concurrency cancel-in-progress; least-privilege `permissions: contents: read`):

1. `setup` - checkout, pnpm, Node 24, cache pnpm store, `pnpm install --frozen-lockfile`.
2. `static` (parallel job) - `pnpm typecheck`, `eslint .`, `prettier --check .`, `knip`, `changeset status --since=origin/main`.
3. `unit` (parallel) - Vitest `--project=unit` (Node/jsdom), coverage artifact.
4. `build` - `pnpm -r build`; upload `dist` artifact; then `publint --strict`, `attw --pack --profile esm-only`, grep `"use client"` in dist, `size-limit`.
5. `storybook-tests` (parallel, matrix sharding 1/2 if slow) - cache `~/.cache/ms-playwright`, `pnpm exec playwright install --with-deps chromium`, `pnpm test-storybook --run --coverage` (render + play + axe via `a11y.test: 'error'`). Upload coverage to Codecov.
6. `e2e` (needs build + `storybook build`) - serve static Storybook, run Playwright projects: chromium, firefox, webkit; projects `forced-colors`, `reduced-motion`, `zoom-400`; `@axe-core/playwright` scenarios; upload traces/report on failure.
7. `react-matrix` (optional) - install React 18.3 vs 19 in a scratch step and run storybook tests for each; plus a minimal Next.js App Router smoke app to validate RSC/'use client' imports.
8. `visual` (optional, PR only, `packages/react/**` paths) - `chromatic --exit-zero-on-changes` or Playwright screenshots.
9. `deploy-storybook` (main only) - `storybook build` -> `actions/deploy-pages` (permissions `pages: write`, `id-token: write`).

Other workflows:
- `release.yml` (section 6.2) on push to main.
- `a11y-nightly.yml` - `schedule` cron; matrix `macos-latest` (VoiceOver via Guidepup) and `windows-latest` (NVDA via Guidepup); `continue-on-error: true` initially; open an issue on failure.
- `scorecard.yml`, `codeql.yml` (weekly + PR), `dependabot.yml`.
- Required status checks on main: static, unit, build, storybook-tests, e2e.

---------------------------------------------------------------------

## 9. Risks and open items

1. tsdown is 0.x and CSS support is experimental: budget time; fallback is separate lightningcss/PostCSS CSS build. [V-doc]
2. `'use client'` preservation in tsdown output must be verified by test. [U]
3. TypeScript 7.0.2 is `latest` but typescript-eslint 8.71.1 peers `<6.1.0`: stay on TS 6.0.x for now. [V-npm]
4. ESLint 10 vs jsx-a11y (peer lists up to ESLint 9, last published 2024-10): stay on ESLint 9. [V-npm]
5. vitest-axe is stale (2025-01): do not depend on it. [V-npm]
6. Trusted publishing bootstrap: confirm whether a package must exist before adding a trusted publisher; if so, first publish manually. Unclear in npm docs. [U]
7. pnpm 12 / Vitest 5 / Vite 8 are new majors: confirm the Storybook 10.6.1 Vite 8 + Vitest 5 combination in a spike before committing; peers allow them [V-npm] but real-world issues are untested here. [U]
8. Guidepup real-SR CI is inherently flaky; keep non-blocking.
9. Option names in config snippets above (tsdown `unbundle`, `deps.neverBundle`, `css`, `attw`) were not all verified against 0.23.1 docs [U].
10. Claims about npm token deprecation timelines found in third-party PRs (e.g., 2FA-bypass tokens removed Jan 2027) are unconfirmed and not relied on. [U]

## 10. Sources (accessed 2026-10-11)

- npm registry (`npm view`) for all versions, engines, peer deps - primary data.
- https://docs.npmjs.com/trusted-publishers
- https://storybook.js.org/docs/writing-tests/accessibility-testing
- https://storybook.js.org/docs/writing-tests/integrations/vitest-addon
- https://storybook.js.org/blog/storybook-10/
- https://tsdown.dev/options/css
- https://tsdown.dev/advanced/benchmark
- https://www.chromatic.com/pricing
- https://raw.githubusercontent.com/changesets/action/main/README.md
- https://cdn.jsdelivr.net/npm/@guidepup/guidepup@0.24.1/README.md (old; secondary)
- https://alan.norbauer.com/articles/tsdown-bundler (secondary, tsup to tsdown)
