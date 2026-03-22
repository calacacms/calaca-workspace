# 🐘 Modern PHP Patterns for CalacaCMS

## Overview
Modern PHP 8.1+ development patterns specifically for CalacaCMS development with strict typing and clean architecture.

## 🎯 Core Principles

### 1. Strict Typing
```php
<?php
declare(strict_types=1);

class ContentManager {
    public function __construct(
        private readonly StorageInterface $storage,
        private readonly CacheInterface $cache,
        private readonly LoggerInterface $logger
    ) {}

    public function getContent(string $id): ?array {
        return $this->storage->find('content', $id);
    }
}
```

### 2. Enums for Constants
```php
enum ContentStatus: string {
    case DRAFT = 'draft';
    case PUBLISHED = 'published';
    case ARCHIVED = 'archived';

    public function isVisible(): bool {
        return $this === self::PUBLISHED;
    }

    public function getLabel(): string {
        return match($this) {
            self::DRAFT => 'Draft',
            self::PUBLISHED => 'Published',
            self::ARCHIVED => 'Archived'
        };
    }
}

enum StorageDriver: string {
    case FLATFILE = 'flatfile';
    case SQLITE = 'sqlite';
    case MYSQL = 'mysql';
    case POSTGRESQL = 'postgresql';
}
```

### 3. Attributes for Metadata
```php
#[Attribute]
class Route {
    public function __construct(
        public readonly string $path,
        public readonly array $methods = ['GET']
    ) {}
}

class ContentController {
    #[Route('/api/content/{id}', methods: ['GET'])]
    public function getContent(string $id): Response {
        $content = $this->contentService->find($id);
        return new JsonResponse($content);
    }
}
```

## 🏗️ Architecture Patterns

### Repository Pattern (CalacaCMS Style)
```php
interface RepositoryInterface {
    public function find(string $id): ?array;
    public function findBy(array $criteria): array;
    public function all(): array;
    public function create(array $data): string;
    public function update(string $id, array $data): bool;
    public function delete(string $id): bool;
}

class PageRepository implements RepositoryInterface {
    public function __construct(
        private readonly StorageInterface $storage
    ) {}

    public function find(string $id): ?array {
        return $this->storage->find('pages', $id);
    }

    public function findBy(array $criteria): array {
        return array_filter($this->all(), function($page) use ($criteria) {
            foreach ($criteria as $key => $value) {
                if (($page[$key] ?? null) !== $value) {
                    return false;
                }
            }
            return true;
        });
    }

    public function all(): array {
        return $this->storage->all('pages');
    }

    public function create(array $data): string {
        return $this->storage->create('pages', $data);
    }

    public function update(string $id, array $data): bool {
        return $this->storage->update('pages', $id, $data);
    }

    public function delete(string $id): bool {
        return $this->storage->delete('pages', $id);
    }
}
```

### Service Layer Pattern
```php
class ContentService {
    public function __construct(
        private readonly RepositoryInterface $repository,
        private readonly CacheInterface $cache,
        private readonly LoggerInterface $logger
    ) {}

    public function publishContent(string $id): void {
        $content = $this->repository->find($id);
        if (!$content) {
            throw new ContentNotFoundException("Content with ID {$id} not found");
        }

        $content['status'] = ContentStatus::PUBLISHED->value;
        $content['published_at'] = date('Y-m-d H:i:s');
        
        $this->repository->update($id, $content);
        $this->cache->clear("content_{$id}");
        
        $this->logger->info('Content published', ['id' => $id, 'title' => $content['title']]);
    }

    public function createPage(array $data): string {
        $validator = new ContentValidator();
        $result = $validator->validate($data);
        
        if (!$result->isValid) {
            throw new ValidationException('Invalid content data', $result->errors);
        }
        
        $data['id'] = $this->generateId();
        $data['status'] = ContentStatus::DRAFT->value;
        $data['created_at'] = date('Y-m-d H:i:s');
        
        return $this->repository->create($data);
    }

    private function generateId(): string {
        return sprintf('%04x%04x-%04x-%04x-%04x-%04x%04x%04x',
            mt_rand(0, 0xffff), mt_rand(0, 0xffff),
            mt_rand(0, 0xffff),
            mt_rand(0, 0x0fff) | 0x4000,
            mt_rand(0, 0x3fff) | 0x8000,
            mt_rand(0, 0xffff), mt_rand(0, 0xffff), mt_rand(0, 0xffff)
        );
    }
}
```

## 🔒 Security Patterns

