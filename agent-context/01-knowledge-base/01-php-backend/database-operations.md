# 🗄️ Database Operations for CalacaCMS

## Overview
Modern database patterns and optimizations for CalacaCMS supporting both flat-file and database storage.

## 🎯 Storage Abstraction Layer

CalacaCMS uses a storage abstraction that supports multiple drivers:

```php
interface StorageInterface {
    public function find(string $type, string $id): ?array;
    public function findBy(string $type, array $criteria): array;
    public function all(string $type): array;
    public function create(string $type, array $data): string;
    public function update(string $type, string $id, array $data): bool;
    public function delete(string $type, string $id): bool;
}
```

### Available Storage Drivers
- **FlatFileStorage** - JSON files (default for development)
- **DatabaseStorage** - MySQL, PostgreSQL, SQLite (production)
- **HybridStorage** - Mix of flat-file and database (future)

## 🏗️ Database Schema Design

### Content Table Structure
```sql
-- Content table for database storage
CREATE TABLE content (
    id VARCHAR(36) PRIMARY KEY,                 -- UUID for consistency with flat-file
    type VARCHAR(50) NOT NULL,                   -- 'page', 'article', 'user'
    slug VARCHAR(255) NOT NULL,
    data JSON NOT NULL,                          -- Store content as JSON (flexible schema)
    status ENUM('draft', 'published', 'archived') DEFAULT 'draft',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_type_status (type, status),
    INDEX idx_slug (slug),
    INDEX idx_created_at (created_at),
    FULLTEXT idx_search (slug, (JSON_EXTRACT(data, '$.title')))
);

-- Categories table (optional)
CREATE TABLE categories (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    parent_id VARCHAR(36) NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (parent_id) REFERENCES categories(id) ON DELETE SET NULL
);

-- Content-Category relationship
CREATE TABLE content_categories (
    content_id VARCHAR(36),
    category_id VARCHAR(36),
    PRIMARY KEY (content_id, category_id),
    FOREIGN KEY (content_id) REFERENCES content(id) ON DELETE CASCADE,
    FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE CASCADE
);
```

### Indexing Strategy for Performance
```sql
-- Composite indexes for common queries
CREATE INDEX idx_content_type_status_created ON content(type, status, created_at);
CREATE INDEX idx_content_category ON content_categories(category_id, content_id);

-- Partial indexes for published content
CREATE INDEX idx_published_pages ON content(created_at) WHERE type = 'page' AND status = 'published';
CREATE INDEX idx_published_articles ON content(created_at) WHERE type = 'article' AND status = 'published';

-- JSON indexes for content fields (MySQL 5.7+)
CREATE INDEX idx_content_title ON content((JSON_EXTRACT(data, '$.title')));
CREATE INDEX idx_content_author ON content((JSON_EXTRACT(data, '$.author')));
```

## 🏗️ Repository Pattern Implementation

### Base Repository Interface
```php
interface RepositoryInterface {
    public function find(string $id): ?array;
    public function findBy(array $criteria): array;
    public function all(): array;
    public function create(array $data): string;
    public function update(string $id, array $data): bool;
    public function delete(string $id): bool;
}
```

### Database Repository Implementation
```php
class DatabaseRepository implements RepositoryInterface {
    public function __construct(
        private PDO $pdo,
        private string $contentType
    ) {}

    public function find(string $id): ?array {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM content WHERE type = :type AND id = :id'
        );
        $stmt->execute(['type' => $this->contentType, 'id' => $id]);
        $result = $stmt->fetch(PDO::FETCH_ASSOC);
        
        return $result ? json_decode($result['data'], true) : null;
    }

    public function findBy(array $criteria): array {
        $where = ['type = :type'];
        $params = ['type' => $this->contentType];
        
        foreach ($criteria as $field => $value) {
            $where[] = "JSON_EXTRACT(data, '$.{$field}') = :{$field}";
            $params[$field] = $value;
        }
        
        $sql = 'SELECT * FROM content WHERE ' . implode(' AND ', $where);
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute($params);
        
        return array_map(fn($row) => json_decode($row['data'], true), $stmt->fetchAll());
    }

    public function create(array $data): string {
        $id = $this->generateId();
        $stmt = $this->pdo->prepare(
            'INSERT INTO content (id, type, slug, data, status) VALUES (:id, :type, :slug, :data, :status)'
        );
        
        $stmt->execute([
            'id' => $id,
            'type' => $this->contentType,
            'slug' => $data['slug'] ?? $id,
            'data' => json_encode($data),
            'status' => $data['status'] ?? 'draft'
        ]);
        
        return $id;
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

## ⚡ Performance Optimization

### 1. Query Optimization
```php
// Efficient content queries with indexes
class ContentRepository {
    public function findPublished(string $type = 'page'): array
    {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM content WHERE type = :type AND status = :status ORDER BY created_at DESC'
        );
        $stmt->execute(['type' => $type, 'status' => 'published']);
        
