# CalacaCMS Cache & Logger Services

## Overview

CalacaCMS includes caching and logging services to help with performance and debugging. Both services implement standard interfaces, making them easy to replace with alternative implementations if needed.

## Cache Service (FileCache)

File-based caching service that stores data in JSON files with expiration support.

### Basic Usage

```php
use CalacaCMS\Cache\FileCache;

// Initialize cache
$cache = new FileCache('/path/to/cache');

// Store a value
$cache->set('article:1', $articleData);

// Store with TTL (Time To Live)
$cache->set('article:1', $articleData, 3600); // 1 hour

// Retrieve a value
$article = $cache->get('article:1');
```

### Advanced Usage

```php
// Remember value (cached if exists, otherwise compute and store)
$article = $cache->remember('article:1', 3600, function() use ($id) {
    return $repository->find($id);
});

// Remember forever
$config = $cache->rememberForever('app:config', function() {
    return $configService->load();
});

// Check if key exists
if ($cache->has('article:1')) {
    // Key exists and is not expired
}

// Delete specific key
$cache->delete('article:1');

// Clear all cache
$cache->clear();
```

### Batch Operations

```php
// Get multiple keys at once
$values = $cache->getMultiple(['article:1', 'article:2', 'config']);

// Set multiple keys at once
$cache->setMultiple([
    'article:1' => $data1,
    'article:2' => $data2,
    'config' => $config,
], 3600);

// Delete multiple keys
$cache->deleteMultiple(['article:1', 'article:2']);
```

### Cache Management

```php
// Clean expired entries
$cleaned = $cache->cleanExpired();
echo "Cleaned {$cleaned} expired cache entries";

// Get cache statistics
$stats = $cache->getStats();
print_r($stats);
// Array (
//     [total_files] => 125
//     [total_size] => 2048576
//     [expired_files] => 15
//     [active_files] => 110
// )

// Get cache path
$cachePath = $cache->getCachePath();
```

### Configuration

```php
// Set default TTL (Time To Live)
$cache->setDefaultTtl(7200); // 2 hours

// Use prefix for cache keys
$cache = new FileCache('/path/to/cache', 'myapp');
// Now keys are stored as "myapp:key"
```

### Cache File Structure

```
/path/to/cache/
├── 00/
│   ├── abc123.json
│   └── def456.json
├── 01/
│   └── ghi789.json
├── 02/
│   └── jkl012.json
└── ...
```

Cache keys are hashed and stored in subdirectories (00-ff) to prevent performance issues with too many files in one directory.

### Cache File Format

Each cache file contains JSON with the following structure:

```json
{
    "value": "cached data",
    "expires": 1678901234,
    "created": 1678896234
}
```

## Logger Service (FileLogger)

File-based logging service that stores log entries in daily rotating log files.

### Basic Usage

```php
use CalacaCMS\Logger\FileLogger;

// Initialize logger
$logger = new FileLogger('/path/to/logs');

// Log messages
$logger->info('User logged in');
$logger->error('Database connection failed');
$logger->warning('Cache miss for key: user:123');
```

### Log Levels

```php
$logger->info('Information message');
$logger->notice('Notice message');
$logger->warning('Warning message');
$logger->error('Error message');
$logger->critical('Critical error');
$logger->debug('Debug message');
```

### Context Data

```php
// Add context to log messages
$logger->info('User logged in', [
    'user_id' => 123,
    'ip' => '192.168.1.1',
    'channel' => 'auth',
]);

$logger->error('Failed to save article', [
    'article_id' => 456,
    'error' => $exception->getMessage(),
    'channel' => 'articles',
]);
```

### Custom Channels

```php
// Set default channel
$logger->setDefaultChannel('auth');

// Override channel per log
$logger->info('User logged in', ['channel' => 'auth']);
$logger->info('Article created', ['channel' => 'articles']);
```

### Log Line Format

```
[2026-03-13 14:30:45] auth.INFO: User logged in {"user_id":123,"ip":"192.168.1.1"}
```

Format: `[timestamp] channel.level: message {context}`

### Reading Logs

```php
// Get in-memory logs for channel
$authLogs = $logger->getLogs('auth');
$articleLogs = $logger->getLogs('articles', 100); // Last 100 logs

// Read specific log file by date
$marchLogs = $logger->readLogFile('2026-03-13');

// Get available log dates
$dates = $logger->getLogDates();
// Array (0 => '2026-03-13', 1 => '2026-03-12', ...)

// Get log file statistics
$stats = $logger->getLogStats();
print_r($stats);
// Array (
//     [total_files] => 30,
//     [total_size] => 1048576,
//     [total_size_human] => '1 MB',
// )
```

