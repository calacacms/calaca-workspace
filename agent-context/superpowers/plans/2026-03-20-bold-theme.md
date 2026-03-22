# Bold Theme Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a switchable "Bold" admin theme (navy + lime-green, no border-radius, Inter Tight headlines, JetBrains Mono labels) alongside the existing default theme, toggled from the Settings page.

**Architecture:** A `data-theme-preset="bold"` attribute on `<html>` activates CSS variable overrides and component style overrides scoped to that selector. An inline anti-flash script sets the attribute from localStorage before first paint. A self-contained Alpine component on the Settings page handles switching and live-updating the attribute.

**Tech Stack:** Tailwind CSS v4, Alpine.js, Twig templates, TypeScript, Bun, `@fontsource/*`

---

## File Map

| File | Change |
|---|---|
| `/package.json` | Add `@fontsource/inter-tight`, `@fontsource/jetbrains-mono` |
| `themes/admin/assets/css/admin.css` | Import 4 new font CSS files |
| `themes/admin/assets/css/themes/index.css` | Append Bold dark + light + system token blocks |
| `themes/admin/assets/css/base/base.css` | Append Bold component overrides in `@layer components` |
| `themes/admin/templates/layouts/layout.twig` | Extend anti-flash `<script>` for preset |
| `themes/admin/templates/pages/settings/index.twig` | Add Appearance section with preset switcher |
| `themes/admin/assets/ts/constants/storage.ts` | Add `THEME_PRESET` key |
| `themes/admin/assets/ts/types/index.ts` | Add `ThemePreset` type + `ThemePresetManager` interface |
| `themes/admin/assets/ts/utils/theme.ts` | Add `presetManager` export |

---

## Task 1: TypeScript foundation — storage key, types, presetManager

**Files:**
- Modify: `themes/admin/assets/ts/constants/storage.ts`
- Modify: `themes/admin/assets/ts/types/index.ts`
- Modify: `themes/admin/assets/ts/utils/theme.ts`

- [ ] **Step 1: Add THEME_PRESET storage key**

In `themes/admin/assets/ts/constants/storage.ts`, add to the `STORAGE_KEYS` object (after the last entry):

```typescript
/** Theme preset: 'default' | 'bold' */
THEME_PRESET: 'calaca-admin-theme-preset',
```

Also add a `THEME_PRESET_CONFIG` constant at the bottom of the file:

```typescript
/** Theme preset configuration */
export const THEME_PRESET_CONFIG = {
  VALID_PRESETS: ['default', 'bold'] as const,
  DEFAULT: 'default' as const,
} as const;
```

- [ ] **Step 2: Add ThemePreset type and ThemePresetManager interface**

In `themes/admin/assets/ts/types/index.ts`, append after the existing `ThemeManager` interface:

```typescript
// Theme preset types
export type ThemePreset = 'default' | 'bold';

export interface ThemePresetManager {
  getPreset(): ThemePreset;
  setPreset(preset: ThemePreset): void;
  applyPreset(preset: ThemePreset): void;
}
```

- [ ] **Step 3: Implement presetManager in theme.ts**

In `themes/admin/assets/ts/utils/theme.ts`, update the import lines at the top to include the new types and constants:

```typescript
import type { Theme, ThemeData, ThemeManager, ThemePreset, ThemePresetManager } from '../types';
import { STORAGE_KEYS, THEME_CONFIG, THEME_PRESET_CONFIG } from '../constants/storage';
```

Then append the following **before** the final `if (typeof document !== 'undefined')` block at the bottom of the file:

```typescript
/**
 * Get the currently stored theme preset
 */
function getPreset(): ThemePreset {
  if (typeof localStorage === 'undefined') {
    return THEME_PRESET_CONFIG.DEFAULT;
  }
  const stored = localStorage.getItem(STORAGE_KEYS.THEME_PRESET);
  if (stored === 'default' || stored === 'bold') {
    return stored;
  }
  return THEME_PRESET_CONFIG.DEFAULT;
}

/**
 * Apply the theme preset to the document root
 */
function applyPreset(preset: ThemePreset): void {
  document.documentElement.setAttribute('data-theme-preset', preset);
}

/**
 * Set and persist the theme preset
 */
function setPreset(preset: ThemePreset): void {
  if (typeof localStorage !== 'undefined') {
    localStorage.setItem(STORAGE_KEYS.THEME_PRESET, preset);
  }
  applyPreset(preset);
}

// Export the preset manager object
export const presetManager: ThemePresetManager = {
  getPreset,
  setPreset,
  applyPreset,
};

// Export individual preset functions (setPreset is also on presetManager)
export { getPreset, applyPreset, setPreset };
```

