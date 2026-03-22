# Bold Theme — Design Spec
**Date:** 2026-03-20
**Status:** Approved by user
**Scope:** Additional admin theme for CalacaCMS, switchable in Settings

---

## 1. Overview

A second admin theme ("Bold") based on the Bold Typography design system, using the same colors as the CalacaCMS landing page for visual brand consistency. It lives alongside the existing default theme; users switch between them in the Settings page. Both dark and light variants are supported.

**Design philosophy:** Type as hero. Extreme scale contrast. Restrained palette. No border-radius anywhere. Sharp edges match sharp typography.

---

## 2. Decisions Made

| Question | Decision |
|---|---|
| Replacement or additional? | Additional — default theme stays intact |
| Dark only or both variants? | Both dark + light variants |
| Depth of change | Full: tokens + components + layout |
| Theme switcher location | Settings page only |
| Implementation approach | `data-theme-preset="bold"` attribute on `<html>`, overrides CSS variables + component classes |

---

## 3. Design Tokens

### Color format note

The existing `index.css` uses `oklch()` throughout. Bold theme tokens are defined in **hex** (matching the landing page source values exactly). This is intentional — both formats are valid CSS custom properties and cascade together without issue. The hex values must **not** be converted to oklch to preserve exact landing-page color matching.

### CSS selector strategy

Three selector blocks are required to cover all theme combinations:

```css
/* Dark mode — explicit */
[data-theme-preset="bold"][data-theme="dark"] { ... }

/* Light mode — explicit */
[data-theme-preset="bold"][data-theme="light"] { ... }

/* System/auto mode — follows OS preference */
@media (prefers-color-scheme: dark) {
  [data-theme-preset="bold"]:not([data-theme="light"]) { ... }
}
@media (prefers-color-scheme: light) {
  [data-theme-preset="bold"]:not([data-theme="dark"]) { ... }
}
```

This ensures Bold works correctly when the user is on `system` auto-detect mode (where `data-theme` may not be set).

### Dark variant

| Token | Value | Purpose |
|---|---|---|
| `--b1` | `#0b0c15` | Content background (from landing page) |
| `--b2` | `#0e0f1a` | Sidebar background |
| `--b3` | `#131420` | Hover states, inputs |
| `--card` | `#131420` | Card background (from landing page) |
| `--bc` | `#FAFAFA` | Primary text |
| `--bc-muted` | `#737373` | Secondary text |
| `--bc-secondary` | `#525252` | Tertiary text |
| `--p` | `#a3ff12` | Accent — lime green (from landing page) |
| `--pc` | `#0b0c15` | Text on accent |
| `--border` | `#1e2035` | Dividers (navy-tinted) |
| `--divider` | `#1e2035` | Same as border for Bold |
| `--radius` | `0px` | No border-radius |
| `--radius-sm` | `0px` | No border-radius |
| `--radius-md` | `0px` | No border-radius |
| `--radius-lg` | `0px` | No border-radius |
| `--radius-xl` | `0px` | No border-radius |
| `--radius-full` | `0px` | Intentionally zeroed — no pill shapes in Bold |
| `--font-mono` | `'JetBrains Mono', ui-monospace, monospace` | Override mono stack |
| `--sidebar-bg` | `#0e0f1a` | Sidebar |
| `--sidebar-text` | `#525252` | Inactive nav items |
| `--sidebar-text-active` | `#a3ff12` | Active nav item |
| `--sidebar-hover` | `rgba(163,255,18,0.05)` | Nav item hover background |
| `--font-display` | `'Inter Tight', 'Inter', system-ui, sans-serif` | Heading/button font stack |
| `--border-hover` | `#2e3050` | Card hover border (navy lighter) |

### Light variant

