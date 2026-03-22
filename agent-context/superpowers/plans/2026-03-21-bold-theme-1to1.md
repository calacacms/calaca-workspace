# Bold Theme 1:1 Replication Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replicate the Bold admin theme reference design 1:1 across sidebar, header, dashboard, and articles table.

**Architecture:** Template-first approach — Twig markup is adjusted to carry the correct CSS classes/structure so that existing scoped Bold CSS selectors (`[data-theme-preset="bold"]`) trigger reliably. Where Bold-specific UI differs structurally from other themes, Alpine.js `x-show` gates visibility on `document.documentElement.getAttribute('data-theme-preset') === 'bold'` (theme preset is client-side only via localStorage). CSS overrides are consolidated (remove data-attribute selectors, add class-based selectors) and gaps filled.

**Tech Stack:** Twig templates, Tailwind CSS v4 (via Vite + `@tailwindcss/vite`), Alpine.js, Bun build toolchain.

**Spec:** `docs/superpowers/specs/2026-03-21-bold-theme-1to1-design.md`

**Build commands (run from project root):**
- Dev: `bun run dev:admin` + `bun run serve` (in separate terminals)
- Build: `bun run build:admin`
- Verify in browser at `http://localhost:8000/admin` with theme preset set to "bold" (via Settings page or localStorage: `localStorage.setItem('calaca-admin-theme-preset', 'bold')`)

---

## File Map

| File | Change type | What changes |
|------|-------------|-------------|
| `themes/admin/assets/css/base/base.css` | Modify | Remove old `[data-stat-number]` selectors; add `.stat-number`, `.stat-label`, `.stat-sub`; add `.bold-hide`, `.sidebar-item-icon`, `.header-icon-btn`, `#page-header` height, `.col-date` overrides |
| `themes/admin/templates/partials/_sidebar.twig` | Modify | Brand icon (C box), nav item square icons + `.bold-hide` on Lucide icons, `nav-item-bar`, `nav-section-label` class on section label buttons |
| `themes/admin/templates/partials/_header.twig` | Modify | Bold breadcrumb variant (x-show), Bold search box variant (x-show), `header-icon-btn` classes on bell + theme toggle |
| `themes/admin/templates/pages/dashboard/dashboard.twig` | Modify | Stat card classes (`stat-number`, `stat-label`, `stat-sub`), lower section dual-variant with x-show |
| `themes/admin/templates/pages/articles/index.twig` | Modify | Badge Alpine binding, `table-container` wrapper, `col-date` class, action link styling |

---

## Task 1: CSS — Consolidate and add Bold overrides

**Files:**
- Modify: `themes/admin/assets/css/base/base.css`

This task fixes CSS before touching any templates — so you can verify styles are correct independently.

- [ ] **Step 1: Open `base.css` and delete obsolete selectors**

Delete these rule blocks entirely (search for each):

1. `[data-stat-number]` — two occurrences (~line 536 and ~line 680). Delete each whole rule block.
2. `[data-stat-label]` — same, delete both occurrences.
3. `[data-theme-preset="bold"] aside nav a` — one block (~line 554–562). Delete it (replaced by `.nav-item` class selectors already present further down).
4. `[data-theme-preset="bold"] #page-sidebar .sidebar-item-icon` — one block (~line 879). Delete it (superseded by the new `.sidebar-item-icon` rule added in Step 3).

**Important:** Do NOT delete `[data-stat-number]` or `[data-stat-label]` occurrences from other theme sections (monochrome, linear) — only delete the ones in the Bold-scoped blocks.

- [ ] **Step 2: Add new stat card classes**

At the end of the `@layer components { ... }` Bold section (before the closing `}` of the last Bold block), add:

```css
  /* ── Stat card classes ── */
  [data-theme-preset="bold"] .stat-number {
    font-family: 'Inter Tight', system-ui, sans-serif !important;
    font-size: 48px !important;
    font-weight: 800 !important;
    letter-spacing: -0.04em !important;
    line-height: 1 !important;
    color: var(--bc) !important;
  }
  [data-theme-preset="bold"] .stat-label {
    font-family: 'JetBrains Mono', monospace !important;
    font-size: 10px !important;
    letter-spacing: 0.12em !important;
    text-transform: uppercase !important;
    color: var(--bc-secondary) !important;
    margin-top: 6px !important;
  }
  [data-theme-preset="bold"] .stat-sub {
    font-size: 12px !important;
    color: var(--bc-muted) !important;
    margin-top: 12px !important;
    padding-top: 12px !important;
    border-top: 1px solid var(--border) !important;
  }
```

- [ ] **Step 3: Add utility classes for Bold-only visibility toggling**

Add after the stat card classes:

```css
  /* ── Bold-only hidden elements ── */
  [data-theme-preset="bold"] .bold-hide {
    display: none !important;
  }

  /* ── Sidebar item icon (14px square) ── */
  .sidebar-item-icon {
    display: none;
    width: 14px;
    height: 14px;
    border: 1.5px solid currentColor;
    flex-shrink: 0;
  }
  [data-theme-preset="bold"] .sidebar-item-icon {
    display: block;
  }

  /* ── Header icon buttons: no border-radius in Bold ── */
  [data-theme-preset="bold"] .header-icon-btn {
    border-radius: 0 !important;
  }

  /* ── Header height: 52px in Bold (overrides Tailwind h-16 = 64px) ── */
  [data-theme-preset="bold"] #page-header {
    height: 52px !important;
  }

  /* ── Table date column: JetBrains Mono ── */
  [data-theme-preset="bold"] table tbody td.col-date {
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
  }
```

- [ ] **Step 4: Build and verify CSS compiles**

```bash
bun run build:admin
```

Expected: no errors, `web/assets/admin/css/admin.css` updated.

- [ ] **Step 5: Commit**

```bash
git add themes/admin/assets/css/base/base.css
git commit -m "feat(bold-theme): consolidate stat selectors, add utility classes"
```

---

## Task 2: Sidebar — Brand icon and nav item squares

**Files:**
- Modify: `themes/admin/templates/partials/_sidebar.twig`

- [ ] **Step 1: Replace the brand logo icon**

First, add these two CSS rules to `base.css` (inside the Bold `@layer components` block, after the utility classes added in Task 1):

```css
  /* Brand letter C: only visible in Bold */
  .bold-brand-letter { display: none; }
  [data-theme-preset="bold"] .bold-brand-letter { display: inline; }

  /* Sidebar brand icon: no radius in Bold */
  [data-theme-preset="bold"] .sidebar-brand-icon { border-radius: 0; }
```

Then find this block in `_sidebar.twig` (lines 23–30):
```twig
<a href="/admin" class="group inline-flex items-center gap-2 font-bold overflow-hidden flex-shrink-0"
   style="color: var(--bc);"
   x-bind:class="sidebarCollapsed ? 'justify-center' : ''">
  <span class="flex items-center justify-center w-8 h-8 rounded-lg bg-primary text-primary-content flex-shrink-0">
    {{ icon('layers', 'size-5') }}
  </span>
  <span x-show="!sidebarCollapsed" x-transition class="truncate whitespace-nowrap">{{ site_name() }}</span>
</a>
```

Replace with:
```twig
<a href="/admin" class="group inline-flex items-center gap-2 font-bold overflow-hidden flex-shrink-0"
   style="color: var(--bc);"
   x-bind:class="sidebarCollapsed ? 'justify-center' : ''">
  <span class="sidebar-brand-icon flex items-center justify-center w-8 h-8 rounded-lg bg-primary text-primary-content flex-shrink-0"
        style="font-size: 12px; font-weight: 800; font-family: var(--font-display, 'Inter Tight', sans-serif);">
    <span class="bold-hide">{{ icon('layers', 'size-5') }}</span>
    <span class="bold-brand-letter">C</span>
  </span>
  <span x-show="!sidebarCollapsed" x-transition class="truncate whitespace-nowrap">{{ site_name() }}</span>
</a>
```

