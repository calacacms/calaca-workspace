# Sprint 2 — Index Templates Semantic Colors

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Vervang alle hardcoded Tailwind kleuren (`slate-*`, `indigo-*`, `green-*`, `yellow-*`, `red-*`) in de drie resterende admin index templates met semantic CSS-variabele klassen.

**Architecture:** Zelfde mapping als Sprint 1. Alle drie templates bevatten identieke patronen en kunnen parallel aangepast worden.

**Tech Stack:** Twig 3.0, Tailwind CSS v4 (`@theme` semantic colors), AlpineJS 3.x

---

## Semantic Color Mapping Reference

| Oud (hardcoded) | Nieuw (semantic) |
|---|---|
| `bg-white` | `bg-base-100` |
| `bg-slate-50` | `bg-base-200/30` |
| `bg-slate-100` | `bg-base-200` |
| `border-slate-50` | `border-border/30` |
| `border-slate-100` | `border-border/50` |
| `border-slate-200` | `border-border` |
| `border-slate-300` | `border-border` |
| `text-slate-900` / `text-slate-800` | `text-base-content` |
| `text-slate-700` | `text-base-content/80` |
| `text-slate-600` | `text-base-content/70` |
| `text-slate-500` | `text-base-content/60` |
| `text-slate-400` | `text-base-content/50` |
| `hover:bg-slate-50` | `hover:bg-base-200/30` |
| `hover:bg-slate-100` | `hover:bg-base-200` |
| `hover:text-slate-600` | `hover:text-base-content/70` |
| `hover:text-slate-900` | `hover:text-base-content` |
| `bg-indigo-600 text-white hover:bg-indigo-700` | `bg-primary text-primary-content hover:bg-primary-focus` |
| `bg-indigo-500` (progress bar fill) | `bg-primary` |
| `hover:text-indigo-600` | `hover:text-primary` |
| `hover:border-indigo-200` | `hover:border-primary/30` |
| `hover:bg-indigo-50/50` | `hover:bg-primary/5` |
| `hover:border-indigo-300` | `hover:border-primary/50` |
| `focus:border-indigo-400` | `focus:border-primary` |
| `bg-green-50 text-green-700` | `bg-success/10 text-success` |
| `bg-yellow-50 text-yellow-700` | `bg-warning/10 text-warning` |
| `bg-slate-100 text-slate-500` (badge) | `bg-base-200 text-base-content/60` |
| `bg-slate-100 text-slate-600` (badge) | `bg-base-200 text-base-content/70` |
| `hover:text-red-600 hover:bg-red-50` | `hover:text-error hover:bg-error/10` |
| `divide-slate-100` | `divide-border/50` |

---

## Task 1: articles/index.twig

**Files:**
- Modify: `calaca/resources/themes/admin/templates/pages/articles/index.twig`

- [ ] **Stap 1: Fix "New Article" button + search bar**

```twig
{# VOOR #}
class="...bg-indigo-600 text-white...hover:bg-indigo-700..."
class="...border-slate-200...focus:border-indigo-400..."
class="...text-slate-400 hover:text-slate-600..."

{# NA #}
class="inline-flex items-center gap-2 px-4 py-2 bg-primary text-primary-content text-sm font-medium rounded-lg hover:bg-primary-focus transition-colors"
class="px-3 py-2 text-sm border border-border rounded-lg bg-base-100 text-base-content focus:outline-none focus:border-primary transition-colors"
class="p-2 text-base-content/50 hover:text-base-content/70 transition-colors"
```

- [ ] **Stap 2: Fix filter selects**

```twig
{# VOOR #}
class="text-sm border border-slate-200 rounded-lg px-3 py-2 focus:outline-none focus:border-indigo-400 transition-colors"

{# NA #}
class="text-sm border border-border rounded-lg px-3 py-2 bg-base-100 text-base-content focus:outline-none focus:border-primary transition-colors"
```