| Token | Value | Purpose |
|---|---|---|
| `--b1` | `#F7F7F2` | Content background (warm white) |
| `--b2` | `#EFEFEA` | Sidebar background |
| `--b3` | `#E5E5E0` | Hover states, inputs |
| `--card` | `#E5E5E0` | Card background |
| `--bc` | `#0b0c15` | Primary text |
| `--bc-muted` | `#525252` | Secondary text |
| `--bc-secondary` | `#737373` | Tertiary text |
| `--p` | `#3a5a00` | Accent — dark olive green (#a3ff12 is unreadable on light) |
| `--pc` | `#F7F7F2` | Text on accent |
| `--border` | `#D4D4CC` | Dividers |
| `--divider` | `#D4D4CC` | Same as border for Bold |
| `--radius` | `0px` | No border-radius |
| `--radius-sm` | `0px` | |
| `--radius-md` | `0px` | |
| `--radius-lg` | `0px` | |
| `--radius-xl` | `0px` | |
| `--radius-full` | `0px` | Intentionally zeroed |
| `--font-mono` | `'JetBrains Mono', ui-monospace, monospace` | Override mono stack |
| `--sidebar-bg` | `#EFEFEA` | Sidebar |
| `--sidebar-text` | `#737373` | Inactive nav items |
| `--sidebar-text-active` | `#2e4700` | Active nav item |
| `--sidebar-hover` | `rgba(58,90,0,0.08)` | Nav item hover background |
| `--font-display` | `'Inter Tight', 'Inter', system-ui, sans-serif` | Heading/button font stack |
| `--border-hover` | `#c0c0b8` | Card hover border (warm grey lighter) |

### Typography

| Role | Font | Weight | Size | Tracking | Line-height |
|---|---|---|---|---|---|
| Page titles | Inter Tight | 800 | `text-5xl` (desktop) | `-0.04em` | `1` |
| Section headings | Inter Tight | 700 | `text-3xl` | `-0.04em` | `1.1` |
| Card stat numbers | Inter Tight | 800 | `text-5xl` | `-0.04em` | `1` |
| Body | Inter | 400 | `text-sm` / `text-base` | `-0.01em` | `1.6` |
| Labels / mono | JetBrains Mono | 500 | `text-[10px]` | `0.12em–0.18em` | — |
| Button text | Inter Tight | 600 | `text-sm` | `0.1em` | — |

**Font loading:**
- `@fontsource/inter` is already installed (root `package.json` dep)
- Add `@fontsource/inter-tight` to the **root** `package.json` — this is a separate package from `@fontsource/inter`
- Add `@fontsource/jetbrains-mono` to the **root** `package.json`
- Import Inter Tight weights 700 and 800 only: `@fontsource/inter-tight/700.css` and `@fontsource/inter-tight/800.css`
- Import JetBrains Mono weights 400 and 500 only: `@fontsource/jetbrains-mono/400.css` and `@fontsource/jetbrains-mono/500.css`
- Add all four imports to `themes/admin/assets/css/admin.css` alongside the existing inter import
- The `--font-mono` token override ensures JetBrains Mono is used for all monospace text in Bold mode
- Inter Tight is applied via a `--font-display` token (see below) and explicit heading/button selectors in `base.css`

**`--font-display` token (add to both dark and light Bold variants):**
```css
--font-display: 'Inter Tight', 'Inter', system-ui, sans-serif;
```
In `base.css`, inside `[data-theme-preset="bold"]`, all heading selectors (`h1`, `h2`, `h3`, `.page-title`, `.card-title`, `.btn-primary`, `.btn`, `.btn-secondary`, `.btn-danger`) receive `font-family: var(--font-display)`. Body text and inputs continue to use `--font-sans` (Inter).

---

## 4. Component Styles

All component overrides live inside `[data-theme-preset="bold"]` in `themes/admin/assets/css/base/base.css`. No Twig markup changes needed for components.

### Buttons

**Primary (`.btn-primary`):**
- No background, no border
- Text color: `var(--p)`, uppercase, `tracking-wider` (`0.1em`)
- Font: Inter Tight 600
- Padding: `py-2 px-0` (no horizontal padding — underline fills the width)
- Animated underline via `::after`: `height: 2px`, `background: var(--p)`, `transform: scaleX(1)` base, `scaleX(1.08)` on hover, `transform-origin: left`, `transition: transform 150ms`
- Active: `transform: translateY(1px)`
- `border-radius: 0`

**Secondary (`.btn` / `.btn-secondary`):**
- `border: 1px solid var(--bc)`, no background fill
- Text: `var(--bc)`, uppercase, `tracking-wider`
- Hover: `background: var(--bc)`, `color: var(--b1)` (full color inversion)
- Transition: `all 150ms`
- `border-radius: 0`

**Ghost (`.btn-ghost`):**
- No border, no background
- Text: `var(--bc-muted)` → `var(--bc)` on hover, transition `color 150ms`
- Underline `::after`: `height: 1px`, `scaleX(0)` → `scaleX(1)` on hover
- `border-radius: 0`

**Danger (`.btn-danger`):**
- `border: 1px solid #ff6b6b`, text `#ff6b6b`, no background
- Hover: `background: #ff6b6b`, `color: var(--b1)`
- `border-radius: 0`

### Inputs / Textarea / Select

- `background: var(--b3)`
- `border: 1px solid var(--border)`, `border-radius: 0`
- Focus: `border-color: var(--p)`, `outline: none`, no ring/glow
- Labels (`.label`): JetBrains Mono, `10px`, `letter-spacing: 0.12em`, uppercase, `color: var(--bc-muted)`
- Transition: `border-color 150ms`

### Cards

Two additional tokens handle hover border colors (add to dark and light variants respectively):
- Dark: `--border-hover: #2e3050`
- Light: `--border-hover: #c0c0b8`

- `background: var(--card)`, `border: 1px solid var(--border)`, `border-radius: 0`
- Hover: `border-color: var(--border-hover)`
- No shadow
- Accent bar: `width: 32px`, `height: 2px`, `background: var(--p)` — sits above card title as a visual anchor
- Featured card: `border: 2px solid var(--p)` with a mono badge (`background: var(--p)`, `color: var(--pc)`, JetBrains Mono, `9px`, uppercase)

### Stat cards (dashboard)

- Stat number: Inter Tight 800, `text-5xl`, `letter-spacing: -0.04em`, `line-height: 1`
- Stat label: JetBrains Mono, `10px`, `letter-spacing: 0.12em`, uppercase, `color: var(--bc-secondary)`
- Sub-line: separated by `border-top: 1px solid var(--border)`, `padding-top: 12px`, `margin-top: 12px`

### Tables

- Header cells: JetBrains Mono, `10px`, `letter-spacing: 0.12em`, uppercase, `color: var(--bc-secondary)`, `font-weight: 500`
- Row hover: `background: var(--b3)`
- Cell borders: `border-bottom: 1px solid var(--border)`
- No outer border, no background on table element

### Badges / Tags

Font: JetBrains Mono, `10px`, `letter-spacing: 0.1em`, uppercase. `border-radius: 0`.

**Dark variant:**

| Variant | Background | Text | Border |
|---|---|---|---|
| Draft | `rgba(163,255,18,0.08)` | `var(--p)` (#a3ff12) | `rgba(163,255,18,0.2)` |
| Published | `rgba(255,255,255,0.05)` | `var(--bc-muted)` | `var(--border)` |
| Error | `rgba(255,107,107,0.08)` | `#ff6b6b` | `rgba(255,107,107,0.2)` |

**Light variant** (separate overrides required — dark rgba values are unreadable on light backgrounds):

| Variant | Background | Text | Border |
|---|---|---|---|
| Draft | `rgba(58,90,0,0.08)` | `var(--p)` (#3a5a00) | `rgba(58,90,0,0.25)` |
| Published | `rgba(0,0,0,0.04)` | `var(--bc-muted)` | `var(--border)` |
| Error | `rgba(180,30,30,0.08)` | `#b41e1e` | `rgba(180,30,30,0.2)` |

### Flash messages

- `border-left: 2px solid var(--p)` (success) or `2px solid #ff6b6b` (error)
- Background: `rgba(163,255,18,0.04)` (success) or `rgba(255,107,107,0.04)` (error)
- `border-radius: 0`

---

## 5. Layout Changes

### Sidebar

- Background: `var(--b2)` (navy dark, not grey)
- Logo icon: `background: var(--p)`, text `var(--pc)` — sharp square (`border-radius: 0`)
- Active indicator: left `2px` border in `var(--p)` (replaces current rounded pill indicator)
- Section labels: JetBrains Mono, `9px`, `letter-spacing: 0.18em`, very muted (`var(--bc-secondary)`)
- Nav item hover: `background: var(--sidebar-hover)`

### Header

- Background: `var(--b1)`, `border-bottom: 1px solid var(--border)`
- Breadcrumb text: JetBrains Mono, `11px`, `letter-spacing: 0.08em`, uppercase
- Search bar: `background: var(--b3)`, `border: 1px solid var(--border)`, `border-radius: 0`

### Page titles

- H1 page title: Inter Tight 800, `text-4xl` (mobile) → `text-5xl` (desktop), `letter-spacing: -0.04em`
- Subtitle: `text-sm`, `color: var(--bc-muted)`

---

## 6. State Management

### Theme preset storage
- Key: `calaca-admin-theme-preset` (new localStorage key)
- Values: `'default'` | `'bold'`
- Default: `'default'`

### Existing localStorage key inconsistency
The codebase currently has two competing keys for light/dark mode: `'themeMode'` (used in `layout.twig` inline script and Alpine x-data) and `'calaca-admin-theme'` (used in `theme.ts` `STORAGE_KEYS.THEME`). **Do not resolve this in the Bold theme PR** — address it separately to avoid scope creep. The new preset key `calaca-admin-theme-preset` is independent of both.

### Application
- On page load: inline `<script>` in `<head>` reads `calaca-admin-theme-preset` from localStorage and sets `data-theme-preset` on `<html>` — same pattern as the existing theme anti-flash script
- Alpine.js: add `themePreset` string to the root `x-data` object in `layout.twig`; watch it to persist to localStorage and update the attribute on `<html>`
- Settings page: radio toggle with two options (see Section 7)

### Anti-flash script (addition to existing inline script in `<head>`)

Add the following **after** the existing `document.documentElement.setAttribute('data-theme', ...)` call, still inside the same `try/catch` block:

```js
const preset = localStorage.getItem('calaca-admin-theme-preset') || 'default';
document.documentElement.setAttribute('data-theme-preset', preset);
```

---

## 7. Settings Page — Theme Preset Switcher

The switcher is a **radio-style card toggle** with two options. It is fully client-side — no Twig variable or PHP controller change is needed. State is read from and written to localStorage only.

- Two clickable cards side by side: "Default" and "Bold"
- Each card shows a small color swatch preview (sidebar color + accent dot)
- Active card: `border: 2px solid var(--p)`
- Inactive card: `border: 1px solid var(--border)`
- Alpine component wraps both cards: `x-data="{ themePreset: localStorage.getItem('calaca-admin-theme-preset') || 'default' }"`
- Card click handler: `@click="themePreset = 'bold'; $el.closest('[x-data]').dispatchEvent(new Event('preset-change'))"`
- `x-watch` (or `$watch`) on `themePreset`: writes to localStorage and sets `data-theme-preset` on `document.documentElement`
- No page reload required — CSS responds to attribute change immediately

---

## 8. TypeScript Changes

### `constants/storage.ts`
Add: `THEME_PRESET: 'calaca-admin-theme-preset'`

### `types/index.ts`
Add a new type and a new interface — do **not** extend the existing `ThemeManager` (which is scoped to light/dark only):
```ts
type ThemePreset = 'default' | 'bold'

interface ThemePresetManager {
  getPreset(): ThemePreset
  setPreset(preset: ThemePreset): void
  applyPreset(preset: ThemePreset): void
}
```

### `utils/theme.ts`
Add and export a `presetManager` object implementing `ThemePresetManager`, using `STORAGE_KEYS.THEME_PRESET`. Keep it separate from the existing `themeManager` export.

---

## 9. Files to Create / Modify

| File | Action | What changes |
|---|---|---|
| `/package.json` (root) | Modify | Add `@fontsource/inter-tight` and `@fontsource/jetbrains-mono` |
| `themes/admin/assets/css/admin.css` | Modify | Import JetBrains Mono 400 + 500 |
| `themes/admin/assets/css/themes/index.css` | Modify | Add Bold token blocks (dark + light + system media queries) |
| `themes/admin/assets/css/base/base.css` | Modify | Add Bold component overrides inside `[data-theme-preset="bold"]` |
| `themes/admin/templates/layouts/layout.twig` | Modify | Add `themePreset` to Alpine x-data, extend anti-flash script, bind `data-theme-preset` |
| `themes/admin/templates/pages/settings/index.twig` | Modify | Add radio card toggle for theme preset |
| `themes/admin/assets/ts/constants/storage.ts` | Modify | Add `THEME_PRESET` key |
| `themes/admin/assets/ts/types/index.ts` | Modify | Add `ThemePreset` type and new `ThemePresetManager` interface |
| `themes/admin/assets/ts/utils/theme.ts` | Modify | Add `getPreset()`, `setPreset()`, `applyPreset()` |

---

## 10. Out of Scope

- Jodit rich text editor internal styling (editor has its own shadow DOM)
- Mobile-specific overrides beyond what token changes provide
- Resolving the existing `'themeMode'` vs `'calaca-admin-theme'` localStorage key inconsistency
- Any changes to the default theme
