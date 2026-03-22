# Sprint 1 — Filosofie Herstel Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Herstel de filosofie-compliance van CalacaCMS op drie fronten: verwijder alle `dark:` Tailwind prefixes en hardcoded kleuren uit admin templates, koppel de FormBuilder aan admin controllers, en update docs naar PHP-reality.

**Architecture:** S1.1 vervangt alle hardcoded Tailwind kleuren in admin templates met semantic classes die werken via CSS variabelen in `themes/index.css`. S1.2 update FormBuilder classes naar semantic en koppelt hem via ContentTypeManager aan ArticleController + PageController. S1.3 is pure docs update.

**Tech Stack:** Twig 3.0, Tailwind CSS v4 (`@theme` semantic colors), PHP 8.1+, CalacaCMS ContentType fluent builder

---

## Semantic Color Mapping Reference

Dit is de mapping die OVERAL consistent toegepast moet worden:

| Oud (hardcoded) | Nieuw (semantic) |
|---|---|
| `bg-white` | `bg-base-100` |
| `bg-slate-50` | `bg-base-200/30` |
| `bg-slate-100` | `bg-base-200` |
| `bg-slate-200` | `bg-base-300` |
| `dark:bg-slate-800` | *(verwijder — theme regelt dit)* |
| `dark:bg-slate-900` | *(verwijder)* |
| `dark:bg-slate-950/25` | *(verwijder)* |
| `text-slate-900` / `text-slate-800` | `text-base-content` |
| `text-slate-700` | `text-base-content/80` |
| `text-slate-600` | `text-base-content/70` |
| `text-slate-500` | `text-base-content/60` |
| `text-slate-400` | `text-base-content/50` |
| `dark:text-slate-400` | *(verwijder)* |
| `dark:text-slate-200` | *(verwijder)* |
| `dark:text-slate-300` | *(verwijder)* |
| `border-slate-100` | `border-border/50` |
| `border-slate-200` | `border-border` |
| `border-slate-200/75` | `border-border/75` |
| `dark:border-slate-700/50` | *(verwijder)* |
| `dark:border-slate-800/75` | *(verwijder)* |
| `divide-slate-100` | `divide-border/50` |
| `divide-slate-200/75` | `divide-border/75` |
| `dark:divide-slate-800/75` | *(verwijder)* |
| `hover:bg-slate-50` | `hover:bg-base-200/30` |
| `dark:hover:bg-slate-950/25` | *(verwijder)* |
| `hover:bg-slate-950/25` | *(verwijder — combined with above)* |
| `bg-indigo-600` / `bg-indigo-700` / `bg-indigo-800` | `bg-primary` |
| `hover:bg-indigo-700` / `hover:bg-indigo-800` | `hover:bg-primary-focus` |
| `dark:bg-indigo-500` / `dark:bg-indigo-600` | *(verwijder)* |
| `text-indigo-600` | `text-primary` |
| `text-white` (op primary button) | `text-primary-content` |
| `dark:text-indigo-400` / `dark:hover:text-indigo-300` | *(verwijder)* |
| `focus:border-indigo-400` | `focus:border-primary` |
| `focus:ring-indigo-300/25` | `focus:ring-primary/25` |
| `dark:focus:ring-indigo-600/50` | *(verwijder)* |
| `bg-indigo-50 text-indigo-500 dark:bg-indigo-950 dark:text-indigo-400` | `bg-primary/10 text-primary` |
| `bg-emerald-50 text-emerald-500 dark:bg-emerald-950 dark:text-emerald-400` | `bg-success/10 text-success` |
| `bg-purple-50 text-purple-500 dark:bg-purple-950 dark:text-purple-400` | `bg-accent/10 text-accent` |
| `bg-orange-50 text-orange-500 dark:bg-orange-950 dark:text-orange-400` | `bg-warning/10 text-warning` |
| `bg-emerald-100 text-emerald-800 dark:bg-emerald-950 dark:text-emerald-300` | `bg-success/15 text-success` |
| `bg-slate-100 text-slate-800 dark:bg-slate-800 dark:text-slate-300` | `bg-base-200 text-base-content` |
| `text-red-500` (required asterisk) | `text-error` |
| `bg-red-50 text-red-600` | `bg-error/10 text-error` |
| `border-red-200` | `border-error/30` |
| `text-gray-700` (in FormBuilder PHP) | `text-base-content/80` |
| `text-gray-600` (help text) | `text-base-content/60` |
| `border-gray-300` (input border) | `border-border` |
| `focus:ring-blue-500` (input focus) | `focus:ring-primary/50` |
| `focus:border-blue-500` | `focus:border-primary` |
| `text-blue-600` (checkbox) | `text-primary` |
| `border-gray-300` (checkbox) | `border-border` |