In Bold: `.bold-hide` hides the layers icon, `.bold-brand-letter` shows the `C`. In other themes: icon visible, `C` hidden.

- [ ] **Step 2: Update the Dashboard nav link (outside the loop)**

Find the Dashboard `<a>` (around line 59–67). It currently looks like:
```twig
<a href="/admin"
   class="flex items-center gap-3 rounded-lg px-2 py-2 text-sm font-medium transition-colors relative"
   style="{{ dash_active ? 'color: var(--sidebar-text-active); background-color: color-mix(in srgb, var(--p) 10%, transparent);' : 'color: var(--sidebar-text);' }}"
   x-bind:class="sidebarCollapsed ? 'justify-center' : ''"
   x-bind:title="sidebarCollapsed ? 'Dashboard' : ''">
  {{ icon('home', 'size-4 flex-shrink-0') }}
  <span x-show="!sidebarCollapsed" x-transition class="truncate">Dashboard</span>
  {% if dash_active %}<span class="absolute left-0 top-1/2 -translate-y-1/2 w-1 h-6 rounded-r-full bg-primary"></span>{% endif %}
</a>
```

Replace with:
```twig
<a href="/admin"
   class="nav-item flex items-center gap-3 rounded-lg px-2 py-2 text-sm font-medium transition-colors relative {{ dash_active ? 'active' : '' }}"
   style="{{ dash_active ? 'color: var(--sidebar-text-active); background-color: color-mix(in srgb, var(--p) 10%, transparent);' : 'color: var(--sidebar-text);' }}"
   x-bind:class="sidebarCollapsed ? 'justify-center' : ''"
   x-bind:title="sidebarCollapsed ? 'Dashboard' : ''">
  <span class="sidebar-item-icon"></span>
  {{ icon('home', 'size-4 flex-shrink-0 bold-hide') }}
  <span x-show="!sidebarCollapsed" x-transition class="truncate">Dashboard</span>
  {% if dash_active %}
    <span class="nav-item-bar absolute left-0 top-1/2 -translate-y-1/2"></span>
    <span class="bold-hide absolute left-0 top-1/2 -translate-y-1/2 w-1 h-6 rounded-r-full bg-primary"></span>
  {% endif %}
</a>
```

- `nav-item` + `active` classes trigger Bold CSS selectors
- `sidebar-item-icon` span: hidden in non-Bold, visible as 14px square in Bold
- Lucide icon gets `.bold-hide`: hidden in Bold, visible in other themes
- `nav-item-bar` span: Bold's 2px × 20px left bar (CSS already present)
- `bold-hide` rounded pill span: only shown in non-Bold (provides the existing rounded indicator)

- [ ] **Step 3: Update loop nav items (Content section, lines ~85–99)**

Apply the same pattern to the `{% for item in [...] %}` loop `<a>` tag:

```twig
<a href="{{ item.href }}"
   class="nav-item flex items-center gap-3 rounded-lg px-2 py-2 text-sm font-medium transition-colors relative {{ item.active ? 'active' : '' }}"
   style="{{ item.active ? 'color: var(--sidebar-text-active); background-color: color-mix(in srgb, var(--p) 10%, transparent);' : 'color: var(--sidebar-text);' }}"
   x-bind:class="sidebarCollapsed ? 'justify-center' : ''"
   x-bind:title="sidebarCollapsed ? '{{ item.label }}' : ''">
  <span class="sidebar-item-icon"></span>
  {{ icon(item.icon, 'size-4 flex-shrink-0 bold-hide') }}
  <span x-show="!sidebarCollapsed" x-transition class="truncate">{{ item.label }}</span>
  {% if item.active %}
    <span class="nav-item-bar absolute left-0 top-1/2 -translate-y-1/2"></span>
    <span class="bold-hide absolute left-0 top-1/2 -translate-y-1/2 w-1 h-6 rounded-r-full bg-primary"></span>
  {% endif %}
</a>
```

