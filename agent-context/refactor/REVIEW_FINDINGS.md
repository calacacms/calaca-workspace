# CalacaCMS Codebase Review Findings

> Generated: 2026-03-13
> Status: In Progress

---

## Security Review

### CRITICAL

1. **PostgreSQLStorage: SQL injection via unquoted schema parameter** (`src/Storage/PostgreSQLStorage.php:68,260,289,303,319`)
   `SET search_path TO {$this->schema}` and subsequent information_schema queries use direct string interpolation for the schema name. A malicious schema value can inject arbitrary SQL.

2. **Hardcoded admin credentials in AuthController** (`src/Controllers/Admin/AuthController.php:123`)
   `authenticateUser()` is a placeholder that hardcodes `admin@example.com` / `admin123`. If this reaches production, any attacker can access the admin panel.

3. **Missing password hashing — plain-text password comparison** (`src/Controllers/Admin/AuthController.php`)
   No `password_hash()` / `password_verify()` anywhere. Passwords are stored and compared as plain text.

4. **Path traversal in content type YAML loading** (`src/Services/YamlParserService.php:124`)
   `$directory . '/' . $name . '.yml'` — `$name` is not sanitized with `basename()`. Calling `getContentType('../../etc/passwd')` can read arbitrary files.

### WARNING

5. **MIME type validation uses client-supplied `$_FILES['type']`** (`src/Services/FileUploadService.php:54`)
   Clients can spoof MIME types to bypass file-type restrictions. Should use `finfo_file()` on the actual file content.

6. **Session fixation — no `session_regenerate_id()` after login** (`src/Controllers/Admin/AuthController.php:71`)
   An attacker can fixate a session ID before the user authenticates, then hijack their session.

7. **Open redirect via unvalidated `$_SESSION['intended_url']`** (`src/Controllers/Admin/AuthController.php:84`)
   Redirect target is stored in session without validating it is internal — enables phishing using the legitimate domain.

8. **CSRF token never rotated, no expiry** (`src/Controllers/Admin/BaseAdminController.php:81`)
   Token stored in plain session, no per-request rotation, no expiration — reduces CSRF protection effectiveness.

9. **Response header injection via `setHeader()`** (`src/Controllers/BaseController.php:174-176`)
   Header names/values are passed through without CRLF validation, enabling response-splitting attacks.

10. **XSS in FormBuilder select options** (`src/Services/FormBuilder.php:299`)
    `<option value="{$optionValue}">` — neither value nor label is passed through `htmlspecialchars()`.

11. **SQL injection in MigrationRunner table names** (`src/Database/MigrationRunner.php:71,85,98`)
    Table name interpolated into CREATE TABLE statements without parameterization in SQL-based drivers.

12. **`$_SERVER['REQUEST_URI']` output without escaping** (`src/Views/Extensions/CmsExtension.php:136`)
    If rendered in a template this is a reflected XSS vector.

### SUGGESTION

13. **Login errors distinguish between unknown user vs wrong password** (`src/Controllers/Admin/AuthController.php:54,62`) — enables user enumeration. Use a single generic message.
14. **No session cookie security flags** — `HttpOnly`, `Secure`, `SameSite` not set anywhere.
15. **Missing security response headers** — no `X-Content-Type-Options`, `X-Frame-Options`, `Content-Security-Policy`, or HSTS.
16. **No brute-force / rate-limiting protection on login** (`src/Controllers/Admin/AuthController.php`).
17. **Storage directories created world-readable (0755)** (`src/CLI/InitCommand.php:53`) — sensitive logs and cache readable by other system users. Use 0700.
18. **`verifyRememberToken()` not implemented** (`src/Controllers/Admin/AuthController.php:138-143`) — remember-me tokens are never verified against storage.

---

## Performance Review

### CRITICAL

1. **FlatFileStorage full-table scans on every query** (`src/Storage/FlatFileStorage.php:88-131`)
   Every `select()` loads the entire JSON file into memory and filters in PHP. No index support. On 1,000+ rows every query reads and deserializes all data.

2. **DashboardController loads ALL articles/pages for statistics** (`src/Controllers/Admin/DashboardController.php:38-61`)
   Calls `findAll()` then filters with `array_filter()` repeatedly. On a large site this pulls tens of thousands of records into memory just to count them.

3. **ArticleRepository.search() uses invalid OR query syntax** (`src/Repositories/ArticleRepository.php:93-102`)
   The nested array WHERE structure is incompatible with FlatFileStorage — likely returns empty results silently, forcing callers to re-query.