### Input Validation
```php
class ContentValidator {
    public function validate(array $data): ValidationResult {
        $errors = [];

        // Title validation
        if (empty($data['title'])) {
            $errors[] = 'Title is required';
        } elseif (strlen($data['title']) > 255) {
            $errors[] = 'Title too long (max 255 characters)';
        }

        // Slug validation
        if (!empty($data['slug'])) {
            if (!preg_match('/^[a-z0-9-]+$/', $data['slug'])) {
                $errors[] = 'Invalid slug format (only lowercase letters, numbers, and hyphens)';
            } elseif (strlen($data['slug']) > 255) {
                $errors[] = 'Slug too long (max 255 characters)';
            }
        }

        // Content validation
        if (isset($data['content']) && strlen($data['content']) > 1000000) {
            $errors[] = 'Content too long (max 1MB)';
        }

        return new ValidationResult(empty($errors), $errors);
    }
}

class ValidationResult {
    public function __construct(
        public readonly bool $isValid,
        public readonly array $errors
    ) {}
}
```

### CSRF Protection
```php
class CsrfProtection {
    public function __construct(
        private readonly SessionService $session
    ) {}

    public function generateToken(): string {
        $token = bin2hex(random_bytes(32));
        $this->session->set('csrf_token', $token);
        return $token;
    }

    public function validateToken(string $token): bool {
        $storedToken = $this->session->get('csrf_token');
        return hash_equals($storedToken ?? '', $token);
    }
}
```

### Rate Limiting
```php
class RateLimiter {
    public function __construct(
        private readonly CacheInterface $cache
    ) {}

    public function isAllowed(string $identifier, int $maxAttempts = 5, int $window = 300): bool {
        $key = "rate_limit_{$identifier}";
        $attempts = $this->cache->get($key, 0);
        
        if ($attempts >= $maxAttempts) {
            return false;
        }
        
        $this->cache->set($key, $attempts + 1, $window);
        return true;
    }

    public function reset(string $identifier): void {
        $this->cache->delete("rate_limit_{$identifier}");
    }
}
```
## ⚡ Performance Patterns

### Caching Strategy
```php
class CachedContentService implements RepositoryInterface {
    public function __construct(
        private readonly RepositoryInterface $repository,
        private readonly CacheInterface $cache
    ) {}

    public function find(string $id): ?array {
        $cacheKey = "content_{$id}";
        
        return $this->cache->get($cacheKey, function() use ($id) {
            return $this->repository->find($id);
        }, 3600); // 1 hour cache
    }

    public function findBy(array $criteria): array {
        $cacheKey = 'content_' . md5(serialize($criteria));
        
        return $this->cache->get($cacheKey, function() use ($criteria) {
            return $this->repository->findBy($criteria);
        }, 1800); // 30 min cache
    }

    public function create(array $data): string {
        $id = $this->repository->create($data);
        $this->cache->clear('content_'); // Clear all content cache
        return $id;
    }

    public function update(string $id, array $data): bool {
        $result = $this->repository->update($id, $data);
        if ($result) {
            $this->cache->delete("content_{$id}");
        }
        return $result;
    }

    public function delete(string $id): bool {
        $result = $this->repository->delete($id);
        if ($result) {
            $this->cache->delete("content_{$id}");
        }
        return $result;
    }

    public function all(): array {
        return $this->cache->get('content_all', function() {
            return $this->repository->all();
        }, 1800);
    }
}
```

### Lazy Loading
```php
class Content {
    private ?array $author = null;
    private ?array $categories = null;

    public function __construct(
        public readonly array $data,
        private readonly RepositoryInterface $repository
    ) {}

    public function getAuthor(): ?array {
        if ($this->author === null && isset($this->data['author_id'])) {
            $this->author = $this->repository->find('users', $this->data['author_id']);
        }
        return $this->author;
    }

    public function getCategories(): array {
        if ($this->categories === null) {
            $this->categories = $this->repository->findBy(['content_id' => $this->data['id']]);
        }
        return $this->categories;
    }
}
```

## 🧪 Testing Patterns