---

## Chunk 1: S1.1 — Semantic Colors in Templates

### Task 1: dashboard.twig — Verwijder dark: + vervang hardcoded kleuren

**Files:**
- Modify: `calaca/resources/themes/admin/templates/pages/dashboard/dashboard.twig`

- [ ] **Stap 1: Vervang hero/heading sectie (regels 11-29)**

```twig
{# VOOR #}
<p class="mt-1 text-sm font-medium text-slate-600 dark:text-slate-400">
<hr class="border-slate-200/75 dark:border-slate-700/50" />
<a class="...bg-indigo-700...dark:bg-indigo-600 dark:hover:bg-indigo-500 dark:focus:ring-indigo-600/50 dark:active:bg-indigo-700">

{# NA #}
<p class="mt-1 text-sm font-medium text-base-content/70">
<hr class="border-border/75" />
<a class="inline-flex items-center justify-center gap-2 rounded-lg border border-transparent bg-primary px-3 py-2 text-sm leading-5 font-semibold text-primary-content hover:bg-primary-focus focus:ring-3 focus:ring-primary/25 active:opacity-90">
```

- [ ] **Stap 2: Vervang stat-cards (regels 40-123)**

Elke stat-card is: `rounded-xl border border-slate-200/75 bg-white px-5 py-7 dark:border-slate-800/75 dark:bg-slate-900`
→ `rounded-xl border border-border/75 bg-base-100 px-5 py-7`

Text in cards: `text-slate-500 dark:text-slate-400` → `text-base-content/60`

Icon containers:
- `bg-indigo-50 text-indigo-500 dark:bg-indigo-950 dark:text-indigo-400` → `bg-primary/10 text-primary`
- `bg-emerald-50 text-emerald-500 dark:bg-emerald-950 dark:text-emerald-400` → `bg-success/10 text-success`
- `bg-purple-50 text-purple-500 dark:bg-purple-950 dark:text-purple-400` → `bg-accent/10 text-accent`
- `bg-orange-50 text-orange-500 dark:bg-orange-950 dark:text-orange-400` → `bg-warning/10 text-warning`

- [ ] **Stap 3: Vervang Recent Articles sectie (regels 129-202)**

Container: `rounded-xl border border-slate-200/75 dark:border-slate-800/75 bg-white dark:bg-slate-900`
→ `rounded-xl border border-border/75 bg-base-100`

Header border: `border-b border-slate-200/75 px-5 py-4 dark:border-slate-800/75`
→ `border-b border-border/75 px-5 py-4`

View all link: `text-indigo-600 hover:text-indigo-700 dark:text-indigo-400 dark:hover:text-indigo-300`
→ `text-primary hover:text-primary-focus`

Table dividers: `divide-y divide-slate-100 dark:divide-slate-800/75`
→ `divide-y divide-border/50`

Table row hover: `border-b border-slate-100 last:border-0 hover:bg-slate-50 dark:border-slate-800/75 dark:hover:bg-slate-950/25`
→ `border-b border-border/50 last:border-0 hover:bg-base-200/30`

Row icon: `bg-slate-100 text-slate-500 dark:bg-slate-800 dark:text-slate-400`
→ `bg-base-200 text-base-content/60`

Article title: `text-slate-800 dark:text-slate-200`
→ `text-base-content`

Link hover: `hover:text-indigo-600 dark:hover:text-indigo-400`
→ `hover:text-primary`

Time/date: `text-slate-500 dark:text-slate-400`
→ `text-base-content/60`

Status badges:
- Published: `bg-emerald-100 text-emerald-800 dark:bg-emerald-950 dark:text-emerald-300`
  → `bg-success/15 text-success`
- Draft: `bg-slate-100 text-slate-800 dark:bg-slate-800 dark:text-slate-300`
  → `bg-base-200 text-base-content`

Empty state: `text-slate-500 dark:text-slate-400`
→ `text-base-content/60`

Article link: `text-indigo-600 hover:text-indigo-700`
→ `text-primary hover:text-primary-focus`