**Important:** The existing export line at the bottom of `theme.ts` is:
```typescript
export { getTheme, applyTheme, setTheme, isDarkMode, themeData, init };
```
Keep that line as-is. The new `export { getPreset, applyPreset, setPreset }` is a separate additional export line.

- [ ] **Step 4: Verify TypeScript compiles**

```bash
cd /Users/macgyver/Dropbox/Code/side-projects/cms/calacacms
bun run build:admin 2>&1 | grep -E "error|warning|✓"
```

Expected: build succeeds with no TypeScript errors.

- [ ] **Step 5: Commit**

```bash
git add themes/admin/assets/ts/constants/storage.ts \
        themes/admin/assets/ts/types/index.ts \
        themes/admin/assets/ts/utils/theme.ts
git commit -m "feat(bold-theme): add ThemePreset type, storage key, and presetManager"
```

---

## Task 2: Font dependencies

**Files:**
- Modify: `/package.json` (root)
- Modify: `themes/admin/assets/css/admin.css`

- [ ] **Step 1: Install font packages**

```bash
cd /Users/macgyver/Dropbox/Code/side-projects/cms/calacacms
bun add @fontsource/inter-tight @fontsource/jetbrains-mono
```

Expected: both packages appear in `package.json` dependencies.

- [ ] **Step 2: Import fonts in admin.css**

In `themes/admin/assets/css/admin.css`, after the existing `@import "@fontsource/inter/index.css";` line, add:

```css
/* Inter Tight - display/headline font for Bold theme */
@import "@fontsource/inter-tight/700.css";
@import "@fontsource/inter-tight/800.css";

/* JetBrains Mono - label/mono font for Bold theme */
@import "@fontsource/jetbrains-mono/400.css";
@import "@fontsource/jetbrains-mono/500.css";
```

- [ ] **Step 3: Verify build still works**

```bash
bun run build:admin 2>&1 | tail -5
```

Expected: CSS builds successfully, no import errors.

- [ ] **Step 4: Commit**

```bash
git add package.json bun.lockb themes/admin/assets/css/admin.css
git commit -m "feat(bold-theme): add Inter Tight and JetBrains Mono font packages"
```

---

## Task 3: CSS design tokens — Bold dark, light, and system variants

**Files:**
- Modify: `themes/admin/assets/css/themes/index.css`

Append the following three blocks to the **end** of `themes/admin/assets/css/themes/index.css`. Do not modify any existing content.

- [ ] **Step 1: Add Bold dark token block**

```css
/* ============================================
   BOLD THEME — DARK VARIANT
   Landing page colors: #0b0c15 bg, #a3ff12 lime accent
   ============================================ */
[data-theme-preset="bold"][data-theme="dark"] {
    /* Accent — lime green from landing page */
    --p:  #a3ff12;
    --pf: #8fe010;
    --pc: #0b0c15;

    /* Base Colors — navy dark from landing page */
    --b1: #0b0c15;   /* content background */
    --b2: #0e0f1a;   /* sidebar background */
    --b3: #131420;   /* hover states, inputs */

    /* Card */
    --card: #131420;

    /* Base Content (Text) */
    --bc: #FAFAFA;
    --bc-secondary: #525252;
    --bc-muted: #737373;

    /* Border & Divider */
    --border: #1e2035;
    --divider: #1e2035;
    --border-hover: #2e3050;

    /* Radius — none in Bold theme */
    --radius-sm:   0px;
    --radius:      0px;
    --radius-md:   0px;
    --radius-lg:   0px;
    --radius-xl:   0px;
    --radius-full: 0px;

    /* Typography */
    --font-display: 'Inter Tight', 'Inter', system-ui, sans-serif;
    --font-mono: 'JetBrains Mono', ui-monospace, SFMono-Regular, monospace;

    /* Sidebar */
    --sidebar-bg:          #0e0f1a;
    --sidebar-text:        #525252;
    --sidebar-text-active: #a3ff12;
    --sidebar-hover:       rgba(163, 255, 18, 0.05);

    /* Shadows — none in Bold theme */
    --shadow-sm: none;
    --shadow:    none;
    --shadow-md: none;
    --shadow-lg: none;
    --shadow-xl: none;
}
```

