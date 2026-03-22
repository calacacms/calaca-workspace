# Bootstrap.php Refactoring Task List

**PRD:** docs/prd-refactor-bootstrap.md
**Target:** Replace static Bootstrap with DI-based Application class
**Estimated Time:** 3.5-4 hours

---

## Phase 1: Create Application Class

**Estimated:** 1 hour

### Task 1.1: Create Application.php skeleton
- [ ] Create `calaca/src/Core/Application.php`
- [ ] Add `use` statements for League Container, Symfony components
- [ ] Define properties: `$container`, `$config`, `$routes`, `$requestContext`
- [ ] Implement `__construct(array $config, Container $container = null)`
- [ ] Add PHPDoc for class and constructor

**Acceptance Criteria:**
- [ ] File exists at `calaca/src/Core/Application.php`
- [ ] Class has `private Container $container` property
- [ ] Class has `private array $config` property
- [ ] Class has `private RouteCollection $routes` property

### Task 1.2: Implement initialize() method
- [ ] Implement `public function initialize(): void`
- [ ] Call `registerServices()` and `registerRoutes()`
- [ ] PHPDoc for method documentation
- [ ] Ensure no static properties (all instance)

**Acceptance Criteria:**
- [ ] `initialize()` method exists
- [ ] Calls `registerServices()` and `registerRoutes()`
- [ ] Method is public and non-static

### Task 1.3: Implement handleRequest() method
- [ ] Implement `public function handleRequest(Request $request): Response`
- [ ] Use Symfony Router with stored `$routes` and `$requestContext`
- [ ] Match route and execute controller method
- [ ] Handle ResourceNotFoundException (return 404)
- [ ] Handle MethodNotAllowedException (return 405)
- [ ] Return proper Response object

**Acceptance Criteria:**
- [ ] `handleRequest()` method matches Bootstrap signature
- [ ] Uses Symfony UrlMatcher with routes
- [ ] Catches route exceptions properly
- [ ] Returns Response for all cases

### Task 1.4: Implement run() convenience method
- [ ] Implement `public function run(): void` (for CLI usage)
- [ ] Create default Request from globals
- [ ] Call `handleRequest()` and send Response

**Acceptance Criteria:**
- [ ] `run()` method exists
- [ ] Creates Request if not provided
- [ ] Calls `handleRequest()` and outputs response

### Task 1.5: Implement service registration method
- [ ] Implement `private function registerServices(): void`
- [ ] Register storage, cache, logger with Container
- [ ] Use service providers (created in Phase 2)

**Acceptance Criteria:**
- [ ] `registerServices()` method exists
- [ ] Uses `$this->container->add()` or similar

### Task 1.6: Implement route registration method
- [ ] Implement `private function registerRoutes(): void`
- [ ] Define all routes from Bootstrap (copy)
- [ ] Use Symfony Route objects
- [ ] Store in `$this->routes` property

**Acceptance Criteria:**
- [ ] `registerRoutes()` method exists
- [ ] All Bootstrap routes are registered
- [ ] `$this->routes` property is populated

---

## Phase 2: Extract Service Providers

**Estimated:** 30 minutes

### Task 2.1: Create ServiceProviderInterface
- [ ] Create `calaca/src/Core/ServiceProviderInterface.php`
- [ ] Define `register(Container $container): void` method
- [ ] Add PHPDoc

**Acceptance Criteria:**
- [ ] Interface created with `register()` method
- [ ] File exists at correct path

### Task 2.2: Create base ServiceProvider class
- [ ] Create `calaca/src/Core/ServiceProvider.php`
- [ ] Implement `register()` as abstract
- [ ] Add `$container` property
- [ ] Add constructor if needed

**Acceptance Criteria:**
- [ ] Abstract ServiceProvider class created
- [ ] Has `register(Container $container)` method

### Task 2.3: Create SessionServiceProvider
- [ ] Create `calaca/src/Core/Providers/SessionServiceProvider.php`
- [ ] Extend ServiceProvider
- [ ] Register `SessionService` singleton
- [ ] Register to Container in `register()` method

