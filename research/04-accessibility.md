# 04 - Component Accessibility Reference

Research date: 2026-10-11 (all "accessed" dates below are 2026-10-11 unless stated).
Confidence tags: [V] = verified by fetching the cited page this session; [K] = from established knowledge of WCAG/APG/ARIA (stable spec text, cited by canonical URL but not re-fetched); [U] = unverified / needs checking before you rely on it.

Search tooling note: web search returned mostly third-party summaries; anything sourced only from a blog is flagged. Issue statuses are snapshots from 2026-10-11 and will drift.

---------------------------------------------------------------------
## 0. Executive findings

1. Headless primitives give you roles, keyboard and focus management; they do NOT give you contrast, focus-ring appearance, target size, text-spacing resilience, forced-colors, reduced motion, accessible names for icon buttons, or error-message wiring. Those are the design system's own responsibility and are where an a11y-first DS earns its credibility.
2. Primitive choice (as of Oct 2026):
   - Base UI: v1.0 stable 2025-12-11, v1.9.0 on 2026-10-09, monthly releases, active a11y labelling in tracker (43 issues labelled `accessibility`, ~9 of newest 25 open) [V]. Has some open touch-screen-reader gaps (Menu on iPhone VO #5794) and a documented Toast timing debate (#4253).
   - React Aria (Components): deepest a11y engineering (explicit SR matrix incl. iOS VO + TalkBack, 30+ locales, RTL, virtual-focus handling, press/hover normalisation) [V]; still has an open backlog of VoiceOver announcement issues (Combobox, Autocomplete, RadioGroup required) [V].
   - Radix Primitives: widest adoption, but maintenance slowed; long-lived "Needs Investigation" a11y bugs (Select + touch SR #1343 open since 2022; Tooltip focus #1800 open) and an unanswered 35-issue third-party audit [V]. Maintenance picture is blog-sourced [U].
   - Recommendation for an a11y-first portfolio piece: React Aria Components or Base UI. If Base UI, budget for your own SR testing of Menu/Select/Combobox on iOS. If Radix, budget for workarounds and say so honestly in the ACR.
3. Automated tests (axe) catch roughly half of issues by volume (Deque: 57% of issue volume, vendor study, 2021) [V], but far fewer success criteria can be fully verified automatically. Keyboard, focus, SR announcement, zoom, forced-colors and touch behaviour need scripted and manual tests.
4. WCAG 2.2 AA is the target; EN 301 549 v4.1.1 (adopts WCAG 2.2 AA) published 2026-09-02 [V]; presumption of conformity only after OJEU citation (date sources conflict, see 1.3).

---------------------------------------------------------------------
## 1. WCAG 2.2 and legal context

### 1.1 Criteria most relevant to a component library

Spec: https://www.w3.org/TR/WCAG22/ ; new-in-2.2 summary https://www.w3.org/WAI/standards-guidelines/wcag/new-in-22/ (accessed 2026-10-11) [V]. Understanding docs pattern: https://www.w3.org/WAI/WCAG22/Understanding/<criterion-slug> [K].
Note: WCAG 2.2 removed 4.1.1 Parsing (obsolete) [K].

| SC | Level | What a component MUST do | Component hot-spots |
|---|---|---|---|
| 1.1.1 Non-text Content | A | Icons/images have text alternative or are hidden (aria-hidden / alt="") when decorative | IconButton, Avatar, Badge icons, Alert icons, Spinner |
| 1.3.1 Info and Relationships | A | Structure conveyed visually is in markup/ARIA: labels tied to controls, group/fieldset+legend, table headers, lists, headings | Field, Radio group, Table, Breadcrumb (nav+ol), Card heading |
| 1.3.2 Meaningful Sequence | A | DOM order = visual/reading order; no positive tabindex | Card, Dialog footer, Table |
| 1.3.4 Orientation | AA | No lock to portrait/landscape | Dialog, Drawer |
| 1.3.5 Identify Input Purpose | AA | Support `autocomplete` attribute on personal-data inputs; forward it | TextField |
| 1.4.1 Use of Color | A | State (error, selected, link) not by color alone | Link in text (underline), Field error (icon+text), Tabs selected, Badge |
| 1.4.3 Contrast (Min) | AA | Text 4.5:1 (3:1 large: 24px or 18.66px bold) incl. disabled-looking-but-not-inert text, placeholder, helper text | Every token pair; verify in light/dark and every state |
| 1.4.4 Resize Text | AA | Text resizes to 200% without loss; use rem, no fixed-height text containers | Button, Input, Badge, Tooltip |
| 1.4.5 Images of Text | AA | No text in images | Logo only |
| 1.4.10 Reflow | AA | No 2-D scroll at 320 CSS px (400% zoom of 1280) except data tables, maps, etc. | Table (allow own scroll container), Dialog (must scroll within), Tabs overflow, Pagination wrap |
| 1.4.11 Non-text Contrast | AA | 3:1 for UI component boundaries needed to identify them, states, focus indicators, and graphical objects (checkbox box, switch track/thumb, input border, slider track) | Input border, Checkbox, Radio, Switch, Select, Progress bar, Spinner |
| 1.4.12 Text Spacing | AA | No loss when user sets line-height 1.5, paragraph 2x, letter 0.12em, word 0.16em | Fixed heights/overflow:hidden on Button, Badge, Tab, Tooltip, Toast; truncation |
| 1.4.13 Content on Hover or Focus | AA | Hover/focus-triggered content is Dismissible (Esc), Hoverable (pointer can move onto it), Persistent | Tooltip, Popover openOnHover, Menu submenus, hover cards |
| 2.1.1 Keyboard | A | All functionality via keyboard | Everything; drag, hover-only, swipe-to-dismiss |
| 2.1.2 No Keyboard Trap | A | Focus can leave; modals trap by design but must have Esc/close | Dialog, Popover, Menu, Combobox, any embedded widget |
| 2.1.4 Character Key Shortcuts | A | Single-key shortcuts can be turned off/remapped or only active on focus | Typeahead is OK when scoped to focused widget; global hotkeys not |
| 2.2.1 Timing Adjustable | A | Time limits can be turned off/adjusted/extended | Toast auto-dismiss (see Base UI #4253), session timeouts |
| 2.2.2 Pause, Stop, Hide | A | Moving/auto-updating content >5s can be paused | Spinner/Skeleton shimmer, carousel, marquee |
| 2.3.1 Three Flashes | A | No >3 flashes/sec | Skeleton shimmer, animations |
| 2.4.1 Bypass Blocks | A | Skip link/landmarks (app level; DS ships SkipLink) | SkipLink |
| 2.4.3 Focus Order | A | Logical focus order incl. portals, dialogs, return of focus | Dialog, Popover, Menu, Toast viewport |
| 2.4.4 / 2.4.6 | A / AA | Link purpose from text/context; headings and labels descriptive | Link ("read more"), Card, Breadcrumb |
| 2.4.7 Focus Visible | AA | Keyboard focus indicator visible on every focusable element | All; never `outline:none` without replacement |
| 2.4.11 Focus Not Obscured (Min) | AA (new) | Focused item not entirely hidden by author content (sticky headers, toasts, cookie banners) | Use `scroll-margin`/`scroll-padding`, Toast placement, sticky Table headers, Dialog footers |
| 2.4.12 Focus Not Obscured (Enhanced) | AAA | No part hidden | Aim for it |
| 2.4.13 Focus Appearance | AAA (new) | Indicator >= 2 CSS px perimeter-equivalent thickness, 3:1 contrast between focused/unfocused states | Adopt as house standard: 2px solid ring, 3:1 vs adjacent colors, 2px offset |
| 2.5.1 Pointer Gestures | A | Multi-point/path gestures have single-pointer alternative | Slider, carousels, map |
| 2.5.2 Pointer Cancellation | A | Activate on up-event (click), not down; allow abort | Button/Menu items. Note Base UI issues #4432/#4433 flagged Slider/ScrollArea vs 2.5.2 [V] |
| 2.5.3 Label in Name | A | Accessible name contains visible label text | Button, IconButton w/ visible text, Tab, Link |
| 2.5.4 Motion Actuation | A | No shake/tilt-only | n/a |
| 2.5.7 Dragging Movements | AA (new) | Every drag has a single-pointer alternative (click/tap) | Slider (click track + keyboard), sortable lists, Table column resize, file drop (also button) |
| 2.5.8 Target Size (Min) | AA (new) | Pointer targets >= 24x24 CSS px, or 24px-diameter circle around its center doesn't intersect another target; exceptions: inline text links, user-agent controls, essential | IconButton, Checkbox/Radio, Pagination, Badge-with-remove, Close buttons, Breadcrumb separators. House standard recommendation: 44px touch where possible (AAA 2.5.5 is 44x44) |
| 3.1.1 / 3.1.2 | A / AA | Language of page/parts (`lang`) | Docs site; locale prop on DS |
| 3.2.1 / 3.2.2 | A | No unexpected context change on focus/input | Select (don't navigate on change), Tabs |
| 3.2.3 / 3.2.4 | AA | Consistent navigation / identification | Same icon = same name across DS (see shadcn audit mention of 3.2.4 [V, blog]) |
| 3.2.6 Consistent Help | A (new) | Repeated help mechanism appears in same relative place | Page-template level; DS can ship a Help slot in layout shells |
| 3.3.1 Error Identification | A | Error identified and described in text | Field |
| 3.3.2 Labels or Instructions | A | Visible labels/instructions | Field |
| 3.3.3 Error Suggestion | AA | Suggest corrections when known | Field error copy guidance |
| 3.3.4 Error Prevention | AA | Reversible/confirmed/checked for legal/financial submissions | AlertDialog confirm patterns |
| 3.3.7 Redundant Entry | A (new) | Don't re-ask info already provided in same process (auto-populate or allow select) | Form templates; `autocomplete` |
| 3.3.8 Accessible Authentication (Min) | AA (new) | No cognitive function test to authenticate (no memorise/transcribe) unless alternative/mechanism; password fields must allow paste and password managers | PasswordInput: don't block paste, `autocomplete="current-password"/"new-password"`, no `onpaste` prevention; show/hide toggle |
| 3.3.9 | AAA (new) | Enhanced version | - |
| 4.1.2 Name, Role, Value | A | Every control exposes name, role, state/value programmatically; changes notified | All widgets (the core of ARIA correctness) |
| 4.1.3 Status Messages | AA | Status messages (not focus-moving) announced via role=status/alert/log or aria-live | Toast, Alert (dynamic), Field error on submit, Progress completion, Pagination "page 3 of 10", combobox result counts |

### 1.2 Reading notes
- 2.4.11 AA vs 2.4.13 AAA: the AA bar is just "not entirely hidden"; the visible ring-quality requirement is AAA. Most reviewers (and your ACR) will still look for 1.4.11 3:1 on the focus indicator, so treat 2.4.13 as target.
- 2.5.8 exceptions: spacing, equivalent (another control meets it), inline, user-agent, essential. Document which of your controls rely on the spacing exception.
- 3.3.8: password fields and 2FA are mostly app-level, but a DS PasswordInput/OTPInput must not break paste/autofill. OTP: use `autocomplete="one-time-code"`, a single input or an input that accepts paste.
- Sources: W3C new-in-2.2 [V] https://www.w3.org/WAI/standards-guidelines/wcag/new-in-22/ ; full TR [K].

### 1.3 EN 301 549 and the European Accessibility Act (EAA)
- EAA (Directive (EU) 2019/882) applicable from 2025-06-28 [V via search summary; K]. Applies to specified consumer products/services (e-commerce, banking, e-books, etc.), with member-state transposition and a microenterprise exemption for services [K, U for exact wording].
- EN 301 549 V3.2.1 (2021) references WCAG 2.1 AA; V4.1.1 adopts WCAG 2.2 AA, reworks user preferences, updates real-time communication (Clause 6), adds EAA mapping annexes; published 2026-09-02 [V: https://www.deque.com/blog/en-301-549-v4-1-1-is-final-what-changed-what-it-means-and-what-you-should-do/ , accessed 2026-10-11].
- Harmonisation/OJEU citation date: Deque says expected ~2026-11-30; another law-firm summary says 2026-12-16 [U: https://www.dwt.com/insights/2026/09/european-accessibility-act-ict-standards-update - only seen as a search snippet]. Until cited, V3.2.1 remains the presumption-of-conformity reference.
- WCAG 2.2 AA conformance != EN 301 549 conformance (EN has extra clauses, e.g. real-time comms, documentation/support, software-specific). A web component library mostly maps to Clause 9 (web = WCAG) and 11 (software) [V Deque statement; K for clause numbers].
- OSS relevance [U]: a free, non-commercial OSS library is likely not itself a "product placed on the market" under the EAA, but adopters shipping covered products are; hence consumers want an ACR/VPAT-style statement from the DS. Verify against the Directive text before claiming anything legal. Do not give legal advice in the docs; say "intended to help adopters meet".

---------------------------------------------------------------------
## 2. Per-component specification

APG index: https://www.w3.org/WAI/ARIA/apg/patterns/ ; ARIA spec https://www.w3.org/TR/wai-aria-1.2/ (1.3 draft exists) [K]. APG pattern URLs below are canonical [K]; APG itself says patterns are illustrative, not normative, and have uneven AT support (https://www.w3.org/WAI/ARIA/apg/ ) [K].

Legend for test columns: AXE = axe-core / jest-axe / @axe-core/playwright catches it; KB = needs scripted keyboard test (Playwright/Testing Library user-event); SR = needs manual SR pass (or an automated SR driver such as Guidepup, [U] maturity).

### 2.1 Foundational rules (apply to all)
- First rule of ARIA: use native element if one exists. Prefer `<button>`, `<a href>`, `<input>`, `<select>`, `<dialog>` where the design allows.
- Name each control: visible label, `aria-labelledby`, or `aria-label`. Name must contain visible text (2.5.3).
- Never `aria-hidden` focusable elements; never nest interactives.
- Disabled: prefer `aria-disabled="true"` + still focusable for buttons where discoverability matters (APG says focusable disabled is acceptable in composites); native `disabled` removes from tab order; disabled controls are exempt from contrast (1.4.3 exception) but document it.

### 2.2 Component tables

#### Button
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/button/
- Roles/props: native `<button type="button">`; toggle: `aria-pressed`; menu button: `aria-haspopup`, `aria-expanded`, `aria-controls`; loading: keep name, set `aria-disabled`/`aria-busy`, announce via status region.
- Keyboard: Enter and Space activate (Space on keyup).
- Focus: visible ring (2.4.7/1.4.11), 24px min target (2.5.8).
- Pitfalls: `div role=button` without key handling; loading spinner replaces label so name vanishes; disabled hides focus; changing label on loading breaks SR (Base UI #4249 "NVDA doesn't announce loading state example" [V]); `aria-pressed` + changing label double-toggles; icon+text where aria-label drifts from visible text (2.5.3).
- AXE: button-name, aria-allowed-attr, color-contrast (only rendered states you test). KB: Enter/Space, disabled behaviour. SR: loading announcement, toggle state phrasing.

#### IconButton
- Required: accessible name (`aria-label` or visually-hidden text), icon `aria-hidden="true" focusable="false"`; tooltip is a supplement, not the name (Base UI doc states trigger needs aria-label matching tooltip [V: https://base-ui.com/react/components/tooltip]).
- Make `label` a REQUIRED prop in TypeScript (no default); lint with eslint-plugin-jsx-a11y.
- Target >= 24px (recommend 40-44px hit area via padding/pseudo-element).
- Pitfalls: label names the icon not the action ("Trash" vs "Delete comment"); same icon different names (3.2.4); toggles (star) must use aria-pressed rather than swapping labels.
- AXE: button-name catches missing name only, NOT bad names. Review icon names manually.

#### Link
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/link/
- Native `<a href>`; Enter activates (not Space). Opening new tab: say so in name/visually-hidden text; `target=_blank` + `rel=noopener`.
- Links in text blocks need non-color cue (underline) 1.4.1 or 3:1 contrast vs surrounding text plus non-color on focus/hover. Link-vs-button: navigation = link, action = button.
- Pitfalls: `<a>` without href (not focusable); router Link wrappers that drop href; "click here" names; `aria-current="page"` for nav.
- AXE: link-name, link-in-text-block. SR: current-page announcement.

#### TextField / Input
- Spec: no APG pattern (native). HTML + ARIA 1.2.
- Props: associated `<label for>`; `aria-describedby` for description + error; `aria-invalid="true"` when invalid; `required` (native) + visible indication that is not color/asterisk-only; `autocomplete` tokens (1.3.5); correct `type`/`inputmode`.
- Keyboard: native.
- Pitfalls: placeholder as label (contrast + disappears); error only by red border; asterisk `aria-hidden` making required invisible to AT (MUI #44740 documents required/invalid placed on hidden input [V via search]); `aria-errormessage` has weak AT support, so prefer `aria-describedby` (React Aria #7425 discusses this [V]); label not wired in custom Field wrappers (recurring real-world bug class e.g. hirobius/hds PR #421 [V]); autofocus on load; `maxLength` silently truncating.
- AXE: label, aria-valid-attr-value, autocomplete-valid. KB: error focus on submit. SR: error + description order, iOS VO announces invalid state oddly (React Aria #9119 [V]).

#### Textarea
- As TextField, plus: auto-grow must not clip (1.4.4/1.4.12), char counter should be polite live region throttled (not per keystroke), resize handle usable (target size UA exception).
- Pitfall: counter announced every keystroke; `aria-live` on counter spam.

#### Checkbox
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/checkbox/
- Native `<input type=checkbox>`; indeterminate set via JS `indeterminate` (maps to mixed). Custom: `role=checkbox`, `aria-checked` true/false/mixed.
- Keyboard: Space toggles.
- Pitfalls: custom box hides native input with `display:none` (removes from AT/tab); target < 24px (2.5.8) - extend label hit area; checked state by color only; group missing fieldset/legend; "select all" mixed state not announced.
- AXE: checkbox name; non-text contrast not reliably. React Aria #9687 notes SR focus-indicator misalignment on Checkbox/Radio with hidden inputs [V].

#### Radio group
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/radio/
- `role=radiogroup` (or fieldset+legend), each `role=radio` + `aria-checked`; roving tabindex.
- Keyboard: Tab enters group (to checked, else first), Arrow keys move AND check, Space checks focused; arrows wrap; RTL flips left/right.
- Pitfalls: required/invalid announced on every radio instead of group (React Aria #9809 [V]); read-only not announced (#2581 [V]); group lacks name; Radix/Headless roving focus + SR "focus selects" mismatch.
- KB test essential (axe can't verify arrow behaviour).

#### Switch
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/switch/
- `role=switch` + `aria-checked` (or native checkbox with role=switch). Label must not change with state (state is conveyed by role).
- Keyboard: Space toggles (Enter optional).
- Pitfalls: label says "Enable X" and state "on" -> "Enable X on" confusion; off/on by color only; uses switch for actions needing submit (use checkbox); 3:1 for track/thumb vs background in both states.
- Forced colors: thumb/track need system colors (see 4.1).

#### Select (native / custom listbox) 
- Native: https://www.w3.org/WAI/ARIA/apg/patterns/combobox/examples/combobox-select-only/ for select-only combobox; native `<select>` is the most robust option on mobile.
- Custom: trigger `role=combobox`, `aria-haspopup=listbox`, `aria-expanded`, `aria-controls`, popup `role=listbox`, `option` + `aria-selected`; focus stays on trigger with `aria-activedescendant` (virtual focus) or moves to options.
- Keyboard: Enter/Space/Down/Alt+Down open; Up/Down move; Home/End; type-ahead; Esc close and return to trigger; Enter selects; Tab closes (APG select-only commits value).
- Pitfalls (library-specific, see 3): touch SR closes popup immediately (Radix #1343 [V]); SR truncates value (Radix #3671 [V via search]); first option no announcement when nothing preselected (Radix #4110 [V via search listing]); required/invalid on hidden native input (MUI #44740); empty value state; label not bound to trigger. Prefer offering native Select as default and custom for advanced.
- AXE: aria-required-children, aria-valid-attr-value (activedescendant target exists), aria-input-field-name. KB + SR essential; axe can't judge virtual focus announcements.

#### Combobox / Autocomplete
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/combobox/ (list autocomplete, with/without inline completion, grid popup)
- Input `role=combobox`, `aria-expanded`, `aria-controls`, `aria-autocomplete` (list|both|none), `aria-activedescendant`; popup listbox/grid/tree/dialog.
- Keyboard: Down opens/moves; Up; Enter accepts; Esc closes, second Esc clears; Alt+Down opens without moving; Home/End within input; Tab behaviour defined.
- Status: announce result count via polite live region (4.1.3) - Base UI #4181 "limited results status message not announced" [V]; React Aria LiveAnnouncer issue #5191 NVDA/JAWS first announce [V].
- Pitfalls: ARIA 1.0 vs 1.1/1.2 combobox structure differences [K: https://github.com/w3c/aria/wiki/Resolving-ARIA-1.1-Combobox-Issues]; selected option not announced (Carbon #21971 [V via search]); multi-select values unread (MUI #40024 [V via search]); VoiceOver announces only item name (React Aria #10220 [V]); grouped sections announced wrong (#5839 [V]); wrapper `div` inside option breaks TalkBack (Base UI #4435 [V]); invalid grid ARIA ownership (Base UI #5560 closed [V]); non-modal popup aria-hides outside but leaves tabbable (Base UI #5528 closed [V]); shadcn Popover+Command example lacks combobox attrs (blog [V, third-party]).
- Test: axe weak; scripted KB + 3 SRs mandatory.

#### Dialog / Modal (and AlertDialog)
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/ ; alert dialog https://www.w3.org/WAI/ARIA/apg/patterns/alertdialog/
- `role=dialog` (or `<dialog>` + showModal), `aria-modal=true`, `aria-labelledby` (title), `aria-describedby` (optional description); AlertDialog `role=alertdialog` requires description, no outside-click dismiss.
- Focus: move into dialog on open (first focusable, or the least destructive action / heading with tabindex=-1 for long content); trap Tab/Shift+Tab; background inert (`inert` attribute preferred over aria-hidden); Esc closes; return focus to trigger (or logical fallback if trigger unmounted).
- Pitfalls: focus does not return if trigger removed (Vaul note in shadcn audit [V, blog]); tooltip on trigger reopens when focus returns (Radix #1800 [V]); "Focus moves to wrong trigger when opened programmatically" (Base UI #4169, closed [V]); aria-hidden on app root also hides live regions so toasts/alerts are silent while modal open (Radix #1820, closed 2026-06-03 [V]); missing title warning; description not read on some VO/macOS versions (Radix #2038 [V]); body scroll lock layout shift; dialog taller than viewport at 400% zoom (1.4.10); sticky footer obscures focus (2.4.11).
- AXE: aria-dialog-name, aria-hidden-focus. KB: trap, Esc, return focus. SR: title/description announced on open.

#### Popover (non-modal dialog / disclosure)
- APG: dialog pattern (non-modal) https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/ ; disclosure https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/
- Trigger: `aria-expanded`, `aria-controls`, `aria-haspopup="dialog"`. Content `role=dialog` with name if it contains interactive content.
- Focus: move into popover on open when interactive; Esc closes and returns; Tab out closes (non-modal); content must stay reachable in DOM order (portal pitfall).
- Hover-open variants must satisfy 1.4.13 and have non-hover (click/focus) path; Base UI recommends Popover openOnHover for info icons on touch [V].
- Pitfalls: portaled content breaks logical Tab order; role=dialog w/o name; outside-click-only dismissal for keyboard.

#### Tooltip
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/tooltip/ (APG pattern is "work in progress" [K])
- Trigger gets `aria-describedby` -> `role=tooltip` (or aria-label if it IS the name); opens on hover and keyboard focus; Esc dismisses; hoverable; persistent until dismissed/blur (1.4.13); never contains interactives or essential info.
- Touch: Base UI disables tooltips on touch and says use Popover for important content [V: https://base-ui.com/react/components/tooltip]. Radix docs do not cover touch/warnings (docs only list keyboard) [V: https://www.radix-ui.com/primitives/docs/components/tooltip].
- Pitfalls: opens on window re-focus/tab switch, and after dialog close returns focus (Radix #1800, open, PR #3964 [V]); SR in modal doesn't announce tooltip (React Aria #8915 [V]); tooltip on disabled button (no focus); tooltip duplicates name causing double reading; `title` attribute is not a substitute.
- AXE: partial. KB + manual hover test needed.

#### Menu / Dropdown Menu
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/ and https://www.w3.org/WAI/ARIA/apg/patterns/menubar/
- Trigger: `aria-haspopup=menu`, `aria-expanded`; `role=menu` > `menuitem`/`menuitemcheckbox`/`menuitemradio`; groups `role=group`; separators.
- Keyboard: Enter/Space/Down open (focus first item), Up opens (last); arrows move; Home/End; type-ahead; Right/Left open/close submenus (flip in RTL); Esc closes and returns; Tab closes. Roving tabindex or activedescendant.
- Do NOT use role=menu for site navigation (use disclosure + links) - APG and GOV.UK guidance [K].
- Pitfalls: iPhone VO closes whole menu after last submenu item (Base UI #5794, open [V]); reverse Tab skipping submenu (Base UI #4463 closed [V]); Radix Menubar dropping aria-expanded in shadcn wrapper [V, blog]; context menu inaccessible to keyboard/touch without alternative (Base UI #4171 closed [V]); advertised shortcuts not wired; items not closing on click when migrating Radix->Base (shadcn migration note [V via search]).
- AXE: aria-required-children/parent, aria-allowed-role. KB + SR (VoiceOver especially) essential.

#### Tabs
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/tabs/
- `tablist` > `tab` (`aria-selected`, `aria-controls`) + `tabpanel` (`aria-labelledby`, tabindex=0 if no focusable content). Roving tabindex.
- Keyboard: Tab into tablist focuses selected tab; Left/Right (Up/Down if vertical) move; Home/End; automatic vs manual activation (manual if panel load is costly); Tab moves to panel.
- Pitfalls: SR cursor focusing a tab auto-activates it in automatic mode (Radix #1047 [V via search]); using tabs as nav links; overflowing tabs hidden at zoom; RTL arrows; migrating Radix->Base changes to manual activation default [V via search].
- AXE: aria-required-children, aria-valid-attr-value. KB essential.

#### Accordion
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/accordion/
- Heading (`h3` etc., level configurable) containing `<button aria-expanded aria-controls>`; panel `role=region aria-labelledby` (omit for many panels >6 per APG) [K].
- Keyboard: Enter/Space toggle; optional Up/Down/Home/End between headers (Base UI aligned Accordion keyboard nav with APG in v1.6.0 [V]).
- Pitfalls: button not inside heading; heading level hard-coded; `display:none` vs `hidden="until-found"`; animation of height breaking reduced motion; nested interactive in header.
- AXE: aria-valid-attr-value, heading-order. 

#### Toast
- APG: no dedicated pattern; use Alert https://www.w3.org/WAI/ARIA/apg/patterns/alert/ ; status role https://www.w3.org/TR/wai-aria-1.2/#status
- Live region container must exist in DOM BEFORE content is injected (NVDA/JAWS) [V via search of PatternFly/Polaris, third-party]; `role=status`/`aria-live=polite` for info/success, `role=alert` for errors only; focus is not moved (APG alert); keyboard path to the region (Radix default hotkey F8 [V]; Base UI maintainer cites F6 [V]); pause timers on hover/focus; do not auto-dismiss toasts with actions or errors (2.2.1); provide persistent alternative (inline message / notification list); toast must not obscure focus (2.4.11); reduced motion.
- Library state: Radix Toast has `type` foreground/background, required `altText` on Action, default 5000ms, timers pause on hover/focus/blur [V: https://www.radix-ui.com/primitives/docs/components/toast]; Base UI open issue #4253 (Undo unreachable in time, 2.1.1/2.2.1) with maintainer position that focus must not move, `timeout:0` available [V: https://github.com/mui/base-ui/issues/4253]; #4229 asks for an a11y guide (open) [V]. React Aria's `UNSTABLE_Toast*` / Toast region support exists [K, U on stability].
- Pitfalls: toast with Undo unreachable; stack of toasts clears the SR queue (Radix doc [V]); aria-hidden app root silences live regions while modal open (Radix #1820 closed [V]); `role=alert` for everything.
- AXE: cannot test announcement. SR test essential (NVDA+Chrome/Firefox, JAWS, VO).

#### Alert / Banner
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/alert/
- Static inline message: no live role needed (`role=region`/plain). Dynamic insertion: `role=status` (polite) or `role=alert` (assertive, errors). Dismiss button named. Don't rely on color; include icon with text alternative or severity text ("Error:").
- Pitfall: `role=alert` on page load for non-critical content (announced at load, disruptive); inserting region and message simultaneously (NVDA/JAWS may miss); contrast of tinted backgrounds.

#### Badge
- No pattern. Purely presentational text; if status conveyed by color, include text. Count badges: provide full context ("3 unread messages") via visually hidden text. Don't announce rapidly changing counts. Contrast 4.5:1 for small text. Reflow/spacing: don't clip.

#### Avatar
- Image with meaningful `alt` (person's name) when it's the only identifier; `alt=""` if the name is adjacent. Initials fallback: expose as text/`aria-label`. Group (AvatarGroup) needs "+3 more" text. Pitfall: broken-image fallback timing; `aria-hidden` on only identifier.

#### Card
- No pattern. Prefer one primary link/button per card with whole-card click achieved by stretched-link pseudo-element so heading link is the accessible name (not `role=link` on a div wrapping everything). Heading level configurable. Avoid nested interactive controls inside a clickable card. Focus ring must show around card and not be clipped by `overflow:hidden`. 2.4.4 link purpose.

#### Skeleton
- Decorative placeholder: `aria-hidden="true"`; container gets `aria-busy="true"` and a status/`aria-live` message ("Loading...") once, not per skeleton. Shimmer animation off under reduced motion (2.2.2/2.3.3). Contrast not required (decorative) but must not be mistaken for content. Pitfall: `aria-busy` support inconsistent; SRs read nothing and user thinks page empty.

#### Spinner / Progress
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/meter/ (meter) ; progressbar role https://www.w3.org/TR/wai-aria-1.2/#progressbar
- `role=progressbar` + `aria-valuenow/min/max` + name; indeterminate omits valuenow; spinner alone: `role=status` + text ("Loading"). Announce start and completion, not every percentage (4.1.3). Under reduced motion: static or slow indicator but still conveying loading.
- Pitfalls: NVDA and JAWS don't announce programmatic label on Meter/Progress (Base UI #4184 open [V]); decorative spinner inside button announcing twice; 3:1 contrast on track and fill (1.4.11).

#### Table
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/table/ ; grid https://www.w3.org/WAI/ARIA/apg/patterns/grid/ ; sortable example https://www.w3.org/WAI/ARIA/apg/patterns/table/examples/sortable-table/
- Native `<table>`, `<caption>`/aria-label, `<th scope>`; sortable headers contain a `<button>` with `aria-sort` on `th` (only on the currently sorted column). Use `role=grid` ONLY for interactive cell navigation (spreadsheets).
- Reflow: scroll container with `tabindex=0`, `role=region`, `aria-labelledby` so keyboard users can scroll it; do not convert to divs (`display:block` on table elements drops semantics in some SR/browser combos [K, U on current status]).
- Pitfalls: `div` tables lacking roles; sort state announced inconsistently; sticky header hides focused cell (2.4.11); row selection checkbox names ("Select row 3"); VO announces unfocused cell in grid mode (React Aria #7734 [V]).
- AXE: td-headers-attr, th-has-data-cells, scope-attr-valid. KB (sort, row actions) + SR for headers.

#### Pagination
- Pattern: `<nav aria-label="Pagination">` > `ul` > links/buttons; current page `aria-current="page"`; Prev/Next named; ellipsis `aria-hidden`; disabled prev/next: keep focusable with `aria-disabled` or remove.
- Status: announce "Page 3 of 10" on change for client-side pagination (4.1.3), and move focus to the results heading/container. Target >= 24px (2.5.8). Names with context ("Go to page 4").
- Pitfall: icon-only prev/next without names; current page by color only.

#### Breadcrumb
- APG: https://www.w3.org/WAI/ARIA/apg/patterns/breadcrumb/
- `<nav aria-label="Breadcrumb">` > `ol` > `li` > `a`; last item `aria-current="page"` (link or plain text); separators via CSS (`::before`) or `aria-hidden`; truncated "..." control must be a button with name and expanded state.
- Pitfall: separators read aloud; collapsed overflow menu not keyboard reachable.

#### Field (label / description / error)
- Spec: HTML + ARIA; reference implementations: GOV.UK error message https://design-system.service.gov.uk/components/error-message/ [K, URL not fetched], React Aria Field/TextField (wires label, description, errorMessage), Base UI Field/Fieldset (auto-association [V: https://base-ui.com/react/overview/accessibility]).
- Structure: `<label for>`; description id and error id both in `aria-describedby`; `aria-invalid` only when invalid; required via `required` + visible text "(required)" or "(optional)"; group of controls: fieldset+legend; error text begins with "Error:" hidden prefix optional; error summary at top with links to fields and focus moved to summary on failed submit (GOV.UK pattern [K]).
- Validate on blur/submit, not on every keystroke; don't clear errors silently; announce new errors via live region or focus move (4.1.3, 3.3.1).
- Pitfalls: id collisions from non-SSR-safe ids (use `useId`); label id not forwarded to custom triggers; error live region injected with content simultaneously; error text color contrast; invalid announced wrongly on iOS VO (React Aria #9119 [V]).
- AXE: label, aria-valid-attr-value (dangling describedby ids caught), but NOT quality of error message.

### 2.3 What each test layer can and cannot catch
| Layer | Catches | Misses |
|---|---|---|
| axe-core (jest-axe, @axe-core/playwright, Storybook addon-a11y) | Missing names, invalid ARIA attr/role/parent-child, duplicate ids, label association, color contrast for rendered state (not hover/focus unless driven), landmarks, heading order, form-field labels, `aria-hidden` on focusable | Whether names are meaningful, keyboard operability, focus order/return, trap, live-region announcements, virtual-focus correctness, target size (partial rule target-size exists in axe 4.x [K, U on exact coverage]), focus-ring contrast, reflow/zoom, text spacing (can be tested by injecting CSS), forced colors, SR phrasing |
| Playwright/Testing Library keyboard tests | Tab order, roving tabindex, Esc/Enter/Space/arrows, focus return, trap, typeahead, RTL arrows, no keyboard trap | Announcements, visual issues |
| Visual regression/Playwright projects | Focus ring present, forced-colors emulation (`forcedColors: 'active'`), `reducedMotion`, `colorScheme`, viewport 320px, 400% zoom approximations, text-spacing CSS injection | Real AT behaviour |
| Manual SR | Announcement quality, live regions, virtual focus, mobile gestures | Scalability |
- Deque vendor study: 57% of issue volume found by axe across 2,000+ audits (2021) [V: https://www.deque.com/blog/automated-testing-study-identifies-57-percent-of-digital-accessibility-issues/]. Use as "automation helps but is partial", not as a conformance claim.
- Optional automation: Guidepup / NVDA automation [U maturity], axe DevTools Pro, Accessibility Insights (manual assessment), Lighthouse (subset of axe).
- Axe false positives with React Aria are documented in its wiki per Adobe [V: https://react-aria.adobe.com/quality].

---------------------------------------------------------------------
## 3. Known gaps: Radix vs React Aria vs Base UI (snapshots 2026-10-11)

### 3.1 Radix Primitives (https://github.com/radix-ui/primitives)
| Issue | Status | Impact | Link |
|---|---|---|---|
| Select instantly closes on open with touch screen readers (VoiceOver iOS) | Open since Apr 2022, "Needs Investigation", Holding milestone; community workaround: preventDefault on `onPointerDownOutside`, larger options | Blocker for iOS VO users; Android TalkBack reportedly also helped by workaround | https://github.com/radix-ui/primitives/issues/1343 [V] |
| Select: VoiceOver cuts last two characters (Safari) | Reported ~Sep 2025; status not verified | SR reads truncated value | https://github.com/radix-ui/primitives/issues/3671 [V via search only] |
| Select: VO item-count bug in Chrome (#3962, Jun 2026) and first option unannounced w/o preselection (#4110, Aug 2026) | Listed in open issues [V via search listing, not individually fetched] | | https://github.com/radix-ui/primitives/issues |
| Tooltip opens when tab regains focus/dialog closes and returns focus to trigger | Open; `Needs Investigation`; PR #3964 linked; assigned Jun 2026 | Unwanted tooltip; stuck tooltip until click elsewhere | https://github.com/radix-ui/primitives/issues/1800 [V] |
| Tooltip docs: touch not covered, only keyboard | Docs gap | Tooltips unreachable on touch | https://www.radix-ui.com/primitives/docs/components/tooltip [V] |
| Dialog/Dropdown: `aria-hidden` on app root hides live regions (toast/alert silent while modal open) | Closed (completed) 2026-06-03; resolution detail not shown; label "Dependency: aria-hidden" | Direct Toast-announcement risk | https://github.com/radix-ui/primitives/issues/1820 [V] |
| Dialog description never read (VO macOS 12 / Edge) | Closed 2024-03-07 "Fixed Upstream"; cause unconfirmed, VO-version dependent | | https://github.com/radix-ui/primitives/issues/2038 [V] |
| Tabs: SR focus selects tab | Reported; status not verified | | https://github.com/radix-ui/primitives/issues/1047 [V via search] |
| Third-party audit by Publicis Sapient: 35 issues (VO Safari macOS/iOS, NVDA Firefox, TalkBack Chrome) | Issue closed and converted to discussion #4174 on 2026-10-06; findings only in a spreadsheet; WCAG compliance question unanswered | Radix makes no published WCAG conformance claim [V that no answer on thread] | https://github.com/radix-ui/primitives/issues/2317 , https://github.com/radix-ui/primitives/discussions/4174 [V] |
| Toast | Docs thorough on type/altText/time limits/F8 hotkey; no confirmed announcement bug found | | https://www.radix-ui.com/primitives/docs/components/toast [V] |
| Maintenance | WorkOS-owned, slowed; shadcn now offers Base UI | Strategic risk | blog-sourced, https://dev.to/mashuktamim/is-your-shadcn-ui-project-at-risk-a-deep-dive-into-radixs-future-45ei [U] |

### 3.2 React Aria / React Aria Components (https://github.com/adobe/react-spectrum)
- Strengths [V: https://react-aria.adobe.com/quality]: SR matrix = VoiceOver macOS (Safari, Chrome); JAWS (Firefox, Chrome); NVDA (Firefox, Chrome); VoiceOver iOS; TalkBack Android Chrome. Press events normalised across mouse/touch/keyboard and hover disabled on touch; 30+ localised string sets, RTL keyboard support, `I18nProvider`.
- Open SR-related issues (from a search of open VoiceOver issues, snapshot) [V: https://github.com/adobe/react-spectrum/issues?q=is%3Aissue+is%3Aopen+VoiceOver]:
  - #10326 Autocomplete: VO doesn't announce focused option; #10220 Combobox only announces item name; #5839 grouped combobox sections; #2578 item count in TalkBack/iOS VO.
  - #9809 RadioGroup isRequired announces invalid on each radio; #9119 invalid TextField on iOS VO; #2581 readonly radio.
  - #8915 tooltip text not announced for trigger inside Modal; #7934 Tags not interactive with VO; #7734 Table VO announces unfocused cell; #8591 DateField segments announced as stepper.
  - #5191 LiveAnnouncer first announce fails NVDA/JAWS; #10310 `useInteractionModality` treats JAWS/NVDA as mouse in Chromium; #4101 sliders tedious on iOS VO.
- Takeaway: highest engineering rigor but virtual focus + live-region announcement quirks exist. Their own docs concede axe false positives [V].

### 3.3 Base UI (https://github.com/mui/base-ui)
- Status [V: https://base-ui.com/react/overview/releases]: v1.0.0 stable 2025-12-11 (new `@base-ui/react` package, 35 components); v1.9.0 2026-10-09; Accordion APG alignment v1.6.0; iOS VoiceOver voice-control fix in 1.0.0-beta.7.
- Tracker: 43 `accessibility` issues, newest 25 sampled: 9 open [V: https://github.com/mui/base-ui/issues?q=is%3Aissue+label%3A%22accessibility%22]. Notable:
  - Open: #5794 iPhone VO closes whole menu after last submenu item (and #5793 backward nav) [V]; #4253 Toast Undo unreachable in time (2.1.1/2.2.1) - maintainers argue focus shouldn't move per APG Alert, F6 hotkey, `timeout: 0`; demo timeout raised 5s->10s in PR #4975 [V]; #4229 Toast a11y guide missing; #4184 NVDA/JAWS don't announce Meter/Progress programmatic label; #4182 command palette focus indicator relies on colour alone; #4226 NumberField min/max; #5583 preview card accessible preview.
  - Closed (evidence of responsiveness): #5560 grid ARIA ownership, #5528 non-modal aria-hide but tabbable, #4435 TalkBack combobox wrapper, #4181 limited-results announcement, #4169 dialog focus wrong trigger, #4171 context menu warning, #4432/#4433 Slider/ScrollArea vs 2.5.2.
- Docs stance [V: https://base-ui.com/react/overview/accessibility]: you own visible focus styling and custom-control names; Field/Fieldset auto-associate labels; suggests APCA "unless you have stricter compliance needs" - note WCAG 2.2 AA conformance requires the WCAG 2 contrast ratio, not APCA.
- Tooltip: disabled on touch by design; use Popover for essential info [V].

### 3.4 Cross-library cautions
- Hidden/native input proxying (Checkbox, Radio, Select) is a recurring source of required/invalid mis-announcement.
- Touch screen reader (VO iOS / TalkBack) is the weakest-covered area for all three; build a manual mobile SR test pass into release criteria.
- Whatever you choose, wrap primitives and pin versions; add regression tests for each bug above you work around.

---------------------------------------------------------------------
## 4. Beyond WCAG

### 4.1 Forced colors / Windows High Contrast
- Use `@media (forced-colors: active)`; browser maps to system colors (Canvas, CanvasText, LinkText, ButtonText, Highlight, HighlightText, GrayText, ButtonFace, Field). Spec: https://www.w3.org/TR/css-color-adjust-1/ ; MDN https://developer.mozilla.org/en-US/docs/Web/CSS/@media/forced-colors [K].
- Rules: backgrounds, box-shadow and background-images are dropped/overridden, so (a) focus ring via `outline` (kept) not box-shadow; (b) give borders (`1px solid transparent` becomes visible) to Button/Input/Card; (c) checkbox/radio/switch glyphs must be real elements/borders or `forced-color-adjust` handled, SVG icons use `currentColor`/`fill: CanvasText`; (d) selected state (Tabs, Menu item) needs non-bg cue (border/underline/outline) because background colours vanish; (e) disabled = `GrayText`; (f) use `forced-color-adjust: none` sparingly, only for swatches.
- Test: Playwright `forcedColors: 'active'` emulation + Chromium rendering emulation; real check on Windows contrast themes [K].

### 4.2 prefers-reduced-motion
- `@media (prefers-reduced-motion: reduce)`: remove parallax/large transforms, shimmer, auto-scrolling, spinners become slower/static; keep opacity fades or instant. Provide tokens (`--motion-duration-*` -> 0.01ms) and a `useReducedMotion` hook; WCAG 2.3.3 (AAA) Animation from Interactions; 2.2.2 for >5s motion [K]. Don't remove essential feedback (progress still shown).

### 4.3 prefers-contrast / theming
- `prefers-contrast: more|less|custom` (Baseline in modern browsers, check caniuse before relying [U]). Offer a high-contrast token layer: stronger borders, text 7:1, ring 4.5:1. Dark mode and `color-scheme` meta so UA controls match. Also `prefers-reduced-transparency` [U support].

### 4.4 Zoom / reflow / text spacing
- Test at 320 CSS px width, 400% zoom on 1280, 200% text-only zoom; no horizontal scroll except data tables (1.4.10). Use `min-height` not `height`, `rem`, wrapping flex, `overflow-wrap`, `scroll-padding` for sticky UI (2.4.11), avoid `vh` traps. Inject the 1.4.12 text-spacing bookmarklet CSS in Playwright and screenshot-diff.
- Tooltips/Popovers must reposition within viewport (collision handling) and stay hoverable.

### 4.5 RTL and i18n
- CSS logical properties (`margin-inline-start`, `inset-inline`), `dir` on root, mirror directional icons (chevrons, back/forward) but not media controls/numbers; arrow-key semantics flip for Tabs, Radio, Menu submenus, Slider (React Aria handles RTL keyboard [V]). `lang` attr and localisable strings for built-in labels (Close, Loading, "Page x of y"): expose all as props/messages. Test with Arabic/Hebrew + text expansion (German ~30%).

### 4.6 Screen reader and browser matrix
| Platform | AT + browser | Priority | Notes |
|---|---|---|---|
| Windows | NVDA + Firefox, NVDA + Chrome | Must | Free; WebAIM survey puts NVDA/JAWS as most used [K, U on 2026 survey figures] |
| Windows | JAWS + Chrome (and Edge) | Must | Licensed (40-min demo mode); live-region and forms-mode quirks |
| macOS | VoiceOver + Safari | Must | Chrome/Firefox secondary; Safari needs Tab/keyboard nav settings |
| iOS | VoiceOver + Safari | Must | Touch exploration; popup/overlay issues (Radix #1343, Base UI #5794) |
| Android | TalkBack + Chrome | Must | |
| Other | Narrator + Edge, Orca, Voice Control / Dragon, Switch Control, Windows magnifier | Spot check | Voice control: label-in-name (2.5.3) critical; Base UI fixed iOS voice-control issue [V] |
- Adobe tests: VO macOS Safari+Chrome, JAWS Firefox+Chrome, NVDA Firefox+Chrome, VO iOS, TalkBack Chrome [V]. Radix doc claim: VO macOS/iOS + TalkBack [V via mirror; U].
- Record in repo: `a11y/screen-reader-matrix.md` with component x AT x browser x date x result x tester x version.

### 4.7 Cognitive accessibility
- W3C COGA: https://www.w3.org/TR/coga-usable/ [K]. Consistent patterns/identification (3.2.3/3.2.4), plain-language labels, error prevention and recovery, no time pressure (2.2.1), avoid CAPTCHA-only, visible instructions, predictable behaviour, avoid distracting motion, chunked forms, status feedback, undo. 3.3.7 and 3.3.8 are the cognitive additions in 2.2.

### 4.8 Icon-only button names
- Name = action + object ("Delete comment"), not glyph ("Trash"); match visible tooltip text; localise; keep icon `aria-hidden`; use TS-required `label`; consistent names per icon across DS; toggles use `aria-pressed` with stable name ("Favourite"); avoid `title` as sole name (unreliable on touch/SR). Lint: eslint-plugin-jsx-a11y, and a story-level axe check plus a codemod/test to assert every `IconButton` has non-empty `label`.

### 4.9 Form error patterns
- Inline: error text adjacent below field, linked by `aria-describedby`, `aria-invalid=true`, icon+text+color, specific suggestion (3.3.3). Summary: on submit failure, render error summary at page top (`role=alert` or move focus to it, `tabindex=-1`), list links to each field (GOV.UK pattern [K, URL https://design-system.service.gov.uk/components/error-summary/ not fetched]). Validate on blur/submit; keep entered data; don't use `aria-live` on every field; group errors (radio group error on fieldset). `aria-errormessage` has inconsistent support -> use describedby (see React Aria #7425 discussion [V]).

---------------------------------------------------------------------
## 5. How exemplary design systems document accessibility

Fetch attempts: GOV.UK /accessibility/ page fetched; it is high-level (says using the DS does not make a service accessible; links to accessibility statement + strategy) [V: https://design-system.service.gov.uk/accessibility/]. Primer overview fetched but gave no structure [V]. Carbon and Spectrum URLs 404'd via my guesses; the descriptions below from knowledge are [K/U] and should be re-verified before quoting.

| System | Pattern observed | Link |
|---|---|---|
| GOV.UK Design System | Per-component "Research on this component" and "Accessibility" notes; publishes an accessibility statement for the frontend and the site; lists known issues and assistive-tech testing results; "Using the DS doesn't make your service accessible" disclaimer [V for disclaimer/statement] | https://design-system.service.gov.uk/accessibility/ ; https://design-system.service.gov.uk/accessibility/accessibility-statement/ [U path] |
| Primer (GitHub) | Dedicated Accessibility guides + per-component "Accessibility" section ("accessibility audit status" tags per component) [K, U] | https://primer.style/product/getting-started/accessibility/ |
| Spectrum / React Aria | "Accessibility" page + per-hook/component Accessibility/Internationalization sections; documented SR test matrix and testing guides [V for matrix] | https://react-aria.adobe.com/quality ; https://spectrum.adobe.com/ |
| Atlassian | Per-component "Accessibility" tab with guidelines, keyboard table, ARIA, and code notes; accessibility foundations section [K, U] | https://atlassian.design/components |
| Carbon (IBM) | Each component has an "Accessibility" tab with Overview/Development/Design sections and a documented accessibility checklist/status per component [K, U] | https://carbondesignsystem.com/components/overview/ |

### 5.1 Recommended "Accessibility" section template for each component page
1. Summary and conformance status (WCAG 2.2 AA: tested / partial / known issues), last-tested date, version.
2. Pattern and semantics: APG link; native element used; roles/states/properties table.
3. Keyboard interaction table (Key | Behaviour | RTL difference).
4. Focus management: initial focus, trap, return, tab order, focus-visible styling tokens.
5. Naming and labelling requirements for consumers (required props, e.g. `aria-label` on IconButton).
6. Screen reader behaviour: expected announcement script per SR (NVDA/JAWS/VO/TalkBack) with tested versions.
7. Visual requirements: contrast pairs (ratio values from tokens), target size, forced-colors, reduced-motion, 400% zoom, text-spacing results.
8. Content guidance (label copy, error copy, don'ts).
9. Test evidence: links to axe test file, keyboard test file, manual checklist result.
10. Known issues and workarounds with upstream issue links (e.g. Radix #1343) and WCAG SC affected.
11. Consumer responsibilities (what the DS cannot guarantee: page-level headings, landmarks, content, custom composition).
12. Changelog of a11y fixes.

### 5.2 Per-component CI checklist (definition of done)
- [ ] axe passes in all states stories (default, hover/focus forced, disabled, invalid, open, loading, dark, forced-colors screenshot)
- [ ] Keyboard script passes (Tab, Shift+Tab, Esc, Enter, Space, arrows, Home/End, typeahead; RTL variant)
- [ ] Focus visible 3:1, not clipped, not obscured by sticky UI
- [ ] Target >= 24px (document spacing exception)
- [ ] Accessible name required by types; no name drift vs visible label
- [ ] 320px reflow, 400% zoom, text-spacing CSS no clipping
- [ ] Reduced motion + forced colors pass
- [ ] Manual SR matrix run (NVDA+FF/Chrome, JAWS+Chrome, VO macOS Safari, VO iOS, TalkBack) for composite widgets
- [ ] Docs a11y section complete; known issues linked

### 5.3 Publishing an honest accessibility conformance statement for an OSS DS
- VPAT is the blank template; completed it becomes an ACR. Four editions: 2.5 508, 2.5 EU (EN 301 549, WCAG 2.1 per ITI page), 2.5 WCAG (WCAG 2.2), 2.5 INT. Terms: Supports / Partially Supports / Does Not Support / Not Applicable (ITI also uses "Not Evaluated" for AAA in WCAG edition [K, U]). Free, no ITI certification or approval process. The VPAT name/form is a registered service mark; do not alter template [V: https://www.itic.org/policy/accessibility/vpat]. Note the ITI page says the EU edition incorporates WCAG 2.1: EN 301 549 v4.1.1 may need a newer edition; check ITI for updates [U].
- Also consider the W3C WCAG-EM and Accessibility Statement generator https://www.w3.org/WAI/planning/statements/ [K] for a web-based statement; GOV.UK publishes a statement in that style [V].
- What to include: scope (components + docs site + versions tested), standard and level (WCAG 2.2 AA; map to EN 301 549 Clause 9/11 if claiming), date and method (automated + manual + SR list with versions), per-SC table with remarks (be specific: "Partially Supports - Select popup unusable with VO iOS when built on Radix, see issue link"), known issues with upstream links, what's out of scope (consumer content, composition, page-level), evaluator identity (self-assessed vs third-party), feedback channel/contact, review cadence (per minor release), and a clear "components are accessible building blocks; conformance of products depends on usage" disclaimer.
- Honesty rules: self-evaluation labelled as such; never claim "WCAG compliant"/"certified"; mark SCs that are page-level as "Applicable to implementers"; do not mark Supports for untestable criteria (e.g., 1.2.x media) -> Not Applicable; keep the ACR in repo (versioned) and publish a human-readable HTML page (not just PDF), accessible itself.
- Consider commissioning a third-party audit for credibility; note third-party audit of Radix exists but unusable (spreadsheet) [V].

---------------------------------------------------------------------
## 6. Recommended decisions for the design system
1. Primitive: React Aria Components (best touch-SR story, i18n/RTL) or Base UI (active, unstyled, Field/Fieldset helpers); avoid committing to Radix for Select/Toast without mitigation. Keep a thin wrapper layer to swap.
2. House standards stricter than AA: focus ring 2px, 3:1 (meets 2.4.13), target 40px default/24px floor, text 4.5:1 incl. muted text, WCAG-2 contrast ratios (not APCA) for conformance claim.
3. Test stack: Storybook + addon-a11y; Vitest/jest-axe unit; Playwright with axe-core + keyboard scripts + `forcedColors`, `reducedMotion`, viewport 320, text-spacing injection; manual SR matrix doc with dates.
4. Ship ACR (VPAT 2.5 WCAG edition) per minor release, with known issues linking upstream issues.
5. Prefer native `<select>` default and `<dialog>`/popover API where feasible; Toast: persistent live-region viewport, no auto-dismiss for actionable/error toasts, F6/F8 hotkey documented.

---------------------------------------------------------------------
## 7. Source list (accessed 2026-10-11)
Fetched [V]:
- WCAG 2.2 what's new: https://www.w3.org/WAI/standards-guidelines/wcag/new-in-22/
- React Aria quality/accessibility: https://react-aria.adobe.com/quality
- Deque EN 301 549 v4.1.1: https://www.deque.com/blog/en-301-549-v4-1-1-is-final-what-changed-what-it-means-and-what-you-should-do/
- Deque automated coverage study: https://www.deque.com/blog/automated-testing-study-identifies-57-percent-of-digital-accessibility-issues/
- ITI VPAT: https://www.itic.org/policy/accessibility/vpat
- Radix issues: #1800 https://github.com/radix-ui/primitives/issues/1800 ; #1343 .../issues/1343 ; #1820 .../issues/1820 ; #2038 .../issues/2038 ; #2317 .../issues/2317 ; discussion #4174 https://github.com/radix-ui/primitives/discussions/4174
- Radix docs: https://www.radix-ui.com/primitives/docs/components/tooltip ; .../toast
- Base UI: https://base-ui.com/react/overview/accessibility ; .../react/components/tooltip ; .../react/overview/releases ; issues https://github.com/mui/base-ui/issues/4253 , /5794 , list https://github.com/mui/base-ui/issues?q=is%3Aissue+label%3A%22accessibility%22
- React Spectrum open VO issues listing: https://github.com/adobe/react-spectrum/issues?q=is%3Aissue+is%3Aopen+VoiceOver
- GOV.UK: https://design-system.service.gov.uk/accessibility/
- shadcn audit (third-party blog): https://thefrontkit.com/blogs/shadcn-ui-accessibility-audit-2026
Search-snippet only [U]: Radix #3671, #3962, #4110, #1047; MUI #44740/#16409/#40024; Carbon #21971; hirobius/hds PR #421; DWT EN 301 549 update; dev.to Radix future; PatternFly/Polaris live-region notes.
Canonical spec URLs used from knowledge [K]: https://www.w3.org/TR/WCAG22/ ; https://www.w3.org/WAI/ARIA/apg/patterns/ (each pattern linked inline) ; https://www.w3.org/TR/wai-aria-1.2/ ; https://www.w3.org/TR/css-color-adjust-1/ ; https://www.w3.org/TR/coga-usable/

## 8. Open items to verify next
- Re-fetch Carbon, Atlassian, Primer, Spectrum component a11y pages for exact structure (my URL guesses 404'd).
- Confirm OJEU citation date for EN 301 549 v4.1.1 (2026-11-30 vs 2026-12-16 conflict) and EAA OSS scope from the Directive text.
- Check Radix #3671/#3962/#4110 statuses; check React Aria Toast stability; check axe-core `target-size` rule coverage; check prefers-contrast / reduced-transparency baseline support.
- Run own SR tests of chosen primitive's Select, Menu, Combobox, Toast on iOS VO + TalkBack before v1.