- [ ] **Step 2: Add Bold light token block**

```css
/* ============================================
   BOLD THEME — LIGHT VARIANT
   Warm white bg, dark olive green accent
   ============================================ */
[data-theme-preset="bold"][data-theme="light"] {
    /* Accent — dark olive (lime is unreadable on white) */
    --p:  #3a5a00;
    --pf: #2e4700;
    --pc: #F7F7F2;

    /* Base Colors — warm white */
    --b1: #F7F7F2;
    --b2: #EFEFEA;
    --b3: #E5E5E0;

    /* Card */
    --card: #E5E5E0;

    /* Base Content (Text) */
    --bc: #0b0c15;
    --bc-secondary: #737373;
    --bc-muted: #525252;

    /* Border & Divider */
    --border: #D4D4CC;
    --divider: #D4D4CC;
    --border-hover: #c0c0b8;

    /* Radius — none */
    --radius-sm:   0px;
    --radius:      0px;
    --radius-md:   0px;
    --radius-lg:   0px;
    --radius-xl:   0px;
    --radius-full: 0px;

    /* Typography */
    --font-display: 'Inter Tight', 'Inter', system-ui, sans-serif;
    --font-mono: 'JetBrains Mono', ui-monospace, SFMono-Regular, monospace;

    /* Sidebar */
    --sidebar-bg:          #EFEFEA;
    --sidebar-text:        #737373;
    --sidebar-text-active: #2e4700;
    --sidebar-hover:       rgba(58, 90, 0, 0.08);

    /* Shadows — none */
    --shadow-sm: none;
    --shadow:    none;
    --shadow-md: none;
    --shadow-lg: none;
    --shadow-xl: none;
}
```

- [ ] **Step 3: Add Bold system/auto blocks**

```css
/* ============================================
   BOLD THEME — SYSTEM (auto-detect OS preference)
   ============================================ */
@media (prefers-color-scheme: dark) {
    [data-theme-preset="bold"]:not([data-theme="light"]) {
        --p:  #a3ff12;
        --pf: #8fe010;
        --pc: #0b0c15;
        --b1: #0b0c15;
        --b2: #0e0f1a;
        --b3: #131420;
        --card: #131420;
        --bc: #FAFAFA;
        --bc-secondary: #525252;
        --bc-muted: #737373;
        --border: #1e2035;
        --divider: #1e2035;
        --border-hover: #2e3050;
        --radius-sm: 0px; --radius: 0px; --radius-md: 0px;
        --radius-lg: 0px; --radius-xl: 0px; --radius-full: 0px;
        --font-display: 'Inter Tight', 'Inter', system-ui, sans-serif;
        --font-mono: 'JetBrains Mono', ui-monospace, SFMono-Regular, monospace;
        --sidebar-bg: #0e0f1a;
        --sidebar-text: #525252;
        --sidebar-text-active: #a3ff12;
        --sidebar-hover: rgba(163, 255, 18, 0.05);
        --shadow-sm: none; --shadow: none; --shadow-md: none;
        --shadow-lg: none; --shadow-xl: none;
    }
}

@media (prefers-color-scheme: light) {
    [data-theme-preset="bold"]:not([data-theme="dark"]) {
        --p:  #3a5a00;
        --pf: #2e4700;
        --pc: #F7F7F2;
        --b1: #F7F7F2;
        --b2: #EFEFEA;
        --b3: #E5E5E0;
        --card: #E5E5E0;
        --bc: #0b0c15;
        --bc-secondary: #737373;
        --bc-muted: #525252;
        --border: #D4D4CC;
        --divider: #D4D4CC;
        --border-hover: #c0c0b8;
        --radius-sm: 0px; --radius: 0px; --radius-md: 0px;
        --radius-lg: 0px; --radius-xl: 0px; --radius-full: 0px;
        --font-display: 'Inter Tight', 'Inter', system-ui, sans-serif;
        --font-mono: 'JetBrains Mono', ui-monospace, SFMono-Regular, monospace;
        --sidebar-bg: #EFEFEA;
        --sidebar-text: #737373;
        --sidebar-text-active: #2e4700;
        --sidebar-hover: rgba(58, 90, 0, 0.08);
        --shadow-sm: none; --shadow: none; --shadow-md: none;
        --shadow-lg: none; --shadow-xl: none;
    }
}
```