### Log Management

```php
// Clear in-memory logs for channel
$logger->clearChannel('auth');

// Rotate logs (keep last N days)
$deleted = $logger->rotateLogs(30); // Keep last 30 days
echo "Deleted {$deleted} log files";

// Get log path
$logPath = $logger->getLogPath();
```

### Log File Structure

```
/path/to/logs/
├── 2026-03-13.log
├── 2026-03-12.log
├── 2026-03-11.log
└── .gitkeep
```

### Configuration

```php
// Set date format
$logger->setDateFormat('Y-m-d H:i:s.v'); // With microseconds

// Set default channel
$logger->setDefaultChannel('myapp');

// Use prefix for log files
$logger = new FileLogger('/path/to/logs', 'myapp');
// Creates files: myapp-2026-03-13.log
```

## Integration Example

Here's how to use both services together in a controller:

```php
use CalacaCMS\Cache\FileCache;
use CalacaCMS\Logger\FileLogger;

class ArticleController
{
    private FileCache $cache;
    private FileLogger $logger;

    public function __construct()
    {
        $this->cache = new FileCache('/path/to/cache');
        $this->logger = new FileLogger('/path/to/logs');
    }

    public function show(int $id): array
    {
        $cacheKey = "article:{$id}";

        $this->logger->info('Loading article', [
            'article_id' => $id,
            'channel' => 'articles',
        ]);

        // Try to get from cache
        $article = $this->cache->get($cacheKey);

        if ($article === null) {
            $this->logger->debug('Cache miss', [
                'cache_key' => $cacheKey,
                'channel' => 'cache',
            ]);

            // Load from database
            $article = $this->repository->find($id);

            if ($article === null) {
                $this->logger->error('Article not found', [
                    'article_id' => $id,
                    'channel' => 'articles',
                ]);

                throw new NotFoundException('Article not found');
            }

            // Store in cache for 1 hour
            $this->cache->set($cacheKey, $article, 3600);
        }

        return $article;
    }
}
```

## Best Practices

### Cache

1. **Use appropriate TTL values**: Don't cache data that changes frequently
2. **Key naming conventions**: Use prefixes to group related items (e.g., `article:1`, `user:profile:123`)
3. **Cache invalidation**: Always delete cache when updating underlying data
4. **Use `remember()` method**: Automatically handles cache misses and stores computed values
5. **Monitor cache hit ratio**: Use stats to optimize caching strategy
6. **Clean expired entries**: Run `cleanExpired()` periodically to free disk space

### Logger

1. **Use appropriate log levels**: Use `info` for normal operations, `warning` for issues that don't stop execution, `error` for failures
2. **Add context**: Include relevant data (IDs, parameters, error details)
3. **Use channels**: Separate logs by component (auth, database, api, etc.)
4. **Rotate logs**: Set up log rotation to prevent disk space issues
5. **Don't log sensitive data**: Never log passwords, tokens, or personal data
6. **Monitor log levels**: Watch for increasing error rates

## Performance Considerations

### Cache

- **File system operations** can be slow for high-traffic sites
- Consider using Redis or Memcached for production
- Cache directory should be on fast storage (SSD)
- Subdirectory structure prevents performance degradation with many files

### Logger

- **Synchronous writes** can impact performance for high-traffic sites
- Consider async logging for production
- Log directory should be on storage separate from cache
- Monitor log file sizes to prevent disk issues

## Alternative Implementations

The interfaces are designed to be easily replaced:

```php
use CalacaCMS\CacheInterface;
use CalacaCMS\LoggerInterface;

// Redis Cache
class RedisCache implements CacheInterface
{
    // Redis implementation
}

// Monolog wrapper
class MonologAdapter implements LoggerInterface
{
    private Monolog\Logger $logger;

    public function info(string $message, array $context = []): void
    {
        $this->logger->info($message, $context);
    }

    // ... other methods
}
```

## Environment Variables

For production, configure cache and logger paths:

```php
// Cache
$_ENV['CACHE_PATH'] = '/var/www/myproject/storage/cache';
$_ENV['CACHE_PREFIX'] = 'myapp';
$_ENV['CACHE_TTL'] = 3600;

// Logger
$_ENV['LOG_PATH'] = '/var/www/myproject/storage/logs';
$_ENV['LOG_PREFIX'] = 'myapp';
$_ENV['LOG_LEVEL'] = 'INFO'; // Filter logs by level
$_ENV['LOG_ROTATION_DAYS'] = 30;
```
