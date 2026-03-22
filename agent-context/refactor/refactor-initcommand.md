# InitCommand Refactoring Documentation

## Overview

The `InitCommand` class has been refactored from a 484-line monolithic class into a modular, testable service architecture. This document explains the new architecture and how to use it.

## Architecture

### Before Refactoring
```
InitCommand (484 lines)
├── Directory creation logic
├── File copying logic
├── Config generation logic
├── User creation logic
├── Migration logic
├── CLI prompt logic
└── Output formatting logic
```

### After Refactoring
```
InitCommand (275 lines) ← Thin CLI adapter
├── InstallationService (orchestration)
│   ├── FileInstallerService (filesystem)
│   ├── ConfigService (configuration)
│   └── UserSetupService (users/migrations)
├── OutputService (formatting)
└── PromptService (user interaction)

Value Objects:
├── InstallationRequest (input validation)
└── InstallationResult (output tracking)
```

## Services

### InstallationService

**Purpose**: Orchestrates the complete installation process.

**Location**: `calaca/src/Services/InstallationService.php`

**Usage**:
```php
use CalacaCMS\Services\InstallationService;
use CalacaCMS\CLI\ValueObjects\InstallationRequest;

$service = new InstallationService('/path/to/target');
$request = InstallationRequest::fromArray([
    'site_name' => 'My Site',
    'site_url' => 'https://example.com',
    'storage_driver' => 'flatfile',
    'storage_config' => ['path' => '/path/to/content'],
    'admin_name' => 'Admin',
    'admin_email' => 'admin@example.com',
    'admin_password' => 'securepassword',
]);

$result = $service->install($request);

if ($result->isSuccess()) {
    echo "Installation successful!\n";
} else {
    echo "Installation failed: " . $result->getFirstError() . "\n";
}
```

**Key Methods**:
- `install(InstallationRequest $request): InstallationResult` - Perform installation
- `validateDirectory(): array` - Check if target directory is suitable

### FileInstallerService

**Purpose**: Handles all filesystem operations.

**Location**: `calaca/src/Services/FileInstallerService.php`

**Key Methods**:
- `createDirectoryStructure(): array` - Create required directories
- `installTemplates(): bool` - Copy template files
- `installPublicAssets(): bool` - Copy CSS/JS assets
- `installContentTypes(): bool` - Copy content type definitions
- `createGitignore(): bool` - Create .gitignore file

### ConfigService

**Purpose**: Generates configuration files.

**Location**: `calaca/src/Services/ConfigService.php`

**Key Methods**:
- `collectStorageConfig(string $driver, array $existing): array` - Interactive config collection
- `generateConfigFile(...): void` - Create config.php
- `generateEntryPoint(string $autoloadPath): void` - Create index.php
- `loadExistingConfig(): array` - Load existing configuration

### UserSetupService

**Purpose**: Admin user creation and database migrations.

**Location**: `calaca/src/Services/UserSetupService.php`

**Key Methods**:
- `runMigrations(string $driver, array $config): bool` - Run DB migrations
- `createAdminUser(...): bool` - Create admin user
- `validateAdminCredentials(...): array` - Validate user input

## Value Objects

### InstallationRequest

**Purpose**: Immutable container for installation parameters with validation.

**Location**: `calaca/src/CLI/ValueObjects/InstallationRequest.php`

**Usage**:
```php
use CalacaCMS\CLI\ValueObjects\InstallationRequest;

// Create from array
$request = InstallationRequest::fromArray([
    'site_name' => 'My Site',
    'site_url' => 'https://example.com',
    'storage_driver' => 'sqlite',
    'storage_config' => ['path' => '/path/to/db.sqlite'],
    'admin_name' => 'Admin',
    'admin_email' => 'admin@example.com',
    'admin_password' => 'password123',
]);

// Or create directly (with validation)
$request = new InstallationRequest(
    siteName: 'My Site',
    siteUrl: 'https://example.com',
    storageDriver: 'flatfile',
    storageConfig: ['path' => '/path/to/content'],
    adminName: 'Admin',
    adminEmail: 'admin@example.com',
    adminPassword: 'password123'
);
```

**Validation Rules**:
- Site name: required, non-empty
- Site URL: required, valid URL format
- Storage driver: must be one of [flatfile, sqlite, mysql, pgsql]
- Admin name: required, non-empty
- Admin email: required, valid email format
- Admin password: required, minimum 8 characters

### InstallationResult

**Purpose**: Immutable container for installation outcome.

**Location**: `calaca/src/CLI/ValueObjects/InstallationResult.php`