4. **TemplateService: `getProjectRoot()` directory-walks on every access** (`src/Views/TemplateService.php:58-77`)
   Project root is detected by traversing parent directories every time it is called — no caching between calls within a request.

5. **FileCache pre-creates all 256 subdirectories on every instantiation** (`src/Cache/FileCache.php:294-302`)
   `mkdir()` called 256 times unconditionally at startup. Should create subdirs lazily.

6. **ContentTypeManager re-parses all YAML files on every request** (`src/Services/ContentTypeManager.php:37-56`)
   No in-memory or opcode caching of parsed content types. Every request hits the filesystem and runs the YAML parser for each type file.

7. **RouterService creates a new `UrlMatcher` instance on every `dispatch()` call** (`src/Routing/RouterService.php:84-95`)
   Both `getMatcher()` and `getAdminMatcher()` construct new instances each invocation instead of caching.

8. **HomeController loads ALL articles to show 3 featured + 6 recent** (`src/Controllers/HomeController.php:37-41`)
   `findFeatured()` loads all articles, filters in PHP, slices to 3. Same for `findRecent(6)`. On 10 K articles: 20 K+ items loaded for a homepage.

### WARNING

9. **BaseRepository.applyOrder() has syntax error — extra closing brace** (`src/Repositories/BaseRepository.php:148`)
   `});` causes a parse-level issue; multi-field ordering silently broken.

10. **FlatFileStorage rollback doesn't deep-copy data** (`src/Storage/FlatFileStorage.php:43-83`)
    `$transactionData = $this->cache` assigns by reference; rollback reverts to the already-mutated array, making transactions ineffective.

11. **FileLogger channel storage is unbounded** (`src/Logger/FileLogger.php:95-116`)
    `$this->channels` array grows without limit during a request. On high-traffic sites this could exhaust memory.

12. **FileLogger.readLogFile() re-runs regex on every line, no caching** (`src/Logger/FileLogger.php:197-235`)
    Large log files parsed line-by-line with uncompiled patterns on every access.

13. **BaseRepository.count() performs full select then counts in PHP** (`src/Repositories/BaseRepository.php:107-111`)
    No COUNT query at the storage layer — loads all matching rows just to return `count($rows)`.

14. **ArticleController.paginate() calls countAll() separately** (`src/Repositories/ArticleRepository.php:79-88`)
    Two full scans per paginated page request: one for count, one for data.

15. **SchemaGenerator.generateIndexes() references undefined `$tableName`** (`src/Services/SchemaGenerator.php:178`)
    Variable not set in scope — index generation fails silently.

### SUGGESTION

16. **StorageFactory.createFromEnv() reads $_ENV multiple times** (`src/Storage/StorageFactory.php:99-128`) — Build config array once.
17. **Bootstrap uses all-static properties** (`src/Core/Bootstrap.php:39-57`) — Prevents state reset between requests/tests.
18. **FileCache uses md5() for every get/set key** (`src/Cache/FileCache.php:263-271`) — Unnecessarily expensive; simpler scheme would suffice.
19. **CmsExtension.slugify() runs 4× preg_replace + iconv per call** (`src/Views/Extensions/CmsExtension.php:44-54`) — Results should be cached.
20. **AdminArticleController.getCategories() returns hardcoded static array on every call** (`src/Controllers/Admin/ArticleController.php:249-257`) — Should be lazy-loaded once.

---

## Code Quality Review

### CRITICAL

1. **Bootstrap: namespace typo `CalacaMS` → `CalacaCMS`** (`src/Core/Bootstrap.php:117`)
   `\CalacaMS\Logger\FileLogger` will throw a fatal class-not-found error at runtime.

2. **Bootstrap: invalid spread operator `new $storageClass(...)`** (`src/Core/Bootstrap.php:99`)
   PHP spread without args is not a valid constructor call pattern — storage driver will fail to instantiate.

3. **BaseRepository.applyFilters() has broken arrow-function syntax** (`src/Repositories/BaseRepository.php:121-148`)
   `=> !isset` is invalid; also extra `});` closing brace (line 148). The entire filter/order pipeline won't execute.

4. **FieldDefinition: misplaced quote in match expression** (`src/Models/FieldDefinition.php:125`)
   `default' =>` — stray quote causes a parse error, preventing the class from loading.