**Acceptance Criteria:**
- [ ] SessionServiceProvider extends ServiceProvider
- [ ] Registers SessionService with container
- [ ] SessionService uses static start() or proper instantiation

### Task 2.4: Create StorageServiceProvider
- [ ] Create `calaca/src/Core/Providers/StorageServiceProvider.php`
- [ ] Extend ServiceProvider
- [ ] Register `StorageFactory::create($config['storage'])`
- [ ] Use config passed to Application constructor

**Acceptance Criteria:**
- [ ] StorageServiceProvider extends ServiceProvider
- [ ] Creates storage instance from config
- [ ] Registers storage with container

### Task 2.5: Create CacheServiceProvider
- [ ] Create `calaca/src/Core/Providers/CacheServiceProvider.php`
- [ ] Extend ServiceProvider
- [ ] Register FileCache instance
- [ ] Use project root from Application config or config array

**Acceptance Criteria:**
- [ ] CacheServiceProvider extends ServiceProvider
- [ ] Registers FileCache with container
- [ ] Uses project root from Application

### Task 2.6: Create LoggerServiceProvider
- [ ] Create `calaca/src/Core/Providers/LoggerServiceProvider.php`
- [ ] Extend ServiceProvider
- [ ] Register FileLogger instance
- [ ] Use project root from Application config

**Acceptance Criteria:**
- [ ] LoggerServiceProvider extends ServiceProvider
- [ ] Registers FileLogger with container
- [ ] Uses project root from Application

### Task 2.7: Create TemplateServiceProvider
- [ ] Create `calaca/src/Core/Providers/TemplateServiceProvider.php`
- [ ] Extend ServiceProvider
- [ ] Register TemplateService (already has initialization logic)
- [ ] Initialize TemplateService with config

**Acceptance Criteria:**
- [ ] TemplateServiceProvider extends ServiceProvider
- [ ] Registers TemplateService with container
- [ ] TemplateService initialized properly

### Task 2.8: Create RepositoryServiceProvider
- [ ] Create `calaca/src/Core/Providers/RepositoryServiceProvider.php`
- [ ] Extend ServiceProvider
- [ ] Register ArticleRepository, PageRepository, UsersRepository
- [ ] Pass storage instance from container

**Acceptance Criteria:**
- [ ] RepositoryServiceProvider extends ServiceProvider
- [ ] Registers all 3 repositories
- [ ] Repositories receive storage from container

### Task 2.9: Create RoutingServiceProvider
- [ ] Create `calaca/src/Core/Providers/RoutingServiceProvider.php`
- [ ] Extend ServiceProvider
- [ ] Register RouteCollection
- [ ] Register RequestContext for URL generation

**Acceptance Criteria:**
- [ ] RoutingServiceProvider extends ServiceProvider
- [ ] Registers RouteCollection
- [ ] Registers RequestContext

### Task 2.10: Create ControllerServiceProvider
- [ ] Create `calaca/src/Core/Providers/ControllerServiceProvider.php`
- [ ] Extend ServiceProvider
- [ ] Register all admin controllers:
  - DashboardController
  - ArticleController (AdminArticleController)
  - PageController (AdminPageController)
  - AuthController (AdminAuthController)
  - MediaController (AdminMediaController)
  - SettingsController
  - UserController (AdminUserController)
- [ ] Register frontend controllers:
  - HomeController
  - ArticleController
  - PageController
- [ ] Pass dependencies to controllers:
  - ArticleRepository, PageRepository (for admin controllers)
  - ContentTypeManager (for admin article/page controllers)

**Acceptance Criteria:**
- [ ] ControllerServiceProvider extends ServiceProvider
- [ ] All 11 controllers registered
- [ ] Controllers receive proper dependencies from container

---

## Phase 3: Update Controllers for Dependency Injection

**Estimated:** 1 hour

