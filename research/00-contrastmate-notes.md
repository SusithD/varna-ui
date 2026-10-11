# ContrastMate: what the design system can build on

Source: https://github.com/SusithD/ContrastMate (read on 2026-10-11). The plugin is v1.1.0, MIT-licensed, built with TypeScript and Preact via `create-figma-plugin`, and tested with Vitest.

## Reusable engine: `src/contrast.ts` (450 lines, no dependencies)
- **WCAG 2.x maths:** sRGB to linear conversion, relative luminance and contrast ratio.
- **Alpha:** contrast that accounts for alpha, and blending of a foreground over a background.
- **Text size:** large-text detection (18pt, or 14pt bold) and font-weight parsing.
- **Levels:** `getWCAGLevel`, `passesWCAG_AA` and `passesWCAG_AAA`.
- **APCA Lc:** `calculateAPCALc`, with constants that match apca-w3 v0.1.9.
- **Tests:** `src/contrast.test.ts` (365 lines). The README badge claims 41 passing tests.

## Other parts
- `src/scanner.ts` (736 lines) walks Figma layers, finds the real background behind text and handles fonts from linked libraries.
- `src/ui.tsx` and `components.tsx` hold the plugin UI, built in Preact and keyboard accessible.

## How the design system can use it
1. **Shared package.** Move `contrast.ts` and its tests into a `contrast` package in the design-system monorepo. Both ContrastMate and the token CI then use the same engine, which tells one connected story across both projects.
2. **Contrast checks in CI.** Every declared foreground/background token pair, in every theme, is checked against its WCAG target, and the build fails if a pair falls short.
3. **Figma.** The plugin, or a new mode in it, validates the Figma kit's variable pairs with the same engine.

## Open questions
- **APCA licence.** The apca-w3 reference implementation has its own licence terms, not plain MIT. Confirm whether re-implementing its constants inside an MIT package is allowed. Until then, mark APCA as "informational, not a conformance claim" (WCAG 3 is still a draft).
- **WCAG version.** The README says WCAG 2.1. The design system should target WCAG 2.2, whose contrast criteria are unchanged.