- [ ] **Stap 3: Fix table container + header**

```twig
{# VOOR #}
class="bg-white rounded-xl border border-slate-100 shadow-sm overflow-hidden"
class="border-b border-slate-100"
class="rounded border-slate-300"
class="...text-slate-500 uppercase..."

{# NA #}
class="bg-base-100 rounded-xl border border-border/50 shadow-sm overflow-hidden"
class="border-b border-border/50"
class="rounded border-border"
class="...text-base-content/50 uppercase..."
```

- [ ] **Stap 4: Fix table rows + status badges**

```twig
{# VOOR #}
class="border-b border-slate-50 hover:bg-slate-50 transition-colors"
class="rounded border-slate-300"
class="font-medium text-slate-900 hover:text-indigo-600 transition-colors"

{# Status badges (Alpine :class) #}
'bg-green-50 text-green-700' : (article.status === 'draft' ? 'bg-yellow-50 text-yellow-700' : 'bg-slate-100 text-slate-500')

class="...text-slate-600..."
class="...text-slate-500..."

{# NA #}
class="border-b border-border/30 hover:bg-base-200/30 transition-colors"
class="rounded border-border"
class="font-medium text-base-content hover:text-primary transition-colors"

{# Status badges #}
'bg-success/10 text-success' : (article.status === 'draft' ? 'bg-warning/10 text-warning' : 'bg-base-200 text-base-content/60')

class="...text-base-content/70..."
class="...text-base-content/60..."
```

- [ ] **Stap 5: Fix action buttons + empty state**

```twig
{# VOOR #}
class="p-1.5 text-slate-400 hover:text-slate-600 hover:bg-slate-100 rounded-lg transition-colors"
class="p-1.5 text-slate-400 hover:text-red-600 hover:bg-red-50 rounded-lg transition-colors"
class="px-4 py-3 text-center text-slate-400"
class="...bg-indigo-600 text-white...hover:bg-indigo-700..."

{# NA #}
class="p-1.5 text-base-content/50 hover:text-base-content/70 hover:bg-base-200 rounded-lg transition-colors"
class="p-1.5 text-base-content/50 hover:text-error hover:bg-error/10 rounded-lg transition-colors"
class="px-4 py-3 text-center text-base-content/50"
class="inline-flex items-center gap-2 px-4 py-2 bg-primary text-primary-content text-sm font-medium rounded-lg hover:bg-primary-focus transition-colors"
```

- [ ] **Stap 6: Verificeer geen hardcoded kleuren meer**

```bash
grep -n "slate-\|indigo-\|green-\|yellow-\|red-" calaca/resources/themes/admin/templates/pages/articles/index.twig
# Verwacht: geen output
```

---

## Task 2: pages/index.twig

**Files:**
- Modify: `calaca/resources/themes/admin/templates/pages/index.twig`

- [ ] **Stap 1: Fix "New Page" button + search bar + filters**

Zelfde patroon als Task 1 Stap 1-2.

- [ ] **Stap 2: Fix table container, header, rows**

Zelfde patroon als Task 1 Stap 3-4.

Aanvullend voor `<code>` element (slug):
```twig
{# VOOR #}
class="text-xs font-mono bg-slate-50 px-2 py-1 rounded text-slate-600"
{# NA #}
class="text-xs font-mono bg-base-200/30 px-2 py-1 rounded text-base-content/70"
```

Status badges (Twig ternary):
```twig
{# VOOR #}
{{ page.status == 'published' ? 'bg-green-50 text-green-700' : (page.status == 'draft' ? 'bg-yellow-50 text-yellow-700' : 'bg-slate-100 text-slate-500') }}
{# NA #}
{{ page.status == 'published' ? 'bg-success/10 text-success' : (page.status == 'draft' ? 'bg-warning/10 text-warning' : 'bg-base-200 text-base-content/60') }}
```

- [ ] **Stap 3: Fix action buttons + empty state + pagination**