Do the same for the System section loop (lines ~119–131) — it's identical structure.

- [ ] **Step 4: Update section labels**

Find the Content section `<button>` (lines ~71–83):
```twig
<button x-show="!sidebarCollapsed" x-transition
        x-on:click="open.content = !open.content"
        type="button"
        class="flex items-center justify-between w-full px-2 pb-1 pt-3 text-[10px] font-semibold uppercase tracking-wider transition-colors hover:text-[var(--bc)]"
        style="color: var(--bc-muted);">
  <span class="flex items-center gap-1.5">
    <span class="-ml-1">{{ icon('minus', 'size-4') }}</span>
    <span>Content</span>
  </span>
  <span x-bind:class="open.content ? '' : '-rotate-90'" class="transition-transform duration-200">
    {{ icon('chevron-down', 'size-3') }}
  </span>
</button>
```

Add `nav-section-label` class and `.bold-hide` to the icons (leave all Alpine attributes untouched):
```twig
<button x-show="!sidebarCollapsed" x-transition
        x-on:click="open.content = !open.content"
        type="button"
        class="nav-section-label flex items-center justify-between w-full px-2 pb-1 pt-3 text-[10px] font-semibold uppercase tracking-wider transition-colors hover:text-[var(--bc)]"
        style="color: var(--bc-muted);">
  <span class="flex items-center gap-1.5">
    <span class="-ml-1 bold-hide">{{ icon('minus', 'size-4') }}</span>
    <span>Content</span>
  </span>
  <span x-bind:class="open.content ? '' : '-rotate-90'" class="transition-transform duration-200 bold-hide">
    {{ icon('chevron-down', 'size-3') }}
  </span>
</button>
```

Apply the same change to the System section `<button>` (lines ~105–116), changing `open.content` to `open.system` and label to `System`.

- [ ] **Step 5: Build and visually verify sidebar**

```bash
bun run build:admin
```

Open browser at `http://localhost:8000/admin`, open DevTools console and run:
```js
localStorage.setItem('calaca-admin-theme-preset', 'bold')
location.reload()
```

Check:
- [ ] Brand: lime `C` box visible, layers icon hidden
- [ ] Nav items: 14px square boxes visible, Lucide icons hidden
- [ ] Dashboard active: lime left bar (2px, no radius), lime text, lime background tint
- [ ] Section labels: JetBrains Mono, tiny uppercase, no chevron/minus in Bold
- [ ] Other themes unaffected (test with `localStorage.setItem('calaca-admin-theme-preset', 'default')`)

- [ ] **Step 6: Commit**

```bash
git add themes/admin/templates/partials/_sidebar.twig themes/admin/assets/css/base/base.css
git commit -m "feat(bold-theme): sidebar brand icon, nav squares, section labels"
```

---

## Task 3: Header — Bold breadcrumb, search, and icon buttons

**Files:**
- Modify: `themes/admin/templates/partials/_header.twig`

- [ ] **Step 1: Add `header-icon-btn` class to bell and theme toggle buttons**

Find the notifications button (line ~90–94):
```twig
<button type="button" class="relative p-2 rounded-lg hover:bg-[var(--b3)] transition-colors"
        style="color: var(--bc-muted);">
```
Add `header-icon-btn` to the class list.

Find the theme toggle button (line ~97–103):
```twig
<button x-on:click="toggleTheme()" type="button" aria-label="Toggle dark mode"
        class="p-2 rounded-lg hover:bg-[var(--b3)] transition-colors"
```
Add `header-icon-btn` to the class list.

- [ ] **Step 2: Wrap breadcrumb nav with x-show variants**

Find the breadcrumb `<nav>` block (lines ~44–60). Wrap it:

