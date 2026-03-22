# Bold Theme — 1:1 Perfect Replication

**Date:** 2026-03-21
**Reference:** `.superpowers/brainstorm/61931-1774023913/components.html`
**Approach:** Option B — Twig templates aanpassen + minimale CSS aanvulling

---

## Scope & Constraints

- De Bold theme tokens (`themes/index.css`) zijn correct en blijven ongewijzigd (`--bc-muted: #737373` is bevestigd aanwezig)
- Bestaande Bold CSS overrides in `base.css` worden uitgebreid/gecorrigeerd; bestaande `[data-stat-number]` data-attribute selectors worden **vervangen** door `.stat-number` class selectors (één consistent patroon)
- Sidebar behoudt collapse-toggle en user-footer functionaliteit
- Dashboard stat cards behouden de icon boxes
- Andere themes (light, dark, linear, monochrome) worden niet geraakt
- **Theme preset is client-side only** (localStorage `calaca-admin-theme-preset`): er is geen server-side PHP variabele. Alle Bold-conditional UI (Featured card, breadcrumb format, etc.) wordt gestuurd via Alpine.js `x-show` die `document.documentElement.getAttribute('data-theme-preset') === 'bold'` checkt — niet via Twig `{% if %}`

---

## Bestanden die wijzigen

| Bestand | Type wijziging |
|---------|---------------|
| `templates/partials/_sidebar.twig` | Brand icon, nav item icons, section labels |
| `templates/partials/_header.twig` | Breadcrumb markup, search box, icon buttons |
| `templates/pages/dashboard/dashboard.twig` | Stat card classes, lower section vervangen |
| `templates/pages/articles/index.twig` | Badge binding + container wrapper |
| `assets/css/base/base.css` | Bold overrides aanvullen/consolideren |

**Niet gewijzigd:** `themes/index.css`, PHP/backend code, andere page templates.

---

## Sectie 1 — Sidebar (`_sidebar.twig`)

### 1.1 Brand logo

**Voor:**
```twig
<span class="flex items-center justify-center w-8 h-8 rounded-lg bg-primary text-primary-content flex-shrink-0">
  {{ icon('layers', 'size-5') }}
</span>
```

**Na:**
```twig
<span class="sidebar-brand-icon flex items-center justify-center w-8 h-8 flex-shrink-0"
      style="background: var(--p); color: var(--pc); font-size: 12px; font-weight: 800; font-family: var(--font-display, 'Inter Tight', sans-serif);">
  C
</span>
```

CSS voor `.sidebar-brand-icon` onder `[data-theme-preset="bold"]`: `border-radius: 0; width: 28px; height: 28px;`
Voor andere themes: de `rounded-lg bg-primary` classes op de span zelf zorgen voor het normale uiterlijk — de Bold CSS overschrijft alleen `border-radius`.

### 1.2 Nav item icons

Canonical aanpak: `<a>` tags **behouden**, Bold-specifieke klassen worden **toegevoegd naast** bestaande Tailwind classes. De bestaande `aside nav a` CSS selectors in `base.css` worden **verwijderd** en vervangen door selectors op de expliciete classes.

De sidebar heeft **drie aparte nav-item contexten** die elk dezelfde behandeling krijgen:

#### A) Dashboard link (buiten loop, lijnen 59–67 in `_sidebar.twig`)
**Voor:**
```twig
<a href="/admin"
   class="flex items-center gap-3 rounded-lg px-2 py-2 text-sm font-medium transition-colors relative"
   style="{{ dash_active ? 'color: var(--sidebar-text-active); background-color: color-mix(in srgb, var(--p) 10%, transparent);' : 'color: var(--sidebar-text);' }}"
   ...>
  {{ icon('home', 'size-4 flex-shrink-0') }}
  <span ...>Dashboard</span>
  {% if dash_active %}<span class="absolute left-0 top-1/2 -translate-y-1/2 w-1 h-6 rounded-r-full bg-primary"></span>{% endif %}
</a>
```