- [ ] **Stap 4: Vervang Quick Actions + System Info (regels 205-255)**

Panel container: `rounded-xl border border-slate-200/75 dark:border-slate-800/75 bg-white dark:bg-slate-900`
→ `rounded-xl border border-border/75 bg-base-100`

Panel header border: `border-b border-slate-200/75 px-5 py-4 dark:border-slate-800/75`
→ `border-b border-border/75 px-5 py-4`

Quick action links: `text-slate-600 hover:bg-slate-50 hover:text-slate-900 dark:text-slate-400 dark:hover:bg-slate-950/25 dark:hover:text-white`
→ `text-base-content/70 hover:bg-base-200/30 hover:text-base-content`

Quick action dividers: `divide-x divide-y divide-slate-200/75 dark:divide-slate-800/75`
→ `divide-x divide-y divide-border/75`

Quick action icons: `text-slate-400 dark:text-slate-500`
→ `text-base-content/50`

System info rows: `border-b border-slate-100 pb-2 dark:border-slate-800/75`
→ `border-b border-border/50 pb-2`

Label text: `text-slate-500 dark:text-slate-400`
→ `text-base-content/60`

Value text: `text-slate-800 dark:text-slate-200`
→ `text-base-content`

Cache badge: `bg-emerald-100 text-emerald-800 dark:bg-emerald-950 dark:text-emerald-300`
→ `bg-success/15 text-success`

- [ ] **Stap 5: Controleer dat er geen dark: meer in het bestand staan**

```bash
grep "dark:" calaca/resources/themes/admin/templates/pages/dashboard/dashboard.twig
# Verwacht: geen output
```

---

### Task 2: auth.twig — Fix body gradient

**Files:**
- Modify: `calaca/resources/themes/admin/templates/layouts/auth.twig`

- [ ] **Stap 1: Vervang body class gradient**

```twig
{# VOOR #}
<body class="min-h-screen flex flex-col items-center justify-center p-6 bg-linear-to-br from-slate-100 to-slate-200 dark:from-slate-900 dark:to-slate-800">

{# NA - gebruik inline style met CSS variabelen #}
<body class="min-h-screen flex flex-col items-center justify-center p-6"
      style="background: linear-gradient(to bottom right, var(--b2), var(--b3));">
```

- [ ] **Stap 2: Controleer geen dark: meer aanwezig**

```bash
grep "dark:" calaca/resources/themes/admin/templates/layouts/auth.twig
# Verwacht: geen output
```

---

### Task 3: articles/edit.twig — Vervang hardcoded kleuren

**Files:**
- Modify: `calaca/resources/themes/admin/templates/pages/articles/edit.twig`

- [ ] **Stap 1: Vervang alle form labels**

```twig
{# VOOR #}
class="block text-sm font-medium text-slate-700 mb-1.5"
{# NA #}
class="block text-sm font-medium text-base-content/80 mb-1.5"
```

- [ ] **Stap 2: Vervang alle form inputs**

```twig
{# VOOR #}
class="w-full px-3 py-2 text-sm border border-slate-200 rounded-lg focus:outline-none focus:border-indigo-400 transition-colors"
{# NA #}
class="w-full px-3 py-2 text-sm border border-border rounded-lg focus:outline-none focus:border-primary transition-colors bg-base-100 text-base-content"
```

- [ ] **Stap 3: Vervang slug URL prefix + generate button**

```twig
{# VOOR #}
<span class="text-sm text-slate-400 flex-shrink-0">
<button...class="...border-slate-200 text-slate-600...hover:bg-slate-50...">
{# NA #}
<span class="text-sm text-base-content/50 flex-shrink-0">
<button...class="px-3 py-2 text-xs font-medium border border-border text-base-content/70 rounded-lg hover:bg-base-200/30 transition-colors flex-shrink-0">
```

- [ ] **Stap 4: Vervang help texts**

```twig
{# VOOR #}
class="text-xs text-slate-400 mt-1"
{# NA #}
class="text-xs text-base-content/50 mt-1"
```

- [ ] **Stap 5: Vervang featured image upload area**

```twig
{# VOOR #}
class="...border-dashed border-slate-200...text-slate-500...hover:border-indigo-300 hover:text-indigo-500..."
{# NA #}
class="flex items-center gap-2 px-4 py-2 text-sm border-2 border-dashed border-border text-base-content/60 cursor-pointer hover:border-primary hover:text-primary transition-colors w-fit rounded-lg"
```