### Unit Testing
```php
class ContentServiceTest extends TestCase {
    private RepositoryInterface $repository;
    private CacheInterface $cache;
    private LoggerInterface $logger;
    private ContentService $service;

    protected function setUp(): void {
        $this->repository = $this->createMock(RepositoryInterface::class);
        $this->cache = $this->createMock(CacheInterface::class);
        $this->logger = $this->createMock(LoggerInterface::class);
        
        $this->service = new ContentService($this->repository, $this->cache, $this->logger);
    }

    public function testPublishContent(): void {
        $content = ['id' => '123', 'title' => 'Test Content', 'status' => 'draft'];
        
        $this->repository->expects($this->once())
            ->method('find')
            ->with('123')
            ->willReturn($content);
            
        $this->repository->expects($this->once())
            ->method('update')
            ->with('123', $this->anything())
            ->willReturn(true);
            
        $this->cache->expects($this->once())
            ->method('clear')
            ->with('content_123');
            
        $this->logger->expects($this->once())
            ->method('info')
            ->with('Content published', $this->anything());

        $this->service->publishContent('123');
        
        $this->assertEquals('published', $content['status']);
        $this->assertArrayHasKey('published_at', $content);
    }

    public function testCreatePageWithInvalidData(): void {
        $this->expectException(ValidationException::class);
        $this->expectExceptionMessage('Invalid content data');
        
        $invalidData = ['content' => 'Some content']; // Missing title
        
        $this->service->createPage($invalidData);
    }
}
```

### Integration Testing
```php
class StorageIntegrationTest extends TestCase {
    private StorageInterface $storage;

    protected function setUp(): void {
        $this->storage = new FlatFileStorage('/tmp/test_content');
        $this->storage->clear();
    }

    public function testFullContentLifecycle(): void {
        // Create
        $data = ['title' => 'Test Page', 'slug' => 'test-page', 'content' => 'Test content'];
        $id = $this->storage->create('pages', $data);
        $this->assertNotEmpty($id);
        
        // Read
        $result = $this->storage->find('pages', $id);
        $this->assertNotNull($result);
        $this->assertEquals('Test Page', $result['title']);
        
        // Update
        $this->storage->update('pages', $id, ['title' => 'Updated Test Page']);
        $updated = $this->storage->find('pages', $id);
        $this->assertEquals('Updated Test Page', $updated['title']);
        
        // Delete
        $this->storage->delete('pages', $id);
        $deleted = $this->storage->find('pages', $id);
        $this->assertNull($deleted);
    }
}
```

## 📦 Dependency Injection

### Bootstrap Service Container (CalacaCMS Style)
```php
class Bootstrap {
    private static array $services = [];
    private static array $singletons = [];

    public static function register(string $name, callable $factory, bool $singleton = false): void {
        self::$services[$name] = ['factory' => $factory, 'singleton' => $singleton];
    }

    public static function get(string $name): object {
        if (isset(self::$singletons[$name])) {
            return self::$singletons[$name];
        }

        if (!isset(self::$services[$name])) {
            throw new InvalidArgumentException("Service '{$name}' not registered");
        }

        $service = self::$services[$name];
        $instance = ($service['factory'])();

        if ($service['singleton']) {
            self::$singletons[$name] = $instance;
        }

        return $instance;
    }

    public static function initialize(): void {
        // Register storage
        self::register('storage', function() {
            $config = self::getConfig('storage');
            return StorageFactory::create($config);
        }, true);

        // Register cache
        self::register('cache', function() {
            return new FileCache('/storage/cache');
        }, true);

        // Register repositories
        self::register('pageRepository', function() {
            return new PageRepository(self::get('storage'));
        });

        self::register('articleRepository', function() {
            return new ArticleRepository(self::get('storage'));
        });

        // Register services
        self::register('contentService', function() {
            return new ContentService(
                self::get('pageRepository'),
                self::get('cache'),
                self::get('logger')
            );
        });
    }
}
```

### Constructor Injection
```php
class PageController {
    public function __construct(
        private readonly PageRepository $repository,
        private readonly ContentService $contentService,
        private readonly ResponseFactory $responseFactory,
        private readonly LoggerInterface $logger
    ) {}

    public function show(string $slug): Response {
        try {
            $criteria = ['slug' => $slug, 'status' => ContentStatus::PUBLISHED->value];
            $page = $this->repository->findBy($criteria);
            
            if (empty($page)) {
                return $this->responseFactory->notFound('Page not found');
            }
            
            return $this->responseFactory->render('page.twig', ['page' => $page[0]]);
        } catch (Exception $e) {
            $this->logger->error('Error loading page', ['slug' => $slug, 'error' => $e->getMessage()]);
            return $this->responseFactory->serverError('An error occurred');
        }
    }
}
```

## 🔄 Event-Driven Architecture

