# CalacaCMS Testing & Quality Assurance

## Overview

This document covers testing, code quality, and quality assurance for CalacaCMS.

## Running Tests

### PHPUnit

CalacaCMS uses PHPUnit 11+ for unit and integration testing.

#### Run All Tests

```bash
# Run all tests
composer test

# Or directly with PHPUnit
phpunit
```

#### Run Specific Test

```bash
# Run specific test class
phpunit tests/Unit/FileCacheTest.php

# Run specific test method
phpunit --filter testSetAndGet tests/Unit/FileCacheTest.php
```

#### Run Test Suites

```bash
# Run unit tests only
phpunit --testsuite=Unit

# Run integration tests only
phpunit --testsuite=Integration
```

#### With Coverage

```bash
# Run tests with coverage report
phpunit --coverage-html coverage/html

# Generate XML coverage for CI
phpunit --coverage-clover coverage/clover.xml

# Generate text coverage
phpunit --coverage-text coverage/coverage.txt
```

### Test Organization

```
tests/
├── bootstrap.php           # PHPUnit bootstrap (autoloading, environment)
├── Unit/                 # Unit tests (isolated, fast)
│   ├── FileCacheTest.php
│   ├── FileLoggerTest.php
│   └── ...
└── Integration/            # Integration tests (full system)
    └── ...
```

#### Unit Tests

- Test individual components in isolation
- Use mocks for dependencies
- Fast execution
- Located in `tests/Unit/`
- Naming: `{Component}Test.php`

Example:
```php
class FileCacheTest extends TestCase
{
    public function testSetAndGet(): void
    {
        $cache = new FileCache('/tmp/test');
        $cache->set('key', 'value');
        $this->assertEquals('value', $cache->get('key'));
    }
}
```

#### Integration Tests

- Test multiple components together
- Use real dependencies where appropriate
- Slower but more realistic
- Located in `tests/Integration/`
- Naming: `{Feature}Test.php`

Example:
```php
class ArticleControllerIntegrationTest extends TestCase
{
    public function testArticleWorkflow(): void
    {
        // Test full article creation, storage, and retrieval
        $controller = new ArticleController();
        $response = $controller->store($articleData);
        $article = $repository->find($response->id);

        $this->assertEquals($articleData['title'], $article->title);
    }
}
```

## Code Quality

### PHP-CS-Fixer

PHP Code Sniffer and Fixer for PSR-12 compliance and code style consistency.

#### Check Code Style

```bash
# Check code style (dry run)
composer lint

# Or directly
php-cs-fixer --dry-run
```

#### Fix Code Style Issues

```bash
# Fix code style automatically
composer fix

# Or directly
php-cs-fixer fix
```

#### Fix Specific Files

```bash
# Check specific file
php-cs-fixer fix --dry-run src/Services/FileUploadService.php

# Fix specific directory
php-cs-fixer fix src/Cache/
```

### PHPStan

Static analysis tool for finding bugs and ensuring code quality.

#### Run Analysis

```bash
# Run PHPStan analysis
composer analyse

# Or directly
phpstan analyse src/
```

#### Analysis Levels

PHPStan has 10 levels of strictness (0-10). CalacaCMS uses level 5.

- **Level 0**: Basic checks (most permissive)
- **Level 1**: Likely false positives
- **Level 2**: Maybe false positives
- **Level 3**: More strict (default)
- **Level 4**: Stricter checks
- **Level 5**: **CalacaCMS default** (recommended)
- **Level 6**: Very strict
- **Level 7**: Extremely strict
- **Level 8**: Unreasonable
- **Level 9**: Paranoia
- **Level 10**: Maximum strictness

#### Check Specific Files

```bash
# Check specific file
phpstan analyse src/Cache/FileCache.php

# Check specific directory
phpstan analyse src/Cache/

# Generate baseline for future runs
phpstan analyse --generate-baseline phpstan-baseline.neon src/
```

#### Using Baseline

After generating a baseline, PHPStan will only report new issues:

```bash
# Analyse with baseline
phpstan analyse --baseline phpstan-baseline.neon src/

# Update baseline
phpstan analyse --generate-baseline phpstan-baseline.neon src/
```

## Test Coverage

CalacaCMS aims for 80%+ code coverage.

### Coverage Reports

Coverage reports are generated in `coverage/` directory:

- `coverage/html/` - HTML report (view in browser)
- `coverage/coverage.txt` - Text summary
- `coverage/clover.xml` - XML for CI tools

### View HTML Coverage

```bash
# Generate and view
phpunit --coverage-html coverage/html
open coverage/html/index.html  # macOS
xdg-open coverage/html/index.html  # Linux
start coverage/html/index.html  # Windows
```

### Coverage Thresholds

Coverage is configured in `phpunit.xml`:

```xml
<coverage>
    <report>
        <html outputDirectory="coverage/html"/>
    </report>
</coverage>
```

## CI/CD Integration

### GitHub Actions

Example GitHub Actions workflow:

```yaml
name: Tests

on: [push, pull_request]

jobs:
    test:
        runs-on: ubuntu-latest

        steps:
            - uses: actions/checkout@v3

            - name: Setup PHP
              uses: shivammathur/setup-php@v2
              with:
                  php-version: '8.1'

            - name: Install dependencies
              run: composer install --no-progress --prefer-dist

            - name: Run tests
              run: composer test

            - name: Generate coverage
              run: phpunit --coverage-clover coverage/clover.xml

            - name: Upload coverage
              uses: codecov/codecov-action@v3
              with:
                  files: coverage/clover.xml
```

### GitLab CI

Example GitLab CI configuration:

```yaml
test:
    image: php:8.1

    script:
        - composer install
        - composer test
        - phpunit --coverage-clover coverage/clover.xml

    coverage: '/^\s*Lines:\s*(\d+\.\d+)%/'
    artifacts:
        reports:
            coverage_report:
                coverage_format: cobertura
                path: coverage/clover.xml
```

## Writing Tests

### Test Structure

```php
<?php

declare(strict_types=1);

namespace CalacaCMS\Tests\Unit;

use CalacaCMS\Cache\FileCache;
use PHPUnit\Framework\TestCase;
use PHPUnit\Framework\Attributes\Test;

/**
 * Component Name Test
 */
class FileCacheTest extends TestCase
{
    private FileCache $cache;

    protected function setUp(): void
    {
        // Setup before each test
        $this->cache = new FileCache('/tmp/test');
    }

    protected function tearDown(): void
    {
        // Cleanup after each test
        $this->cache->clear();
    }

    #[Test]
    public function testSomething(): void
    {
        // Arrange
        $key = 'test_key';
        $value = 'test_value';

        // Act
        $this->cache->set($key, $value);
        $result = $this->cache->get($key);

        // Assert
        $this->assertEquals($value, $result);
    }
}
```

### Test Naming Conventions

- Test class: `{Component}Test.php`
- Test method: `test{Feature}()` or `test{Feature}_{Scenario}()`
- Use `declare(strict_types=1);` at top
- Use PHP 8.1+ attributes: `#[Test]`

### Assertion Methods

Use appropriate PHPUnit assertions:

```php
$this->assertEquals($expected, $actual);
$this->assertNotEquals($notExpected, $actual);
$this->assertTrue($condition);
$this->assertFalse($condition);
$this->assertNull($actual);
$this->assertNotNull($actual);
$this->assertIsArray($actual);
$this->assertIsInt($actual);
$this->assertInstanceOf(ClassName::class, $actual);
$this->assertArrayHasKey($key, $array);
$this->assertContains($needle, $haystack);
$this->assertEmpty($actual);
$this->assertCount($expectedCount, $actual);
```

### Data Providers

Use data providers for testing multiple scenarios:

```php
public static function cacheKeyProvider(): array
{
    return [
        'simple_key' => ['simple_key'],
        'key_with_underscores' => ['key_with_underscores'],
        'key_with_numbers' => ['key_123'],
    ];
}

#[Test]
#[DataProvider('cacheKeyProvider')]
public function testCacheKey(string $key): void
{
    $this->cache->set($key, 'value');
    $this->assertEquals('value', $this->cache->get($key));
}
```

### Mocking Dependencies

Use PHPUnit's built-in mocking:

```php
public function testServiceWithMockedDependency(): void
{
    // Create mock
    $mockStorage = $this->createMock(StorageInterface::class);

    // Set mock expectations
    $mockStorage->expects($this->once())
        ->method('select')
        ->with('articles', ['id' => 1])
        ->willReturn([['id' => 1, 'title' => 'Test']]);

    // Inject mock into service
    $service = new ArticleService($mockStorage);

    // Test service behavior
    $result = $service->getArticle(1);

    $this->assertEquals('Test', $result['title']);
}
```

## Best Practices

### Testing

1. **Test one thing at a time**: Each test should verify a single behavior
2. **Use descriptive test names**: `testUserCanLoginWithValidCredentials` instead of `testLogin`
3. **Arrange-Act-Assert pattern**: Clear separation of setup, execution, and verification
4. **Test edge cases**: Null values, empty arrays, boundary conditions
5. **Test error conditions**: Verify proper exception handling
6. **Keep tests fast**: Unit tests should run in milliseconds
5. **Maintain independence**: Tests should not depend on each other
6. **Use test doubles appropriately**: Mock external dependencies in unit tests
7. **Aim for high coverage**: Target 80%+ code coverage

### Code Quality

1. **Run linter before commits**: Always run `composer lint` before pushing
2. **Fix style issues automatically**: Use `composer fix` instead of manual fixes
3. **Run static analysis**: Use `composer analyse` to catch bugs early
4. **Address warnings**: Don't ignore PHPStan warnings without justification
5. **Review failed CI checks**: Don't bypass quality gates
6. **Keep dependencies updated**: Run `composer update` regularly

### Coverage

1. **Write tests for new code**: Every new feature needs tests
2. **Target critical paths**: Focus on business logic and security
3. **Monitor coverage trends**: Watch for decreasing coverage
4. **Review uncovered code**: Determine if it needs tests or can be refactored
5. **Avoid test-only code**: Don't add code just for coverage

## Troubleshooting

### PHPUnit Issues

**Test not found:**
```bash
# Check namespace matches directory
# Should be: namespace CalacaCMS\Tests\Unit;
# File location: tests/Unit/FileCacheTest.php
```

**Autoloader issues:**
```bash
# Check composer autoload is updated
composer dump-autoload

# Check tests/bootstrap.php loads autoloader
```

**Slow tests:**
```bash
# Profile slow tests
phpunit --report-useless-tests --report-untested

# Consider splitting slow integration tests
```

### PHP-CS-Fixer Issues

**Rule conflicts:**
```bash
# Check .php-cs-fixer.php for conflicting rules
# Temporarily disable conflicting rules in config
```

**Large reformatting:**
```bash
# Review changes before committing
php-cs-fixer fix --dry-run
git diff
```

### PHPStan Issues

**False positives:**
```bash
# Use ignoreErrors in phpstan.neon for known issues
# Update baseline after review
phpstan analyse --generate-baseline phpstan-baseline.neon src/
```

**Missing stub files:**
```bash
# Add stub files for third-party libraries
# Located in stubs/ directory
```

## Continuous Improvement

### Review Metrics

Track and improve:
- Test execution time
- Code coverage percentage
- PHPStan error count
- PHP-CS-Fixer violations
- Bug reports from production

### Quality Gates

Set up quality gates in CI:
- Minimum 80% coverage
- Zero PHPStan errors
- Zero PHP-CS-Fixer violations
- All tests passing

### Regular Reviews

- Weekly code review meetings
- Monthly quality metrics review
- Quarterly architecture review
- Annual technical debt assessment