**Na:**
```twig
<a href="/admin"
   class="nav-item flex items-center gap-3 rounded-lg px-2 py-2 text-sm font-medium transition-colors relative {{ dash_active ? 'active' : '' }}"
   style="{{ dash_active ? 'color: var(--sidebar-text-active); background-color: color-mix(in srgb, var(--p) 10%, transparent);' : 'color: var(--sidebar-text);' }}"
   ...>
  <span class="sidebar-item-icon flex-shrink-0"></span>
  {{ icon('home', 'size-4 flex-shrink-0 bold-hide') }}
  <span ...>Dashboard</span>
  {% if dash_active %}<span class="nav-item-bar absolute left-0 top-1/2 -translate-y-1/2"></span>{% endif %}
</a>
```

#### B) Content & System loop items (twee `{% for %}` loops)
**Voor:**
```twig
<a href="{{ item.href }}"
   class="flex items-center gap-3 rounded-lg px-2 py-2 text-sm font-medium transition-colors relative"
   style="{{ item.active ? 'color: var(--sidebar-text-active); background-color: color-mix(in srgb, var(--p) 10%, transparent);' : 'color: var(--sidebar-text);' }}"
   ...>
  {{ icon(item.icon, 'size-4 flex-shrink-0') }}
  <span ...>{{ item.label }}</span>
  {% if item.active %}<span class="absolute left-0 top-1/2 -translate-y-1/2 w-1 h-6 rounded-r-full bg-primary"></span>{% endif %}
</a>
```

**Na (zelfde patroon als Dashboard link):**
```twig
<a href="{{ item.href }}"
   class="nav-item flex items-center gap-3 rounded-lg px-2 py-2 text-sm font-medium transition-colors relative {{ item.active ? 'active' : '' }}"
   style="{{ item.active ? 'color: var(--sidebar-text-active); background-color: color-mix(in srgb, var(--p) 10%, transparent);' : 'color: var(--sidebar-text);' }}"
   ...>
  <span class="sidebar-item-icon flex-shrink-0"></span>
  {{ icon(item.icon, 'size-4 flex-shrink-0 bold-hide') }}
  <span ...>{{ item.label }}</span>
  {% if item.active %}<span class="nav-item-bar absolute left-0 top-1/2 -translate-y-1/2"></span>{% endif %}
</a>
```

**Regels:**
- `.bold-hide`: `[data-theme-preset="bold"] .bold-hide { display: none !important; }` — Lucide icon verborgen in Bold, `.sidebar-item-icon` square zichtbaar
- `.sidebar-item-icon { display: none; }` in base — alleen zichtbaar via Bold selector
- `.active` class triggert `#page-sidebar .nav-item.active` Bold CSS
- `nav-item-bar`: `2px × 20px`, geen border-radius (CSS al aanwezig)
- De `w-1 h-6 rounded-r-full bg-primary` span wordt vervangen door `nav-item-bar` span

### 1.3 Section labels (Content / System)

De bestaande `<button>` structuur (lijnen 71–83 in `_sidebar.twig`) heeft `x-show="!sidebarCollapsed"`, `x-transition`, en een geanimeerde chevron via `x-bind:class="open.content ? '' : '-rotate-90'"`. Deze Alpine bindings **blijven volledig behouden**.

Aanpak: class `nav-section-label` toevoegen aan de **bestaande `<button>`** — geen markup rewrite.

**Voor:**
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

**Na (alleen class toegevoegd, rest ongewijzigd):**
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

- `.nav-section-label` class triggert Bold CSS (JetBrains Mono, 9px, letter-spacing, etc.)
- `.bold-hide` verbergt minus-icon en chevron in Bold theme (ze staan niet in de reference)
- `x-show`, `x-transition`, `x-on:click`, `x-bind:class` blijven 100% intact
- Zelfde aanpak voor het System label (lijnen 105–116)

---

## Sectie 2 — Header (`_header.twig`)

### 2.1 Breadcrumb

**Aanpak:** Beide breadcrumb varianten staan in de DOM. Alpine.js `x-show` schakelt op basis van het actieve theme preset (uit `localStorage`/`data-theme-preset` attribuut). Geen Twig conditional nodig.

