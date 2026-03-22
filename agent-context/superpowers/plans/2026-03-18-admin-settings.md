# Admin Settings Page Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implement a functional `/admin/settings` page where site administrators can configure General, Email, and Regional settings persisted to `calaca/config/settings.json`.

**Architecture:** A new `SettingsService` handles JSON file I/O (read/write with `LOCK_EX`, directory creation, merging defaults with saved values). A new `SettingsController` handles HTTP (GET renders form, POST validates and saves). Bootstrap.php wires both and replaces the hardcoded site globals with service-sourced values. Config.php site values remain as the fallback for Bootstrap's two Twig globals only.

**Tech Stack:** PHP 8.1+, PHPUnit 10, Twig 3, Symfony HttpFoundation, DaisyUI/Tailwind CSS

**Spec:** `docs/superpowers/specs/2026-03-18-admin-settings-design.md`

---

## File Map

| Action | Path | Responsibility |
|--------|------|----------------|
| Create | `calaca/src/Services/SettingsService.php` | Read/write settings.json, merge with hardcoded defaults |
| Create | `calaca/src/Controllers/Admin/SettingsController.php` | GET index + POST update |
| Create | `calaca/tests/Unit/SettingsServiceTest.php` | Unit tests for SettingsService |
| Modify | `calaca/themes/admin/templates/pages/settings/index.twig` | Replace placeholder with real form |
| Modify | `calaca/src/Core/Bootstrap.php` | Register controller, routes, load globals from service; add strict_types |
| Modify | `calaca/src/Controllers/Admin/DashboardController.php` | Remove `settings()` stub; add strict_types |

---

## Task 1: SettingsService

**Files:**
- Create: `calaca/src/Services/SettingsService.php`
- Create: `calaca/tests/Unit/SettingsServiceTest.php`

- [ ] **Step 1: Write the failing tests**

Create `calaca/tests/Unit/SettingsServiceTest.php`:

```php
<?php

declare(strict_types=1);

namespace CalacaCMS\Tests\Unit;

use CalacaCMS\Services\SettingsService;
use PHPUnit\Framework\TestCase;
use PHPUnit\Framework\Attributes\Test;

class SettingsServiceTest extends TestCase
{
    private string $projectRoot;
    private SettingsService $service;

    protected function setUp(): void
    {
        $this->projectRoot = sys_get_temp_dir() . '/calacacms_settings_test_' . uniqid();
        mkdir($this->projectRoot, 0777, true);
        $this->service = new SettingsService($this->projectRoot);
    }

    protected function tearDown(): void
    {
        $settingsPath = $this->projectRoot . '/calaca/config/settings.json';
        if (file_exists($settingsPath)) {
            unlink($settingsPath);
        }
        $configDir = $this->projectRoot . '/calaca/config';
        if (is_dir($configDir)) {
            rmdir($configDir);
        }
        $calacaDir = $this->projectRoot . '/calaca';
        if (is_dir($calacaDir)) {
            rmdir($calacaDir);
        }
        rmdir($this->projectRoot);
    }

    #[Test]
    public function testAllReturnsDefaults(): void
    {
        $settings = $this->service->all();

        $this->assertSame('My Website', $settings['site_title']);
        $this->assertSame('Europe/Amsterdam', $settings['timezone']);
        $this->assertSame('d-m-Y', $settings['date_format']);
        $this->assertSame('H:i', $settings['time_format']);
    }

    #[Test]
    public function testGetReturnsSingleValue(): void
    {
        $this->assertSame('My Website', $this->service->get('site_title'));
        $this->assertSame('fallback', $this->service->get('nonexistent_key', 'fallback'));
    }

    #[Test]
    public function testSaveCreatesDirectoryAndFile(): void
    {
        $this->service->save(['site_title' => 'Test Site', 'site_url' => 'http://example.com']);

        $path = $this->projectRoot . '/calaca/config/settings.json';
        $this->assertFileExists($path);

        $data = json_decode(file_get_contents($path), true);
        $this->assertSame('Test Site', $data['site_title']);
    }

    #[Test]
    public function testSavedValuesOverrideDefaults(): void
    {
        $this->service->save(['site_title' => 'My Custom Site', 'site_url' => 'http://custom.com']);

        $fresh = new SettingsService($this->projectRoot);
        $this->assertSame('My Custom Site', $fresh->get('site_title'));
    }

    #[Test]
    public function testSaveStripsInvalidDateFormat(): void
    {
        $this->service->save(['date_format' => 'invalid-format', 'site_url' => 'http://example.com', 'site_title' => 'Test']);

        $fresh = new SettingsService($this->projectRoot);
        $this->assertSame('d-m-Y', $fresh->get('date_format'));
    }

    #[Test]
    public function testSaveStripsInvalidTimeFormat(): void
    {
        $this->service->save(['time_format' => 'invalid', 'site_url' => 'http://example.com', 'site_title' => 'Test']);

        $fresh = new SettingsService($this->projectRoot);
        $this->assertSame('H:i', $fresh->get('time_format'));
    }

    #[Test]
    public function testSaveStripsInvalidTimezone(): void
    {
        $this->service->save(['timezone' => 'Not/ATimezone', 'site_url' => 'http://example.com', 'site_title' => 'Test']);

        $fresh = new SettingsService($this->projectRoot);
        $this->assertSame('Europe/Amsterdam', $fresh->get('timezone'));
    }

    #[Test]
    public function testGetSettingsPathPointsToCorrectLocation(): void
    {
        $expected = $this->projectRoot . '/calaca/config/settings.json';
        $this->assertSame($expected, $this->service->getSettingsPath());
    }
}
```