- [ ] **Stap 6: Vervang sidebar panels (bg-white + border-slate-100)**

```twig
{# VOOR #}
class="bg-white rounded-xl border border-slate-100 shadow-sm p-5"
{# NA #}
class="bg-base-100 rounded-xl border border-border/50 shadow-sm p-5"
```

- [ ] **Stap 7: Vervang panel headings**

```twig
{# VOOR #}
class="text-sm font-semibold text-slate-900 mb-4"
{# NA #}
class="text-sm font-semibold text-base-content mb-4"
```

- [ ] **Stap 8: Vervang Save Draft + Publish buttons**

```twig
{# VOOR - Save Draft #}
class="flex-1 px-4 py-2 text-sm font-medium border border-slate-200 text-slate-700 rounded-lg hover:bg-slate-50 transition-colors"
{# NA #}
class="flex-1 px-4 py-2 text-sm font-medium border border-border text-base-content/80 rounded-lg hover:bg-base-200/30 transition-colors"

{# VOOR - Publish #}
class="flex-1 px-4 py-2 text-sm font-medium bg-indigo-600 text-white rounded-lg hover:bg-indigo-700 transition-colors"
{# NA #}
class="flex-1 px-4 py-2 text-sm font-medium bg-primary text-primary-content rounded-lg hover:bg-primary-focus transition-colors"
```

- [ ] **Stap 9: Vervang panel dividers + category/tag links**

```twig
{# VOOR #}
class="flex gap-3 pt-4 border-t border-slate-100"
{# NA #}
class="flex gap-3 pt-4 border-t border-border/50"

{# VOOR - Add new category link #}
class="mt-1 text-xs text-indigo-600 hover:text-indigo-700"
{# NA #}
class="mt-1 text-xs text-primary hover:text-primary-focus"
```

- [ ] **Stap 10: Vervang image container + remove button**

```twig
{# VOOR #}
class="h-24 w-auto rounded-lg border border-slate-200"
class="...bg-red-50 text-red-600...border border-red-200"
{# NA #}
class="h-24 w-auto rounded-lg border border-border"
class="absolute -top-2 -right-2 px-2 py-0.5 text-xs font-medium bg-error/10 text-error rounded-lg hover:bg-error/20 transition-colors border border-error/30"
```

---

### Task 4: pages/edit.twig — Vervang hardcoded kleuren

**Files:**
- Modify: `calaca/resources/themes/admin/templates/pages/pages/edit.twig`

- [ ] **Stap 1: Pas dezelfde vervangingen toe als in articles/edit.twig**

Zelfde mappings als Task 3 stap 1-9. Aanvullend:

```twig
{# file: input voor og_image #}
{# VOOR #}
class="text-sm text-slate-500 file:mr-3 file:py-1.5 file:px-3 file:rounded-lg file:border file:border-slate-200 file:text-xs file:font-medium file:text-slate-600 hover:file:bg-slate-50 file:transition-colors"
{# NA #}
class="text-sm text-base-content/60 file:mr-3 file:py-1.5 file:px-3 file:rounded-lg file:border file:border-border file:text-xs file:font-medium file:text-base-content/70 hover:file:bg-base-200/30 file:transition-colors"

{# Social image thumbnail #}
{# VOOR #}
class="h-16 w-auto rounded-lg border border-slate-200"
{# NA #}
class="h-16 w-auto rounded-lg border border-border"
```

- [ ] **Stap 2: Controleer beide edit templates zijn vrij van hardcoded kleur-patronen**

```bash
grep -n "slate-\|indigo-\|emerald-\|purple-\|orange-" \
  calaca/resources/themes/admin/templates/pages/articles/edit.twig \
  calaca/resources/themes/admin/templates/pages/pages/edit.twig
# Verwacht: geen output (of alleen semantic/allowed patterns)
```

---

## Chunk 2: S1.2 — FormBuilder Koppelen

### Task 5: FormBuilder.php — Update naar semantic classes

**Files:**
- Modify: `calaca/src/Services/FormBuilder.php`

- [ ] **Stap 1: Update buildTextField() (regel ~100-116)**