- [ ] **Step 4: Verify build**

```bash
bun run build:admin 2>&1 | tail -5
```

Expected: CSS compiles without errors.

- [ ] **Step 5: Commit**

```bash
git add themes/admin/assets/css/themes/index.css
git commit -m "feat(bold-theme): add CSS design tokens for dark, light, and system variants"
```

---

## Task 4: CSS component overrides

**Files:**
- Modify: `themes/admin/assets/css/base/base.css`

Append the following to the **end** of `themes/admin/assets/css/base/base.css`, after all existing content.

- [ ] **Step 1: Add Bold component overrides**

```css
/* ============================================
   BOLD THEME — COMPONENT OVERRIDES
   All scoped to [data-theme-preset="bold"]
   ============================================ */

@layer components {

  /* ── Headings use Inter Tight ── */
  [data-theme-preset="bold"] h1,
  [data-theme-preset="bold"] h2,
  [data-theme-preset="bold"] h3 {
    font-family: var(--font-display);
    letter-spacing: -0.04em;
  }

  /* ── Primary button: text + animated underline, no background ── */
  [data-theme-preset="bold"] .btn-primary {
    background-color: transparent;
    color: var(--p);
    border: none;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    font-family: var(--font-display);
    font-weight: 600;
    padding-left: 0;
    padding-right: 0;
    position: relative;
    transform: none;
  }
  [data-theme-preset="bold"] .btn-primary::after {
    content: '';
    position: absolute;
    bottom: 4px;
    left: 0;
    right: 0;
    height: 2px;
    background: var(--p);
    transform: scaleX(1);
    transform-origin: left;
    transition: transform 150ms;
  }
  [data-theme-preset="bold"] .btn-primary:hover:not(.disabled) {
    transform: none;
    background-color: transparent;
  }
  [data-theme-preset="bold"] .btn-primary:hover:not(.disabled)::after {
    transform: scaleX(1.08);
  }
  [data-theme-preset="bold"] .btn-primary:active:not(.disabled) {
    transform: translateY(1px);
  }

  /* ── Secondary / base button: outline → invert on hover ── */
  [data-theme-preset="bold"] .btn,
  [data-theme-preset="bold"] .btn-secondary {
    background-color: transparent;
    color: var(--bc);
    border: 1px solid var(--bc);
    border-radius: 0;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    font-family: var(--font-display);
    font-weight: 600;
    transition: all 150ms;
  }
  [data-theme-preset="bold"] .btn:hover:not(.disabled),
  [data-theme-preset="bold"] .btn-secondary:hover:not(.disabled) {
    background-color: var(--bc);
    color: var(--b1);
    transform: none;
  }

  /* ── Ghost button: hover underline ── */
  [data-theme-preset="bold"] .btn-ghost {
    background-color: transparent;
    color: var(--bc-muted);
    border: none;
    font-family: var(--font-display);
    position: relative;
    transition: color 150ms;
  }
  [data-theme-preset="bold"] .btn-ghost::after {
    content: '';
    position: absolute;
    bottom: 4px;
    left: 0;
    right: 0;
    height: 1px;
    background: currentColor;
    transform: scaleX(0);
    transform-origin: left;
    transition: transform 150ms;
  }
  [data-theme-preset="bold"] .btn-ghost:hover:not(.disabled) {
    color: var(--bc);
    background-color: transparent;
    transform: none;
  }
  [data-theme-preset="bold"] .btn-ghost:hover:not(.disabled)::after {
    transform: scaleX(1);
  }

  /* ── Danger button: outline red ── */
  [data-theme-preset="bold"] .btn-danger {
    background-color: transparent;
    color: #ff6b6b;
    border: 1px solid #ff6b6b;
    font-family: var(--font-display);
    transition: all 150ms;
  }
  [data-theme-preset="bold"] .btn-danger:hover:not(.disabled) {
    background-color: #ff6b6b;
    color: var(--b1);
    transform: none;
  }

  /* ── Inputs: sharp corners, lime focus ── */
  [data-theme-preset="bold"] .input,
  [data-theme-preset="bold"] .textarea,
  [data-theme-preset="bold"] .select {
    background-color: var(--b3);
    border-color: var(--border);
    border-radius: 0;
    transition: border-color 150ms;
  }
  [data-theme-preset="bold"] .input:focus,
  [data-theme-preset="bold"] .textarea:focus,
  [data-theme-preset="bold"] .select:focus {
    border-color: var(--p);
    box-shadow: none;
    outline: none;
  }

  /* ── Flash messages: left accent border only ── */
  [data-theme-preset="bold"] .flash-message {
    border-radius: 0;
    border-top: none;
    border-right: none;
    border-bottom: none;
    border-left-width: 2px;
    border-left-style: solid;
  }
  [data-theme-preset="bold"] .flash-message.success {
    border-left-color: var(--p);
    background-color: color-mix(in srgb, var(--p) 4%, transparent);
  }
  [data-theme-preset="bold"] .flash-message.error {
    border-left-color: var(--er);
    background-color: color-mix(in srgb, var(--er) 4%, transparent);
  }
  [data-theme-preset="bold"] .flash-message.warning {
    border-left-color: var(--wa);
    background-color: color-mix(in srgb, var(--wa) 4%, transparent);
  }
  [data-theme-preset="bold"] .flash-message.info {
    border-left-color: var(--in);
    background-color: color-mix(in srgb, var(--in) 4%, transparent);
  }

  /* ── Modal: sharp corners ── */
  [data-theme-preset="bold"] .modal {
    border-radius: 0;
  }

  /* ── Labels: JetBrains Mono uppercase ── */
  [data-theme-preset="bold"] label,
  [data-theme-preset="bold"] .label {
    font-family: var(--font-mono);
    font-size: 10px;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    color: var(--bc-muted);
  }

  /* ── Stat cards: oversized Inter Tight numbers ── */
  [data-theme-preset="bold"] [data-stat-number] {
    font-family: var(--font-display);
    font-size: 3rem;
    font-weight: 800;
    letter-spacing: -0.04em;
    line-height: 1;
  }

  /* ── Cards: sharp, --border-hover on hover ── */
  [data-theme-preset="bold"] .card,
  [data-theme-preset="bold"] [class*="rounded"] {
    border-radius: 0;
  }

  /* ── Tables: JetBrains Mono headers, b3 row hover ── */
  [data-theme-preset="bold"] table thead th {
    font-family: var(--font-mono);
    font-size: 10px;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    color: var(--bc-secondary);
    font-weight: 500;
    border-bottom: 1px solid var(--border);
  }
  [data-theme-preset="bold"] table tbody tr:hover td {
    background-color: var(--b3);
  }
  [data-theme-preset="bold"] table tbody td {
    border-bottom: 1px solid var(--border);
    font-size: 0.875rem;
    color: var(--bc-muted);
  }
  [data-theme-preset="bold"] table tbody td:first-child {
    color: var(--bc);
    font-weight: 500;
  }

  /* ── Badges: sharp, dark/light specific rgba ── */
  [data-theme-preset="bold"] .badge,
  [data-theme-preset="bold"] [class*="badge"] {
    border-radius: 0;
    font-family: var(--font-mono);
    font-size: 10px;
    letter-spacing: 0.1em;
    text-transform: uppercase;
  }
  /* Draft badge — explicit dark */
  [data-theme-preset="bold"][data-theme="dark"] .badge-draft {
    background: rgba(163, 255, 18, 0.08);
    color: var(--p);
    border-color: rgba(163, 255, 18, 0.2);
  }
  /* Draft badge — system dark */
  @media (prefers-color-scheme: dark) {
    [data-theme-preset="bold"]:not([data-theme="light"]) .badge-draft {
      background: rgba(163, 255, 18, 0.08);
      color: var(--p);
      border-color: rgba(163, 255, 18, 0.2);
    }
  }
  /* Draft badge — explicit light */
  [data-theme-preset="bold"][data-theme="light"] .badge-draft {
    background: rgba(58, 90, 0, 0.08);
    color: var(--p);
    border-color: rgba(58, 90, 0, 0.25);
  }
  /* Draft badge — system light */
  @media (prefers-color-scheme: light) {
    [data-theme-preset="bold"]:not([data-theme="dark"]) .badge-draft {
      background: rgba(58, 90, 0, 0.08);
      color: var(--p);
      border-color: rgba(58, 90, 0, 0.25);
    }
  }

  /* ── Sidebar: 2px left accent bar replaces rounded pill ── */
  [data-theme-preset="bold"] #page-sidebar a[style*="sidebar-text-active"],
  [data-theme-preset="bold"] #page-sidebar .sidebar-item-active {
    border-radius: 0;
  }
  /* The sidebar active pill indicator span gets restyled via token cascade:
     --sidebar-text-active drives text color, --sidebar-hover drives bg.
     The existing left border indicator (absolute w-1 rounded-r-full) needs
     border-radius zeroed — achieved by the global --radius-full: 0px token. */

}
```