- [ ] **Step 2: Run tests to verify they fail**

```bash
cd /Users/macgyver/Dropbox/Code/side-projects/cms/calacacms/calaca
composer test -- --filter SettingsServiceTest
```

Expected: errors like `Class "CalacaCMS\Services\SettingsService" not found`

- [ ] **Step 3: Implement SettingsService**

Create `calaca/src/Services/SettingsService.php`:

```php
<?php

declare(strict_types=1);

namespace CalacaCMS\Services;

/**
 * SettingsService - Reads and writes site settings to calaca/config/settings.json.
 *
 * Value priority (highest wins):
 *   1. Values saved in settings.json
 *   2. Hardcoded defaults
 */
class SettingsService
{
    private const ALLOWED_DATE_FORMATS = ['d-m-Y', 'Y-m-d', 'm/d/Y', 'd F Y'];
    private const ALLOWED_TIME_FORMATS = ['H:i', 'g:i A'];

    private const DEFAULTS = [
        'site_title'   => 'My Website',
        'site_tagline' => '',
        'site_url'     => 'http://localhost:8000',
        'admin_email'  => '',
        'timezone'     => 'Europe/Amsterdam',
        'date_format'  => 'd-m-Y',
        'time_format'  => 'H:i',
    ];

    private string $projectRoot;

    public function __construct(string $projectRoot)
    {
        $this->projectRoot = rtrim($projectRoot, '/');
    }

    /**
     * Return all settings: defaults merged with saved values.
     */
    public function all(): array
    {
        return array_merge(self::DEFAULTS, $this->load());
    }

    /**
     * Return a single setting value.
     */
    public function get(string $key, mixed $default = null): mixed
    {
        return $this->all()[$key] ?? $default;
    }

    /**
     * Save settings to JSON file.
     * Creates calaca/config/ directory if it does not exist.
     * Only persists keys that are in DEFAULTS; validates enum fields.
     */
    public function save(array $settings): void
    {
        $sanitized = [];
        foreach (self::DEFAULTS as $key => $_) {
            if (!array_key_exists($key, $settings)) {
                continue;
            }
            $sanitized[$key] = $this->sanitizeValue($key, trim((string) $settings[$key]));
        }

        $dir = dirname($this->getSettingsPath());
        if (!is_dir($dir)) {
            mkdir($dir, 0755, true);
        }

        file_put_contents(
            $this->getSettingsPath(),
            json_encode($sanitized, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES),
            LOCK_EX
        );
    }

    /**
     * Return absolute path to settings.json.
     */
    public function getSettingsPath(): string
    {
        return $this->projectRoot . '/calaca/config/settings.json';
    }

    /**
     * Load raw values from settings.json (empty array if file absent).
     */
    private function load(): array
    {
        $path = $this->getSettingsPath();
        if (!file_exists($path)) {
            return [];
        }

        $decoded = json_decode((string) file_get_contents($path), true);
        return is_array($decoded) ? $decoded : [];
    }

    /**
     * Validate and sanitize a single value. Falls back to default on invalid input.
     */
    private function sanitizeValue(string $key, string $value): string
    {
        return match ($key) {
            'date_format' => in_array($value, self::ALLOWED_DATE_FORMATS, true)
                ? $value
                : self::DEFAULTS['date_format'],
            'time_format' => in_array($value, self::ALLOWED_TIME_FORMATS, true)
                ? $value
                : self::DEFAULTS['time_format'],
            'timezone'    => in_array($value, \DateTimeZone::listIdentifiers(), true)
                ? $value
                : self::DEFAULTS['timezone'],
            default       => $value,
        };
    }
}
```