```twig
{# Bold breadcrumb: ADMIN · CURRENT PAGE #}
<div class="hidden sm:flex items-center gap-2"
     x-show="document.documentElement.getAttribute('data-theme-preset') === 'bold'"
     x-cloak
     style="font-family: var(--font-mono, 'JetBrains Mono', monospace); font-size: 11px; letter-spacing: 0.08em; text-transform: uppercase; color: var(--bc-muted);">
  Admin &middot;
  <span style="color: var(--bc);">{{ breadcrumbs|last.label|upper }}</span>
</div>

{# Default breadcrumb: icon + chevron trail #}
<nav class="hidden sm:flex items-center gap-2 text-sm min-w-0" aria-label="Breadcrumb"
     x-show="document.documentElement.getAttribute('data-theme-preset') !== 'bold'"
     x-cloak>
  {# ... existing breadcrumb loop untouched ... #}
</nav>
```

Note: `breadcrumbs|last.label` already exists in the template (used for mobile title on line ~64). `|upper` converts to uppercase for the Bold format.

Also wrap the mobile title span (line ~63–65) to show/hide between Bold and default:

```twig
{# Bold mobile: uppercase page name #}
<span class="sm:hidden font-medium truncate"
      x-show="document.documentElement.getAttribute('data-theme-preset') === 'bold'"
      x-cloak
      style="font-family: var(--font-mono); font-size: 11px; letter-spacing: 0.08em; text-transform: uppercase; color: var(--bc);">
  {{ breadcrumbs|last.label|upper }}
</span>

{# Default mobile: normal page name #}
<span class="sm:hidden font-medium truncate"
      x-show="document.documentElement.getAttribute('data-theme-preset') !== 'bold'"
      x-cloak
      style="color: var(--bc);">
  {{ breadcrumbs|last.label }}
</span>
```

- [ ] **Step 3: Wrap search input with Bold/default variants**

Find the search `<div>` (lines ~71–80). Wrap it:

```twig
{# Bold search box #}
<div class="hidden md:flex header-search items-center"
     x-show="document.documentElement.getAttribute('data-theme-preset') === 'bold'"
     x-cloak
     style="background: var(--b3); border: 1px solid var(--border); border-radius: 0; padding: 6px 12px; font-size: 12px; color: var(--bc-muted); font-family: var(--font-mono, 'JetBrains Mono', monospace); gap: 8px; width: 180px;">
  ⌘ Search...
</div>

{# Default search input #}
<div class="hidden md:flex items-center relative"
     x-show="document.documentElement.getAttribute('data-theme-preset') !== 'bold'"
     x-cloak>
  {# ... existing search input untouched ... #}
</div>
```

- [ ] **Step 4: Build and visually verify header**

```bash
bun run build:admin
```

Check in browser (Bold preset active):
- [ ] Breadcrumb shows `ADMIN · DASHBOARD` in JetBrains Mono uppercase
- [ ] Search box is flat, dark, `⌘ Search...` placeholder, 180px wide
- [ ] Bell and theme toggle have no border-radius
- [ ] Header height is 52px

- [ ] **Step 5: Commit**

```bash
git add themes/admin/templates/partials/_header.twig
git commit -m "feat(bold-theme): bold breadcrumb, flat search, header icon styling"
```

---

## Task 4: Dashboard — Stat card classes

**Files:**
- Modify: `themes/admin/templates/pages/dashboard/dashboard.twig`

- [ ] **Step 1: Update stat number `<dt>` elements**

There are 4 stat cards. For each, find the `<dt>` with `x-text` and add `stat-number` class:

**Before (all 4 instances, same pattern):**
```twig
<dt x-show="!loading" x-text="stats.total_articles || 0" class="text-2xl font-extrabold"></dt>
```

**After:**
```twig
<dt x-show="!loading" x-text="stats.total_articles || 0" class="stat-number text-2xl font-extrabold"></dt>
```

Do the same for `total_pages`, `total_media`, `total_users`.

- [ ] **Step 2: Update stat label `<dd>` elements**

Find the label `<dd>` (text like "Total Articles") and add `stat-label` class:

**Before:**
```twig
<dd x-show="!loading" class="text-xs font-medium text-base-content/60">Total Articles</dd>
```

**After:**
```twig
<dd x-show="!loading" class="stat-label text-xs font-medium text-base-content/60">Total Articles</dd>
```

