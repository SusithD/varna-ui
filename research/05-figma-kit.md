# 05 - Figma kit and design-to-code (research as of 2026-10-11)

Access date for all URLs: 2026-10-11. Labels: VERIFIED = read on a primary page; SECONDARY = third-party/forum/search summary; UNVERIFIED = could not confirm. Search tooling returned snippets for several items, so primary-source confirmation is flagged per item.

## TL;DR
- Professional plan is enough for a first-class Figma kit: Variables, modes (10 per collection, SECONDARY), aliasing, scoping, typography variables, Slots (GA), Dev Mode annotations, Community publishing, Plugin API access to variables.
- NOT available on Professional: Variables REST API (Enterprise only, read and write), Extended collections (Enterprise), Code Connect (Organization/Enterprise).
- Best pipeline for a Pro-plan portfolio project: code (DTCG JSON in repo) is the source of truth -> Style Dictionary/Terrazzo builds CSS vars -> a custom Plugin-API plugin (sibling of ContrastMate) imports DTCG into Figma variables and validates contrast pairs. This avoids Enterprise REST and Tokens Studio Pro.
- The strongest portfolio story: "Token Contrast Auditor" - reads the kit's own variables from inside Figma via Plugin API, checks every foreground/background pair per mode (WCAG 2.x + APCA), writes a report, and the same check runs in CI against the JSON. Same source tokens, two enforcement points.

## 1. Figma Variables

### Capabilities
- Types: Color, Number (FLOAT), String, Boolean. Typography variables bind to font family (String), size, weight/style, line height, letter spacing, paragraph spacing (Number). Line height via variables must be fixed numbers, not percentages (SECONDARY, 2024-era posts; no 2026 change found). Composite text styles remain styles; common pattern = variables as primitives, text styles referencing them (Primer does exactly this: color+size as variables, text/shadow still styles). https://frontendmasters.com/blog/figma-typography-variables/ ; https://primer.style/guides/figma
- Collections and modes: modes per collection depend on plan; forum (Figma staff pinned reply) says Pro 10, Org 20; Enterprise commonly cited as 40 (SECONDARY; official help page does not list numbers). Older posts saying 4 are outdated. https://help.figma.com/hc/en-us/articles/15343816063383-Modes-for-variables ; https://forum.figma.com/suggest-a-feature-11/launched-all-plans-should-offer-more-than-4-variable-modes-13979/index15.html
- Aliasing: variable -> variable, including across collections (primitive -> semantic -> component). Plugin API: `createVariableAlias`, `setBoundVariable*`. https://developers.figma.com/docs/plugins/api/figma-variables/ (VERIFIED)
- Scoping: `variable.scopes` controls which pickers show the variable (e.g. only text fill, frame fill); `hiddenFromPublishing` hides primitives from library consumers; `codeSyntax` (WEB/ANDROID/iOS) stores the CSS var name for Dev Mode. https://developers.figma.com/docs/plugins/api/Variable/ (VERIFIED). Exact VariableScope enum values not retrieved (check https://developers.figma.com/docs/plugins/api/VariableScope/). Tokens Studio syncs scoping and code syntax as of its 2026-02-04 changelog (SECONDARY).
- Extended collections (multi-brand): parent collection extended per brand, overriding selected values, inheriting the rest. Help article says Enterprise plan. Plugin API: `extendLibraryCollectionByKeyAsync`, `ExtendedVariableCollection`. Rollout began ~Nov 2025 after being announced at Schema 2025. https://help.figma.com/hc/en-us/articles/36346281624471-Extend-a-variable-collection ; https://developers.figma.com/docs/plugins/api/ExtendedVariableCollection/ ; https://forum.figma.com/ask-the-community-7/native-variable-export-feature-47831
  - Pro workaround for multi-brand: one collection with one mode per brand/theme (light, dark, and brand modes up to the 10-mode cap), or separate collections for brand and theme. Combined brand x theme matrices explode on Pro; keep the kit to light/dark (+ maybe high-contrast) and demonstrate theming in code.