- [ ] **Step 4: Run tests to verify they pass**

```bash
cd /Users/macgyver/Dropbox/Code/side-projects/cms/calacacms/calaca
composer test -- --filter SettingsServiceTest
```

Expected: 8 tests, all green. (Assertion count will be higher than 8 due to multiple assertions per test.)

- [ ] **Step 5: Commit**

```bash
cd /Users/macgyver/Dropbox/Code/side-projects/cms/calacacms/calaca
git add src/Services/SettingsService.php tests/Unit/SettingsServiceTest.php
git commit -m "feat: add SettingsService for reading and writing site settings"
```

---

## Task 2: SettingsController + DashboardController cleanup

**Files:**
- Create: `calaca/src/Controllers/Admin/SettingsController.php`
- Modify: `calaca/src/Controllers/Admin/DashboardController.php`

- [ ] **Step 1: Create SettingsController**

Create `calaca/src/Controllers/Admin/SettingsController.php`:

```php
<?php

declare(strict_types=1);

namespace CalacaCMS\Controllers\Admin;

use CalacaCMS\Repositories\ArticleRepository;
use CalacaCMS\Repositories\PageRepository;
use CalacaCMS\Services\SettingsService;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

/**
 * Admin Settings Controller
 */
class SettingsController extends BaseAdminController
{
    private SettingsService $settingsService;

    public function __construct(
        ArticleRepository $articleRepository,
        PageRepository $pageRepository,
        SettingsService $settingsService,
        ?Request $request = null,
    ) {
        parent::__construct($articleRepository, $pageRepository, $request);
        $this->settingsService = $settingsService;
    }

    /**
     * Display the settings form.
     */
    public function index(): Response
    {
        $this->requireAuth();

        return $this->html($this->render('@admin/pages/settings/index.twig', [
            'page_title' => 'Settings',
            'settings'   => $this->settingsService->all(),
            'csrf_token' => $this->generateCsrfToken(),
            'timezones'  => $this->groupedTimezones(),
        ]));
    }

    /**
     * Handle settings form submission.
     */
    public function update(): Response
    {
        $this->requireAuth();

        $token = $_POST['_token'] ?? null;
        if (!$this->validateCsrfToken($token)) {
            return new Response('Invalid CSRF token.', 403);
        }

        $raw = [
            'site_title'   => trim(strip_tags($_POST['site_title'] ?? '')),
            'site_tagline' => trim(strip_tags($_POST['site_tagline'] ?? '')),
            'site_url'     => trim(strip_tags($_POST['site_url'] ?? '')),
            'admin_email'  => trim(strip_tags($_POST['admin_email'] ?? '')),
            'timezone'     => trim($_POST['timezone'] ?? ''),
            'date_format'  => trim($_POST['date_format'] ?? ''),
            'time_format'  => trim($_POST['time_format'] ?? ''),
        ];

        $errors = $this->validateSettings($raw);
        if ($errors !== []) {
            $this->setFlash('error', implode(' ', $errors));
            return $this->redirect('/admin/settings');
        }

        $this->settingsService->save($raw);
        $this->setFlash('success', 'Settings saved successfully.');
        return $this->redirect('/admin/settings');
    }

    /**
     * Validate settings POST data. Returns array of error strings.
     *
     * @return string[]
     */
    private function validateSettings(array $data): array
    {
        $errors = [];

        if ($data['site_title'] === '') {
            $errors[] = 'Site title is required.';
        }

        if ($data['site_url'] === '') {
            $errors[] = 'Site URL is required.';
        } elseif (!filter_var($data['site_url'], FILTER_VALIDATE_URL)) {
            $errors[] = 'Site URL must be a valid URL (e.g. http://example.com).';
        }

        if ($data['admin_email'] !== '' && !filter_var($data['admin_email'], FILTER_VALIDATE_EMAIL)) {
            $errors[] = 'Admin email must be a valid email address.';
        }

        return $errors;
    }

    /**
     * Return timezones grouped by continent for the <select> element.
     *
     * @return array<string, string[]>
     */
    private function groupedTimezones(): array
    {
        $groups = [];
        foreach (\DateTimeZone::listIdentifiers() as $tz) {
            $slash = strpos($tz, '/');
            $continent = $slash !== false ? substr($tz, 0, $slash) : 'Other';
            $groups[$continent][] = $tz;
        }
        ksort($groups);
        return $groups;
    }
}
```