**Usage**:
```php
use CalacaCMS\CLI\ValueObjects\InstallationResult;

// Check success
if ($result->isSuccess()) {
    // Access metadata
    $siteName = $result->getMetadata('site_name');
}

// Check for warnings
if ($result->hasWarnings()) {
    foreach ($result->warnings as $warning) {
        echo "Warning: $warning\n";
    }
}

// Handle errors
if ($result->hasErrors()) {
    echo "Errors: " . $result->getErrorsAsString("\n") . "\n";
}
```

## CLI Services

### OutputService

**Purpose**: Consistent CLI formatting with colors and symbols.

**Location**: `calaca/src/CLI/OutputService.php`

**Usage**:
```php
use CalacaCMS\CLI\OutputService;

$output = new OutputService();
$output->success('Operation completed');
$output->error('Something went wrong');
$output->warning('Please check this');
$output->info('Processing...');
$output->printHeader();
$output->printStep(1, 'Installation');
$output->printSuccessSummary([...]);
```

### PromptService

**Purpose**: Interactive user input collection.

**Location**: `calaca/src/CLI/PromptService.php`

**Usage**:
```php
use CalacaCMS\CLI\PromptService;

$prompt = new PromptService();

// Ask for text input
$name = $prompt->ask('Your name', 'Default Name');

// Ask for password (hidden input)
$password = $prompt->askPassword('Password');

// Ask for yes/no confirmation
if ($prompt->confirm('Continue?', true)) {
    // User confirmed
}
```

## Dependency Injection

The refactored `InitCommand` supports full dependency injection:

```php
use CalacaCMS\CLI\InitCommand;
use CalacaCMS\CLI\OutputService;
use CalacaCMS\CLI\PromptService;
use CalacaCMS\Services\InstallationService;
use CalacaCMS\Services\ConfigService;

// For testing - inject mock services
$mockOutput = new OutputService();
$mockPrompt = new PromptService();
$mockInstaller = new InstallationService('/tmp/test', $mockOutput);
$mockConfig = new ConfigService('/tmp/test', $mockPrompt);

$command = new InitCommand(
    targetDir: '/tmp/test',
    output: $mockOutput,
    prompt: $mockPrompt,
    installationService: $mockInstaller,
    configService: $mockConfig
);

$command->run();
```

## Testing Strategy

### Unit Testing

Each service can be tested independently:

```php
// Test InstallationRequest validation
$this->expectException(\InvalidArgumentException::class);
new InstallationRequest(
    siteName: '',
    siteUrl: 'https://example.com',
    // ... other required params
);

// Test OutputService
$output = new OutputService();
ob_start();
$output->success('Test');
$content = ob_get_clean();
$this->assertStringContainsString('✓', $content);
```

### Integration Testing

Test the complete installation flow:

```php
$service = new InstallationService('/tmp/test-install');
$request = InstallationRequest::fromArray([...]);
$result = $service->install($request);

$this->assertTrue($result->isSuccess());
$this->assertFileExists('/tmp/test-install/config/config.php');
$this->assertFileExists('/tmp/test-install/index.php');
```

## Migration Guide

If you have code that depends on the old `InitCommand`:

**Old Code** (before refactoring):
```php
InitCommand::execute([]);
```

**New Code** (after refactoring):
```php
// Still works - backward compatible
InitCommand::execute([]);

// Or use instance-based approach for testing
$command = new InitCommand('/path/to/target');
$command->run();
```

## Refactoring Metrics

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| InitCommand Lines | 484 | 275 | 43% reduction |
| Classes | 1 | 8 | Better separation |
| Testability | Low | High | Full DI support |
| Reusability | None | High | Services usable elsewhere |
| Maintainability | Low | High | Single responsibility |

## Future Improvements

1. **Add comprehensive unit tests** for each service
2. **Add integration tests** for complete installation flow
3. **Support programmatic installation** (non-interactive mode)
4. **Add rollback capability** for failed installations
5. **Support installation profiles** (pre-configured setups)
6. **Add progress callbacks** for long-running operations

## Related Files

- `calaca/src/CLI/InitCommand.php` - Main CLI command
- `calaca/src/CLI/OutputService.php` - Output formatting
- `calaca/src/CLI/PromptService.php` - User prompts
- `calaca/src/CLI/ValueObjects/InstallationRequest.php` - Input VO
- `calaca/src/CLI/ValueObjects/InstallationResult.php` - Output VO
- `calaca/src/Services/InstallationService.php` - Orchestrator
- `calaca/src/Services/FileInstallerService.php` - Filesystem
- `calaca/src/Services/ConfigService.php` - Configuration
- `calaca/src/Services/UserSetupService.php` - Users/DB