        return array_map(fn($row) => json_decode($row['data'], true), $stmt->fetchAll());
    }

    public function search(string $query): array
    {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM content WHERE status = :status AND MATCH(slug, JSON_EXTRACT(data, "$$.title")) AGAINST(:query IN BOOLEAN MODE)'
        );
        $stmt->execute(['status' => 'published', 'query' => $query]);
        
        return array_map(fn($row) => json_decode($row['data'], true), $stmt->fetchAll());
    }
}
```

### 2. Pagination Implementation
```php
class PaginatedContentService {
    public function getPaginatedContent(string $type = 'page', int $page = 1, int $limit = 10): array
    {
        $offset = ($page - 1) * $limit;
        
        // Get paginated results
        $stmt = $this->pdo->prepare(
            'SELECT * FROM content WHERE type = :type AND status = :status ORDER BY created_at DESC LIMIT :limit OFFSET :offset'
        );
        $stmt->execute([
            'type' => $type,
            'status' => 'published',
            'limit' => $limit,
            'offset' => $offset
        ]);
        
        $content = array_map(fn($row) => json_decode($row['data'], true), $stmt->fetchAll());
        
        // Get total count
        $countStmt = $this->pdo->prepare(
            'SELECT COUNT(*) FROM content WHERE type = :type AND status = :status'
        );
        $countStmt->execute(['type' => $type, 'status' => 'published']);
        $total = $countStmt->fetchColumn();
        
        return [
            'content' => $content,
            'pagination' => [
                'current_page' => $page,
                'per_page' => $limit,
                'total' => $total,
                'total_pages' => ceil($total / $limit)
            ]
        ];
    }
}
```

### 3. Caching Strategy
```php
class CachedRepository implements RepositoryInterface {
    public function __construct(
        private RepositoryInterface $repository,
        private CacheInterface $cache
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
}
```

## 🔒 Security Patterns

### 1. SQL Injection Prevention
```php
// Always use prepared statements
class SecureRepository implements RepositoryInterface {
    public function findBySlug(string $slug): ?array {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM content WHERE type = :type AND slug = :slug AND status = :status'
        );
        $stmt->execute([
            'type' => $this->contentType,
            'slug' => $slug,
            'status' => 'published'
        ]);
        
        $result = $stmt->fetch(PDO::FETCH_ASSOC);
        return $result ? json_decode($result['data'], true) : null;
    }

    public function search(string $query): array {
        // Validate and sanitize search query
        $query = trim($query);
        if (strlen($query) < 3 || strlen($query) > 100) {
            return [];
        }
        
        $stmt = $this->pdo->prepare(
            'SELECT * FROM content WHERE type = :type AND status = :status AND MATCH(slug, JSON_EXTRACT(data, "$$.title")) AGAINST(:query IN BOOLEAN MODE)'
        );
        $stmt->execute([
            'type' => $this->contentType,
            'status' => 'published',
            'query' => $query
        ]);
        
        return array_map(fn($row) => json_decode($row['data'], true), $stmt->fetchAll());
    }
}
```

### 2. Input Validation
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

## 📊 Performance Monitoring

### 1. Query Logging
```php
class DatabaseLogger {
    public function __construct(private LoggerInterface $logger) {}
    
    public function logQuery(string $sql, array $params, float $executionTime): void {
        if ($executionTime > 1.0) { // Log slow queries
            $this->logger->warning('Slow query detected', [
                'sql' => $sql,
                'params' => $params,
                'execution_time' => $executionTime
            ]);
        }
    }
}

// Usage with PDO wrapper
class LoggingPDO extends PDO {
    public function __construct(
        private PDO $pdo,
        private DatabaseLogger $logger
    ) {
        parent::__construct('sqlite::memory:');
    }
    
    public function prepare(string $query, array $options = []): PDOStatement|false {
        $start = microtime(true);
        $stmt = $this->pdo->prepare($query, $options);
        $this->logger->logQuery($query, [], microtime(true) - $start);
        return $stmt;
    }
}
```

### 2. Connection Management
```php
class DatabaseConnectionFactory {
    private static array $connections = [];
    
    public static function create(array $config): PDO {
        $key = md5(serialize($config));
        
        if (!isset(self::$connections[$key])) {
            $dsn = match($config['driver']) {
                'mysql' => "mysql:host={$config['host']};dbname={$config['database']};charset=utf8mb4",
                'pgsql' => "pgsql:host={$config['host']};dbname={$config['database']}",
                'sqlite' => "sqlite:" . $config['path'],
                default => throw new InvalidArgumentException("Unsupported driver: {$config['driver']}")
            };
            
            self::$connections[$key] = new PDO($dsn, $config['username'] ?? null, $config['password'] ?? null, [
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
                PDO::ATTR_EMULATE_PREPARES => false,
                PDO::ATTR_PERSISTENT => true
            ]);
        }
        
        return self::$connections[$key];
    }
}
```

## 🧪 Testing Database Operations

### 1. Unit Tests with SQLite In-Memory
```php
class DatabaseRepositoryTest extends TestCase {
    private PDO $pdo;
    private DatabaseRepository $repository;

