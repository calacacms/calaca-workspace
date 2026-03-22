# Semantic Color System

## Overview

CalacaCMS uses a Daisy UI-inspired semantic color system based on OKLCH CSS variables. This enables automatic theme switching without manual `dark:` classes.

**Important**: CalacaCMS uses OKLCH colors with Daisy UI naming convention. OKLCH (OK Lightness Chroma Hue) provides a balanced, accessible color palette that works well across both light and dark themes.

## How It Works

1. **CSS Variables**: Colors are defined as OKLCH values in CSS custom properties
2. **Daisy UI Naming**: Uses short variable names (`--p`, `--s`, `--a`, `--n`, `--b1`, etc.)
3. **Theme Switching**: Change CSS variable values to switch themes
4. **Automatic Dark Mode**: No manual `dark:` classes needed

## Color Variables

### Daisy UI Naming Convention

```css
/* Primary Colors */
--p: oklch(52% 0.25 264);      /* Primary */
--pf: oklch(47% 0.23 264);    /* Primary Focus */
--pc: oklch(100% 0 0);       /* Primary Content */

/* Secondary Colors */
--s: oklch(55% 0 0);         /* Secondary */
--sf: oklch(45% 0 0);        /* Secondary Focus */
--sc: oklch(100% 0 0);       /* Secondary Content */

/* Accent Colors */
--a: oklch(65% 0.15 230);     /* Accent */
--af: oklch(55% 0.15 230);    /* Accent Focus */
--ac: oklch(100% 0 0);       /* Accent Content */

/* Neutral Colors */
--n: oklch(20% 0 0);         /* Neutral */
--nf: oklch(12% 0 0);        /* Neutral Focus */
--nc: oklch(98% 0 0);        /* Neutral Content */

/* Base Colors */
--b1: oklch(100% 0 0);       /* Base-100 - main content area */
--b2: oklch(96% 0 0);        /* Base-200 - sidebar, page bg */
--b3: oklch(91% 0 0);        /* Base-300 - hover states */

/* Base Content */
--bc: oklch(15% 0 0);        /* Base Content - primary text */
--bc-secondary: oklch(52% 0 0); /* Secondary text */
--bc-muted: oklch(68% 0 0);   /* Muted text */
```

### Status Colors (Semantic)

```css
/* Info */
--in: oklch(65% 0.15 230);      /* Info */
--inf: oklch(55% 0.15 230);    /* Info Focus */
--inc: oklch(100% 0 0);       /* Info Content */

/* Success */
--su: oklch(65% 0.2 160);      /* Success */
--suf: oklch(55% 0.18 160);    /* Success Focus */
--suc: oklch(100% 0 0);       /* Success Content */

/* Warning */
--wa: oklch(75% 0.15 80);      /* Warning */
--waf: oklch(65% 0.15 80);     /* Warning Focus */
--wac: oklch(20% 0 0);        /* Warning Content */

/* Error */
--er: oklch(58% 0.22 25);      /* Error */
--erf: oklch(50% 0.22 25);     /* Error Focus */
--erc: oklch(100% 0 0);       /* Error Content */

/* Border & Divider */
--border: oklch(88% 0 0);     /* Border */
--divider: oklch(93% 0 0);    /* Divider */
```

## Semantic Classes

### Background Classes

```css
.bg-primary { background-color: var(--p); }
.bg-secondary { background-color: var(--s); }
.bg-accent { background-color: var(--a); }
.bg-neutral { background-color: var(--n); }
.bg-base-100 { background-color: var(--b1); }
.bg-base-200 { background-color: var(--b2); }
.bg-base-300 { background-color: var(--b3); }
```

### Text Classes

```css
.text-primary-content { color: var(--pc); }
.text-secondary-content { color: var(--sc); }
.text-accent-content { color: var(--ac); }
.text-neutral-content { color: var(--nc); }
.text-base-content { color: var(--bc); }
```

### Focus Classes

```css
.focus:ring-primary-focus { --tw-ring-color: var(--pf); }
.focus:ring-secondary-focus { --tw-ring-color: var(--sf); }
.focus:ring-accent-focus { --tw-ring-color: var(--af); }
.focus:ring-neutral-focus { --tw-ring-color: var(--nf); }
```

### Status Classes

```css
.bg-info { background-color: var(--in); }
.bg-success { background-color: var(--su); }
.bg-warning { background-color: var(--wa); }
.bg-error { background-color: var(--er); }

.text-info-content { color: var(--inc); }
.text-success-content { color: var(--suc); }
.text-warning-content { color: var(--wac); }
.text-error-content { color: var(--erc); }
```

## Theme System

### Light Theme (Default)

