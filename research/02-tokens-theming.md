# 02 - Design Tokens and Theming (research as of 11 Oct 2026)

Legend: [V] verified from a fetched source on 2026-10-11 (URL cited); [S] secondary/community source; [U] unverified or from prior knowledge - check before relying on it.

## 0. Executive recommendations

1. Use **DTCG 2025.10 JSON files in git as the source of truth** (hand-authored plus script-generated palettes). Figma Variables are a downstream *consumer* (and a design-time editing surface), not the origin. Reason: the Figma Variables REST API needs an Enterprise plan [V]; a portfolio project cannot rely on it, and git gives review, history, CI and a public artifact.
2. Build with **Style Dictionary v5** (latest 5.6.1, npm, 2026-10-10 [V]) for CSS custom properties plus TS; optionally evaluate **Terrazzo 2.x** (npm @terrazzo/cli 2.7.1 [V]) which natively supports DTCG resolvers and ships an `a11y/min-contrast` lint rule.
3. Three tiers: **primitive -> semantic -> component**, with semantic tokens carrying all theming. Palettes generated in **OKLCH** to **target contrast ratios** (Leonardo-style), so contrast is guaranteed by construction and then **verified in CI** with a pair manifest.
4. Claim **WCAG 2.2 AA (target AAA for body text)** compliance. Use APCA as an *informational* second signal only. Do not claim "APCA compliance" (APCA is not in the WCAG 3 draft; see section 5).
5. Ship modes: light, dark, high-contrast (`prefers-contrast: more`), forced-colors (system colors), reduced motion, density, plus a script-aware typography layer (Latin / Sinhala / Tamil).

## 1. DTCG format - status and shape

### 1.1 Status

| Item | Finding | Source |
|---|---|---|
| Stable release | "First stable version 2025.10", announced 28 Oct 2025. Versioning is date-based (2025.10), **not 1.0**. FAQ also calls it "v1". | [V] https://www.designtokens.org/ ; https://www.w3.org/community/design-tokens/2025/10/28/design-tokens-specification-reaches-first-stable-version/ (accessed 2026-10-11) |
| Standing | Community Group report, published as "Candidate Recommendation" per CG definitions, **not on the W3C Standards Track and not a W3C Recommendation**; intended for implementation. | [V] same + https://www.designtokens.org/faq/ |
| Modules | Format, Color, Resolver (each "Module 2025.10"). Drafts at designtokens.org/tr/drafts are dated Sept 2026 and say "do not implement the draft"; implement the published 2025.10. | [V] https://www.designtokens.org/tr/drafts/format/ ; /color/ ; /resolver/ |
| Open issues that may change types | Gradient `type` (linear/radial) issue 101; `lineHeight` number vs dimension issue 102. | [V] format draft |

### 1.2 File format essentials (Format module)

- Token = object with `$value`; `$type` required but inheritable from a parent group; tools must not guess types. Optional `$description`, `$extensions` (reverse-domain keys, tools must preserve), `$deprecated` (bool or string). Names cannot start with `$` or contain `{`, `}`, `.`. [V]
- Groups: objects without `$value`; may carry `$type`, `$description`, `$extends` (inheritance, token-level replace), `$deprecated`; `$root` gives a group a base token. [V]
- Aliases: `{group.token}` curly syntax (whole token) and JSON-Pointer `$ref` (RFC 6901) to reach inside a value, e.g. `#/colors/blue/$value/components/0`. Both must be supported; cycles are errors. [V]
- Dimension: `{ "value": 16, "unit": "px" | "rem" }`; duration `{ "value": 200, "unit": "ms" | "s" }`; `cubicBezier` = 4 numbers (x in [0,1]). [V]
- Composites: `shadow` (color, offsetX, offsetY, blur, spread; array allowed), `border` (color, width, style), `transition` (duration, delay, timingFunction), `strokeStyle`, `gradient` (stops with `color`, `position` 0-1), `typography` (fontFamily, fontSize, fontWeight, letterSpacing, lineHeight - lineHeight is a unitless number, multiplier of fontSize). [V]
- Color module: `$value = { colorSpace, components[], alpha?, hex? }`. Spaces: srgb, srgb-linear, display-p3, a98-rgb, prophoto-rgb, rec2020, hsl, hwb, lab, lch, oklab, **oklch**, xyz-d65, xyz-d50; `none` allowed for powerless components; `hex` is a sRGB fallback. [V] https://www.designtokens.org/tr/drafts/color/
- Resolver module: a `*.resolver.json` with `version: "2025.10"`, `sets` (named token sources, inline or `$ref` files), `modifiers` (named contexts such as light/dark with optional `default`), and `resolutionOrder`. Input like `{ "theme": "dark" }` selects contexts; later entries override earlier; N permutations = product of context counts. Modifiers cannot reference modifiers. [V] https://www.designtokens.org/tr/drafts/resolver/