**Na (beide varianten naast elkaar):**
```twig
{# Bold breadcrumb — alleen zichtbaar als theme preset = bold #}
<div class="header-breadcrumb"
     x-show="document.documentElement.getAttribute('data-theme-preset') === 'bold'"
     x-cloak
     style="font-family: var(--font-mono); font-size: 11px; letter-spacing: 0.08em; text-transform: uppercase; color: var(--bc-muted); display: flex; align-items: center; gap: 8px;">
  Admin ·
  <span style="color: var(--bc);">{{ current_page_title|default('Dashboard') }}</span>
</div>

{# Standaard breadcrumb — verborgen in Bold #}
<div x-show="document.documentElement.getAttribute('data-theme-preset') !== 'bold'"
     x-cloak>
  {# bestaande breadcrumb loop — ongewijzigd #}
</div>
```

`current_page_title`: controleer of dit al als Twig globaal beschikbaar is in `TemplateService.php`. Zo niet, voeg toe als globale variabele die de paginatitel retourneert (kan afgeleid worden van de route of template block title).

### 2.2 Search box

```twig
<div class="header-search" style="background: var(--b3); border: 1px solid var(--border); padding: 6px 12px; font-size: 12px; color: var(--bc-muted); font-family: var(--font-mono); display: flex; align-items: center; gap: 8px; width: 180px;">
  ⌘ Search...
</div>
```

In Bold theme: flat, geen border-radius. CSS override: `[data-theme-preset="bold"] .header-search { border-radius: 0; }`

### 2.3 Header action icons

**Notificaties:**
```twig
<button type="button" class="header-icon-btn p-2" style="color: var(--bc-secondary); background: none; border: none; cursor: pointer;">
  {{ icon('bell', 'size-4') }}
</button>
```

**Theme switcher:** bestaand toggle component, class `header-icon-btn` toevoegen.

CSS: `[data-theme-preset="bold"] .header-icon-btn { border-radius: 0; }`

### 2.4 Header container

CSS aanvulling: `[data-theme-preset="bold"] #page-header { height: 52px; }` (huidige Tailwind `h-16` = 64px overschrijven).

---

## Sectie 3 — Dashboard stat cards (`dashboard.twig`)

### 3.1 Klasse aanpak

Canonical: `<dt>` krijgt class `stat-number`, `<dd>` voor label krijgt `stat-label`, sub-text `<div>` krijgt `stat-sub`. De bestaande `[data-stat-number]` en `[data-stat-label]` selectors in `base.css` worden **verwijderd** en vervangen door `.stat-number` en `.stat-label`.

**Voor:**
```twig
<dt x-show="!loading" x-text="stats.total_articles || 0" class="text-2xl font-extrabold"></dt>
<dd x-show="!loading" class="text-xs font-medium text-base-content/60">Total Articles</dd>
```

**Na:**
```twig
<dt x-show="!loading" x-text="stats.total_articles || 0" class="stat-number text-2xl font-extrabold"></dt>
<dd x-show="!loading" class="stat-label text-xs font-medium text-base-content/60">Total Articles</dd>
```

Sub-text div:
```twig
<div x-show="!loading" class="stat-sub mt-2 text-xs font-medium text-base-content/60">...</div>
```

### 3.2 Bold CSS voor stat classes