### Task 3.1: Update AdminArticleController
- [ ] Add constructor to accept dependencies via Container
- [ ] Accept: ArticleRepository, PageRepository, ContentTypeManager
- [ ] Remove any static Bootstrap references if present

**Acceptance Criteria:**
- [ ] Constructor accepts 3 dependencies
- [ ] Properties are `private Repository`, etc. (not static)
- [ ] No calls to `Bootstrap::getRepository()`

### Task 3.2: Update AdminPageController
- [ ] Add constructor to accept dependencies via Container
- [ ] Accept: ArticleRepository, PageRepository, ContentTypeManager

**Acceptance Criteria:**
- [ ] Constructor accepts 3 dependencies
- [ ] No calls to `Bootstrap::getRepository()`

### Task 3.3: Update AdminDashboardController
- [ ] Add constructor to accept dependencies via Container
- [ ] Accept: ArticleRepository, PageRepository

**Acceptance Criteria:**
- [ ] Constructor accepts 2 dependencies
- [ ] No calls to `Bootstrap::getRepository()`

### Task 3.4: Update AdminAuthController
- [ ] Add constructor to accept dependencies via Container
- [ ] Accept: UsersRepository, RateLimiter, ArticleRepository, PageRepository

**Acceptance Criteria:**
- [ ] Constructor accepts 4 dependencies
- [ ] No calls to `Bootstrap::getRepository()`

### Task 3.5: Update AdminMediaController
- [ ] Add constructor to accept dependencies via Container
- [ ] Accept: ArticleRepository, PageRepository, FileUploadService
- [ ] Add MediaRepository (currently missing)

**Acceptance Criteria:**
- [ ] Constructor accepts 4 dependencies
- [ ] No calls to `Bootstrap::getRepository()`

### Task 3.6: Update AdminSettingsController
- [ ] Add constructor to accept dependencies via Container
- [ ] Accept: ArticleRepository, PageRepository

**Acceptance Criteria:**
- [ ] Constructor accepts 2 dependencies
- [ ] No calls to `Bootstrap::getRepository()`

### Task 3.7: Update AdminUserController
- [ ] Add constructor to accept dependencies via Container
- [ ] Accept: UsersRepository, ArticleRepository, PageRepository

**Acceptance Criteria:**
- [ ] Constructor accepts 3 dependencies
- [ ] No calls to `Bootstrap::getRepository()`

### Task 3.8: Update HomeController
- [ ] Add constructor to accept dependencies via Container
- [ ] Accept: ArticleRepository, PageRepository

**Acceptance Criteria:**
- [ ] Constructor accepts 2 dependencies
- [ ] No calls to `Bootstrap::getRepository()`

### Task 3.9: Update ArticleController
- [ ] Add constructor to accept dependencies via Container
- [ ] Accept: ArticleRepository

**Acceptance Criteria:**
- [ ] Constructor accepts 1 dependency
- [ ] No calls to `Bootstrap::getRepository()`

### Task 3.10: Update PageController
- [ ] Add constructor to accept dependencies via Container
- [ ] Accept: PageRepository

**Acceptance Criteria:**
- [ ] Constructor accepts 1 dependency
- [ ] No calls to `Bootstrap::getRepository()`

---

## Phase 4: Migrate Entry Point

**Estimated:** 30 minutes

### Task 4.1: Update web/index.php
- [ ] Read current `web/index.php`
- [ ] Add composer autoloader require
- [ ] Load `.env` file (keep this separate)
- [ ] Create Application instance with loaded config
- [ ] Call `$application->handleRequest()` instead of `Bootstrap::handleRequest()`
- [ ] Send Response and exit

**Acceptance Criteria:**
- [ ] `web/index.php` uses new Application class
- [ ] Config loaded from config/.env
- [ ] Application handles request correctly
- [ ] Response sent to browser

### Task 4.2: Mark Bootstrap as deprecated
- [ ] Add `@deprecated` tag to Bootstrap class
- [ ] Add comment referencing new Application class
- [ ] Add migration notice (backward compatible)