Do same for Pages, Media Files, Users labels.

- [ ] **Step 3: Update sub-text `<div>` elements**

Find each sub-text div (e.g., "0 published · 1 draft") and add `stat-sub` class:

**Before:**
```twig
<div x-show="!loading" class="mt-2 text-xs font-medium text-base-content/60">
```

**After:**
```twig
<div x-show="!loading" class="stat-sub mt-2 text-xs font-medium text-base-content/60">
```

Do for all 4 cards (articles, pages, media, users).

- [ ] **Step 4: Build and visually verify stat cards**

```bash
bun run build:admin
```

Check (Bold preset):
- [ ] Numbers are 48px Inter Tight, weight 800
- [ ] Labels are 10px JetBrains Mono uppercase
- [ ] Sub-text has top border separator, 12px muted text
- [ ] Icons still visible (not removed)

- [ ] **Step 5: Commit**

```bash
git add themes/admin/templates/pages/dashboard/dashboard.twig
git commit -m "feat(bold-theme): stat card number/label/sub classes"
```

---

## Task 5: Dashboard — Lower section Bold variant

**Files:**
- Modify: `themes/admin/templates/pages/dashboard/dashboard.twig`

- [ ] **Step 1: Wrap existing lower section**

Find the `<!-- Content Sections -->` div (line ~127):
```twig
<!-- Content Sections -->
<div class="grid grid-cols-1 gap-4 lg:grid-cols-2">
```

Wrap it with an `x-show` guard:
```twig
<!-- Content Sections — default themes -->
<div class="grid grid-cols-1 gap-4 lg:grid-cols-2"
     x-show="document.documentElement.getAttribute('data-theme-preset') !== 'bold'"
     x-cloak>
  {# ... existing content untouched ... #}
</div>
```

- [ ] **Step 2: Add Bold lower section after the existing one**

After the closing `</div>` of the existing lower section, add:

```twig
<!-- Content Sections — Bold theme -->
<div x-show="document.documentElement.getAttribute('data-theme-preset') === 'bold'"
     x-cloak
     class="grid grid-cols-1 lg:grid-cols-3"
     style="gap: 2px;">

  <!-- Quick Actions card -->
  <div style="background: var(--card); border: 1px solid var(--border); padding: 24px;">
    <div style="width: 32px; height: 2px; background: var(--p); margin-bottom: 16px;"></div>
    <div style="font-family: var(--font-display, 'Inter Tight', sans-serif); font-size: 18px; font-weight: 700; letter-spacing: -0.03em; color: var(--bc);">Quick Actions</div>
    <div style="font-size: 13px; color: var(--bc-muted); margin-top: 6px; line-height: 1.6;">Common tasks to get you started quickly.</div>
    <div style="margin-top: 16px; display: flex; flex-direction: column; gap: 8px;">
      <a href="{{ url('admin.page_create') }}"
         style="font-size: 13px; color: var(--bc-secondary); text-decoration: none; display: block; transition: color 0.15s;"
         onmouseover="this.style.color='var(--bc)'"
         onmouseout="this.style.color='var(--bc-secondary)'">New Page</a>
      <a href="{{ url('admin.media') }}"
         style="font-size: 13px; color: var(--bc-secondary); text-decoration: none; display: block; border-bottom: 1px solid currentColor; padding-bottom: 1px; transition: color 0.15s;"
         onmouseover="this.style.color='var(--bc)'"
         onmouseout="this.style.color='var(--bc-secondary)'">Upload Media</a>
      <a href="{{ url('admin.users') }}"
         style="font-size: 13px; color: var(--bc-secondary); text-decoration: none; display: block; transition: color 0.15s;"
         onmouseover="this.style.color='var(--bc)'"
         onmouseout="this.style.color='var(--bc-secondary)'">Manage Users</a>
    </div>
  </div>

  <!-- System Info card -->
  <div style="background: var(--card); border: 1px solid var(--border); padding: 24px;">
    <div style="width: 32px; height: 2px; background: var(--p); margin-bottom: 16px;"></div>
    <div style="font-family: var(--font-display, 'Inter Tight', sans-serif); font-size: 18px; font-weight: 700; letter-spacing: -0.03em; color: var(--bc);">System Info</div>
    <div style="font-size: 13px; color: var(--bc-muted); margin-top: 6px; line-height: 1.6;">Server and version details.</div>
    <div style="margin-top: 16px;">
      <div style="display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid var(--border);">
        <span style="font-family: var(--font-mono, 'JetBrains Mono', monospace); font-size: 10px; letter-spacing: 0.08em; text-transform: uppercase; color: var(--bc-secondary);">Version</span>
        <span style="color: var(--bc); font-size: 12px;">{{ cms_version|default('1.0.0') }}</span>
      </div>
      <div style="display: flex; justify-content: space-between; padding: 8px 0;">
        <span style="font-family: var(--font-mono, 'JetBrains Mono', monospace); font-size: 10px; letter-spacing: 0.08em; text-transform: uppercase; color: var(--bc-secondary);">PHP</span>
        <span style="color: var(--bc); font-size: 12px;">{{ php_version|default(constant('PHP_VERSION')) }}</span>
      </div>
    </div>
  </div>

  <!-- Featured card (Bold only) -->
  <div style="background: var(--card); border: 2px solid var(--p); padding: 24px;">
    <div style="font-family: var(--font-mono, 'JetBrains Mono', monospace); font-size: 9px; letter-spacing: 0.15em; text-transform: uppercase; color: var(--pc); background: var(--p); display: inline-block; padding: 3px 8px; margin-bottom: 14px;">Featured</div>
    <div style="font-family: var(--font-display, 'Inter Tight', sans-serif); font-size: 18px; font-weight: 700; letter-spacing: -0.03em; color: var(--bc);">Bold Theme Active</div>
    <div style="font-size: 13px; color: var(--bc-muted); margin-top: 6px; line-height: 1.6;">You are viewing the Bold Typography theme.</div>
  </div>

</div>
```

- [ ] **Step 3: Build and visually verify lower section**

```bash
bun run build:admin
```

Check (Bold preset):
- [ ] 3 cards in a row with 2px gap
- [ ] Lime accent bar (32×2px) at top of Quick Actions and System Info cards
- [ ] Featured card has 2px lime border and FEATURED badge
- [ ] Quick Actions has 3 links (New Page, Upload Media, Manage Users)
- [ ] System Info shows version + PHP version
- [ ] Switch to non-Bold preset: existing Recent Articles + Quick Actions grid shows correctly

- [ ] **Step 4: Commit**

```bash
git add themes/admin/templates/pages/dashboard/dashboard.twig
git commit -m "feat(bold-theme): dual lower section (bold 3-card / default recent articles)"
```

---

## Task 6: Articles table — Badges and container

**Files:**
- Modify: `themes/admin/templates/pages/articles/index.twig`

- [ ] **Step 1: Wrap the table in a `table-container` div**

Find the `<div class="overflow-x-auto ...">` wrapping the `<table>`. Add `table-container` class to this wrapper div, or wrap the entire outer card div:

Find the outermost article-list container div and add `table-container` class:
```twig
<div class="table-container overflow-hidden rounded-xl border border-border/75 bg-base-100">
```

The Bold CSS for `.table-container` (`background: var(--b2); border: 1px solid var(--border); padding: 0`) will apply in Bold, the existing Tailwind classes provide the non-Bold appearance.

- [ ] **Step 2: Add `col-date` class to the date `<td>`**

Find the date column `<td>` (line ~232):
```twig
<td class="hidden sm:table-cell px-4 py-3 text-base-content/60" x-text="new Date(...)..."></td>
```

Add `col-date` class:
```twig
<td class="col-date hidden sm:table-cell px-4 py-3 text-base-content/60" x-text="new Date(...)..."></td>
```

- [ ] **Step 3: Update badge Alpine binding to support both themes**

