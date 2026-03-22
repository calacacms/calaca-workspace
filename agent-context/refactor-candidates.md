# Refactor Candidates

> Generated: 2026-03-20

## Priority Matrix

| # | File | Lines | Tech Debt | Impact | Risk | Complexity | Score | Priority | Status |
|---|------|-------|-----------|--------|------|------------|-------|----------|--------|
| 1 | `Core/Bootstrap.php` | 537 | 5 | 5 | 4 | 4 | **18** | CRITICAL | ⬜ |
| 2 | `Core/Application.php` | 514 | 4 | 5 | 4 | 4 | **17** | CRITICAL | ⬜ |
| 3 | `Controllers/Admin/MediaController.php` | 418 | 4 | 3 | 3 | 4 | **14** | HIGH | ⬜ |
| 4 | `Models/FieldDefinition.php` | 361 | 3 | 3 | 2 | 3 | **11** | HIGH | ⬜ |
| 5 | `Database/MigrationRunner.php` | 358 | 3 | 3 | 3 | 3 | **12** | HIGH | ⬜ |
| 6 | `Cache/FileCache.php` | 340 | 3 | 2 | 2 | 3 | **10** | MEDIUM | ⬜ |
| 7 | `Controllers/Admin/UserController.php` | 336 | 3 | 2 | 2 | 2 | **9** | MEDIUM | ⬜ |
| 8 | `Logger/FileLogger.php` | 335 | 3 | 2 | 2 | 3 | **10** | MEDIUM | ⬜ |
| 9 | `Repositories/MediaRepository.php` | 318 | 2 | 2 | 2 | 2 | **8** | MEDIUM | ⬜ |
| 10 | `Controllers/Admin/ArticleController.php` | 201 | 1 | 2 | 2 | 1 | **6** | LOW | ✅ |

**Scoring:** Tech Debt (×0.4) + Impact (×0.3) + Risk (×0.2) + Complexity (×0.1). Scale 1–5.

---

## Architectural Violations

| File | Issue | Severity |
|------|-------|----------|
| `Core/Bootstrap.php` | God Object — 20+ static methods acting as a service locator; mixes initialization, routing, and dependency management | HIGH |
| `Core/Bootstrap.php` | Parallel implementation with `Application.php` — two competing bootstrapping systems create confusion | HIGH |
| `Core/Application.php` | Monolithic class handles DI registration, routing, and request dispatch in a single file | HIGH |
| `Controllers/Admin/MediaController.php` | Controller does file I/O, folder management, API responses, and validation — violates SRP | HIGH |
| `Storage/FlatFileStorage.php` | Internally spawns 6 sub-service objects (CacheManager, TransactionManager, QueryProcessor, etc.) — excessive orchestration in a storage class | MEDIUM |
| `Controllers/Admin/ArticleController.php` | Duplicate CRUD pattern with `PageController` — 80%+ identical logic | MEDIUM |
| `Controllers/Admin/PageController.php` | Duplicate CRUD pattern with `ArticleController` — 80%+ identical logic | RESOLVED |

---

## Missing Test Coverage

### Controllers (0% covered)
- `Controllers/Admin/MediaController.php`
- `Controllers/Admin/UserController.php`
- `Controllers/Admin/ArticleController.php`
- `Controllers/Admin/PageController.php`
- `Controllers/Admin/AuthController.php`
- `Controllers/Admin/DashboardController.php`
- `Controllers/BaseController.php`
- `Controllers/Admin/BaseAdminController.php`
- `Controllers/HomeController.php`

### Core Infrastructure (0% covered)
- `Core/Bootstrap.php`
- `Core/Application.php`
- `Core/Providers/*`

### Services (0% covered)
- `Services/InstallationService.php`
- `Services/FileInstallerService.php`
- `Services/PageDataPreparer.php`
- `Services/ArticleDataPreparer.php`
- `Services/PageService.php`

### Database/Storage (0% covered)
- `Storage/BasePdoStorage.php`
- `Storage/MySQLStorage.php`
- `Storage/PostgreSQLStorage.php`
- `Storage/SQLiteStorage.php`
- `Database/Migration.php`

---

## Statistics

| Metric | Value |
|--------|-------|
| Total source files | 114 |
| Total lines of code | 17,961 |
| Average file size | 157 lines |
| Files > 300 lines | 10 |
| Files with test coverage | 22 |
| Files without tests | 92 |
| Test coverage (by file count) | ~19% |
| Deeply nested lines (20+ spaces) | 120 |
| TODO / FIXME comments | 0 |
| Duplicate controller patterns | 3 pairs |

---

## Phase 1 Recommendations

1. ✅ **`Core/Bootstrap.php`** — Migrate remaining static usage to `Application.php` DI container, then delete Bootstrap. This eliminates the God Object and unifies the two parallel bootstrapping paths. **COMPLETED**
2. ✅ **`Core/Application.php`** — Split into: `ContainerProvider` (DI wiring), `RouterProvider` (route definitions), and `RequestHandler` (dispatch). Each ≤ 150 lines. **COMPLETED**
3. ✅ **`Controllers/Admin/MediaController.php`** — Extract folder operations to `FolderController`, validation to `MediaValidator`, keeping only HTTP orchestration in the controller. **COMPLETED**
4. ✅ **`Controllers/Admin/ArticleController.php` + `PageController.php`** — Introduce a `BaseCrudController` with shared create/store/edit/update/delete logic, reducing both files to content-type-specific overrides only. **COMPLETED**
5. **`Models/FieldDefinition.php`** — Extract field-type-specific logic into `FieldDefinition\TypeRegistry` or individual value objects per field type.

---

## Duplicate Code Hotspots

| Pattern | Files Affected | Duplication % |
|---------|---------------|---------------|
| CRUD create/store/edit/update/delete | ArticleController, PageController | ~0% (resolved via BaseCrudController) |
| Flash + redirect after write | All admin controllers (15+ occurrences each) | High |
| CSRF token validation block | All write methods across 4 controllers | ~90% |
| `jsonSuccess` / `jsonError` response helpers | All admin controllers (13+ calls each) | Controlled (via BaseAdminController) |