- [ ] **Step 2: Remove settings() stub from DashboardController and add strict_types**

In `calaca/src/Controllers/Admin/DashboardController.php`:

**Add** `declare(strict_types=1);` after the opening `<?php` line (line 1):
```php
<?php

declare(strict_types=1);

namespace CalacaCMS\Controllers\Admin;
```

**Remove** the entire `settings()` method and its docblock (currently lines 72–81):
```php
    /**
     * Settings page (placeholder)
     */
    public function settings(): Response
    {
        $this->requireAuth();
        return $this->html($this->render('@admin/pages/settings/index.twig', [
            'page_title' => 'Settings',
        ]));
    }
```

- [ ] **Step 3: Run existing tests**

```bash
cd /Users/macgyver/Dropbox/Code/side-projects/cms/calacacms/calaca
composer test
```

Expected: all existing tests pass.

- [ ] **Step 4: Commit**

```bash
cd /Users/macgyver/Dropbox/Code/side-projects/cms/calacacms/calaca
git add src/Controllers/Admin/SettingsController.php src/Controllers/Admin/DashboardController.php
git commit -m "feat: add SettingsController, remove DashboardController settings stub"
```

---

## Task 3: Bootstrap Wiring

**Files:**
- Modify: `calaca/src/Core/Bootstrap.php`

- [ ] **Step 1: Add strict_types declaration to Bootstrap.php**

`Bootstrap.php` currently lacks `declare(strict_types=1)`. Add it after the opening `<?php`:

```php
<?php

declare(strict_types=1);

namespace CalacaCMS\Core;
```

- [ ] **Step 2: Add SettingsController and SettingsService imports**

After the existing `use` statements (around line 34), add:
```php
use CalacaCMS\Controllers\Admin\SettingsController;
use CalacaCMS\Services\SettingsService;
```

- [ ] **Step 3: Update the admin.settings route and add POST route**

In `initializeRouting()`, find:
```php
        $admin->add('admin.settings',      new Route('/admin/settings',               ['_controller' => 'admin.dashboard::settings']));
```

Replace with:
```php
        $admin->add('admin.settings',        new Route('/admin/settings',        ['_controller' => 'admin.settings::index']));
        $admin->add('admin.settings_update', new Route('/admin/settings/update', ['_controller' => 'admin.settings::update'], [], [], '', [], ['POST']));
```

- [ ] **Step 4: Register SettingsController in initializeControllers()**

In `initializeControllers()`, after the `admin.auth` controller block, add:
```php
        self::$controllers['admin.settings'] = new SettingsController(
            self::$repositories['articles'],
            self::$repositories['pages'],
            new SettingsService(TemplateService::getProjectRoot()),
        );
```

- [ ] **Step 5: Replace hardcoded site globals with SettingsService values**

In `initializeTemplates()`, find these three lines (currently 222–225):
```php
        TemplateService::addGlobal('app_debug', $debug);
        TemplateService::addGlobal('site_name', self::$config['site']['name'] ?? 'My Website');
        TemplateService::addGlobal('site_url', self::$config['site']['url'] ?? 'http://localhost:8000');
        TemplateService::addGlobal('vite_dev', (bool) ($_ENV['VITE_DEV'] ?? getenv('VITE_DEV')));
```

Replace with:
```php
        $siteSettings = new SettingsService($projectRoot);
        TemplateService::addGlobal('app_debug', $debug);
        TemplateService::addGlobal('site_name', $siteSettings->get('site_title', self::$config['site']['name'] ?? 'My Website'));
        TemplateService::addGlobal('site_url',  $siteSettings->get('site_url',   self::$config['site']['url']  ?? 'http://localhost:8000'));
        TemplateService::addGlobal('vite_dev', (bool) ($_ENV['VITE_DEV'] ?? getenv('VITE_DEV')));
```