5. **SchemaGenerator: undefined variable `$tableName`** (`src/Services/SchemaGenerator.php:178`)
   Variable is never set in scope; should be `$contentType->getTableName()`. Index generation silently produces broken SQL.

6. **SchemaGenerator: extra quote in SQL DEFAULT value** (`src/Services/SchemaGenerator.php:46`)
   Mismatched quotes break generated CREATE TABLE statements.

7. **StorageFactory.createFromEnv(): match expression result never assigned** (`src/Storage/StorageFactory.php:107-125`)
   Driver-specific config (host, port, dbname) is computed but the result is discarded — MySQL/PostgreSQL drivers always get an empty config.

8. **RouterService: `Request` class not imported** (`src/Routing/RouterService.php:51-54`)
   `Request::create()` called without use statement — fatal error on every request dispatch.

### WARNING

9. **`RepositoryInterface` in wrong namespace** (`src/Core/RepositoryInterface.php:3`)
   Declared as `CalacaCMS\` instead of `CalacaCMS\Core\` — violates CLAUDE.md architecture spec.

10. **Dutch comments throughout codebase** — explicit project rule is English-only
    - `src/Repositories/BaseRepository.php:8,18,32,42,62,72,81,88,104,115,130`
    - `src/Repositories/ArticleRepository.php:6,10,23,25,33,41,65,91`
    - `src/Core/StorageInterface.php:6-34`
    - `src/Routing/RouterService.php:45,58,98`

11. **SQL storage drivers duplicate ~50 lines of select/insert/update/delete** (`src/Storage/MySQLStorage.php:89-142`, `src/Storage/SQLiteStorage.php:89-142`)
    Should be extracted to an abstract `BasePdoStorage`.

12. **FlatFileStorage WHERE matching uses loose `!=` comparison** (`src/Storage/FlatFileStorage.php:94-101`)
    Type coercion can cause false-positive matches between `"0"` and `false`.

13. **FormBuilder calls methods not on FieldDefinition interface** (`src/Services/FormBuilder.php:63-65`)
    `isMultiple()`, `getPlaceholder()`, `getHelp()` — runtime errors when rendering any form.

14. **YamlParserService custom parser used despite symfony/yaml in composer.json** (`src/Services/YamlParserService.php:20`)
    Custom parser has acknowledged incomplete nested-object support (line 85 comment). symfony/yaml is already a dependency.

15. **BaseController: repeated `session_start()` without headers-sent guard** (`src/Controllers/BaseController.php:109-138`)
    Silent session failures can mask authentication bugs.

16. **MigrationRunner: catches generic `\Exception`** (`src/Database/MigrationRunner.php:166`)
    Swallows unexpected errors with a misleading generic message.

17. **Magic numbers/strings with no constants**
    - `FileCache.php:15` — TTL `3600`
    - `FileLogger.php:15` — date format `'Y-m-d H:i:s'`
    - `ArticleController.php:287-288` — max upload size `5 * 1024 * 1024` and MIME list

### SUGGESTION

18. **CacheFactory pattern not used — Bootstrap hardcodes FileCache** (`src/Core/Bootstrap.php:105-108`) — inconsistent with StorageFactory approach.
19. **FileLogger.rotateLogs() never called automatically** (`src/Logger/FileLogger.php:304-326`) — logs grow unbounded.
20. **FormBuilder uses heredoc with hardcoded Tailwind CSS classes** (`src/Services/FormBuilder.php:97+`) — should use Twig templates.
21. **InitCommand.mkdir() return values unchecked** (`src/CLI/InitCommand.php:52-55`) — silent failure during `calacacms init`.
22. **ArticleController imports unused `RedirectResponse`** (`src/Controllers/Admin/ArticleController.php:6`).
23. **Missing return type declarations on controller actions** — claimed PHP 8.1 strict-types project.

## Test Coverage Review

> 9 test files / 2,477 lines of tests cover ~25% of 60+ source files.

### CRITICAL — Zero coverage on critical paths

1. **All 4 storage drivers completely untested** (~1,492 LOC)
   - `FlatFileStorage.php` (352 LOC) — file I/O, transactions, JSON error handling, table sanitization
   - `SQLiteStorage.php` (300 LOC) — PDO connection, error recovery
   - `MySQLStorage.php` (318 LOC) — multi-param connections, charset handling
   - `PostgreSQLStorage.php` (354 LOC) — schema support, sequences
   - `StorageFactory.php` (168 LOC) — driver selection, `createFromEnv()`, config validation

2. **Authentication flow has zero tests** (`src/Controllers/Admin/AuthController.php` — 145 LOC)
   Login, logout, remember-me cookie, CSRF generation, and the hardcoded placeholder credentials are all untested.

3. **MigrationRunner completely untested** (`src/Database/MigrationRunner.php` — 358 LOC)
   run, rollback, status, reset, drop — all migration workflows untested. Failed migration recovery and transaction handling untested.

4. **All 4 CLI commands untested** (`src/CLI/` — 230+ LOC)
   `MigrateCommand`, `InitCommand`, `AdminCommand`, `ServeCommand`.

5. **AuthMiddleware completely untested** (`src/Middleware/AuthMiddleware.php` — 102 LOC)
   Session initialization, public route detection, role checks, redirect logic.

6. **FileUploadService completely untested** (`src/Services/FileUploadService.php` — 271 LOC)
   MIME validation, size limits, path traversal guard, `deleteFile()`, `listFiles()`.

7. **ContentTypeManager completely untested** (`src/Services/ContentTypeManager.php` — 333 LOC)
   YAML directory parsing, field factory, all 10 type-specific configuration methods.

8. **All 10 controllers untested** — includes all admin CRUD and all frontend rendering.

### WARNING — Inadequate existing tests

9. **RouterServiceTest uses `assertTrue(true)` placeholder assertions** (`tests/RouterServiceTest.php:38-39,49-50`)
   Tests pass unconditionally without verifying any behavior.

10. **FileCacheTest TTL test uses `sleep(3)`** (`tests/FileCacheTest.php:56-60`)
    Brittle — system load can cause spurious failures. Time should be mockable.

11. **ArticleRepository and PageRepository have no tests** despite having custom query logic (`findFeatured()`, `search()`, `paginate()`).

12. **FieldDefinition.validate() and validateRule() untested** (`src/Models/FieldDefinition.php:132-261`)
    9 validation rule types with no test coverage.

### SUGGESTION — Missing edge cases

13. No integration tests for: storage driver switching, migration workflows end-to-end, authentication login→session→logout cycle, file upload→storage→retrieval.
14. No security-boundary tests: CSRF bypass attempts, session fixation, path traversal, SQL injection through repositories.
15. No tests for corrupted data recovery (malformed JSON in FlatFileStorage, failed PDO connections).
16. No tests for Unicode/emoji input, very long strings, or empty/null edge cases across services.

---

## Consensus Summary

> Status: Complete — 2026-03-13

### The application cannot run as-is

Multiple **compile/runtime-fatal** issues across code quality findings will crash the app before a single request completes:
- Namespace typo in `Bootstrap.php:117` (`CalacaMS` → `CalacaCMS`)
- Invalid spread operator `new $storageClass(...)` in `Bootstrap.php:99`
- Parse error in `FieldDefinition.php:125` (stray quote in match expression)
- Broken `BaseRepository::applyFilters()` (invalid syntax, extra brace)
- Missing `use` import for `Request` in `RouterService.php:51`

### Top priorities by risk

| # | Finding | Category | Severity |
|---|---------|----------|----------|
| 1 | Hardcoded admin credentials (`admin123`) | Security | CRITICAL |
| 2 | Plain-text password storage/comparison | Security | CRITICAL |
| 3 | Bootstrap namespace typo + broken instantiation | Code Quality | CRITICAL |
| 4 | Path traversal in YAML content type loader | Security | CRITICAL |
| 5 | PostgreSQL SQL injection via schema parameter | Security | CRITICAL |
| 6 | BaseRepository filter/order pipeline broken | Code Quality | CRITICAL |
| 7 | StorageFactory config silently discarded | Code Quality | CRITICAL |
| 8 | Zero test coverage on storage, auth, migrations | Test Coverage | CRITICAL |
| 9 | FlatFileStorage full scans — no pagination at storage layer | Performance | CRITICAL |
| 10 | Session fixation on login, open redirect | Security | WARNING |

### Systemic themes
- **Dutch comments** violate the explicit English-only project rule across ~15 files.
- **Missing abstractions**: SQL drivers duplicate ~50 lines; no `BasePdoStorage`; custom YAML parser when `symfony/yaml` is already a dependency.
- **No integration tests** for any critical workflow (auth, migrations, file uploads, routing).
- **FlatFileStorage is fundamentally unscalable** for production use beyond a few hundred records without query-level pagination and indexing.