### 1.3 Example (valid 2025.10 shape)

```json
{
  "color": {
    "$type": "color",
    "blue": {
      "600": {
        "$value": { "colorSpace": "oklch", "components": [0.52, 0.19, 259], "hex": "#2b5fd9" },
        "$description": "Primitive. Do not use in components."
      }
    }
  },
  "space": {
    "$type": "dimension",
    "4": { "$value": { "value": 0.25, "unit": "rem" } }
  },
  "motion": {
    "duration": { "$type": "duration", "fast": { "$value": { "value": 120, "unit": "ms" } } },
    "easing": { "$type": "cubicBezier", "standard": { "$value": [0.2, 0, 0, 1] } }
  },
  "text": {
    "body": {
      "$type": "typography",
      "$value": {
        "fontFamily": ["Inter", "Noto Sans Sinhala", "Noto Sans Tamil", "sans-serif"],
        "fontSize": { "value": 1, "unit": "rem" },
        "fontWeight": 400,
        "letterSpacing": { "value": 0, "unit": "rem" },
        "lineHeight": 1.5
      }
    }
  }
}
```

Resolver example (themes):

```json
{
  "version": "2025.10",
  "sets": { "foundation": { "sources": [{ "$ref": "tokens/primitives.json" }, { "$ref": "tokens/semantic.base.json" }] } },
  "modifiers": {
    "theme": {
      "default": "light",
      "contexts": {
        "light": [{ "$ref": "tokens/theme.light.json" }],
        "dark": [{ "$ref": "tokens/theme.dark.json" }],
        "hc-light": [{ "$ref": "tokens/theme.hc-light.json" }],
        "hc-dark": [{ "$ref": "tokens/theme.hc-dark.json" }]
      }
    },
    "density": {
      "default": "comfortable",
      "contexts": { "compact": [{ "$ref": "tokens/density.compact.json" }], "comfortable": [{ "$ref": "tokens/density.comfortable.json" }] }
    }
  },
  "resolutionOrder": [{ "$ref": "#/sets/foundation" }, { "$ref": "#/modifiers/theme" }, { "$ref": "#/modifiers/density" }]
}
```

Caveat [U]: tool support for the resolver varies (Terrazzo supports it; Style Dictionary does not consume `.resolver.json` natively as far as I could verify). With Style Dictionary, express modes as separate `source` sets per theme and loop over them in the build script (see section 2.4).

## 2. Tooling

### 2.1 Comparison