### Native DTCG import/export
- Shipped: Figma help documents "Export mode / Export modes" (right-click mode or collection) and "Import mode" accepting DTCG JSON. Import mapping: Color (HSL/sRGB) -> Color; Dimension (px only) and Duration (s only) -> Number; Font family -> String; Number -> Number or Boolean with `com.figma.type: "boolean"`; cross-collection aliases via `com.figma.aliasData` extension; nested groups become slash names; duplicates normalised to same name -> first wins. (VERIFIED: Modes help page.)
- Timeline: announced at Schema 2025 along with extended collections; rolling out in late Nov-Dec 2025, some users (even Enterprise) still lacked "Export modes" on 2025-12-04; early exports lacked `description` (SECONDARY forum). The help page gives no per-plan breakdown for import/export and no date. Treat Pro availability as UNVERIFIED: open a Pro file and right-click a mode to check. Colour import is sRGB/HSL only, so Display P3/OKLCH tokens from DTCG 2025.10 will not round-trip. (inference from the type table)
- DTCG itself: first stable spec 2025.10 published 2025-10-28 (Format, Color, Resolver modules); not a W3C Standard. https://www.w3.org/community/design-tokens/2025/10/28/design-tokens-specification-reaches-first-stable-version/ (SECONDARY via search; page title confirmed)
- Plan: use native import as a convenience, but do not depend on it for CI.

