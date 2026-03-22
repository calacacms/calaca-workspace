# CalacaCMS Storage Drivers

## Overview

CalacaCMS supports multiple storage drivers through a unified interface. This allows you to choose the storage solution that best fits your needs and easily switch between them.

## Supported Storage Drivers

### 1. FlatFileStorage (Default)
JSON-based file storage for simple projects and development.

**Pros:**
- No database setup required
- Easy to inspect and edit data
- Great for small projects and prototypes
- Portable (just copy files)

**Cons:**
- Limited performance for large datasets
- No advanced features (transactions are simulated)
- Not suitable for production with high traffic

**Usage:**
```php
use CalacaCMS\Storage\FlatFileStorage;

$storage = new FlatFileStorage('/path/to/storage');
```

### 2. SQLiteStorage
Lightweight SQL database perfect for small to medium projects.

**Pros:**
- Zero configuration (serverless)
- SQL support with ACID compliance
- Great for embedded applications
- Easy backup (single file)

**Cons:**
- Limited concurrent write performance
- Not ideal for high-traffic production sites

**Usage:**
```php
use CalacaCMS\Storage\SQLiteStorage;

$storage = new SQLiteStorage('/path/to/database.sqlite');
```

### 3. MySQLStorage
Full-featured MySQL database for production applications.

**Pros:**
- High performance and scalability
- Advanced features (transactions, indexes, etc.)
- Production-ready
- Large ecosystem and tools

**Cons:**
- Requires MySQL server setup
- More configuration needed

**Usage:**
```php
use CalacaCMS\Storage\MySQLStorage;

$storage = new MySQLStorage(
    'localhost',    // host
    'calacacms',    // database name
    'username',     // username
    'password',     // password
    'utf8mb4'       // charset (optional)
);
```

### 4. PostgreSQLStorage
Enterprise-grade PostgreSQL database.

**Pros:**
- Advanced SQL features
- Excellent performance
- ACID compliant
- Production-ready

**Cons:**
- Requires PostgreSQL server setup
- More complex configuration

**Usage:**
```php
use CalacaCMS\Storage\PostgreSQLStorage;

$storage = new PostgreSQLStorage(
    'localhost',    // host
    'calacacms',    // database name
    'username',     // username
    'password',     // password
    '5432',         // port (optional)
    'public'        // schema (optional)
);
```

## Storage Interface

All storage drivers implement the `StorageInterface` with the following methods:

```php
interface StorageInterface
{
    public function getConnection(): mixed;
    public function beginTransaction(): void;
    public function commit(): void;
    public function rollback(): void;
    public function select(string $table, array $where = [], array $order = [], int $limit = 0, int $offset = 0): array;
    public function insert(string $table, array $data): int;
    public function update(string $table, array $data, array $where = []): bool;
    public function delete(string $table, array $where = []): bool;
}
```

## Storage Factory

The `StorageFactory` provides a convenient way to create storage drivers:

### From Configuration Array

```php
use CalacaCMS\Storage\StorageFactory;

// FlatFile
$config = [
    'driver' => 'flatfile',
    'path' => '/path/to/storage',
];
$storage = StorageFactory::create($config);

// SQLite
$config = [
    'driver' => 'sqlite',
    'path' => '/path/to/database.sqlite',
];
$storage = StorageFactory::create($config);

// MySQL
$config = [
    'driver' => 'mysql',
    'host' => 'localhost',
    'database' => 'calacacms',
    'username' => 'root',
    'password' => '',
];
$storage = StorageFactory::create($config);

// PostgreSQL
$config = [
    'driver' => 'postgresql',
    'host' => 'localhost',
    'database' => 'calacacms',
    'username' => 'postgres',
    'password' => '',
];
$storage = StorageFactory::create($config);
```

### From Environment Variables

```php
use CalacaCMS\Storage\StorageFactory;

// Set environment variables
$_ENV['STORAGE_DRIVER'] = 'mysql';
$_ENV['DB_HOST'] = 'localhost';
$_ENV['DB_DATABASE'] = 'calacacms';
$_ENV['DB_USERNAME'] = 'root';
$_ENV['DB_PASSWORD'] = '';

$storage = StorageFactory::createFromEnv();
```

### Supported Environment Variables

- `STORAGE_DRIVER`: flatfile, sqlite, mysql, postgresql
- `STORAGE_PATH`: Path for FlatFile or SQLite
- `DB_HOST`: Database host
- `DB_PORT`: Database port (PostgreSQL)
- `DB_DATABASE`: Database name
- `DB_USERNAME`: Database username
- `DB_PASSWORD`: Database password
- `DB_CHARSET`: Database charset (MySQL)
- `DB_SCHEMA`: Database schema (PostgreSQL)

## Basic Operations

### Select Data

