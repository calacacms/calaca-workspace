# Bootstrap.php Refactoring PRD

**Generated:** 2026-03-19
**Target File:** calaca/src/Core/Bootstrap.php (506 lines)
**Priority:** CRITICAL (Score: 4.1)

---

## Problem Statement

Bootstrap.php is a **God class** that violates multiple architectural principles:

1. **Static Method Anti-pattern**: All methods are static, making unit testing impossible
2. **Service Locator Pattern**: Uses `getRepository()` and `getController()` factory methods instead of proper dependency injection
3. **Mixed Responsibilities**: Handles configuration, storage, cache, logger, routing, templates, repositories, controllers - 11+ concerns in one class
4. **Global State**: Uses private static properties to hold singletons, creating hidden dependencies
5. **Tight Coupling**: All controllers are instantiated within Bootstrap, making it impossible to inject dependencies
6. **No Dependency Injection**: Controllers receive dependencies as constructor parameters, but Bootstrap instantiates them with its own resources

**Impact:**
- **Risk**: 5 (Critical) - Changes affect entire application startup
- **Testability**: Cannot mock services or controllers due to static methods
- **Maintainability**: Adding new services requires modifying Bootstrap directly
- **Extensibility**: Difficult to override initialization logic
- **Modern PHP**: Violates DI container best practices (League Container ^4.0)

---

## Current Architecture Issues

### Static State Management
```php
private static ?StorageInterface $storage = null;
private static ?CacheInterface $cache = null;
private static ?LoggerInterface $logger = null;
private static array $repositories = [];
private static array $contentTypes = [];
private static array $config = [];
private static ?RouteCollection $routes = null;
private static ?RequestContext $requestContext = null;
private static ?UsersRepository $usersRepository = null;
private static array $controllers = [];
```

### Service Locator Pattern
```php
// Bad: Service locator
public static function getRepository(string $name): RepositoryInterface {
    return self::$repositories[$name] ?? throw new \RuntimeException(...);
}

// Bad: Controller factory
public static function getController(string $name): object {
    return self::$controllers[$name] ?? throw new \RuntimeException(...);
}
```

### Manual Dependency Instantiation
```php
// Controllers instantiated manually with manual dependencies
self::$controllers['admin.articles'] = new AdminArticleController(
    self::$repositories['articles'],
    self::$repositories['pages'],
    null,
    $contentTypeManager
);
```

---

## Proposed Solution

### New Architecture: Application Class with Dependency Injection

Create a new `Application` class that:
1. **Uses League Container** for dependency injection
2. **Holds services as instance properties** (not static)
3. **Provides controllers via Container** (not manual instantiation)
4. **Implements proper initialization lifecycle**
5. **Makes the entire application testable**

```
Before:
Bootstrap::handleRequest($request)
    └─> Manual controller instantiation
        └─> Static service access

After:
$application = new Application($config);
$application->handleRequest($request)
    └─> Container resolves controller
        └─> Properly injected dependencies
```

---

## Implementation Plan

### Phase 1: Create Application Class
**Estimated:** 1 hour
- Create `calaca/src/Core/Application.php`
- Accept config array in constructor
- Use League Container internally
- Provide methods: `handleRequest()`, `run()`
- Implement service registration with Container

**Key Methods:**
```php
class Application {
    private Container $container;
    private array $config;
    private RouteCollection $routes;
    private RequestContext $requestContext;

    public function __construct(array $config, Container $container = null)
    public function handleRequest(Request $request): Response
    public function run(): void
    private function initialize(): void
    private function registerServices(): void
    private function registerRoutes(): void
}
```

### Phase 2: Extract Service Providers
**Estimated:** 30 minutes
- Create `ServiceProviderInterface` and base `ServiceProvider` abstract class
- Create service providers for:
  - `SessionServiceProvider`
  - `StorageServiceProvider`
  - `CacheServiceProvider`
  - `LoggerServiceProvider`
  - `TemplateServiceProvider`
  - `RepositoryServiceProvider`
  - `RoutingServiceProvider`
  - `ControllerServiceProvider`

**Purpose:** Centralize service registration logic

### Phase 3: Update Controllers for DI
**Estimated:** 1 hour
- Update all controller constructors to accept dependencies via Container
- Remove manual instantiation from Bootstrap
- Controllers remain as-is but receive properly injected services
- Key changes:
  ```php
  // Remove this:
  self::$controllers['admin.media'] = new AdminMediaController(...);

  // Controllers will be resolved by Container
  $container->get(AdminMediaController::class);
  ```

### Phase 4: Migrate Entry Point
**Estimated:** 30 minutes
- Update `web/index.php` to use Application class
- Keep `.env` loading (separate concern)
- Replace `Bootstrap::handleRequest()` with `$application->handleRequest()`
- Ensure backward compatibility

### Phase 5: Deprecate Bootstrap
**Estimated:** 15 minutes
- Mark Bootstrap as `@deprecated`
- Add migration guide comments
- Keep minimal wrapper for backward compatibility

### Phase 6: Testing & Verification
**Estimated:** 1 hour
- Test all routes still work
- Verify controllers receive dependencies
- Check session/storage/cache initialization
- Verify .env overrides still apply
- Manual browser testing of admin panel

---

## Risk Mitigation

1. **Backward Compatibility**: Keep Bootstrap with @deprecated tag
2. **Incremental Rollout**: Can revert by reverting index.php changes
3. **Feature Flag**: Optional flag to use old Bootstrap vs new Application
4. **Comprehensive Testing**: Test all functionality before committing
5. **Container Caching**: Reuse League Container (no performance impact)

---

## Success Criteria

- [ ] Application class created with proper DI
- [ ] All service providers implemented
- [ ] Controllers updated to receive DI
- [ ] index.php updated to use Application
- [ ] All existing tests pass
- [ ] No regressions in manual testing
- [ ] Bootstrap marked as deprecated with migration guide
- [ ] Code compiles with zero PHPStan errors (level 5)

---

## Dependencies

- League\Container: ^4.0 (already in use)
- All existing services: No changes required
- Controllers: Constructor updates only
- Symfony Routing/HttpFoundation: No changes required

---

**Total Estimated Time:** 3.5-4 hours
**Risk Level:** HIGH (requires touching application entry point)
**Reward:** Modern, testable architecture aligned with DI container best practices