**Acceptance Criteria:**
- [ ] Bootstrap has `@deprecated` PHPDoc
- [ ] Comment references Application class
- [ ] File compiles with deprecation warning

---

## Phase 5: Deprecate Bootstrap

**Estimated:** 15 minutes

### Task 5.1: Add migration notice to Bootstrap
- [ ] Add @see Application class reference
- [ ] Explain why deprecated (DI architecture)
- [ ] Add migration date

**Acceptance Criteria:**
- [ ] Class marked as @deprecated
- [ ] Migration notice present in PHPDoc
- [ ] References new Application class

### Task 5.2: Keep minimal backward compatibility wrapper
- [ ] Keep Bootstrap class (don't delete)
- [ ] Add static `initialize()` method that creates Application internally
- [ ] Delegate all static methods to internal Application instance

**Acceptance Criteria:**
- [ ] Bootstrap still exists (for backward compat)
- [ ] New `initialize()` method creates Application
- [ ] Static methods delegate to internal Application instance

---

## Phase 6: Testing & Verification

**Estimated:** 1 hour

### Task 6.1: Test homepage
- [ ] Visit http://localhost:8000/
- [ ] Verify page renders correctly
- [ ] Check no PHP errors
- [ ] Verify templates load

**Acceptance Criteria:**
- [ ] Homepage loads without errors
- [ ] Article listings visible
- [ ] No console errors

### Task 6.2: Test article page
- [ ] Visit http://localhost:8000/articles
- [ ] Verify article list loads
- [ ] Click an article and verify detail page

**Acceptance Criteria:**
- [ ] Article list loads correctly
- [ ] Article detail page loads
- [ ] Routing works for /articles/{id}

### Task 6.3: Test admin panel
- [ ] Visit http://localhost:8000/admin
- [ ] Verify login page loads
- [ ] Test login functionality
- [ ] Verify dashboard loads after login

**Acceptance Criteria:**
- [ ] Admin panel accessible
- [ ] Login works
- [ ] Dashboard displays correctly

### Task 6.4: Test article management (admin)
- [ ] Create test article in admin
- [ ] Edit existing article
- [ ] Delete article
- [ ] Verify MediaRepository operations

**Acceptance Criteria:**
- [ ] Article CRUD operations work
- [ ] Data saved to content/
- [ ] No regressions from previous Bootstrap

### Task 6.5: Test media management (admin)
- [ ] Upload test image
- [ ] Upload test document
- [ ] Delete media file
- [ ] Verify folder operations

**Acceptance Criteria:**
- [ ] File upload works
- [ ] Media deletion works
- [ ] Folder creation works
- [ ] API endpoints return correct data

### Task 6.6: Run existing tests
- [ ] Run `composer test`
- [ ] Verify all tests pass
- [ ] Check test coverage
- [ ] Fix any failing tests

**Acceptance Criteria:**
- [ ] `composer test` passes
- [ ] No test regressions
- [ ] Coverage maintained or improved

### Task 6.7: Verify .env overrides
- [ ] Set APP_DEBUG=true in .env
- [ ] Verify debug mode enabled
- [ ] Set custom SITE_NAME
- [ ] Verify config takes precedence

**Acceptance Criteria:**
- [ ] .env overrides work
- [ ] Config from .env takes precedence over config.php
- [ ] Debug mode toggles correctly

### Task 6.8: Verify PHPStan compatibility
- [ ] Run PHPStan level 5 on Application class
- [ ] Run PHPStan on service providers
- [ ] Run PHPStan on updated controllers
- [ ] Fix any type errors

**Acceptance Criteria:**
- [ ] PHPStan passes with zero errors
- [ ] No new type issues introduced
- [ ] Level 5 analysis completes

---

## Summary

- **Total Tasks:** 62
- **Total Estimated Time:** 3.5-4 hours
- **Phases:** 6
- **Critical Path:** Touching application entry point (index.php)

## Blocking Issues

None identified at this time.

---

**Generated:** 2026-03-19