```twig
{# Pagination VOOR #}
class="flex items-center justify-between px-4 py-3 border-t border-slate-100 mt-4"
class="...border border-slate-200...text-slate-600 hover:bg-slate-50..."
class="text-sm text-slate-500"

{# Pagination NA #}
class="flex items-center justify-between px-4 py-3 border-t border-border/50 mt-4"
class="px-4 py-2 text-sm border border-border text-base-content/70 rounded-lg hover:bg-base-200/30 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
class="text-sm text-base-content/60"
```

- [ ] **Stap 4: Verificeer geen hardcoded kleuren meer**

```bash
grep -n "slate-\|indigo-\|green-\|yellow-\|red-" calaca/resources/themes/admin/templates/pages/index.twig
# Verwacht: geen output
```

---

## Task 3: media/index.twig

**Files:**
- Modify: `calaca/resources/themes/admin/templates/pages/media/index.twig`

- [ ] **Stap 1: Fix toolbar buttons**

```twig
{# Upload button VOOR #}
class="...bg-indigo-600 text-white...hover:bg-indigo-700..."
{# NA #}
class="inline-flex items-center gap-2 px-4 py-2 bg-primary text-primary-content text-sm font-medium rounded-lg hover:bg-primary-focus transition-colors"

{# New Folder button VOOR #}
class="...border border-slate-200 text-slate-700...hover:bg-slate-50..."
{# NA #}
class="inline-flex items-center gap-2 px-4 py-2 border border-border text-base-content/80 text-sm font-medium rounded-lg hover:bg-base-200/30 transition-colors"
```

- [ ] **Stap 2: Fix view toggle + filter selects + search**

```twig
{# View toggle VOOR #}
class="flex rounded-lg border border-slate-200 overflow-hidden"
:class="view === 'grid' ? 'bg-slate-100 text-slate-900' : 'text-slate-400 hover:text-slate-600'"
class="...border-l border-slate-200..."
{# NA #}
class="flex rounded-lg border border-border overflow-hidden"
:class="view === 'grid' ? 'bg-base-200 text-base-content' : 'text-base-content/50 hover:text-base-content/70'"
class="...border-l border-border..."

{# Filter selects: zelfde als Task 1 Stap 2 #}
```

- [ ] **Stap 3: Fix breadcrumbs**

```twig
{# VOOR #}
class="flex items-center gap-1 text-sm text-slate-500"
class="hover:text-indigo-600 transition-colors"
{# NA #}
class="flex items-center gap-1 text-sm text-base-content/60"
class="hover:text-primary transition-colors"
```

- [ ] **Stap 4: Fix grid view cards (folders + files)**

```twig
{# Folder card VOOR #}
class="...bg-white...border-slate-100...hover:border-indigo-200..."
class="text-xs font-medium text-slate-700..."
class="text-xs text-slate-400"
{# NA #}
class="group relative bg-base-100 rounded-xl border border-border/50 p-4 cursor-pointer hover:border-primary/30 hover:shadow-sm transition-all flex flex-col items-center gap-2"
class="text-xs font-medium text-base-content/80 text-center truncate w-full"
class="text-xs text-base-content/50"

{# File card VOOR #}
class="...bg-white...border-slate-100...hover:border-indigo-200..."
class="rounded border-slate-300"
class="aspect-square bg-slate-50 flex..."
class="text-xs font-medium text-slate-700 truncate"
class="text-xs text-slate-400"
{# NA #}
class="group relative bg-base-100 rounded-xl border border-border/50 overflow-hidden cursor-pointer hover:border-primary/30 hover:shadow-sm transition-all"
class="rounded border-border"
class="aspect-square bg-base-200/30 flex..."
class="text-xs font-medium text-base-content/80 truncate"
class="text-xs text-base-content/50"
```

- [ ] **Stap 5: Fix list view table**