| Tool | Status (Oct 2026) | DTCG | Strengths | Gaps / risks |
|---|---|---|---|---|
| **Style Dictionary** (Amazon, now community-run) | v5.6.1 published 2026-10-10 [V npm + https://github.com/amzn/style-dictionary/releases]. v4 released 2024-06-28, v5 2025-05-16 [V npm timestamps] | v4 first-class DTCG; v5 adopted 2025.10 as base format; v5.3 added all 14 DTCG color spaces and oklch/oklab/lch/P3 output transforms; v5.4 added dimension object values [S: search summary of changelogs]. Docs still say "2025.10 does not have full support yet" [V https://styledictionary.com/info/dtcg/] | Mature, huge ecosystem, custom transforms/formats/hooks, CSS/SCSS/TS/iOS/Android, multi-file multi-theme builds | No native resolver; composite tokens need expand/transform config; you write theme loops yourself |
| **Terrazzo** (formerly Cobalt UI) | @terrazzo/cli 2.7.1 [V npm]. Site: "resolvers and other 2025.10 features" in 2.0 [V https://terrazzo.app/docs/] | Claims full DTCG; resolver as successor to modes | Plugins: CSS, JS/TS, Sass, Tailwind v4, Swift, Storybook, Vanilla Extract [V]; CSS plugin maps resolver contexts to media queries/classes/selectors [V search]; built-in lint incl. `a11y/min-contrast` (pairs, AA/AAA, `largeText`) [V source of @terrazzo/parser 0.8.1: https://app.unpkg.com/@terrazzo/parser@0.8.1/files/src/lint/plugin-core/rules/a11y-min-contrast.ts] | Smaller ecosystem; config syntax changed between 1.x and 2.x [S]; contrast rule options must be re-checked in 2.x [U] |
| **Tokens Studio** (Figma plugin + platform) | Active; Feb 2026 added variable scoping and custom code syntax sync; May 2026 server-side alias resolution [S https://feedback.tokens.studio/changelog] | DTCG opt-in per provider (since 2.0, Aug 2024); needs Style Dictionary 4 downstream; companion `@tokens-studio/sd-transforms` 2.0.3 [V npm] | Best-in-class token authoring in Figma incl. math, modifiers; Git sync | Composite tokens are skipped when exported to Figma variables (Figma variables are single-value only) [S https://docs.tokens.studio/figma/export/variables-skipped]; paid tiers for some sync features [U] |
| **Figma Variables** | Core Figma feature; modes, aliases, scopes, code syntax | Native DTCG export/import announced at Config 2025; rollout uneven; reports of missing `$description`, `$extensions` with Figma IDs, and failed round-trip imports (Dec 2025 - Apr 2026) [S forum.figma.com threads - single reports, unconfirmed] | Designers' native tool; works on all plans in the editor | **REST Variables API (read and write) requires Enterprise plan and Full seat; write needs `file_variables:write`; variables must be published** [V https://developers.figma.com/docs/rest-api/variables/] |
| **Figma Plugin API** | Available on all plans [S] | n/a | Plugin can read/write variables (this is how Tokens Studio and open-source sync plugins work) | You maintain a plugin; no server-side automation |
| **Specify** | Sunset (sources disagree on 2024 vs 2025; shut down by early 2025 per DTCG mailing list) [S] | - | - | Do not adopt |
| **Supernova** | Operating per 2026 roundups [S]; no shutdown news found | Imports Figma variables; export pipelines to Git | Docs plus pipelines | Commercial; adds a vendor between Figma and git |

Version-sensitive facts (npm "latest" on 2026-10-11) [V]: style-dictionary 5.6.1, @terrazzo/cli 2.7.1, @tokens-studio/sd-transforms 2.0.3, @radix-ui/colors 3.0.0, @adobe/leonardo-contrast-colors 1.1.0, apca-w3 0.1.9, culori 4.0.2, colorjs.io 0.7.1, @adobe/spectrum-tokens 15.5.0.

### 2.2 Source-of-truth decision

| Option | Direction | Pros | Cons |
|---|---|---|---|
| A. Figma is source | Figma Variables -> REST/export -> DTCG -> CSS | Designers lead | Enterprise-only API; native export incomplete; contrast generation logic cannot live in Figma; not reproducible in CI |
| **B. DTCG in git is source (recommended)** | git DTCG -> build -> CSS+TS; git DTCG -> Figma Variables via plugin/import | Reviewable, testable, free, OSS-friendly, supports generated palettes and contrast CI, works for any contributor | Designers edit via PRs or a plugin; need a sync step to Figma |
| C. Tokens Studio as source | TS JSON (DTCG) in git <-> Figma | Good authoring UX, two-way | Plugin lock-in for authoring; composite tokens not representable as Figma vars |

Recommendation: **B, with Figma as a pushed mirror.** The author already writes Figma plugins (ContrastMate), so a small "DS Token Sync" plugin (or Tokens Studio in read-mostly mode) that imports the built DTCG JSON into Figma Variables collections (Primitives, Semantic with Light/Dark/HC modes) is natural, and it works on a free/pro plan through the Plugin API. If a Figma-side edit must come back, round-trip through a PR using the plugin's export, validated by the same JSON schema and contrast CI. Treat native Figma DTCG export as a convenience to test, not to depend on [S].

### 2.3 Pipeline

```
tokens/
  primitives/ (generated palettes + raw scales)   <- scripts/generate-palettes.ts (Leonardo/culori)
  semantic/   (base, light, dark, hc-light, hc-dark, forced-colors map)
  component/
        | validate (JSON schema, alias cycles, $type present)
        | contrast gate (pairs.json)  <-- CI fails here
        v
  Style Dictionary 5 (or Terrazzo)
        |-> dist/css/tokens.css (+ per-theme blocks)
        |-> dist/ts/tokens.ts (typed names + resolved values)
        |-> dist/json/tokens.resolved.json
        |-> dist/figma/variables.json  -> Figma plugin import
        v
  Storybook (reads tokens.css) + axe tests + Chromatic/visual
```

### 2.4 Style Dictionary v5 config sketch (multi-theme)

```js
import StyleDictionary from 'style-dictionary';

const themes = ['light', 'dark', 'hc-light', 'hc-dark'];

for (const theme of themes) {
  const sd = new StyleDictionary({
    source: ['tokens/primitives/**/*.json', 'tokens/semantic/base.json', `tokens/semantic/${theme}.json`, 'tokens/component/**/*.json'],
    usesDtcg: true,
    platforms: {
      css: {
        transformGroup: 'css',
        prefix: 'ds',
        buildPath: 'dist/css/',
        files: [{
          destination: `${theme}.css`,
          format: 'css/variables',
          filter: (t) => t.filePath.includes('/semantic/') || t.filePath.includes('/component/'),
          options: { selector: `[data-theme="${theme}"]`, outputReferences: true }
        }]
      }
    }
  });
  await sd.buildAllPlatforms();
}
```

Then a small post-step wraps `light` as `:root`, `dark` also in `@media (prefers-color-scheme: dark)`, HC in `@media (prefers-contrast: more)`, and a hand-written `@media (forced-colors: active)` block (section 3.4). Verify option names against the v5 docs [U]; `usesDtcg` and `outputReferences` exist in v4/v5 [S/prior knowledge].

## 3. Token architecture

### 3.1 Tiers and what other systems do

| System | Tiers / naming | Notes | Source |
|---|---|---|---|
| Primer (GitHub) | Base -> Functional -> Component/pattern. Name blocks: prefix, namespace, pattern, variant, property (required), scale. Examples `base-color-green-5`, `bgColor-muted`, `fgColor-default`, `button-primary-bgColor-hover`. Modifiers: default/muted/emphasis; sizes xsmall-xxlarge; density condensed/normal/spacious | [V] https://primer.style/product/primitives/token-names/ |
| Material 3 | Reference (`md.ref.*`) -> System (`md.sys.color.on-surface`) -> Component (`md.comp.*`; in CSS not prefixed `comp`). "on-" tokens are the foreground for a matching container | [S] Material Web docs via search; official m3.material.io not fetched |
| Spectrum (Adobe) | Global -> Alias -> Component; component token -> alias -> global -> value. Source repo `adobe/spectrum-tokens` (npm @adobe/spectrum-tokens 15.5.0 [V]); newer "Design Data" spec draft uses a structured `name` object [S] | https://spectrum.adobe.com/page/design-tokens ; https://opensource.adobe.com/spectrum-design-data/spec/token-format |
| Atlassian | name = foundation + property + modifiers (role, emphasis, state), e.g. `color.background.danger.bold.hovered`; elevation surfaces `elevation.surface.{sunken,default,raised,overlay}`; docs note names may still evolve | [V/S] https://atlassian.design/foundations/tokens/design-tokens/ |
| Radix Colors | 12-step scales per hue, light and dark, alpha (A1-A12) and P3 variants; v3.0.0 reworked all colors [V changelog via search] | see 3.2 |

### 3.2 Radix 12-step scale (what each step is for) [V https://www.radix-ui.com/colors/docs/palette-composition/understanding-the-scale]

| Step | Use |
|---|---|
| 1 | App background |
| 2 | Subtle background |
| 3 | UI element background (normal) |
| 4 | UI element background (hover) |
| 5 | UI element background (active/selected) |
| 6 | Subtle borders, non-interactive (separators, cards) |
| 7 | Subtle borders, interactive |
| 8 | Stronger borders, focus rings |
| 9 | Solid backgrounds (highest chroma) |
| 10 | Solid background hover |
| 11 | Low-contrast text (guaranteed APCA Lc 60 on step 2) |
| 12 | High-contrast text (guaranteed APCA Lc 90 on step 2) |

Adopt the same *role-per-step* idea for your own OKLCH scale; it makes semantic mapping mechanical and contrast pairs predictable. Note the Radix guarantee is stated in APCA, not WCAG ratio, so you still need to re-verify with WCAG 2 math (steps 11/12 on step 1-2 usually pass 4.5:1 / 7:1 but this must be tested, not assumed) [U].

### 3.3 Recommended architecture for this DS

Tier 1 - Primitive (never used in components; not exposed in docs as public API):
`color.{hue}.{1..12}`, `color.{hue}.a{1..12}` (optional), `space.{0..16}`, `size.*`, `radius.*`, `font.family.*`, `font.size.*`, `duration.*`, `easing.*`, `shadow.*`.

Tier 2 - Semantic (theme-aware; this is the public API):
```
color.bg.{canvas,subtle,surface,raised,overlay,sunken}
color.bg.{neutral,accent,success,warning,danger}.{subtle,default,strong}  (+ .hover/.active)
color.fg.{default,muted,subtle,disabled,on-accent,link,danger,success}
color.border.{subtle,default,strong,focus,danger}
color.focus.ring            (non-text, >= 3:1 on adjacent colours)
color.status.{success,warning,danger,info}.{bg,fg,border}
elevation.shadow.{1..4}
motion.duration.{instant,fast,moderate,slow}  motion.easing.{standard,emphasized}
space.{inline,stack,inset}.{xs..xl}   (density-aware)
text.{display,heading-1..4,body,body-sm,label,code}  (typography composites)
```
Naming rule: `category.property.role[.emphasis][.state]`, lower-kebab segments, dot-separated in JSON, CSS var `--ds-color-fg-muted`. Avoid colour names in semantic tokens (no `blue`), use roles.

Tier 3 - Component (only when a component needs a decision not expressible semantically): `button.primary.bg.hover` -> alias to `color.bg.accent.strong.hover`. Keep this tier thin; most components should use tier 2 directly to limit token count. Component tokens are the layer consumers override.

Rules: components may reference tier 2 and 3 only; tier 2 may reference tier 1; lint-enforced (custom Style Dictionary preprocessor or JSON-schema check). Every semantic colour token is defined for **every** mode or the build fails.

### 3.4 Modes

| Mode | Mechanism | Token handling |
|---|---|---|
| Light / dark | `[data-theme]` attribute plus `@media (prefers-color-scheme: dark)` default; CSS `light-dark()` is an option for single-declaration themes [U: browser support now broad, verify caniuse] | Separate semantic files per theme; dark is *designed* (not inverted) - lower chroma, surface elevation by lightness steps (Atlassian-style surfaces) |
| High contrast | `@media (prefers-contrast: more)`; plus explicit `data-contrast="more"` toggle | Separate `hc-light`/`hc-dark` semantic sets targeting AAA (7:1 text, 4.5:1 UI) and thicker borders; generated by the same palette script with higher target ratios |
| Forced colors (Windows HCM) | `@media (forced-colors: active)` | Map semantic tokens to **system colors in pairs**: text -> `CanvasText`/`Canvas`, links -> `LinkText`, buttons -> `ButtonText`/`ButtonFace`, disabled -> `GrayText`, selected -> `Highlight`/`HighlightText`, focus ring -> `Highlight`. Only real system-colour keywords (not `transparent`/named/hex). Shadows vanish in this mode, so give controls a real `border` using a system colour. Use `forced-color-adjust: none` only for swatches/data-colour [V https://developer.mozilla.org/docs/Web/CSS/@media/forced-colors ; Firefox HCM checklist https://firefox-source-docs.mozilla.org/accessible/HCMMediaQueries.html ; S https://www.smashingmagazine.com/2022/03/windows-high-contrast-colors-mode-css-custom-properties]. `-ms-high-contrast` is deprecated. Test via DevTools Rendering "forced-colors: active" and Playwright `forcedColors: 'active'` |
| Reduced motion | `@media (prefers-reduced-motion: reduce)` | Redefine `--ds-motion-duration-*` to ~0.01ms (or swap to opacity-only), never remove state changes; also expose `data-motion` override. WCAG 2.3.3 (AAA) animation from interactions; 2.2.2 for auto-playing >5s content |
| Density | `[data-density="compact"]` | Semantic `space.inset.*` and `size.control.height.*` differ per density (Primer: condensed/normal/spacious). Keep touch target >= 24x24 CSS px (WCAG 2.5.8 AA) in compact [U verify SC text] |

### 3.5 Colour spaces and palette generation

- Author in **OKLCH** (perceptually uniform lightness, predictable hue/chroma); DTCG 2025.10 can store `oklch` natively [V]. Emit sRGB `hex` fallback and, optionally, `oklch()` / `color(display-p3 ...)` in CSS with gamut-mapping (use Style Dictionary 5.3+ oklch/P3 transforms [S]); keep a `hex` fallback in the token for older tools and for Figma (Figma variables are sRGB/P3 but not OKLCH [U]).
- **Leonardo** (`@adobe/leonardo-contrast-colors` 1.1.0 [V npm]): generates colours by **target contrast ratio** against a background; `Theme`, `Color`, `BackgroundColor` classes; supports WCAG 2 and APCA modes and OKLCH/LAB/LCH/CAM02 interpolation spaces [S https://leonardocolor.io/api.html ; npm README]. Fit: define semantic steps by *ratio* (e.g. fg.muted = 4.5:1 on canvas) rather than by hex, then run for light and dark and HC themes (HC = same script with ratio multiplier).
- Alternative (no Adobe dependency, matches ContrastMate's maths): binary-search OKLCH lightness per hue with `culori`/`colorjs.io` until WCAG ratio >= target, clamping chroma into sRGB gamut. A port of this exact method exists in Elixir and describes Leonardo's algorithm [S https://hexdocs.pm/color/Color.Palette.Contrast.html]. This is ~60 lines and showcases the author's domain skill.

## 4. Contrast

### 4.1 WCAG 2.2 requirements [V https://www.w3.org/TR/WCAG22/]

| SC | Level | Requirement |
|---|---|---|
| 1.4.3 Contrast (Minimum) | AA | Text 4.5:1; large text 3:1 (>= 18pt, or >= 14pt bold) |
| 1.4.6 Contrast (Enhanced) | AAA | Text 7:1; large text 4.5:1 |
| 1.4.11 Non-text Contrast | AA | UI component boundaries/states and meaningful graphics 3:1 against adjacent colours |
| 2.4.11 Focus Not Obscured (Min) | AA | New in 2.2 |
| 2.4.13 Focus Appearance | AAA | Focus indicator size and 3:1 change (so a 3:1 ring is good practice) |

Exemptions: disabled controls and pure decoration are exempt, but still aim for legibility. Placeholder text and borders that *identify* an input need 3:1 (1.4.11) [U: interpretation, widely held].

### 4.2 APCA and WCAG 3

- WCAG 3 is still a **Working Draft** (latest confirmed: March 2026), incomplete, "intended to develop into a W3C Standard in a few years"; WCAG 2 will not be deprecated for several years after [V search of https://www.w3.org/WAI/WCAG3 and https://www.w3.org/WAI/news/2026-03-03/wcag3].
- **APCA is not in the current draft**: it was removed from the July 2023 draft; the editor's note says the contrast algorithm is "yet to be determined" (Roselli, April 2026) [S https://adrianroselli.com/2026/04/wcag3-contrast-as-of-april-2026.html ; https://www.yatil.net/blog/wcag-3-is-not-ready-yet]. Bridge-PCA (Myndex) is a WCAG-2-compatible variant with no W3C standing [S npm bridge-pca].
- **Position for the DS:** claim conformance to **WCAG 2.2 AA** (document AAA where met, e.g., HC theme). Publish APCA Lc values as *supplementary, non-normative* data (nice differentiator, consistent with ContrastMate), worded "also meets APCA Lc >= X (informational)". Never write "APCA compliant". Pin `apca-w3` version (0.1.9 [V]) because the algorithm constants have changed historically.

### 4.3 How to guarantee contrast

1. **By construction**: palette generator targets ratios (3.3).
2. **By declaration**: `tokens/a11y/pairs.json` lists every fg/bg pair and the minimum ratio, with a role.
3. **By CI**: a test resolves each theme (light, dark, hc-light, hc-dark) to final sRGB, composites alpha over the declared background, computes WCAG 2 ratio (and APCA Lc informational), and fails below threshold. Add a **coverage check**: every `color.fg.*` and `color.border.*` token must appear in at least one pair (or be allow-listed), so new tokens cannot ship untested.
4. **By component tests**: axe-core (jest-axe / @axe-core/playwright / Storybook test-runner) in each theme catches real rendered contrast, including states if you force them; axe cannot see hover/focus contrast on its own, which is why step 3 matters.
5. **By snapshot of the resolved matrix**: publish the pair matrix into the docs site (auto-generated), reviewing diffs in PRs.

Pair manifest:

```json
{
  "pairs": [
    { "fg": "color.fg.default",   "bg": "color.bg.canvas",  "min": 7,   "kind": "text" },
    { "fg": "color.fg.muted",     "bg": "color.bg.canvas",  "min": 4.5, "kind": "text" },
    { "fg": "color.fg.muted",     "bg": "color.bg.surface", "min": 4.5, "kind": "text" },
    { "fg": "color.fg.on-accent", "bg": "color.bg.accent.default", "min": 4.5, "kind": "text" },
    { "fg": "color.border.strong","bg": "color.bg.canvas",  "min": 3,   "kind": "non-text" },
    { "fg": "color.focus.ring",   "bg": "color.bg.canvas",  "min": 3,   "kind": "non-text" },
    { "fg": "color.focus.ring",   "bg": "color.bg.accent.default", "min": 3, "kind": "non-text" }
  ],
  "themeOverrides": { "hc-light": { "text": 7, "non-text": 4.5 }, "hc-dark": { "text": 7, "non-text": 4.5 } }
}
```

Test (Vitest + culori; reads the *resolved* tokens produced by the build):

```ts
import { wcagContrast, parse, formatHex, blend } from 'culori'; // culori 4.x
import { APCAcontrast, sRGBtoY } from 'apca-w3';                 // informational only
import pairs from '../tokens/a11y/pairs.json';
import { resolveTheme } from './helpers';                         // returns Map<tokenPath, "#rrggbb" | rgba>

const themes = ['light', 'dark', 'hc-light', 'hc-dark'] as const;

describe.each(themes)('contrast: %s', (theme) => {
  const t = resolveTheme(theme);
  it.each(pairs.pairs)('$fg on $bg >= $min', ({ fg, bg, min, kind }) => {
    const need = pairs.themeOverrides[theme]?.[kind] ?? min;
    const bgc = parse(t.get(bg)!)!;
    const fgc = flatten(parse(t.get(fg)!)!, bgc);                  // composite alpha over bg
    expect(wcagContrast(fgc, bgc)).toBeGreaterThanOrEqual(need);
  });
});
```
(`wcagContrast` exists in culori and is what Terrazzo's rule uses [V source]; `flatten` is a small alpha-over helper you write. Import names for apca-w3 should be verified [U].)

Terrazzo alternative: the built-in `a11y/min-contrast` lint rule takes `pairs` of fg/bg token IDs, `level` AA/AAA and per-pair `largeText` [V 0.8.1 source; re-check 2.x options]. It does not cover non-text 3:1, themes beyond what the resolver yields, or alpha, so keep the custom test as the authority regardless.

## 5. Typography, spacing, multi-script, motion

### 5.1 Type and spacing

| Topic | Recommendation |
|---|---|
| Units | `rem` for font sizes and spacing so user font-size preferences scale (WCAG 1.4.4, 1.4.12). DTCG dimension supports `rem` [V]. Avoid `px` for text; borders/hairlines may be px |
| Scale | Modular or hand-tuned steps; 12/14/16/18/20/24/30/36/48 as primitives; semantic `text.*` composites |
| Fluid type | `clamp(min, preferred, max)` with a `rem + vw` preferred value, e.g. `clamp(1.125rem, 1rem + 0.5vw, 1.375rem)`; always keep a `rem` term in the preferred value so zoom still enlarges text (1.4.4). DTCG has no native clamp type: store min/max/vw as primitives and have a Style Dictionary custom transform emit `clamp()`, or put the expression in `$extensions` [U: no DTCG standard way] |
| Line height | Unitless multipliers (DTCG typography `lineHeight` is a number [V]). Body 1.5 (WCAG 1.4.12 requires content survive 1.5 line spacing, 2x paragraph spacing, 0.12em letter, 0.16em word spacing without loss) [U exact values from WCAG 1.4.12; verify]. Headings 1.15-1.3 for Latin |
| Spacing | 4pt base, 8pt rhythm; tokens `space.1=0.25rem ... space.16=4rem`; component sizes on 4pt steps; semantic `space.inline/stack/inset` that change with density |
| Radius/elevation | Few steps (none, sm, md, lg, full); shadows as `shadow` composites per elevation level, with dark-mode surface-lightness elevation |

### 5.2 Multi-script (Latin, Sinhala, Tamil)

Findings: Sinhala and Tamil glyphs extend well above/below Latin ascender/descender; at line-height 1.1 several tall scripts clip, 1.6 resolved it in one community test (single non-authoritative test) [S https://discourse.suttacentral.net/t/testing-for-line-height-with-tall-scripts/6196]; metrics vary heavily per font so test with shipped files; avoid fixed heights/`overflow:hidden` on text; Noto Sans Sinhala needs complex-text shaping support [S https://fonts.google.com/noto/specimen/Noto%2BSans%2BSinhala/about]; Tamil often exceeds Latin ascender/descender in bi-script fonts [S https://typedrawers.com/discussion/4554].

Token design:

```json
{
  "font": {
    "$type": "fontFamily",
    "family": {
      "latin":   { "$value": ["Inter", "system-ui", "sans-serif"] },
      "sinhala": { "$value": ["Noto Sans Sinhala", "Iskoola Pota", "sans-serif"] },
      "tamil":   { "$value": ["Noto Sans Tamil", "Latha", "sans-serif"] }
    }
  },
  "text": {
    "body": {
      "lineHeight": {
        "$type": "number",
        "latin":  { "$value": 1.5 },
        "sinhala": { "$value": 1.75 },
        "tamil":   { "$value": 1.7 }
      }
    }
  }
}
```
(Values 1.7-1.75 are starting points to be tuned against actual fonts [U].)

CSS emission: set `:lang(si)` and `:lang(ta)` rules that swap `--ds-font-family-body` and `--ds-line-height-body`; build a single stack `Inter, "Noto Sans Sinhala", "Noto Sans Tamil", system-ui` so mixed-script strings fall back per glyph; self-host subsetted WOFF2 with `unicode-range` and `font-display: swap` plus `size-adjust`/`ascent-override` to limit layout shift [U]. Add Storybook stories with real Sinhala/Tamil strings and an RTL/long-text "stress" toggle; add a visual test for clipping. Increase minimum body size a notch for Indic scripts (fine conjunct detail) [U, design judgement].

### 5.3 Motion tokens

```json
{
  "motion": {
    "duration": {
      "$type": "duration",
      "instant":  { "$value": { "value": 0,   "unit": "ms" } },
      "fast":     { "$value": { "value": 120, "unit": "ms" } },
      "moderate": { "$value": { "value": 200, "unit": "ms" } },
      "slow":     { "$value": { "value": 320, "unit": "ms" } }
    },
    "easing": {
      "$type": "cubicBezier",
      "standard":   { "$value": [0.2, 0, 0, 1] },
      "decelerate": { "$value": [0, 0, 0, 1] },
      "accelerate": { "$value": [0.3, 0, 1, 1] }
    }
  }
}
```
```css
@media (prefers-reduced-motion: reduce) {
  :root { --ds-motion-duration-fast: 0.01ms; --ds-motion-duration-moderate: 0.01ms; --ds-motion-duration-slow: 0.01ms; }
}
```
Prefer replacing movement (slide/scale/parallax) with fades rather than killing feedback entirely. Provide a `reducedMotion` override in a `data-motion` attribute for Storybook.

## 6. Recommended final stack (summary table)

| Concern | Decision |
|---|---|
| Format | DTCG 2025.10 JSON (`*.tokens.json`) in git, validated by JSON schema |
| Source of truth | Git (DTCG). Figma Variables = generated mirror via own plugin (all plans) |
| Build | Style Dictionary 5.x (CSS vars, TS, resolved JSON, Figma import JSON). Re-evaluate Terrazzo 2.x for resolver and lint if SD theme loops get awkward |
| Palettes | OKLCH, generated to target ratios (Leonardo or custom culori search); 12 role-based steps, Radix-style |
| Tiers | primitive -> semantic (public API) -> component (thin) |
| Modes | light, dark, hc-light, hc-dark, forced-colors (system colours), reduced motion, density, script/lang |
| Contrast | WCAG 2.2 AA claimed (AAA in HC); APCA informational only; pair manifest + coverage check in CI; axe in Storybook |
| Figma | Variable collections Primitives / Semantic (modes Light, Dark, HC Light, HC Dark) / Component; code syntax set to the CSS var names; scopes set per token (Tokens Studio added code-syntax and scoping sync Feb 2026 [S]) |
| npm | `@scope/tokens` (JSON + CSS + TS) separate from `@scope/react`; semver where renaming/removing a token = major, `$deprecated` for one minor |

## 7. Open questions / unverified items to resolve before building

- Exact Style Dictionary v5 option names and whether a DTCG-resolver preprocessor exists (check https://styledictionary.com).
- Terrazzo 2.x `a11y/min-contrast` option names and whether it handles non-text contrast and themes.
- Whether Figma native DTCG export now includes descriptions/modes and round-trips reliably (test on a sample file).
- Browser support status for `light-dark()`, `color-mix()`, relative colour syntax and gamut-mapping of `oklch()` in your support matrix (check caniuse).
- WCAG 1.4.12 text-spacing exact numbers and 2.5.8 target size (24x24) - confirm in the spec text.
- Final Sinhala/Tamil font choice and measured line-height values.
- Radix Colors 3.0 internal colour space (only "P3 versions added" confirmed).

## Sources (accessed 2026-10-11)

- DTCG home https://www.designtokens.org/ ; announcement https://www.w3.org/community/design-tokens/2025/10/28/design-tokens-specification-reaches-first-stable-version/ ; FAQ https://www.designtokens.org/faq/
- Format https://www.designtokens.org/tr/drafts/format/ ; Color https://www.designtokens.org/tr/drafts/color/ ; Resolver https://www.designtokens.org/tr/drafts/resolver/ ; published TR https://www.designtokens.org/tr/2025.10/
- Style Dictionary DTCG page https://styledictionary.com/info/dtcg/ ; releases https://github.com/amzn/style-dictionary/releases ; npm registry (version/time data)
- Terrazzo https://terrazzo.app/docs/ ; lint rule source https://app.unpkg.com/@terrazzo/parser@0.8.1/files/src/lint/plugin-core/rules/a11y-min-contrast.ts
- Figma REST Variables https://developers.figma.com/docs/rest-api/variables/ ; Figma forum threads on native export https://forum.figma.com/ask-the-community-7/native-variable-export-feature-47831
- Tokens Studio https://feedback.tokens.studio/changelog ; https://docs.tokens.studio/figma/export/variables-skipped
- Radix https://www.radix-ui.com/colors/docs/palette-composition/understanding-the-scale ; https://radix-ui.com/colors/docs/overview/releases
- Primer https://primer.style/product/primitives/token-names/ ; Spectrum https://spectrum.adobe.com/page/design-tokens ; Atlassian https://atlassian.design/foundations/tokens/design-tokens/
- WCAG 2.2 https://www.w3.org/TR/WCAG22/ ; WCAG 3 https://www.w3.org/WAI/WCAG3 ; https://www.w3.org/WAI/news/2026-03-03/wcag3 ; https://adrianroselli.com/2026/04/wcag3-contrast-as-of-april-2026.html ; https://www.yatil.net/blog/wcag-3-is-not-ready-yet
- Leonardo https://leonardocolor.io/api.html ; npm @adobe/leonardo-contrast-colors
- Forced colors https://developer.mozilla.org/docs/Web/CSS/@media/forced-colors ; https://firefox-source-docs.mozilla.org/accessible/HCMMediaQueries.html ; https://www.smashingmagazine.com/2022/03/windows-high-contrast-colors-mode-css-custom-properties
- Tall scripts https://discourse.suttacentral.net/t/testing-for-line-height-with-tall-scripts/6196 ; https://typedrawers.com/discussion/4554