```php
// VOOR
$requiredStar = $required ? '<span class="text-red-500">*</span>' : '';
$helpHtml = $help ? "<p class=\"mt-1 text-sm text-gray-600\">{$help}</p>" : '';
class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500 sm:text-sm"

// NA
$requiredStar = $required ? '<span class="text-error">*</span>' : '';
$helpHtml = $help ? "<p class=\"mt-1 text-sm text-base-content/60\">{$help}</p>" : '';
class="w-full px-3 py-2 border border-border rounded-lg bg-base-100 text-base-content text-sm focus:outline-none focus:border-primary focus:ring-2 focus:ring-primary/20 transition-colors"
```

- [ ] **Stap 2: Update alle label classes (textareaField, numberField, dateField, booleanField)**

Vervang in ALLE buildX methodes:
- `class="block text-sm font-medium text-gray-700 mb-1"` → `class="block text-sm font-medium text-base-content/80 mb-1.5"`
- `class="mt-1 text-sm text-gray-600"` → `class="mt-1 text-sm text-base-content/60"`

- [ ] **Stap 3: Update buildTextareaField() border/focus**

```php
// NA
class="w-full px-3 py-2 border border-border rounded-lg bg-base-100 text-base-content text-sm focus:outline-none focus:border-primary focus:ring-2 focus:ring-primary/20 transition-colors resize-y"
```

- [ ] **Stap 4: Update buildBooleanField()**

```php
// VOOR
class="w-4 h-4 text-blue-600 rounded border-gray-300 focus:ring-blue-500 focus:border-blue-500"

// NA
class="w-4 h-4 text-primary rounded border-border focus:ring-primary focus:border-primary"
```

- [ ] **Stap 5: Update buildSelectField() en buildFileField()**

Zelfde border/focus patroon als TextField.

- [ ] **Stap 6: Update wrapField() error styling**

```php
// Zoek naar error klassen in wrapField() en update:
// 'border-red-500' → 'border-error'
// 'text-red-500' → 'text-error'
// 'text-red-600' → 'text-error'
```

- [ ] **Stap 7: Verwijder wrapForm() — FormBuilder mag geen `<form>` tag genereren**

De `<form>` met CSRF token hoort in de Twig template, niet in FormBuilder. Vervang `wrapForm()` door directe return van `$formHtml`:

```php
// build() methode — verander return:
// VOOR: return $this->wrapForm($formHtml);
// NA:   return $formHtml;
```

Verwijder de `wrapForm()` private methode volledig.

---

### Task 6: Maak Article content type definitie

**Files:**
- Create: `calaca/stubs/types/article.php` (stub voor nieuwe projecten)

- [ ] **Stap 1: Maak de stub file**

```php
<?php

declare(strict_types=1);

use CalacaCMS\ContentTypes\ContentTypeDefinitionInterface;
use CalacaCMS\Models\ContentType;
use CalacaCMS\Models\Fields\TextField;
use CalacaCMS\Models\Fields\TextareaField;
use CalacaCMS\Models\Fields\SelectField;
use CalacaCMS\Models\Fields\FileField;
use CalacaCMS\Models\Fields\BooleanField;

return new class implements ContentTypeDefinitionInterface {
    public static function define(): ContentType
    {
        return ContentType::create('article', 'Articles', 'Blog articles and news posts')
            ->field(TextField::make('title')->label('Title')->required()->placeholder('Article title'))
            ->field(TextField::make('slug')->label('Slug (URL)')->placeholder('auto-generated-from-title'))
            ->field(TextareaField::make('excerpt')->label('Excerpt')->help('Brief summary shown in listings (optional)'))
            ->field(TextareaField::make('content')->label('Content')->required())
            ->field(FileField::make('featured_image')->label('Featured Image')->accept('image/*'))
            ->field(TextField::make('image_caption')->label('Image Caption'));
    }
};
```

- [ ] **Stap 2: Maak dezelfde file beschikbaar voor de lokale dev setup**

```bash
# Controleer waar het calaca/types/ directory is (relatief aan project root)
ls /Users/macgyver/Dropbox/Code/side-projects/cms/calacacms/calaca/
# Als er al een types/ dir is, kopieer stub daarheen
```

---

### Task 7: Maak Page content type definitie

**Files:**
- Create: `calaca/stubs/types/page.php`

- [ ] **Stap 1: Maak de stub file**