Find the badge span (lines ~226–229):
```twig
<span class="inline-flex px-2 py-0.5 rounded-full text-xs font-medium"
      :class="article.status === 'published' ? 'bg-success/10 text-success' : (article.status === 'draft' ? 'bg-warning/10 text-warning' : 'bg-base-200 text-base-content/60')"
      x-text="article.status.charAt(0).toUpperCase() + article.status.slice(1)">
</span>
```

Replace with (adds `isBold` check to the x-data of the closest ancestor, or inline):

First, add `isBold` to the top-level `x-data` on the wrapping div (line ~8):
```twig
<div x-data="{ loading: true, articles: [], categories: [], error: null, isBold: document.documentElement.getAttribute('data-theme-preset') === 'bold' }"
```

Then update the badge span:
```twig
<span class="badge"
      :class="isBold
        ? { 'badge-published': article.status === 'published', 'badge-draft': article.status === 'draft', 'badge-error': article.status === 'error' }
        : { 'inline-flex px-2 py-0.5 rounded-full text-xs font-medium': true, 'bg-success/10 text-success': article.status === 'published', 'bg-warning/10 text-warning': article.status === 'draft', 'bg-base-200 text-base-content/60': !['published','draft'].includes(article.status) }"
      x-text="isBold ? article.status.toUpperCase() : article.status.charAt(0).toUpperCase() + article.status.slice(1)">
</span>
```

- [ ] **Step 4: Build and visually verify articles table**

```bash
bun run build:admin
```

Navigate to `/admin/articles` (Bold preset):
- [ ] Table headers: JetBrains Mono 10px uppercase, muted color
- [ ] Row titles: white text, weight 500
- [ ] Date column: JetBrains Mono 11px
- [ ] DRAFT badge: lime tint, lime text, sharp corners
- [ ] PUBLISHED badge: muted, sharp corners
- [ ] ERROR badge: red tint, red text, sharp corners
- [ ] Switch to non-Bold: colored badges (green/yellow) with rounded-full appear correctly

- [ ] **Step 5: Commit**

```bash
git add themes/admin/templates/pages/articles/index.twig
git commit -m "feat(bold-theme): table-container, col-date, theme-aware badge binding"
```

---

## Task 7: Final verification pass

- [ ] **Step 1: Build production assets**

```bash
bun run build:admin
```

- [ ] **Step 2: Full browser check — Bold dark**

Set preset to bold, theme to dark:
```js
localStorage.setItem('calaca-admin-theme-preset', 'bold')
localStorage.setItem('themeMode', 'dark')
location.reload()
```

Check all 6 sections against the reference screenshots:
- [ ] Sidebar: C box, square icons, active bar, section labels
- [ ] Header: `ADMIN · DASHBOARD` breadcrumb, flat search, sharp icon buttons, 52px height
- [ ] Dashboard stats: 48px numbers, mono labels, border sub-text
- [ ] Dashboard cards: 3-col, accent bars, Featured with lime border
- [ ] Articles table: all Bold styling
- [ ] Buttons: primary (underline), secondary (outline), ghost (no border), danger (red outline)
- [ ] Flash messages: left border style

- [ ] **Step 3: Full browser check — non-Bold (regression check)**

```js
localStorage.setItem('calaca-admin-theme-preset', 'default')
location.reload()
```

- [ ] Dashboard: Recent Articles + Quick Actions grid visible
- [ ] Sidebar: Lucide icons visible, rounded nav items
- [ ] Articles: colored rounded badges
- [ ] Header: normal breadcrumb with icons

- [ ] **Step 4: Full browser check — Bold light**

```js
localStorage.setItem('calaca-admin-theme-preset', 'bold')
localStorage.setItem('themeMode', 'light')
location.reload()
```

- [ ] Lime → olive green accent
- [ ] White/cream background
- [ ] All components match light variant

- [ ] **Step 5: Final commit**

```bash
git add -A
git commit -m "feat(bold-theme): complete 1:1 reference implementation verified"
```