Note: `$projectRoot` is already declared earlier in `initializeTemplates()` via `TemplateService::getProjectRoot()`.

- [ ] **Step 6: Run all tests**

```bash
cd /Users/macgyver/Dropbox/Code/side-projects/cms/calacacms/calaca
composer test
```

Expected: all tests pass.

- [ ] **Step 7: Commit**

```bash
cd /Users/macgyver/Dropbox/Code/side-projects/cms/calacacms/calaca
git add src/Core/Bootstrap.php
git commit -m "feat: wire SettingsController and SettingsService into Bootstrap"
```

---

## Task 4: Settings Template + .gitignore

**Files:**
- Modify: `calaca/themes/admin/templates/pages/settings/index.twig`
- Modify: `.gitignore` in the **repository root** (`calacacms/.gitignore`)

- [ ] **Step 1: Replace placeholder with real settings form**

Replace the entire content of `calaca/themes/admin/templates/pages/settings/index.twig`:

```twig
{% extends '@admin/layouts/layout.twig' %}
{% from '@admin/macros/icons.twig' import icon %}

{% block admin_title %}Settings — {{ site_name }}{% endblock %}
{% block page_title %}Settings{% endblock %}

{% block admin_content %}
{% include '@admin/partials/_flash.twig' %}

<form method="post" action="/admin/settings/update" class="max-w-2xl space-y-6">
    <input type="hidden" name="_token" value="{{ csrf_token }}">

    {# General #}
    <div class="bg-base-100 rounded-xl border border-border/50 shadow-sm p-6">
        <h2 class="text-sm font-semibold text-base-content mb-5 flex items-center gap-2">
            {{ icon('globe', 'size-4') }} General
        </h2>
        <div class="space-y-4">
            <div>
                <label for="site_title" class="block text-sm font-medium text-base-content/80 mb-1.5">
                    Site Title <span class="text-error">*</span>
                </label>
                <input type="text" id="site_title" name="site_title"
                    value="{{ settings.site_title|default('') }}" required
                    placeholder="My Website"
                    class="w-full px-3 py-2 text-sm border border-border rounded-lg bg-base-100 text-base-content focus:outline-none focus:border-primary focus:ring-2 focus:ring-primary/20 transition-colors">
            </div>
            <div>
                <label for="site_tagline" class="block text-sm font-medium text-base-content/80 mb-1.5">Tagline</label>
                <input type="text" id="site_tagline" name="site_tagline"
                    value="{{ settings.site_tagline|default('') }}"
                    placeholder="A short description of your site"
                    class="w-full px-3 py-2 text-sm border border-border rounded-lg bg-base-100 text-base-content focus:outline-none focus:border-primary focus:ring-2 focus:ring-primary/20 transition-colors">
            </div>
            <div>
                <label for="site_url" class="block text-sm font-medium text-base-content/80 mb-1.5">
                    Site URL <span class="text-error">*</span>
                </label>
                <input type="url" id="site_url" name="site_url"
                    value="{{ settings.site_url|default('') }}" required
                    placeholder="http://localhost:8000"
                    class="w-full px-3 py-2 text-sm border border-border rounded-lg bg-base-100 text-base-content focus:outline-none focus:border-primary focus:ring-2 focus:ring-primary/20 transition-colors">
            </div>
        </div>
    </div>

    {# Email #}
    <div class="bg-base-100 rounded-xl border border-border/50 shadow-sm p-6">
        <h2 class="text-sm font-semibold text-base-content mb-5 flex items-center gap-2">
            {{ icon('mail', 'size-4') }} Email
        </h2>
        <div>
            <label for="admin_email" class="block text-sm font-medium text-base-content/80 mb-1.5">Admin Email</label>
            <input type="email" id="admin_email" name="admin_email"
                value="{{ settings.admin_email|default('') }}"
                placeholder="admin@example.com"
                class="w-full px-3 py-2 text-sm border border-border rounded-lg bg-base-100 text-base-content focus:outline-none focus:border-primary focus:ring-2 focus:ring-primary/20 transition-colors">
            <p class="text-xs text-base-content/50 mt-1.5">Used for system notifications.</p>
        </div>
    </div>

    {# Regional #}
    <div class="bg-base-100 rounded-xl border border-border/50 shadow-sm p-6">
        <h2 class="text-sm font-semibold text-base-content mb-5 flex items-center gap-2">
            {{ icon('clock', 'size-4') }} Regional
        </h2>
        <div class="space-y-4">
            <div>
                <label for="timezone" class="block text-sm font-medium text-base-content/80 mb-1.5">Timezone</label>
                <select id="timezone" name="timezone"
                    class="w-full px-3 py-2 text-sm border border-border rounded-lg bg-base-100 text-base-content focus:outline-none focus:border-primary focus:ring-2 focus:ring-primary/20 transition-colors">
                    {% for continent, zones in timezones %}
                        <optgroup label="{{ continent }}">
                            {% for tz in zones %}
                                <option value="{{ tz }}" {% if settings.timezone == tz %}selected{% endif %}>
                                    {{ tz|replace({'_': ' '}) }}
                                </option>
                            {% endfor %}
                        </optgroup>
                    {% endfor %}
                </select>
            </div>
            <div>
                <label for="date_format" class="block text-sm font-medium text-base-content/80 mb-1.5">Date Format</label>
                <select id="date_format" name="date_format"
                    class="w-full px-3 py-2 text-sm border border-border rounded-lg bg-base-100 text-base-content focus:outline-none focus:border-primary focus:ring-2 focus:ring-primary/20 transition-colors">
                    {% set formats = {'d-m-Y': '31-12-2026', 'Y-m-d': '2026-12-31', 'm/d/Y': '12/31/2026', 'd F Y': '31 December 2026'} %}
                    {% for value, example in formats %}
                        <option value="{{ value }}" {% if settings.date_format == value %}selected{% endif %}>
                            {{ example }} ({{ value }})
                        </option>
                    {% endfor %}
                </select>
            </div>
            <div>
                <label for="time_format" class="block text-sm font-medium text-base-content/80 mb-1.5">Time Format</label>
                <select id="time_format" name="time_format"
                    class="w-full px-3 py-2 text-sm border border-border rounded-lg bg-base-100 text-base-content focus:outline-none focus:border-primary focus:ring-2 focus:ring-primary/20 transition-colors">
                    <option value="H:i" {% if settings.time_format == 'H:i' %}selected{% endif %}>14:30 (24-hour)</option>
                    <option value="g:i A" {% if settings.time_format == 'g:i A' %}selected{% endif %}>2:30 PM (12-hour)</option>
                </select>
            </div>
        </div>
    </div>

    {# Save button #}
    <div class="flex justify-end">
        <button type="submit"
            class="px-5 py-2 text-sm font-medium bg-primary text-primary-content rounded-lg hover:bg-primary/90 transition-colors">
            Save Settings
        </button>
    </div>
</form>
{% endblock %}
```