### REST API vs Plugin API
- Variables REST API (GET local/published, POST variables): plan Enterprise for both; POST needs Full seat + admin role; scopes `file_variables:read|write`; guests cannot use. https://developers.figma.com/docs/rest-api/variables/ (VERIFIED)
- Plugin API has no such plan gate; it can read/write variables in any file you can edit (Southleft's skills note REST is Enterprise-only so they read through the Plugin API). Implication: all automation on Pro must run inside a plugin (or via a plugin-fed export). https://www.skills.sh/southleft/skills-for-figma/export-tokens-figma (SECONDARY)
- Plugin manifest note: sync getters deprecated; with `documentAccess: "dynamic-page"` use the `...Async` methods (VERIFIED).
- Figma's reference repo for REST sync via GitHub Actions needs Enterprise: https://github.com/figma/variables-github-action-example (SECONDARY snippet).
- Rate limits for Pro full seats: forum reports of aggressive `Retry-After` on REST for Pro (UNVERIFIED, anecdotal).

## 2. Building a professional UI kit

### Architecture (recommended)
1. Separate files or pages: `Foundations` (variables, type, icons, effects) published as a library; `Components` library consuming it; `Docs/Cover` pages. Primer splits into Primitives / Web / Icons / Templates libraries so consumers enable only what they need. https://primer.style/guides/figma (VERIFIED)
2. Three variable layers: primitives (hidden from publishing, scopes empty) -> semantic (`color/bg/surface`, `color/text/primary`, scopes set) -> component tokens (only where necessary). Names match CSS custom properties 1:1 (Polaris pattern; set `codeSyntax.WEB`).
3. Components mirror React props: one Figma component per React component; variant properties = enums (size, intent), boolean properties = booleans (disabled, loading), text properties = string content, instance-swap = icon props, **Slots** = children/composition. Keep property names identical to prop names. Primer: "use components as-is, rely on properties not overrides".
4. Slots: Figma Slots went open beta 2026-03-05 and GA by 2026-06-10 (plugin API: `SlotNode`, `componentNode.createSlot()`, `SLOT` component property type, child-count limits). Full seats only. https://developers.figma.com/docs/plugins/updates/2026/06/10/update/ (VERIFIED for GA/API; March date SECONDARY forum). Good fit for Card/Dialog/Field composition. Confirm availability on your Pro Full seat.
5. Auto layout: use padding/gap/size bound to number variables, min/max width/height for fluid components (Input, Button label truncation), wrap for chip groups. Bind every spacing/radius/stroke to variables so a token change propagates.
6. States as variants only for what code truly has (hover, focus-visible, pressed, disabled); focus ring drawn from a variable-bound stroke/effect and verified at 3:1 non-text contrast.
7. Naming: `Component/Variant` layers, slash-delimited variable names, PascalCase component names equal to the React export, `property=value` variant naming. Add a `_` prefix or hide-from-publishing for internal building blocks.
8. Docs page per component: purpose, anatomy, variants matrix, states, usage do/don't, accessibility notes (keyboard, roles, name/label), link to Storybook/docs (component `documentationLinks`/description).

### Dev Mode and annotations
- Dev Mode (GA Jan 2024) supports measurements and annotations (categories, filter by category, auto-hiding at zoom); Plugin API can create annotations in bulk. Annotate what a dev cannot infer: interaction, keyboard, announce text. https://www.figma.com/blog/dev-mode-ga/ ; https://help.figma.com/hc/en-us/articles/20774752502935 (SECONDARY snippets)
- Accessibility annotation kits to learn from or interoperate with:
  - CVS Health Web/iOS Accessibility Annotation Kit - free on Community, CC-BY-4.0 repo, tested with 65+ design teams over 2 years. https://github.com/cvs-health/annotations (VERIFIED)
  - Indeed A11y Annotation Kit (Stephanie Hagadorn): callouts, focus order, keyboard interactions (SECONDARY).
  - Intopia, Fluent Accessibility Notation, US VA design system reuses CVS kit (SECONDARY): https://design.va.gov/accessibility/accessibility-annotations
  - Recommendation: do not rebuild; include per-component "Accessibility spec" frames using the CVS pattern (landmarks, heading level, focus order, name/role/value, keyboard map) and credit them. Consider a small set of your own annotation components bound to your tokens, listing CC-BY attribution.

### Publishing to Community
- Need can-edit access, publish from the editor via Share > Publish to Community; content must follow Community Guidelines. Free files default CC BY 4.0 (commercial use allowed, attribution required); creators may state additional licences on the listing (no in-product selector, per a 2021 staff reply - dated). Paid files need seller approval and reviews; help page notes Figma is not currently approving new sellers (SECONDARY snippet - verify). https://help.figma.com/hc/en-us/articles/360042296374-Figma-Community-copyright-and-licensing ; https://help.figma.com/hc/en-us/articles/360040035974-Publish-Community-files
- Practical: publish one clean Community file (cover, getting started, foundations, components) as CC BY 4.0, plus the React repo under MIT; state in README that kit is CC BY 4.0 and code MIT. Community files cannot be live-linked as team libraries across the Community boundary the same way a team library is: consumers duplicate the file (note: general Figma behaviour; verify), so ship versioned releases and a changelog page. Primer notes only employees can contribute to the Figma library "due to how Figma handles open source files" - Figma has no PR workflow for design files; accept issues/discussions on GitHub.

## 3. Figma Code Connect
- What: maps code components to Figma components so Dev Mode and the Figma MCP server show your real code snippets instead of generated ones. Two routes: Code Connect UI (in Figma, optionally GitHub-connected) and CLI (`.figma.tsx`/template `.figma.ts` files, `npx figma connect publish`). React, React Native, HTML (web components/Angular/Vue), SwiftUI, Compose supported; template files framework-agnostic. https://developers.figma.com/docs/code-connect/ ; https://developers.figma.com/docs/code-connect/react (VERIFIED/SECONDARY)
- Plan: Organization or Enterprise AND a Full or Dev seat. Not available on Professional for custom mappings (Figma support answer, Dec 2025). https://help.figma.com/hc/en-us/articles/23920389749655-Code-Connect (VERIFIED); https://forum.figma.com/report-a-problem-6/can-you-enable-code-connect-ui-for-our-workspace-please-48177 (SECONDARY)
- Cost: third-party pricing lists Organization Full $55 / Dev $25 per month annual; Professional Full $16 / Dev $12 annual (SECONDARY; verify at figma.com/pricing). Organization annual-only.
- Figma MCP server: Pro Dev/Full seats get 200 tool calls/day and 10/min; Org same daily with 15/min; Enterprise 600/day; Starter 6 per month; `add_code_connect_map` exempt from limits. https://developers.figma.com/docs/figma-mcp-server/rate-limits-access/ (VERIFIED). Whether CC-related MCP tools work on Pro without mappings: UNVERIFIED.
- OSS usage: I could not find a verified OSS design system's public `.figma.tsx` set (Primer example not found). UNVERIFIED; check github.com/primer/react and github.com/figma/code-connect issues before citing anything.
- Alternatives on Pro:
  1. Storybook Connect (Figma plugin from Chromatic/Storybook): links Figma components/variants to published stories; Figma-side link on instances. Needs Chromatic-published Storybook (free tier exists - verify). https://help.figma.com/hc/en-us/articles/360045003494 and Storybook design-integrations docs.
  2. `@storybook/addon-designs`: embed Figma frames in stories (works regardless of file sharing). https://storybook.js.org/docs/sharing/design-integrations
  3. Component description/documentation links in Figma pointing to docs URL; matching property names; `codeSyntax` on variables so Dev Mode shows `var(--ds-color-bg-surface)`.
  4. Write the `.figma.tsx` files anyway (they are small) and commit them with a note "publishes on Org plan" - signals competence but cannot be tested without a seat. Consider a one-month Org trial or asking a team with Org seat to validate; label it untested otherwise.
  5. Figma's curated UI kits get free code snippets on Pro; irrelevant for custom kit.

## 4. Token sync pipelines

| Option | Pro plan? | Direction | Pros | Cons |
|---|---|---|---|---|
| Native Figma export/import (mode JSON, DTCG) | Probably (UNVERIFIED) | Manual both | Zero tooling, DTCG | One mode/file, manual, sRGB-only colour, no CI |
| Tokens Studio plugin | Free tier + paid | Two-way via GitHub | Mature, themes, DTCG format option, Style Dictionary | Multi-file/folder sync is Pro (paid); non-local variables/styles Pro; subscription (SECONDARY: Free EUR0, Team EUR39/mo annual) https://tokens.studio/pricing ; https://docs.tokens.studio/token-storage/remote/sync-git-github |
| Figma REST variables API + GitHub Actions | No (Enterprise) | Two-way | Fully automated | Enterprise only |
| Custom plugin (Plugin API) + DTCG repo | Yes | Pull/push on demand | Free, you own format, perfect portfolio piece, runs contrast checks | Manual trigger (no headless CI unless user runs it); maintenance |
| Community DTCG plugins (e.g. Token Press, Design Token Exporter, davo/figma-variables-import) | Yes | Mostly one-way | Fast start | Quality varies; one admits non-compliant DTCG parser (SECONDARY) |

Recommended source of truth: `tokens/*.json` (DTCG 2025.10) in the repo -> Style Dictionary v4 or Terrazzo -> CSS custom properties (+ TS types). Terrazzo also imports from Figma but needs Enterprise REST. https://terrazzo.app/docs/guides/import-from-figma (SECONDARY)

### Extending ContrastMate: "Token Contrast Auditor" (proposed)
Because the Plugin API can read variables on any plan, a sibling feature inside ContrastMate (or a mode) can do what REST would need Enterprise for.

Feature sketch:
1. Load collections via `figma.variables.getLocalVariableCollectionsAsync()`; resolve each mode using `resolveForConsumer` so aliases flatten.
2. Pair definition file `contrast-pairs.json` (or derive by naming/scope convention: `color/text/*` x `color/bg/*`, `color/border/*` vs surfaces for 3:1, `color/focus/ring`), stored in plugin data or repo. Pair rules carry required level (AA 4.5, AA large 3, AAA 7, non-text 3, APCA Lc thresholds).
3. Run for every mode (light/dark/high-contrast), compute WCAG 2.x ratio and APCA Lc, handle alpha by compositing over the paired background.
4. Output: matrix view (rows = pairs, columns = modes) with pass/fail, nearest passing value suggestion (adjust lightness in OKLCH), click to select/bind the variable; optional "fix" writes new value to the variable.
5. Export report JSON/markdown and a `contrast-report.json` consumed by a CI script in the repo (same pairs file, same maths published as a tiny shared package) so Figma and code cannot drift; badge in README.
6. DTCG import/export in the same plugin (round-trip with `$extensions` for scopes/codeSyntax; preserve `com.figma.aliasData` semantics) - gives full Pro-plan sync without Tokens Studio.
7. Annotate: optional step that places annotation/measurement-style stickers on component frames showing actual contrast for text-on-fill instances (uses annotations API).
Portfolio angle: accessibility-first system whose accessibility is machine-checked at design time (plugin) and build time (CI) from one token source. ContrastMate is the shipped proof of ability to ship a plugin.
Constraints: plugin is user-triggered (no headless Figma on Pro); keep contrast math in a shared TS module used by both the plugin and CI.

## 5. OSS design systems with public Figma kits
- Radix Themes: no official Figma kit; resources page lists one unofficial community kit (Victor Allegret). Lesson: code-first systems lose designers without a first-party kit. https://www.radix-ui.com/themes/docs/overview/resources (VERIFIED)
- shadcn/ui: community kits only (Obra shadcn/ui MIT with plugin, Pietro Schirano's kit, paid shadcn/studio etc.), listed in shadcn docs. Lesson: kit quality and naming parity vary; a first-party matching kit is a differentiator. https://shadcn-vue.com/docs/figma (SECONDARY)
- Primer (GitHub): official libraries split (Primitives, Web, Octicons, Templates), variables with scoping and light/dark modes, components match Primer React/ViewComponents/CSS; contributions only by GitHub employees via branches; sync is by convention and process (no documented automation). https://primer.style/guides/figma (VERIFIED)
- Polaris (Shopify): primitive + semantic tokens since v12, Figma variables for color/space/size, names mirroring CSS vars (`--p-`), tokens published as npm package, Figma plugin for components (SECONDARY). https://polaris.shopify.com/previous-releases/version-12
- Atlassian: token-based Figma libraries rebuilt for light/dark, token Figma plugin to apply/preview tokens, design-code parity as stated goal (2022 sources; current pipeline not verified). https://community.developer.atlassian.com/t/introducing-design-tokens-new-colour-foundations-and-dark-mode/62258
- GOV.UK: official team does not maintain a Figma kit; community kit referenced in resources, with no support statement; several departments publish their own kits. https://design-system.service.gov.uk/community/resources-and-tools/ (VERIFIED via snippet)
- Untitled UI: commercial; colour/spacing/radius/width/typography/effects variables, dark mode, foundations page; no code sync described (SECONDARY). https://untitledui.com/changelog
- CVS annotation kit: model for docs + open licence + GitHub wiki/discussions.
- What makes them good: foundations separated from components, semantic naming, scoped variables, light/dark modes, components using properties not overrides, docs inside the file, public changelog, licence clarity. Sync reality: tokens sync automatically (where at all); components sync by parity review. A 2026 vendor article makes the same claim (SECONDARY): https://atomize.tools/blog/figma-design-system-parity-code-sync/

## 6. Professional plan: what is possible
| Need | On Pro? |
|---|---|
| Variables, modes (10), aliasing, scoping, typography variables, code syntax | Yes |
| Plugin API variable read/write, annotations API | Yes |
| Slots, component properties, auto layout min/max | Yes (Slots needs Full seat) |
| Team library publishing (Pro supports shared libraries) | Yes (verify seat/team setup) |
| Publish to Community, free, CC BY 4.0 | Yes |
| Dev Mode (Dev seat or Full seat on paid plan) | Yes, paid seat; Pro Dev seat ~$12 annual (SECONDARY) |
| Figma MCP server (200 calls/day, 10/min) | Yes |
| Extended collections | No (Enterprise) |
| Variables REST API (read/write) | No (Enterprise) |
| Code Connect custom mappings (UI or CLI) | No (Org/Enterprise) |
| Native DTCG export/import | Unverified; test in your file |

## Recommendations
1. Make DTCG JSON in the repo the source of truth; Style Dictionary/Terrazzo for CSS vars; hide primitives, scope semantics, set `codeSyntax.WEB` to the CSS variable name.
2. Build the kit in two libraries (Foundations, Components) plus a Community-published bundle; props-to-properties 1:1, Slots for composition, 2 modes (light/dark) plus optional high-contrast.
3. Ship Storybook with `addon-designs` and Storybook Connect for Pro-compatible linking; commit `.figma.tsx` files labelled untested, or borrow an Org seat for a trial month to validate and screenshot Dev Mode snippets for the portfolio.
4. Build "Token Contrast Auditor" (and DTCG round-trip) as a ContrastMate mode; share contrast maths with a CI check; document the story as "accessibility enforced from tokens to pixels".
5. Include CVS-style accessibility annotation frames in each component doc page; credit CVS (CC-BY-4.0).
6. License: kit CC BY 4.0 on Community (default), code MIT; version and changelog page in the file.

## Items to verify manually
- Native Export/Import mode availability on your Pro file.
- Exact mode cap on Pro (10 vs other) and VariableScope enum.
- Slots on your Pro Full seat; Community seller status (irrelevant if free).
- Whether Storybook Connect needs a paid Chromatic tier.
- Any OSS repo with real Code Connect files.
- Current pricing at https://www.figma.com/pricing.