- [ ] **Step 2: Build and verify**

```bash
bun run build:admin 2>&1 | tail -5
```

Expected: builds without errors.

- [ ] **Step 3: Visual smoke test**

Run the dev server and open the browser:

```bash
bun run start
```

Open `http://localhost:8000/admin`. In the browser console, run:

```js
document.documentElement.setAttribute('data-theme-preset', 'bold');
document.documentElement.setAttribute('data-theme', 'dark');
```

Expected: page turns navy-dark with lime accent. Buttons should lose their background fills, sidebar should show lime active state.

Then test light:

```js
document.documentElement.setAttribute('data-theme', 'light');
```

Expected: page turns warm white with dark olive green accent.

Reset with:

```js
document.documentElement.setAttribute('data-theme-preset', 'default');
```

- [ ] **Step 4: Commit**

```bash
git add themes/admin/assets/css/base/base.css
git commit -m "feat(bold-theme): add component overrides for buttons, inputs, flash messages"
```

---

## Task 5: Layout — anti-flash script

**Files:**
- Modify: `themes/admin/templates/layouts/layout.twig`

- [ ] **Step 1: Extend the anti-flash inline script**

In `layout.twig`, locate the existing anti-flash `<script>` block (lines ~11–19). It currently looks like:

```html
<script>
    (() => {
        try {
            const stored = localStorage.getItem('themeMode');
            const isDark = stored ? stored === 'dark' : matchMedia('(prefers-color-scheme: dark)').matches;
            document.documentElement.setAttribute('data-theme', isDark ? 'dark' : 'light');
        } catch (_) {}
    })();
</script>
```