```twig
{# Container VOOR #}
class="bg-white rounded-xl border border-slate-100 shadow-sm overflow-hidden"
{# NA #}
class="bg-base-100 rounded-xl border border-border/50 shadow-sm overflow-hidden"

{# Header/rows: zelfde patroon als Task 1 Stap 3-5 #}
{# Type badge VOOR #}
class="...bg-slate-100 text-slate-600..."
{# NA #}
class="inline-flex px-2 py-0.5 rounded-full text-xs font-medium bg-base-200 text-base-content/70"
```

- [ ] **Stap 6: Fix sidebar panels (Selected Items + Storage + Quick Upload)**

```twig
{# Panel container VOOR #}
class="bg-white rounded-xl border border-slate-100 shadow-sm p-5"
{# NA #}
class="bg-base-100 rounded-xl border border-border/50 shadow-sm p-5"

{# Panel heading VOOR #}
class="text-sm font-semibold text-slate-900 mb-3"
{# NA #}
class="text-sm font-semibold text-base-content mb-3"

{# Storage bar VOOR #}
class="h-2 bg-slate-100 rounded-full overflow-hidden"
class="h-full bg-indigo-500 rounded-full"
{# NA #}
class="h-2 bg-base-200 rounded-full overflow-hidden"
class="h-full bg-primary rounded-full"

{# Quick upload drop zone VOOR #}
class="...border-dashed border-slate-200...hover:border-indigo-300 hover:bg-indigo-50/50..."
class="text-xs font-medium text-slate-600"
class="text-xs text-slate-400"
{# NA #}
class="flex flex-col items-center gap-2 p-4 border-2 border-dashed border-border rounded-xl text-center cursor-pointer hover:border-primary/50 hover:bg-primary/5 transition-all"
class="text-xs font-medium text-base-content/70"
class="text-xs text-base-content/50"

{# Sidebar buttons VOOR #}
class="...border border-slate-200 text-slate-600...hover:bg-slate-50..."
{# NA #}
class="px-3 py-1.5 text-xs font-medium border border-border text-base-content/70 rounded-lg hover:bg-base-200/30 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
```

- [ ] **Stap 7: Fix upload modal**

```twig
{# Modal container VOOR #}
class="relative bg-white rounded-2xl shadow-2xl w-full max-w-lg"
class="...border-b border-slate-100"
class="text-base font-semibold text-slate-900"
class="...text-slate-400 hover:text-slate-600 hover:bg-slate-100..."

{# Drop zone in modal VOOR #}
class="...border-dashed border-slate-200...hover:border-indigo-300 hover:bg-indigo-50/50..."
class="text-sm font-medium text-slate-600"
class="text-xs text-slate-400"

{# Progress bar VOOR #}
class="h-2 bg-slate-100 rounded-full overflow-hidden"
class="h-full bg-indigo-500 rounded-full transition-all"
class="text-xs text-slate-500 text-center"

{# Upload folder select VOOR #}
class="block text-sm font-medium text-slate-700 mb-1.5"
class="...border border-slate-200...focus:border-indigo-400..."

{# Cancel button VOOR #}
class="...border border-slate-200 text-slate-700...hover:bg-slate-50..."

{# NA — zelfde semantic mapping als vorige stappen #}
```

- [ ] **Stap 8: Verificeer geen hardcoded kleuren meer**

```bash
grep -n "slate-\|indigo-\|green-\|yellow-\|red-" calaca/resources/themes/admin/templates/pages/media/index.twig
# Verwacht: geen output
```

---

## Verificatie Checklist (na alles)

- [ ] `grep -r "slate-\|indigo-\|green-[0-9]\|yellow-[0-9]\|red-[0-9]" calaca/resources/themes/admin/templates --include="*.twig"` → leeg
- [ ] `grep -r "dark:" calaca/resources/themes/admin/templates --include="*.twig"` → leeg
- [ ] Alle 3 index templates tonen correcte kleuren in zowel light als dark mode
