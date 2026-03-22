# Tailwind v4 Modular Theme System - Design Document

**Document Version:** 1.0
**Date:** 2026-03-21
**Status:** Approved
**Author:** AI Agent Team

---

## Executive Summary

Refactor CalacaCMS admin theme CSS architecture from a monolithic 1400+ line theme override system to a modular, semantic variable-based architecture. This enables easy theme addition, reduces code duplication, and leverages Tailwind CSS v4's modern features.

---

## Problem Statement

### Current Issues

1. **Non-DRY Theme Overrides:** `base.css` contains ~1400 lines of theme-specific overrides. Each new theme requires duplicating this pattern.

2. **Tight Coupling:** Component styles are tightly coupled to theme definitions, making it difficult to add new themes without modifying component CSS.

3. **Maintenance Burden:** Adding a new theme requires:
   - Finding all component class definitions
   - Writing theme-specific overrides for each
   - Ensuring consistency across all components

4. **Mixed Concerns:** Theme variables and component styles are mixed in the same files.

### Example of Current Problem

```css
/* base.css - 100+ lines just for button theme overrides */
[data-theme-preset="bold"] .btn-primary {
    display: inline-flex;
    background: none !important;
    border: none !important;
    /* ... 20 more lines */
}

[data-theme-preset="monochrome"] .btn-primary {
    background-color: var(--p);
    border: none;
    border-radius: 0;
    /* ... 20 more lines */
}

[data-theme-preset="linear"] .btn-primary {
    background-color: var(--p);
    box-shadow: 0 0 0 1px rgba(94,106,210,0.5);
    /* ... 20 more lines */
}
```

---

## Proposed Solution

### Architecture Principles

1. **Separation of Concerns:** Theme variables separate from component definitions
2. **Semantic Abstraction:** Components use semantic variables (`var(--btn-primary-bg)`), not direct theme colors
3. **Theme-as-Package:** Each theme is self-contained in its own file
4. **Progressive Enhancement:** Inline utilities for layout, CSS classes for complex/stateful components

### New File Structure

```
themes/admin/assets/css/
├── admin.css                    # Entry point
├── components/
│   └── skeleton.css             # Keep as-is (complex animations)
└── base/
    ├── base.css                 # Semantic component primitives
    └── themes/                  # NEW: Modular theme directory
        ├── _index.css           # Imports all theme files
        ├── _light.css           # Light theme variables
        ├── _dark.css            # Dark theme variables
        ├── _bold.css            # Bold theme variables
        ├── _monochrome.css      # Monochrome theme variables
        └── _linear.css          # Linear theme variables
```

---

## Component Refactoring Strategy

### Semantic Variable Naming

Components reference semantic variables, not theme colors directly:

```css
/* Component Definition - base.css */
.btn-primary {
    background-color: var(--btn-primary-bg);
    color: var(--btn-primary-color);
    border-radius: var(--btn-primary-radius);
    font-family: var(--btn-primary-font);
    text-transform: var(--btn-primary-transform);
    padding: var(--btn-primary-padding);
}

/* Theme Definition - _bold.css */
[data-theme-preset="bold"][data-theme="dark"] {
    --btn-primary-bg: transparent;
    --btn-primary-color: #a3ff12;
    --btn-primary-radius: 0;
    --btn-primary-font: 'Inter Tight', sans-serif;
    --btn-primary-transform: uppercase;
    --btn-primary-padding: 8px 0;
}
```

### Template Migration Strategy

| Category | Current | Target | Example |
|----------|---------|--------|---------|
| Layout | `.flex` | Inline Tailwind | `class="flex gap-2"` |
| Spacing | `.p-4` | Inline Tailwind | `class="p-4 mb-2"` |
| Stateful Components | `.btn-primary` | Keep CSS class | `class="btn btn-primary"` |
| Complex Animations | `.skeleton` | Keep CSS class | `class="skeleton"` |

**Rationale:** Inline utilities for simple properties, CSS classes for:
- Interactive states (hover, focus, active)
- Complex animations
- Theme-dependent styles
- Reusable patterns

---

## Migration Plan

### Phase 1: File Structure (Non-Breaking)

1. Create `base/themes/` directory
2. Split `themes/index.css` into separate theme files
3. Update imports in `admin.css`

### Phase 2: Component Refactoring