- [ ] **Step 2: Add settings.json to .gitignore**

The git repository root is `calacacms/`. Add this line to `calacacms/.gitignore` (create the file if it doesn't exist):
```
calaca/config/settings.json
```

- [ ] **Step 3: Manual smoke test**

Start the dev server and visit `http://localhost:8000/admin/settings`. Verify:
- Form renders with 3 section cards (General, Email, Regional)
- Timezone dropdown is populated with continents as `<optgroup>`
- Saving valid data redirects back with a green success flash
- Saved values are pre-filled on reload
- Submitting an invalid URL shows a red error flash

- [ ] **Step 4: Run all tests**

```bash
cd /Users/macgyver/Dropbox/Code/side-projects/cms/calacacms/calaca
composer test
```

Expected: all tests green.

- [ ] **Step 5: Commit**

```bash
cd /Users/macgyver/Dropbox/Code/side-projects/cms/calacacms/calaca
git add ../themes/admin/templates/pages/settings/index.twig ../.gitignore
git commit -m "feat: implement admin settings form template"
```

---

## Done

All tasks complete when:
- `composer test` passes with no failures
- `/admin/settings` renders the full form
- Saving settings writes `calaca/config/settings.json`
- Saved values survive a page reload
- Invalid input (bad URL, bad email) shows a flash error
- `calaca/config/settings.json` is in `.gitignore`