Add the preset line **after** the `setAttribute('data-theme', ...)` call, still inside the `try` block:

```html
<script>
    (() => {
        try {
            const stored = localStorage.getItem('themeMode');
            const isDark = stored ? stored === 'dark' : matchMedia('(prefers-color-scheme: dark)').matches;
            document.documentElement.setAttribute('data-theme', isDark ? 'dark' : 'light');
            const preset = localStorage.getItem('calaca-admin-theme-preset') || 'default';
            document.documentElement.setAttribute('data-theme-preset', preset);
        } catch (_) {}
    })();
</script>
```

**Important:** Do not touch or "fix" the existing `'themeMode'` key. The key inconsistency with `'calaca-admin-theme'` is a known issue deferred to a separate PR.

- [ ] **Step 2: Build and verify no-flash**

```bash
bun run build:admin 2>&1 | tail -5
```

Then open `http://localhost:8000/admin` in the browser. In Application → Local Storage, set `calaca-admin-theme-preset` to `bold`. Hard-reload the page (Cmd+Shift+R). Expected: page loads with Bold theme immediately, no flash of default theme.

- [ ] **Step 3: Commit**

```bash
git add themes/admin/templates/layouts/layout.twig
git commit -m "feat(bold-theme): add preset anti-flash script to layout head"
```

---

## Task 6: Settings page — theme preset switcher

**Files:**
- Modify: `themes/admin/templates/pages/settings/index.twig`

- [ ] **Step 1: Add Appearance section before the Save button**

In `settings/index.twig`, locate the save button section:

```twig
{# Save button #}
<div class="flex justify-end">
```

Insert the following **before** that save button block:

```twig
{# Appearance — Theme Preset #}
<div class="bg-base-100 rounded-xl border border-border/50 shadow-sm p-6"
     x-data="{
       themePreset: localStorage.getItem('calaca-admin-theme-preset') || 'default',
       setPreset(preset) {
         this.themePreset = preset;
         localStorage.setItem('calaca-admin-theme-preset', preset);
         document.documentElement.setAttribute('data-theme-preset', preset);
       }
     }">
  <h2 class="text-sm font-semibold text-base-content mb-1 flex items-center gap-2">
    {{ icon('palette', 'size-4') }} Appearance
  </h2>
  <p class="text-xs text-base-content/50 mb-5">Choose the visual style of the admin interface.</p>

  <div class="flex gap-3">

    {# Default theme card #}
    <button type="button"
            @click="setPreset('default')"
            class="flex-1 p-4 text-left transition-colors cursor-pointer"
            :style="themePreset === 'default'
              ? 'border: 2px solid var(--p);'
              : 'border: 1px solid var(--border);'">
      <div class="flex gap-2 mb-3">
        <span class="w-5 h-5 inline-block" style="background: oklch(52% 0.25 264);"></span>
        <span class="w-5 h-5 inline-block" style="background: oklch(96% 0 0); border: 1px solid var(--border);"></span>
      </div>
      <div class="text-sm font-semibold text-base-content">Default</div>
      <div class="text-xs text-base-content/50 mt-1">Indigo accent, clean greys</div>
    </button>

    {# Bold theme card #}
    <button type="button"
            @click="setPreset('bold')"
            class="flex-1 p-4 text-left transition-colors cursor-pointer"
            :style="themePreset === 'bold'
              ? 'border: 2px solid var(--p);'
              : 'border: 1px solid var(--border);'">
      <div class="flex gap-2 mb-3">
        <span class="w-5 h-5 inline-block" style="background: #a3ff12;"></span>
        <span class="w-5 h-5 inline-block" style="background: #0b0c15;"></span>
      </div>
      <div class="text-sm font-semibold text-base-content">Bold</div>
      <div class="text-xs text-base-content/50 mt-1">Lime accent, dark navy — matches landing page</div>
    </button>

  </div>
</div>
```

- [ ] **Step 2: Visual test — switcher works live**

```bash
bun run start
```

Open `http://localhost:8000/admin/settings`. Verify:
1. Two cards visible: "Default" and "Bold"
2. The currently active card has a thicker accent border
3. Clicking "Bold" immediately changes the page theme (no reload)
4. Clicking "Default" restores the original theme
5. Hard-reloading after selecting "Bold" preserves the choice

- [ ] **Step 3: Commit**

```bash
git add themes/admin/templates/pages/settings/index.twig
git commit -m "feat(bold-theme): add theme preset switcher to settings page"
```

---

## Task 7: Final integration check and cleanup

- [ ] **Step 1: Full build**

```bash
cd /Users/macgyver/Dropbox/Code/side-projects/cms/calacacms
bun run build:admin
```

Expected: exits 0, no errors.

- [ ] **Step 2: Check all pages in Bold dark mode**

Start the dev server:

```bash
bun run start
```

Set Bold dark via Settings, then visit:
- `/admin` — dashboard stat cards, sidebar lime active state, page title Inter Tight
- `/admin/articles` — table rows, Draft badge lime, Published badge muted
- `/admin/articles/new` (or edit) — inputs have sharp corners, lime focus border
- `/admin/settings` — switcher cards render correctly in Bold style

- [ ] **Step 3: Check Bold light mode**

Switch to light mode via the existing dark/light toggle while Bold is active. Verify:
- Background becomes warm white (#F7F7F2)
- Accent becomes dark olive (#3a5a00)
- Draft badge uses dark olive tones, not neon lime

- [ ] **Step 4: Check persistence across page loads**

In Application → Local Storage, verify `calaca-admin-theme-preset` is set to `bold`. Hard-reload every page. Expected: Bold theme applies immediately on each load without flash.

- [ ] **Step 5: Reset to default and verify**

Go to Settings, click "Default". Verify theme reverts to indigo/grey. Hard-reload — stays on default.

- [ ] **Step 6: Add .superpowers to .gitignore if not already present**

```bash
grep -q ".superpowers" .gitignore || echo ".superpowers/" >> .gitignore
git add .gitignore
```

- [ ] **Step 7: Final commit**

```bash
git add .gitignore
git commit -m "feat(bold-theme): complete Bold theme implementation

- Landing page colors: #0b0c15 navy bg, #a3ff12 lime accent
- data-theme-preset attribute system with anti-flash script
- Inter Tight headlines, JetBrains Mono labels
- Zero border-radius throughout
- Settings page switcher with live preview
- Dark, light, and system/auto variants all covered"
```