1. Refactor component classes to use semantic variables
2. Define semantic variables for each theme
3. Test each theme for visual consistency

### Phase 3: Template Migration

1. Audit template usage of custom classes
2. Convert layout/spacing to inline Tailwind
3. Keep complex/stateful components as CSS classes
4. Remove unused CSS classes

### Phase 4: Cleanup

1. Remove redundant theme-specific overrides
2. Verify all themes work correctly
3. Document new theme addition process

---

## Theme Variable Specification

### Required Variables Per Theme

Each theme MUST define these variables:

```css
/* Colors */
--color-primary, --color-primary-focus, --color-primary-content
--color-secondary, --color-secondary-focus, --color-secondary-content
--color-accent, --color-accent-focus, --color-accent-content
--color-neutral, --color-neutral-focus, --color-neutral-content

/* Base Colors */
--color-base-100, --color-base-200, --color-base-300
--color-base-content, --color-base-content-secondary, --color-base-content-muted

/* Semantic States */
--color-info, --color-success, --color-warning, --color-error

/* Borders */
--color-border, --color-border-hover, --color-divider

/* Typography */
--font-sans, --font-mono, --font-display, --font-body

/* Spacing */
--radius-sm, --radius, --radius-md, --radius-lg, --radius-xl, --radius-full

/* Shadows */
--shadow-sm, --shadow, --shadow-md, --shadow-lg, --shadow-xl

/* Component-Specific (optional) */
--btn-primary-*
--input-*
--card-*
```

---

## Future Theme Addition Process

Adding a new theme is now a 3-step process:

### Step 1: Create Theme File

```bash
touch themes/admin/assets/css/base/themes/_nouveau.css
```

### Step 2: Define Variables

```css
/* base/themes/_nouveau.css */
[data-theme-preset="nouveau"][data-theme="dark"] {
    --color-primary: #ff6b35;
    --color-primary-content: #ffffff;
    --radius: 24px;
    --font-display: 'Outfit', sans-serif;
    /* ... other variables */
}
```

### Step 3: Import and Use

```css
/* base/themes/_index.css */
@import "./_nouveau.css";
```

```twig
{# Template #}
<html data-theme-preset="nouveau" data-theme="dark">
```

**No component CSS required** - all components automatically use the new variables.

---

## Success Criteria

- [ ] All existing themes (light, dark, bold, monochrome, linear) work identically
- [ ] New theme file structure in place
- [ ] Component classes use semantic variables only
- [ ] Template classes converted where appropriate
- [ ] Redundant theme overrides removed
- [ ] Documentation updated
- [ ] Can add new theme by creating single CSS file

---

## Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Visual regression | High | Side-by-side testing before/after, screenshots |
| Breaking changes | Medium | Phased rollout, keep old CSS during transition |
| Missed edge cases | Medium | Comprehensive testing of all interactive states |
| Performance | Low | CSS variables are performant, Tailwind purges unused |

---

## Open Questions

1. Should we remove `@apply` directive usage in favor of pure Tailwind?
   - Decision: Keep `@apply` for component primitives, use inline for templates

2. Should skeleton loading be converted to inline utilities?
   - Decision: No, keep as CSS class due to complex animations

3. Should we create Tailwind plugin for theme-specific utilities?
   - Decision: No, CSS variables are sufficient and more portable

---

## Appendix A: Component Classes to Convert

### Keep as CSS Classes (Complex/Stateful)
- `.btn`, `.btn-primary`, `.btn-secondary`, etc.
- `.input`, `.textarea`, `.select`
- `.checkbox`, `.radio`
- `.modal` (and all modal sub-classes)
- `.flash-message` (all variants)
- `.skeleton` (all variants)
- `.badge` (theme-specific)

### Convert to Inline (Simple/Layout)
- `.card` → `rounded-xl border bg-base-100`
- `.field` → `space-y-2` (or similar)
- Layout containers
- Spacing utilities
- Typography hierarchy (when not theme-specific)

---

## Appendix B: Testing Checklist

For each theme (light, dark, bold, monochrome, linear):
- [ ] Buttons render correctly
- [ ] Inputs focus states work
- [ ] Modal displays properly
- [ ] Flash messages show correct colors
- [ ] Skeleton animations work
- [ ] Hover/focus/active states correct
- [ ] Dark/light mode toggle works
- [ ] All pages render without visual regressions

---

**End of Design Document**