```css
:root,
[data-theme="light"] {
    /* Primary Colors — Indigo */
    --p: oklch(52% 0.25 264);
    --pf: oklch(47% 0.23 264);
    --pc: oklch(100% 0 0);
    
    /* Base Colors — Zinc */
    --b1: oklch(100% 0 0);       /* white — content area */
    --b2: oklch(96% 0 0);        /* zinc-100 — sidebar, page bg */
    --b3: oklch(91% 0 0);        /* zinc-200 — hover states */
    
    /* Base Content (Text) */
    --bc: oklch(15% 0 0);        /* zinc-900 — primary text */
    --bc-secondary: oklch(52% 0 0); /* zinc-500 — secondary text */
    --bc-muted: oklch(68% 0 0);  /* zinc-400 — muted text */
}
```

### Dark Theme

```css
[data-theme="dark"],
.dark {
    /* Primary Colors — Indigo (lighter for dark bg) */
    --p: oklch(65% 0.25 264);
    --pf: oklch(72% 0.22 264);
    --pc: oklch(100% 0 0);
    
    /* Base Colors — Zinc dark */
    --b1: oklch(15% 0 0);        /* zinc-900 — main content bg */
    --b2: oklch(22% 0 0);        /* zinc-800 — sidebar bg */
    --b3: oklch(32% 0 0);        /* zinc-700 — hover states */
    
    /* Base Content (Text) */
    --bc: oklch(98% 0 0);        /* zinc-50 — primary text */
    --bc-secondary: oklch(68% 0 0); /* zinc-400 — secondary text */
    --bc-muted: oklch(52% 0 0);  /* zinc-500 — muted text */
}
```

### Auto Theme Detection

```css
@media (prefers-color-scheme: dark) {
    :root:not([data-theme]):not(.dark) {
        /* Dark theme variables applied automatically */
    }
}
```

All elements using semantic classes will automatically switch to the appropriate theme.

## Usage Examples

### Card Component

```html
<!-- Automatically adapts to theme -->
<div class="bg-base-100 text-base-content border border-border rounded-lg shadow">
  <div class="p-6">
    <h3 class="text-lg font-semibold">Card Title</h3>
    <p class="text-base-content/80">Card description</p>
  </div>
</div>
```

### Form Elements

```html
<!-- Input automatically adapts to theme -->
<input class="bg-base-200 text-base-content border border-border rounded-md px-3 py-2" 
       placeholder="Enter text..." />

<!-- Button variants -->
<button class="bg-primary text-primary-content hover:opacity-90 px-4 py-2 rounded">
  Save
</button>
<button class="bg-error text-error-content hover:opacity-90 px-4 py-2 rounded">
  Delete
</button>
```

### Status Indicators

```html
<span class="bg-success text-success-content px-2 py-1 rounded-full text-xs">Published</span>
<span class="bg-warning text-warning-content px-2 py-1 rounded-full text-xs">Draft</span>
<span class="bg-error text-error-content px-2 py-1 rounded-full text-xs">Error</span>
```

## Benefits

1. **No Dark Classes**: No need for `dark:bg-gray-800`
2. **Consistent Theming**: One theme change affects all components
3. **Daisy UI Convention**: Familiar naming convention for developers
4. **OKLCH Colors**: Better perceptual uniformity and accessibility
5. **Auto Detection**: Respects system color scheme preferences
6. **DRY**: Colors defined once, used everywhere

## Theme Configuration

Themes are configured in `themes/admin/assets/css/themes/index.css` using OKLCH values:

- **Light theme**: Applied by default to `:root` and `[data-theme="light"]`
- **Dark theme**: Applied to `[data-theme="dark"]` and `.dark`
- **Auto detection**: Applied via `@media (prefers-color-scheme: dark)`

The theme system uses Daisy UI naming convention and OKLCH color space for better color consistency.

## Migration from Hardcoded Colors

### Before (Don't do this)

```html
<div class="bg-white dark:bg-slate-900 text-slate-900 dark:text-white">
  <button class="bg-blue-500 dark:bg-blue-600 text-white">
```

### After (Do this)

```html
<div class="bg-base-100 text-base-content">
  <button class="bg-primary text-primary-content">
```

### Key Changes
- Replace `bg-white/dark:bg-slate-900` with `bg-base-100`
- Replace `text-slate-900/dark:text-white` with `text-base-content`
- Replace `bg-blue-500/dark:bg-blue-600` with `bg-primary`
- Replace `text-white` with `text-primary-content`

## Resources

- [Tailwind v4 Setup](./tailwind-v4-setup.md)
- [Admin Components](./admin-components.md)
- [PROJECT_SSOT.md](../../PROJECT_SSOT.md)
- [Theme File](../../../themes/admin/assets/css/themes/index.css) - Source theme definitions