```php
// Get all records
$articles = $storage->select('articles');

// Filter with WHERE
$published = $storage->select('articles', ['status' => 'published']);

// Sort with ORDER BY
$articles = $storage->select('articles', [], ['created_at' => 'DESC']);

// Paginate with LIMIT and OFFSET
$articles = $storage->select('articles', [], ['created_at' => 'DESC'], 10, 0);

// Combined
$articles = $storage->select(
    'articles',                           // table
    ['status' => 'published'],            // where
    ['created_at' => 'DESC'],            // order
    10,                                  // limit
    0                                    // offset
);
```

### Insert Data

```php
$id = $storage->insert('articles', [
    'title' => 'My Article',
    'slug' => 'my-article',
    'content' => 'Article content...',
    'status' => 'draft',
]);
```

### Update Data

```php
$success = $storage->update(
    'articles',                          // table
    ['status' => 'published'],           // data to update
    ['id' => 1]                         // where conditions
);
```

### Delete Data

```php
$success = $storage->delete('articles', ['id' => 1]);
```

## Transactions

All storage drivers support transactions:

```php
$storage->beginTransaction();

try {
    $storage->insert('articles', $data1);
    $storage->insert('comments', $data2);
    $storage->commit();
} catch (\Exception $e) {
    $storage->rollback();
    throw $e;
}
```

## Additional Methods (Driver-Specific)

### SQL Storage Drivers (SQLite, MySQL, PostgreSQL)

```php
// Execute raw SQL
$stmt = $storage->execute('SELECT * FROM articles WHERE status = ?', ['published']);

// Get all tables
$tables = $storage->getTables();

// Drop table
$storage->dropTable('articles');

// Check if table exists
$exists = $storage->tableExists('articles');

// Get table columns
$columns = $storage->getTableColumns('articles');
```

### FlatFileStorage

```php
// Get storage path
$path = $storage->getStoragePath();

// Get all tables (JSON files)
$tables = $storage->getTables();

// Drop table (delete file)
$storage->dropTable('articles');

// Clear cache
$storage->clearCache();
```

## Migrations

CalacaCMS includes a migration system for managing database schema changes.

### Creating a Migration

Create a new file in `src/Database/Migrations/`:

```php
<?php

namespace CalacaCMS\Database\Migrations;

use CalacaCMS\Database\Migration;
use CalacaCMS\Core\StorageInterface;

class Migration_20260312000000_CreateArticlesTable extends Migration
{
    private ?StorageInterface $storage = null;

    public function setStorage(StorageInterface $storage): void
    {
        $this->storage = $storage;
    }

    public function up(): void
    {
        $this->storage->execute('
            CREATE TABLE IF NOT EXISTS articles (
                id INT AUTO_INCREMENT PRIMARY KEY,
                title VARCHAR(255) NOT NULL,
                slug VARCHAR(255) NOT NULL UNIQUE,
                content TEXT,
                status VARCHAR(50) DEFAULT "draft",
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        ');
    }

    public function down(): void
    {
        $this->storage->execute('DROP TABLE IF EXISTS articles');
    }
}
```

### Running Migrations

```php
use CalacaCMS\Database\MigrationRunner;
use CalacaCMS\Storage\StorageFactory;

$storage = StorageFactory::createFromEnv();
$runner = new MigrationRunner($storage, '/path/to/migrations');

// Run all pending migrations
$runner->migrate();

// Rollback last migration
$runner->rollback();

// Rollback specific number of migrations
$runner->rollback(3);

// Show status
$status = $runner->getStatus();
print_r($status);

// Reset (rollback all then re-run)
$runner->reset();
```

### Migration CLI Command

```bash
# Run pending migrations
calacacms migrate run

# Rollback last migration
calacacms migrate rollback

# Rollback 3 migrations
calacacms migrate rollback --step=3

# Show migration status
calacacms migrate status

# Reset all migrations
calacacms migrate reset

# Drop all tables
calacacms migrate drop

# Use specific storage driver
calacacms migrate run --driver=mysql

# Specify custom paths
calacacms migrate run --path=/custom/path --migrations-path=/custom/migrations
```

## Best Practices

1. **Start with FlatFileStorage** for development and testing
2. **Use SQLite** for small to medium production sites
3. **Use MySQL or PostgreSQL** for high-traffic production applications
4. **Always use transactions** when performing multiple related operations
5. **Use migrations** for all schema changes
6. **Index frequently queried columns** for better performance
7. **Validate configuration** before creating storage with `StorageFactory::validateConfig()`

## Switching Storage Drivers

Because all drivers implement the same interface, you can easily switch between them:

```php
// Original config
$config = ['driver' => 'flatfile', 'path' => '/storage'];
$storage = StorageFactory::create($config);

// Switch to MySQL
$config = [
    'driver' => 'mysql',
    'host' => 'localhost',
    'database' => 'calacacms',
    'username' => 'root',
    'password' => '',
];
$storage = StorageFactory::create($config);
```

Note: You'll need to run migrations for the new storage driver to set up the database schema.