```css
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

### 3.3 Grid

`grid-cols-4` met `gap: 2px` (via bestaande Bold grid gap override).

---

## Sectie 4 — Dashboard lower section (`dashboard.twig`)

De lower section krijgt twee varianten naast elkaar in de DOM, geschakeld via `x-show` — conform de breadcrumb aanpak in sectie 2. De bestaande content (Recent Articles, Quick Actions grid, System Info) blijft intact voor andere themes.

```twig
{# Bestaande lower section — verborgen in Bold #}
<div x-show="document.documentElement.getAttribute('data-theme-preset') !== 'bold'" x-cloak>
  {# ... bestaande grid cols-2 met Recent Articles + System Info ongewijzigd ... #}
</div>

{# Bold lower section — alleen zichtbaar in Bold #}
<div x-show="document.documentElement.getAttribute('data-theme-preset') === 'bold'" x-cloak>
```

Dan de 3-card grid:

```twig
<div class="grid grid-cols-3" style="gap: 2px;">

  {# Card 1 — Quick Actions #}
  <div class="card-bold" style="background: var(--card); border: 1px solid var(--border); padding: 24px;">
    <div class="card-accent-bar"></div>
    <div class="card-title" style="font-family: var(--font-display); font-size: 18px; font-weight: 700; letter-spacing: -0.03em;">Quick Actions</div>
    <div class="card-body" style="font-size: 13px; color: var(--bc-muted); margin-top: 6px; line-height: 1.6;">Common tasks to get you started quickly.</div>
    <div style="margin-top: 16px; display: flex; flex-direction: column; gap: 8px;">
      <a href="{{ url('admin.page_create') }}" class="btn-ghost-link" style="text-align: left; font-size: 13px; color: var(--bc-secondary);">New Page</a>
      <a href="{{ url('admin.media') }}" class="btn-ghost-link" style="text-align: left; font-size: 13px; color: var(--bc-secondary);">Upload Media</a>
      <a href="{{ url('admin.users') }}" class="btn-ghost-link" style="text-align: left; font-size: 13px; color: var(--bc-secondary);">Manage Users</a>
    </div>
  </div>

  {# Card 2 — System Info #}
  <div class="card-bold" style="background: var(--card); border: 1px solid var(--border); padding: 24px;">
    <div class="card-accent-bar"></div>
    <div class="card-title" style="font-family: var(--font-display); font-size: 18px; font-weight: 700; letter-spacing: -0.03em;">System Info</div>
    <div class="card-body" style="font-size: 13px; color: var(--bc-muted); margin-top: 6px; line-height: 1.6;">Server and version details.</div>
    <div style="margin-top: 16px;">
      <div style="display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid var(--border);">
        <span style="font-family: var(--font-mono); font-size: 10px; letter-spacing: 0.08em; text-transform: uppercase; color: var(--bc-secondary);">Version</span>
        <span style="color: var(--bc); font-size: 12px;">{{ cms_version|default('1.0.0') }}</span>
      </div>
      <div style="display: flex; justify-content: space-between; padding: 8px 0;">
        <span style="font-family: var(--font-mono); font-size: 10px; letter-spacing: 0.08em; text-transform: uppercase; color: var(--bc-secondary);">PHP</span>
        <span style="color: var(--bc); font-size: 12px;">{{ php_version|default('8.1+') }}</span>
      </div>
    </div>
  </div>

  {# Card 3 — Featured (zichtbaar in Bold, verborgen in andere themes via Alpine) #}
  <div x-show="document.documentElement.getAttribute('data-theme-preset') === 'bold'"
       x-cloak
       class="card-bold" style="background: var(--card); border: 2px solid var(--p); padding: 24px;">
    <div style="font-family: var(--font-mono); font-size: 9px; letter-spacing: 0.15em; text-transform: uppercase; color: var(--pc); background: var(--p); display: inline-block; padding: 3px 8px; margin-bottom: 14px;">Featured</div>
    <div class="card-title" style="font-family: var(--font-display); font-size: 18px; font-weight: 700; letter-spacing: -0.03em;">Bold Theme Active</div>
    <div class="card-body" style="font-size: 13px; color: var(--bc-muted); margin-top: 6px; line-height: 1.6;">You are viewing the Bold Typography theme.</div>
  </div>
  {# Fallback placeholder voor niet-Bold themes #}
  <div x-show="document.documentElement.getAttribute('data-theme-preset') !== 'bold'"
       x-cloak
       style="background: var(--b2); border: 1px solid var(--border); padding: 24px;">
  </div>

</div>
```

</div>
{# Einde Bold lower section #}


---

## Sectie 5 — Articles table (`articles/index.twig`)

### 5.1 Badge rendering

**Huidig (exacte broncode, lijn 226–229 in `articles/index.twig`):**
```twig
<span class="inline-flex px-2 py-0.5 rounded-full text-xs font-medium"
      :class="article.status === 'published' ? 'bg-success/10 text-success' : (article.status === 'draft' ? 'bg-warning/10 text-warning' : 'bg-base-200 text-base-content/60')"
      x-text="article.status.charAt(0).toUpperCase() + article.status.slice(1)">
</span>
```

**Na:**
```twig
<span class="badge"
      :class="{
        'badge-published': article.status === 'published',
        'badge-draft': article.status === 'draft',
        'badge-error': article.status === 'error'
      }"
      x-text="article.status.toUpperCase()">
</span>
```

- `inline-flex`, `px-2`, `py-0.5`, `rounded-full`, `text-xs`, `font-medium` worden vervangen door `.badge` class (Bold CSS definieert de stijl volledig)
- Drie-weg `:class` binding wordt vervangen door object-syntax met drie keys
- `x-text` gewijzigd naar `.toUpperCase()` (badge text is uppercase in Bold)
- In andere themes: `.badge` class heeft minimale basis styling (geen bold-specifieke overrides), waardoor de bestaande kleuren via `:class` leidend zijn — **maar** de kleur-classes (`bg-success/10 text-success` etc.) worden verwijderd. Oplossing: voeg beide patronen samen als de Bold CSS scoped genoeg is, of voeg een Alpine check toe: `:class="isBold ? { 'badge-published': ..., ... } : { 'bg-success/10 text-success': ..., ... }"`

**Aanbeveling**: gebruik de `isBold` Alpine data property (gedefinieerd op de parent container):
```html
<div x-data="{ isBold: document.documentElement.getAttribute('data-theme-preset') === 'bold' }">
```

Dan:
```twig
<span class="badge"
      :class="isBold
        ? { 'badge-published': article.status === 'published', 'badge-draft': article.status === 'draft', 'badge-error': article.status === 'error' }
        : { 'bg-success/10 text-success': article.status === 'published', 'bg-warning/10 text-warning': article.status === 'draft', 'bg-base-200 text-base-content/60': !['published','draft'].includes(article.status) }"
      x-text="isBold ? article.status.toUpperCase() : article.status.charAt(0).toUpperCase() + article.status.slice(1)">
</span>
```

### 5.2 Extra kolommen

De bestaande extra kolommen (checkbox, category, author) **blijven behouden** — ze worden niet verwijderd. Bold CSS targe het gedrag van `th` en `td` generiek, dus alle kolommen passen automatisch aan.

### 5.3 Actions

```twig
<td>
  <a href="..." class="btn-ghost text-xs">Edit</a>
  <a href="..." class="text-xs" style="color: #ff6b6b;">Delete</a>
</td>
```

### 5.4 Table container wrapper

De `<table>` wrappen in:
```twig
<div class="table-container">
  <table ...>
```

CSS `[data-theme-preset="bold"] .table-container` al aanwezig in base.css.

### 5.5 Datum kolom

Via CSS: `[data-theme-preset="bold"] table tbody td.col-date { font-family: var(--font-mono); font-size: 11px; }` — class `col-date` toevoegen aan de datum `<td>`.

---

## Sectie 6 — CSS aanvullingen in `base.css`

### Te verwijderen (consolidatie)
- `[data-stat-number]` selectors (vervangen door `.stat-number`)
- `[data-stat-label]` selectors (vervangen door `.stat-label`)
- `[data-theme-preset="bold"] aside nav a` generieke selector (vervangen door `.nav-item` class selector)

### Toe te voegen
```css
/* Stat card classes */
[data-theme-preset="bold"] .stat-number { ... } /* zie sectie 3.2 */
[data-theme-preset="bold"] .stat-label  { ... }
[data-theme-preset="bold"] .stat-sub    { ... }

/* Header height */
[data-theme-preset="bold"] #page-header { height: 52px !important; }

/* Bold-only hidden elements */
[data-theme-preset="bold"] .bold-hide { display: none !important; }

/* Sidebar item icon (square) — verborgen buiten Bold */
.sidebar-item-icon { display: none; }
[data-theme-preset="bold"] .sidebar-item-icon {
  display: block;
  width: 14px; height: 14px;
  border: 1.5px solid currentColor;
  flex-shrink: 0;
}

/* Header icon buttons */
[data-theme-preset="bold"] .header-icon-btn { border-radius: 0; }

/* Date column mono */
[data-theme-preset="bold"] table tbody td.col-date {
  font-family: var(--font-mono);
  font-size: 11px;
}
```

### Bestaande overrides die correct zijn (geen wijziging nodig)
- Alle badge classes (`badge-draft`, `badge-published`, `badge-error`)
- Button overrides (`.btn-primary`, `.btn-secondary`, `.btn-ghost`, `.btn-danger`)
- Input/select/textarea overrides
- Flash message overrides
- Card `.rounded-xl.border` overrides
- Sidebar `.nav-item`, `.nav-item-bar`, `.nav-section-label` styling
- Scrollbar styling
- Grid gap override
- Table `th`/`td` styling
