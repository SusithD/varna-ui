# varna-ui: status (paused 2026-10-11)

varna (වර්ණ) means colour in Sinhala. varna-ui is a small, accessibility-evidenced React design system on Base UI. Its tokens are gated on contrast, it has high-contrast and forced-colors themes, it supports Sinhala and Tamil type, and it has a matching Figma kit.

## Done
- **Research.** Seven sourced reports are in `research/`, about 2,000 lines in total. The key claims were checked against npm, GitHub and the source repos.
  - 00: notes on reusing ContrastMate.
  - 01: landscape.
  - 02: tokens and theming.
  - 03: tooling and release.
  - 04: accessibility.
  - 05: Figma kit.
  - 06: hiring and positioning.
- **Decision brief.** `research/BRIEF.md` covers positioning, the decision table, v1 scope with a definition of done, differentiators, milestones, risks and spikes, and a ledger of verified claims.
- **Owner decisions** are recorded in BRIEF §7:
  1. React 19 only.
  2. ContrastMate v2 uses `apca-w3` with a licence notice. APCA stays informational.
  3. The demo is a booking flow in `apps/demo`.
  4. Calendar and DatePicker ship in v1.1.
  5. A Tailwind `@theme` bridge ships from v0.1.
  6. The name is `varna-ui`, with packages `@varna-ui/tokens`, `@varna-ui/react` and `@varna-ui/contrast`. Lead with the Sinhala meaning, because "varna" also has a caste meaning in Sanskrit and Hindi.
- **Public repo** at https://github.com/SusithD/varna-ui, with the research pushed.
- **Names:** `varna-ui` and the `@varna-ui/*` packages are free on npm. GitHub org `varna-ui` is free.

## Key findings
- **Primitives:** Base UI (`@base-ui/react` 1.9.0). It is shadcn's default since July 2026 and the base of Cal.com's coss ui. Radix is slower to release, and its Select has a touch screen-reader bug open since 2022 (#1343).
- **Tokens:** DTCG 2025.10 JSON in git is the source of truth, built with Style Dictionary 5.6.1. Figma Variables are synced by a custom plugin, because the REST API is Enterprise-only.
- **Contrast:** WCAG 2.2 AA is the CI gate. APCA is informational only, because `apca-w3` is under a "Limited W3 License".
- **Toolchain pins:** TypeScript 6.0.x, because typescript-eslint supports only versions below 6.1. ESLint 9, because jsx-a11y supports only up to 9. Also Storybook 10.6.1, Vite 8.3.4, Vitest 5.0.3, tsdown 0.23.1 and pnpm 12.11.2.
- **Figma on the Professional plan:** Variables, modes, Slots and Community publishing all work. Code Connect needs an Organization plan.
- **Cal.com:** the Senior Product Design Engineer role asks for a repo or component library. Its repo is still public and MIT. Its design system, coss ui, is AGPL, so read its docs but don't copy its code.

## Next (when resumed)
1. Write `EXECUTION.md`, the task plan for the M0 spike and v0.1. The planning agent was stopped before writing it.
2. M0 spike, weeks 1–2:
   - monorepo scaffold and CI;
   - tsdown ESM with `'use client'` and CSS output;
   - Storybook 10 with Vite 8 and the Vitest addon, with axe set to `error`;
   - Style Dictionary DTCG to CSS variables per theme, plus forced-colors and the Tailwind bridge;
   - the contrast gate;
   - Base UI Select and Menu keyboard tests with axe;
   - Sinhala and Tamil line-height.
3. v0.1, about weeks 3–8:
   - `@varna-ui/contrast`, ported from ContrastMate without the APCA constants;
   - `@varna-ui/tokens`;
   - `@varna-ui/react`: Button, Field and Input, Checkbox, Radio and Switch;
   - Storybook on GitHub Pages.

## Needs the owner
- Run `npm login` before the first publish.
- Test Select and Menu with iOS VoiceOver and TalkBack on a real phone (script to follow).
