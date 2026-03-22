# Admin Settings Page — Design Spec

**Date:** 2026-03-18
**Status:** Approved

## Overview

Implement a functional admin settings page at `/admin/settings` that allows the site administrator to configure general, email, and regional settings. Settings are persisted in `calaca/config/settings.json`.

## Architecture

### New Files

| File | Purpose |
|------|---------|
| `src/Controllers/Admin/SettingsController.php` | HTTP handler: GET index, POST update |
| `src/Services/SettingsService.php` | Read/write `settings.json`, merge with defaults |
| `themes/admin/templates/pages/settings/index.twig` | Settings form (3 section cards) |

### Modified Files

| File | Change |
|------|--------|
| `src/Core/Bootstrap.php` | Reassign existing `admin.settings` route, add POST route, inject SettingsService, wire globals after templates init |
| `src/Controllers/Admin/DashboardController.php` | Remove stub `settings()` method |

---

## SettingsService

Responsible for all file I/O. No HTTP concerns.

```
calaca/config/settings.json  ←→  SettingsService  ←→  SettingsController
```

**Constructor:** receives `string $projectRoot` (the project root path).

**Methods:**
- `all(): array` — returns merged array: config.php site values → hardcoded defaults → settings.json overrides
- `get(string $key, mixed $default = null): mixed` — single value lookup from `all()`
- `save(array $settings): void` — creates `calaca/config/` directory if absent (`mkdir` recursive), writes JSON atomically: `file_put_contents` with `LOCK_EX` flag. Validates allowed values before writing.
- `getSettingsPath(): string` — returns absolute path to `settings.json`

**Value priority (highest wins):**
1. Values in `settings.json` (user-saved)
2. Values from `config.php` `site` key (existing install)
3. Hardcoded defaults below

**Hardcoded defaults:**
```json
{
  "site_title":   "My Website",
  "site_tagline": "",
  "site_url":     "http://localhost:8000",
  "admin_email":  "",
  "timezone":     "Europe/Amsterdam",
  "date_format":  "d-m-Y",
  "time_format":  "H:i"
}
```

**Allowed values (validated in `save()`):**
- `date_format`: `["d-m-Y", "Y-m-d", "m/d/Y", "d F Y"]`
- `time_format`: `["H:i", "g:i A"]`
- `timezone`: any value in `DateTimeZone::listIdentifiers()`

Settings are stored as a flat JSON object. No nesting.

---

## SettingsController

Extends `BaseAdminController`.

**Constructor:**
```php
public function __construct(
    ArticleRepository $articleRepository,
    PageRepository $pageRepository,
    SettingsService $settingsService,
)
```
The third parameter is `SettingsService`. Bootstrap instantiates this explicitly.

### `index(): Response`
- Calls `requireAuth()`
- Loads `SettingsService::all()`
- Generates CSRF token via `generateCsrfToken()`
- Renders `@admin/pages/settings/index.twig`

### `update(): Response`
- Calls `requireAuth()`
- Validates CSRF token — returns 403 if invalid
- Strips and trims all POST values
- Validates:
  - `site_title`: required, non-empty after trim
  - `site_url`: required, passes `filter_var($url, FILTER_VALIDATE_URL)`
  - `admin_email`: if not empty, passes `filter_var($email, FILTER_VALIDATE_EMAIL)`
  - `timezone`: in `DateTimeZone::listIdentifiers()`
  - `date_format`: in allowed set
  - `time_format`: in allowed set
- On validation failure: redirect to `/admin/settings` with flash error
- On success: calls `SettingsService::save()`, redirects with flash success

**Known v1 limitation:** Single CSRF token per session. Two simultaneous settings tabs will cause the second save to fail CSRF validation. Acceptable for a single-admin CMS.

---

## Routes

```
GET  /admin/settings         → admin.settings::index    (replaces existing stub)
POST /admin/settings/update  → admin.settings::update   (new, methods: ['POST'])
```

In `Bootstrap::initializeRouting()`:
- Replace existing `admin.settings` route (currently pointing to `admin.dashboard::settings`) with `admin.settings::index`
- Add `admin.settings_update` route with `methods: ['POST']`

In `Bootstrap::initializeControllers()`:
- Add `admin.settings` controller key → new `SettingsController` instance

---

## Template — `settings/index.twig`

Replaces the existing placeholder. Extends `@admin/layouts/layout.twig`. Three section cards, single-column layout (`max-w-2xl` centered). Flash messages via existing `{% include '@admin/partials/_flash.twig' %}`.

### Section 1: General
| Field | Type | Name |
|-------|------|------|
| Site Title | text (required) | `site_title` |
| Tagline | text | `site_tagline` |
| Site URL | url | `site_url` |

### Section 2: Email
| Field | Type | Name |
|-------|------|------|
| Admin Email | email | `admin_email` |

### Section 3: Regional
| Field | Type | Options |
|-------|------|---------|
| Timezone | select | `DateTimeZone::listIdentifiers()` grouped by continent |
| Date Format | select | d-m-Y / Y-m-d / m/d/Y / d F Y |
| Time Format | select | H:i (24h) / g:i A (12h) |

Footer: single **Save Settings** button + hidden CSRF input.

---

## Bootstrap Integration

In `initializeTemplates()`, after `TemplateService::initialize()`, load settings and set globals:

```php
$settings = new SettingsService($projectRoot);
TemplateService::addGlobal('site_name', $settings->get('site_title', 'My Website'));
TemplateService::addGlobal('site_url',  $settings->get('site_url',   'http://localhost:8000'));
```

This replaces the existing hardcoded `self::$config['site']` globals. Twig is initialized at this point so `addGlobal()` is safe.

---

## Settings File Location

```
calaca/
└── config/
    └── settings.json   ← created on first save; directory created by SettingsService if absent
```

Add `calaca/config/settings.json` to `.gitignore` (the directory itself can be committed).

---

## Out of Scope (v1)

- Per-user preferences
- SMTP configuration / mail sending
- Social media links
- API keys or webhooks
- Inline field-level validation errors (flash message sufficient)
- Tabs UI (single scrollable page)
- Concurrent write safety beyond `LOCK_EX` (acceptable for single-admin CMS)