    protected function setUp(): void {
        // Create in-memory SQLite database
        $this->pdo = new PDO('sqlite::memory:');
        $this->pdo->exec("
            CREATE TABLE content (
                id VARCHAR(36) PRIMARY KEY,
                type VARCHAR(50) NOT NULL,
                slug VARCHAR(255) NOT NULL,
                data JSON NOT NULL,
                status VARCHAR(20) DEFAULT 'draft',
                created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
                updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
            );
            CREATE INDEX idx_type_status ON content(type, status);
            CREATE INDEX idx_slug ON content(slug);
        ");
        
        $this->repository = new DatabaseRepository($this->pdo, 'page');
    }

    public function testCreateAndFind(): void {
        $data = [
            'title' => 'Test Page',
            'slug' => 'test-page',
            'content' => 'Test content',
            'status' => 'published'
        ];
        
        $id = $this->repository->create($data);
        $result = $this->repository->find($id);
        
        $this->assertNotNull($result);
        $this->assertEquals('Test Page', $result['title']);
        $this->assertEquals('test-page', $result['slug']);
    }

    public function testFindByCriteria(): void {
        // Create test data
        $this->repository->create(['title' => 'Page 1', 'slug' => 'page-1', 'status' => 'published']);
        $this->repository->create(['title' => 'Page 2', 'slug' => 'page-2', 'status' => 'draft']);
        
        $published = $this->repository->findBy(['status' => 'published']);
        $draft = $this->repository->findBy(['status' => 'draft']);
        
        $this->assertCount(1, $published);
        $this->assertCount(1, $draft);
        $this->assertEquals('Page 1', $published[0]['title']);
        $this->assertEquals('Page 2', $draft[0]['title']);
    }
}
```

### 2. Integration Tests with Test Database
```php
class StorageIntegrationTest extends TestCase {
    private StorageInterface $storage;

    protected function setUp(): void {
        // Use test database configuration
        $config = [
            'driver' => 'sqlite',
            'path' => ':memory:'
        ];
        
        $this->storage = StorageFactory::create($config);
        $this->initializeDatabase($this->storage);
    }

    public function testStorageOperations(): void {
        // Test create
        $data = ['title' => 'Test', 'content' => 'Content'];
        $id = $this->storage->create('pages', $data);
        $this->assertNotEmpty($id);
        
        // Test find
        $result = $this->storage->find('pages', $id);
        $this->assertNotNull($result);
        $this->assertEquals('Test', $result['title']);
        
        // Test update
        $this->storage->update('pages', $id, ['title' => 'Updated Test']);
        $updated = $this->storage->find('pages', $id);
        $this->assertEquals('Updated Test', $updated['title']);
        
        // Test delete
        $this->storage->delete('pages', $id);
        $deleted = $this->storage->find('pages', $id);
        $this->assertNull($deleted);
    }
}
```

## 📋 Best Practices Checklist

- [ ] Always use prepared statements for database queries
- [ ] Implement proper indexing strategy for common queries
- [ ] Use connection pooling/persistence for performance
- [ ] Cache frequently accessed data with appropriate TTL
- [ ] Monitor and log slow queries (>1 second)
- [ ] Implement pagination for large datasets
- [ ] Validate all input data before database operations
- [ ] Use transactions for complex multi-table operations
- [ ] Implement database migrations for schema changes
- [ ] Test database operations with in-memory databases
- [ ] Use UUIDs for consistency between flat-file and database storage
- [ ] Store flexible schema data as JSON in database
- [ ] Implement proper error handling and logging

## 🚀 Performance Tips

1. **Use database indexes** for common query patterns
2. **Implement query caching** where appropriate
3. **Use connection persistence** for better performance
4. **Monitor slow queries** regularly and optimize them
5. **Use database views** for complex queries
6. **Implement read replicas** for scaling (future)
7. **Use database partitioning** for large tables (future)
8. **Store content as JSON** for flexibility like flat-files
9. **Use UUIDs** instead of auto-increment for consistency
10. **Implement proper connection management** with factory pattern

## 🔧 Storage Driver Configuration

### FlatFile Storage (Default)
```json
{
  "storage": {
    "driver": "flatfile",
    "path": "content"
  }
}
```

### Database Storage (Production)
```json
{
  "storage": {
    "driver": "mysql",
    "host": "localhost",
    "database": "calacacms",
    "username": "user",
    "password": "secret"
  }
}
```

### SQLite Storage (Development)
```json
{
  "storage": {
    "driver": "sqlite",
    "path": "storage/database.sqlite"
  }
}
```