### Event System
```php
interface EventDispatcherInterface {
    public function dispatch(object $event): void;
    public function addListener(string $eventName, callable $listener): void;
}

class EventDispatcher implements EventDispatcherInterface {
    private array $listeners = [];

    public function dispatch(object $event): void {
        $eventName = get_class($event);
        
        foreach ($this->listeners[$eventName] ?? [] as $listener) {
            $listener($event);
        }
    }

    public function addListener(string $eventName, callable $listener): void {
        $this->listeners[$eventName][] = $listener;
    }
}

// Event classes
class ContentPublishedEvent {
    public function __construct(
        public readonly array $content,
        public readonly DateTimeImmutable $publishedAt
    ) {}
}

class ContentCreatedEvent {
    public function __construct(
        public readonly string $id,
        public readonly array $data
    ) {}
}
```

### Event Listeners
```php
class CacheInvalidationListener {
    public function __construct(private readonly CacheInterface $cache) {}

    public function onContentChanged(object $event): void {
        if ($event instanceof ContentPublishedEvent || $event instanceof ContentCreatedEvent) {
            $this->cache->clear('content_');
        }
    }
}

class SearchIndexListener {
    public function __construct(private readonly SearchService $search) {}

    public function onContentPublished(ContentPublishedEvent $event): void {
        $this->search->indexContent($event->content);
    }
}
```

### Service with Events
```php
class ContentService {
    public function __construct(
        private readonly RepositoryInterface $repository,
        private readonly EventDispatcherInterface $dispatcher
    ) {}

    public function publishContent(string $id): void {
        $content = $this->repository->find($id);
        if (!$content) {
            throw new ContentNotFoundException("Content with ID {$id} not found");
        }

        $content['status'] = ContentStatus::PUBLISHED->value;
        $content['published_at'] = date('Y-m-d H:i:s');
        
        $this->repository->update($id, $content);
        
        // Dispatch events
        $this->dispatcher->dispatch(new ContentPublishedEvent($content, new DateTimeImmutable()));
    }
}
```

## 📋 Best Practices Checklist

- [ ] Always use `declare(strict_types=1)`
- [ ] Implement proper error handling with try-catch blocks
- [ ] Use dependency injection with constructor injection
- [ ] Validate all input data before processing
- [ ] Implement caching where appropriate for performance
- [ ] Write comprehensive unit tests for all services
- [ ] Use interfaces for all abstractions
- [ ] Implement event-driven patterns for loose coupling
- [ ] Follow PSR-12 coding standards
- [ ] Use type hints everywhere (parameters and return types)
- [ ] Use readonly properties for immutable data
- [ ] Implement proper logging for debugging and monitoring
- [ ] Use enums instead of string constants
- [ ] Implement proper exception handling with custom exceptions

## 🚀 Performance Tips

1. **Use readonly properties** for immutable data structures
2. **Implement lazy loading** for expensive operations and relationships
3. **Cache frequently accessed data** with appropriate TTL
4. **Use database indexes** for common query patterns
5. **Implement connection pooling** for database connections
6. **Use OPcache** in production for PHP code caching
7. **Profile your code** regularly to identify bottlenecks
8. **Use array functions** instead of loops where possible
9. **Implement proper error handling** to avoid expensive exception stack traces
10. **Use event-driven architecture** to decouple expensive operations

## 🔧 CalacaCMS Specific Patterns

### Storage Factory Pattern
```php
class StorageFactory {
    public static function create(array $config): StorageInterface {
        return match($config['driver']) {
            'flatfile' => new FlatFileStorage($config['path']),
            'sqlite' => new DatabaseStorage('sqlite:' . $config['path']),
            'mysql' => new DatabaseStorage("mysql:host={$config['host']};dbname={$config['database']}", $config['username'], $config['password']),
            'postgresql' => new DatabaseStorage("pgsql:host={$config['host']};dbname={$config['database']}", $config['username'], $config['password']),
            default => throw new InvalidArgumentException("Unsupported storage driver: {$config['driver']}")
        };
    }
}
```

### Bootstrap Service Registration
```php
// In Bootstrap::initialize()
self::register('storage', function() {
    $config = self::getConfig('storage');
    return StorageFactory::create($config);
}, true);

self::register('pageRepository', function() {
    return new PageRepository(self::get('storage'));
});

self::register('contentService', function() {
    return new ContentService(
        self::get('pageRepository'),
        self::get('cache'),
        self::get('logger')
    );
});
```

## 🔧 Tools & Libraries Used by CalacaCMS

- **Composer** - Dependency management
- **PHPStan** - Static analysis (level 5)
- **PHPUnit** - Testing framework (281 tests)
- **Symfony Routing** - URL routing and HTTP components
- **Twig** - Template engine
- **League Container** - Dependency injection
- **Monolog** - Logging framework
- **vlucas/phpdotenv** - Environment configuration