```php
<?php

declare(strict_types=1);

use CalacaCMS\ContentTypes\ContentTypeDefinitionInterface;
use CalacaCMS\Models\ContentType;
use CalacaCMS\Models\Fields\TextField;
use CalacaCMS\Models\Fields\TextareaField;

return new class implements ContentTypeDefinitionInterface {
    public static function define(): ContentType
    {
        return ContentType::create('page', 'Pages', 'Static pages')
            ->field(TextField::make('title')->label('Title')->required()->placeholder('Page title'))
            ->field(TextField::make('slug')->label('Slug (URL)')->placeholder('auto-generated-from-title'))
            ->field(TextField::make('subtitle')->label('Subtitle')->help('Optional subtitle shown below the page title'))
            ->field(TextareaField::make('intro')->label('Introduction')->help('Brief intro shown before main content (optional)'))
            ->field(TextareaField::make('content')->label('Content')->required());
    }
};
```

---

### Task 8: Voeg ContentTypeManager toe aan BaseAdminController

**Files:**
- Modify: `calaca/src/Controllers/Admin/BaseAdminController.php`
- Modify: `calaca/src/Core/Bootstrap.php`

- [ ] **Stap 1: Voeg ContentTypeManager dependency toe aan BaseAdminController**

```php
// Voeg use toe
use CalacaCMS\Services\ContentTypeManager;

// Update constructor
public function __construct(
    ArticleRepository $articleRepository,
    PageRepository $pageRepository,
    ?Request $request = null,
    ?ContentTypeManager $contentTypeManager = null
) {
    parent::__construct($request);
    $this->articleRepository = $articleRepository;
    $this->pageRepository = $pageRepository;
    $this->contentTypeManager = $contentTypeManager ?? new ContentTypeManager();
}

// Voeg property toe
protected ContentTypeManager $contentTypeManager;
```

- [ ] **Stap 2: Update Bootstrap::initializeControllers()**

```php
// Voeg initialisatie van ContentTypeManager toe bovenaan initializeControllers():
$contentTypeManager = new ContentTypeManager();
$contentTypeManager->loadContentTypes();

// Update AdminArticleController instantiatie:
self::$controllers['admin.articles'] = new AdminArticleController(
    self::$repositories['articles'],
    self::$repositories['pages'],
    null,
    $contentTypeManager
);

// Update AdminPageController instantiatie:
self::$controllers['admin.pages'] = new AdminPageController(
    self::$repositories['articles'],
    self::$repositories['pages'],
    null,
    $contentTypeManager
);
```

---

### Task 9: Wire FormBuilder in ArticleController

**Files:**
- Modify: `calaca/src/Controllers/Admin/ArticleController.php`

- [ ] **Stap 1: Voeg FormBuilder gebruik toe aan create() methode**

```php
// Voeg use toe bovenaan
use CalacaCMS\Services\FormBuilder;

// In create() methode, voeg toe vóór $data definitie:
$articleContentType = $this->contentTypeManager->getContentType('article');
$formHtml = '';
if ($articleContentType) {
    $builder = new FormBuilder($articleContentType);
    $formHtml = $builder->build();
}

// Voeg 'form_html' toe aan $data array:
$data = [
    // ... bestaande keys ...
    'form_html' => $formHtml,
];
```

- [ ] **Stap 2: Zelfde in edit() methode**

```php
$articleContentType = $this->contentTypeManager->getContentType('article');
$formHtml = '';
if ($articleContentType) {
    $builder = new FormBuilder($articleContentType);
    $formHtml = $builder->build($article ?? []);
}
$data['form_html'] = $formHtml;
```

---

### Task 10: Wire FormBuilder in PageController

**Files:**
- Modify: `calaca/src/Controllers/Admin/PageController.php`

- [ ] **Stap 1: Zelfde pattern als ArticleController**

```php
use CalacaCMS\Services\FormBuilder;

// In create() en edit():
$pageContentType = $this->contentTypeManager->getContentType('page');
$formHtml = '';
if ($pageContentType) {
    $builder = new FormBuilder($pageContentType);
    $formHtml = $builder->build($page ?? []);
}
$data['form_html'] = $formHtml;
```

---

### Task 11: Update articles/edit.twig om form_html te gebruiken

**Files:**
- Modify: `calaca/resources/themes/admin/templates/pages/articles/edit.twig`

- [ ] **Stap 1: Vervang de main content fields door FormBuilder output**

Vervang het blok van stap 16 t/m 78 (title, slug, excerpt, content, featured_image, image_caption) met:

```twig
{# Main content velden — gegenereerd door FormBuilder #}
{% if form_html %}
    {{ form_html|raw }}
{% else %}
    {# Fallback: hardcoded fields als ContentType niet beschikbaar is #}
    <div>
        <label for="title" class="block text-sm font-medium text-base-content/80 mb-1.5">
            Title <span class="text-error">*</span>
        </label>
        <input type="text" id="title" name="title" value="{{ article.title|default('') }}" required
            class="w-full px-3 py-2 text-sm border border-border rounded-lg focus:outline-none focus:border-primary bg-base-100 text-base-content transition-colors">
    </div>
    {# ... overige fallback velden als nodig #}
{% endif %}
```

Voeg de slug JavaScript toe onder de FormBuilder output (het blijft relevant):

```twig
<script>
    // Slug auto-generation from title field (if generated by FormBuilder)
    const titleField = document.querySelector('[name="title"]');
    const slugField = document.querySelector('[name="slug"]');
    if (titleField && slugField) {
        titleField.addEventListener('input', () => {
            slugField.value = titleField.value
                .toLowerCase()
                .replace(/[^a-z0-9]+/g, '-')
                .replace(/^-+|-+$/g, '');
        });
    }
</script>
```

---

### Task 12: Update pages/edit.twig om form_html te gebruiken

**Files:**
- Modify: `calaca/resources/themes/admin/templates/pages/pages/edit.twig`

- [ ] **Stap 1: Zelfde aanpak als articles/edit.twig**

Vervang main fields (title, slug, subtitle, intro, content) met `{{ form_html|raw }}` + fallback.

---

## Chunk 3: S1.3 — Documentatie Update

### Task 13: Update philosophy docs — YAML → PHP

**Files:**
- Modify: `docs/01-knowledge-base/00-philosophy/chaincms-philosophy.md`

- [ ] **Stap 1: Zoek en vervang YAML referenties**

Zoek naar:
```
`.calaca/content_types/*.yml`
`YamlParserService`
YAML files
```

Vervang door PHP equivalenten:
```
Content types worden gedefinieerd als PHP files in `calaca/types/*.php`.
Elke file retourneert een anonieme class die `ContentTypeDefinitionInterface` implementeert.
```

- [ ] **Stap 2: Update de code voorbeelden in de docs**

```markdown
### 2. Definieer content types in PHP

\`\`\`php
// calaca/types/project.php
return new class implements ContentTypeDefinitionInterface {
    public static function define(): ContentType {
        return ContentType::create('project', 'Projects')
            ->field(TextField::make('title')->required())
            ->field(TextareaField::make('description'))
            ->field(FileField::make('image')->accept('image/*'))
            ->field(SelectField::make('category')->options(['web', 'design', 'branding']))
            ->field(BooleanField::make('featured'));
    }
};
\`\`\`
```

---

### Task 14: Update CLAUDE.md — Reflect PHP reality

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Stap 1: Zoek YAML referenties in Content Types System sectie**

```markdown
# VOOR
Content types are defined in YAML files (`/path/to/project/.calaca/content_types/*.yml`). The system includes:
- `YamlParserService` - Parses YAML files

# NA
Content types are defined as PHP files in `calaca/types/*.php`. Each file returns an anonymous class implementing `ContentTypeDefinitionInterface`. The system includes:
```

- [ ] **Stap 2: Update de field types lijst**

Controleer dat de CLAUDE.md field types (`text, textarea, number, date, boolean, select, file`) nog correct zijn.

---

## Verificatie Checklist (na alles)

- [ ] `grep -r "dark:" calaca/resources/themes/admin/templates --include="*.twig"` → leeg
- [ ] `grep -r "bg-slate-\|text-slate-\|border-slate-\|bg-indigo-\|text-indigo-" calaca/resources/themes/admin/templates --include="*.twig"` → leeg (of alleen niet-semantic patterns die we bewust laten staan)
- [ ] `grep -r "text-gray-\|border-gray-\|text-blue-\|focus:ring-blue-" calaca/src/Services/FormBuilder.php` → leeg
- [ ] `grep -r "YAML\|YamlParser\|content_types/*.yml" docs/01-knowledge-base CLAUDE.md` → leeg
- [ ] FormBuilder rendeert correct HTML wanneer ContentType beschikbaar is
- [ ] Admin edit forms tonen FormBuilder output in main area
