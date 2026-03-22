# Tailwind CSS v4 Setup for Admin UI

## Overview

CalacaCMS uses Tailwind CSS v4 for the admin interface with a content-aware theming system.

## Key Differences from Tailwind v3

- **No `@tailwind` directives**: Use `@import "tailwindcss";` instead
- **CSS-first configuration**: Configuration lives in CSS, not `tailwind.config.js`
- **Semantic color classes**: `bg-primary`, `text-foreground` instead of hardcoded colors
- **Content-aware themes**: No manual `dark:` classes needed

## Installation

Already configured in `themes/admin/` directory:

```bash
# From project root
bun install                    # Install all dependencies
bun run dev:admin              # Admin theme development (port 5173)
bun run build:admin            # Build admin theme for production
bun run dev                    # Full development (admin + default themes)
bun run build                  # Build all themes
```

## CSS Structure

```css
/* themes/admin/assets/css/admin.css */
@import "tailwindcss";
@import "basecoat-css";

/* Inter Variable Font - one file for all weights (100-900) */
@import "@fontsource/inter/index.css";

@import "./themes/index.css";
@import "./base/base.css";

/* Tell Tailwind where to scan for classes */
@source "../../templates/**/*.twig";

@theme {
    /* Semantic colors for Tailwind v4 */
    --color-primary: var(--p);
    --color-primary-focus: var(--pf);
    --color-primary-content: var(--pc);
    
    --color-secondary: var(--s);
    --color-secondary-focus: var(--sf);
    --color-secondary-content: var(--sc);
    
    --color-accent: var(--a);
    --color-accent-focus: var(--af);
    --color-accent-content: var(--ac);
    
    --color-neutral: var(--n);
    --color-neutral-focus: var(--nf);
    --color-neutral-content: var(--nc);
    
    --color-base-100: var(--b1);
    --color-base-200: var(--b2);
    --color-base-300: var(--b3);
    --color-base-content: var(--bc);
}
```

## Semantic Color Classes

Use these semantic classes instead of hardcoded Tailwind colors:

### Backgrounds
- `bg-primary` - Primary brand color (`var(--p)`)
- `bg-secondary` - Secondary background (`var(--s)`)
- `bg-accent` - Accent color (`var(--a)`)
- `bg-neutral` - Neutral background (`var(--n)`)
- `bg-base-100` - Base background level 1 (`var(--b1)`)
- `bg-base-200` - Base background level 2 (`var(--b2)`)
- `bg-base-300` - Base background level 3 (`var(--b3)`)

### Text
- `text-primary-content` - Primary text color (`var(--pc)`)
- `text-secondary-content` - Secondary text (`var(--sc)`)
- `text-accent-content` - Accent text (`var(--ac)`)
- `text-neutral-content` - Neutral text (`var(--nc)`)
- `text-base-content` - Base text color (`var(--bc)`)

### Focus States
- `focus:ring-primary-focus` - Primary focus ring (`var(--pf)`)
- `focus:ring-secondary-focus` - Secondary focus ring (`var(--sf)`)
- `focus:ring-accent-focus` - Accent focus ring (`var(--af)`)
- `focus:ring-neutral-focus` - Neutral focus ring (`var(--nf)`)

## Component Patterns

### Card

```html
<div class="bg-base-100 text-base-content rounded-lg border border-neutral shadow">
  <div class="p-6">
    <h3 class="text-lg font-semibold">Card Title</h3>
    <p class="text-neutral-content">Card description</p>
  </div>
</div>
```

### Form Input

```html
<input 
  type="text" 
  class="bg-base-200 text-base-content border border-neutral rounded-md px-3 py-2
         focus:outline-none focus:ring-2 focus:ring-primary-focus"
  placeholder="Enter text..."
>
```

### Button Variants

```html
<!-- Primary -->
<button class="bg-primary text-primary-content hover:opacity-90 px-4 py-2 rounded">
  Primary Action
</button>

<!-- Secondary -->
<button class="bg-secondary text-secondary-content hover:opacity-90 px-4 py-2 rounded">
  Secondary
</button>

<!-- Accent -->
<button class="bg-accent text-accent-content hover:opacity-90 px-4 py-2 rounded">
  Accent
</button>
```

## Best Practices

1. **Use semantic classes**: `bg-primary` not `bg-blue-500`
2. **Basecoat CSS integration**: Theme variables are provided by basecoat-css
3. **CSS variables**: Use Basecoat variables (`--p`, `--s`, `--a`, `--n`, `--b1`, etc.)
4. **Consistent patterns**: Use the same patterns across all components
5. **Content-aware themes**: No manual `dark:` classes needed

## Build Commands

```bash
# Development with watch (admin theme only)
bun run dev:admin

# Development with watch (all themes)
bun run dev

# Build for production (admin theme only)
bun run build:admin

# Build all themes for production
bun run build

# Full development server (PHP + Vite)
bun run start

# Type checking
cd themes/admin && bunx tsc --noEmit
```

## File Locations

- **Source CSS**: `themes/admin/assets/css/admin.css`
- **CSS Components**: `themes/admin/assets/css/components/`
- **CSS Themes**: `themes/admin/assets/css/themes/`
- **CSS Base**: `themes/admin/assets/css/base/`
- **Built CSS**: `web/assets/admin/css/admin.css`
- **TypeScript**: `themes/admin/assets/ts/`
- **Templates**: `themes/admin/templates/`

## Resources

- [PROJECT_SSOT.md](../../PROJECT_SSOT.md) - Complete project documentation
- [Admin Components](./admin-components.md) - Component library
- [Semantic Colors](./semantic-colors.md) - Color system details
