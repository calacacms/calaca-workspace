# Refactoring Task List: InitCommand.php

> **PRD:** docs/prd-refactor-initcommand.md
> **Status:** ⬜ Not Started
> **Total Phases:** 8
> **Estimated Time:** 9.5 hours

---

## Phase 1: Create OutputService (Utility Layer)

**Goal:** Extract CLI formatting logic into dedicated service
**Time:** 30 minutes | **Complexity:** LOW | **Risk:** LOW

### Tasks

- [ ] **1.1** Create `calaca/src/CLI/OutputService.php`
  - [ ] Add class declaration with namespace `CalacaCMS\CLI`
  - [ ] Define color constants (GREEN, RED, YELLOW, BLUE, MAGENTA, CYAN)
  - [ ] Define symbol constants (CHECK, CROSS, ARROW)

- [ ] **1.2** Extract `printHeader()` method
  - [ ] Move ASCII art logo from InitCommand
  - [ ] Move title line
  - [ ] Make instance method `printHeader(): void`
  - [ ] Add color codes using constants

- [ ] **1.3** Extract `printStep()` method
  - [ ] Move from InitCommand
  - [ ] Make instance method `printStep(int $number, string $title): void`
  - [ ] Add separator line

- [ ] **1.4** Extract `ok()` method
  - [ ] Move from InitCommand
  - [ ] Rename to `success(string $message): void`
  - [ ] Use color constants

- [ ] **1.5** Add new output methods
  - [ ] `error(string $message): void` - red error messages
  - [ ] `warning(string $message): void` - yellow warnings
  - [ ] `info(string $message): void` - cyan informational messages

- [ ] **1.6** Update InitCommand to use OutputService
  - [ ] Create OutputService instance in execute()
  - [ ] Replace all echo statements with OutputService calls
  - [ ] Test output formatting

- [ ] **1.7** Create `calaca/tests/CLI/OutputServiceTest.php`
  - [ ] Test color output (capture stdout)
  - [ ] Test printHeader format
  - [ ] Test printStep format
  - [ ] Test success/error/warning methods

**Deliverable:** OutputService.php (~80 lines), OutputServiceTest.php

---

## Phase 2: Create PromptService (Interaction Layer)

**Goal:** Extract user interaction logic into dedicated service
**Time:** 45 minutes | **Complexity:** LOW | **Risk:** LOW

### Tasks

- [ ] **2.1** Create `calaca/src/CLI/PromptService.php`
  - [ ] Add class declaration
  - [ ] Define prompt style constants

- [ ] **2.2** Extract `prompt()` method
  - [ ] Move from InitCommand
  - [ ] Make instance method `ask(string $question, string $default = ''): string`
  - [ ] Keep readline() logic
  - [ ] Keep hint display with default value

- [ ] **2.3** Extract `promptPassword()` method
  - [ ] Move from InitCommand
  - [ ] Make instance method `askPassword(string $question): string`
  - [ ] Keep Unix stty -echo logic
  - [ ] Keep Windows fallback

- [ ] **2.4** Add new prompt methods
  - [ ] `confirm(string $question, bool $default = false): bool` - yes/no prompts
  - [ ] `select(string $question, array $options, mixed $default): mixed` - menu selection
  - [ ] Add validation support for confirm/select

- [ ] **2.5** Update InitCommand to use PromptService
  - [ ] Create PromptService instance in execute()
  - [ ] Replace all prompt() calls with $prompt->ask()
  - [ ] Replace promptPassword() with $prompt->askPassword()
  - [ ] Update reinstall confirmation to use confirm()

- [ ] **2.6** Create `calaca/tests/CLI/PromptServiceTest.php`
  - [ ] Mock readline() for testing
  - [ ] Test ask() with and without defaults
  - [ ] Test askPassword() masking
  - [ ] Test confirm() yes/no parsing
  - [ ] Test select() option validation

**Deliverable:** PromptService.php (~100 lines), PromptServiceTest.php

---

## Phase 3: Create FileInstallerService (File Operations)

**Goal:** Extract filesystem operations into dedicated service
**Time:** 1 hour | **Complexity:** MEDIUM | **Risk:** LOW

### Tasks

- [ ] **3.1** Create `calaca/src/Services/FileInstallerService.php`
  - [ ] Add namespace `CalacaCMS\Services`
  - [ ] Constructor accepts `targetDir` and `packageDir`

- [ ] **3.2** Extract `createDirectories()` method
  - [ ] Move directory list from InitCommand
  - [ ] Make instance method `createDirectoryStructure(): void`
  - [ ] Create each directory with 0755 permissions
  - [ ] Return array of created paths

- [ ] **3.3** Extract template installation
  - [ ] Move `copyTemplates()` logic
  - [ ] Make method `installTemplates(): void`
  - [ ] Use source: resources/templates, dest: templates

- [ ] **3.4** Extract public assets installation
  - [ ] Move `copyPublicAssets()` logic
  - [ ] Make method `installPublicAssets(): void`
  - [ ] Use source: public, dest: public

- [ ] **3.5** Extract content types installation
  - [ ] Move `copyContentTypes()` logic
  - [ ] Make method `installContentTypes(): void`
  - [ ] Use source: resources/types, dest: calaca/types

- [ ] **3.6** Extract `recursiveCopy()` method
  - [ ] Move from InitCommand
  - [ ] Make instance method `recursiveCopy(string $src, string $dst): void`
  - [ ] Keep RecursiveDirectoryIterator logic
  - [ ] Add error handling for permission errors

- [ ] **3.7** Extract `createGitignore()` method
  - [ ] Move from InitCommand
  - [ ] Make instance method `createGitignore(): void`
  - [ ] Check if exists before creating
  - [ ] Write standard .gitignore content

- [ ] **3.8** Add status reporting
  - [ ] Return `array<string, string>` with operation results
  - [ ] Format: ['operation' => 'success|error', 'path' => 'target']
  - [ ] Throw exceptions on critical failures

- [ ] **3.9** Create `calaca/tests/Services/FileInstallerServiceTest.php`
  - [ ] Test directory creation (use temp directory)
  - [ ] Test file copying (mock filesystem)
  - [ ] Test recursive copy
  - [ ] Test gitignore creation
  - [ ] Test error handling (read-only directories)

**Deliverable:** FileInstallerService.php (~150 lines), FileInstallerServiceTest.php

---

## Phase 4: Create ConfigService (Configuration Generation)

**Goal:** Extract config generation logic into dedicated service
**Time:** 1.5 hours | **Complexity:** MEDIUM | **Risk:** MEDIUM

### Tasks

- [ ] **4.1** Create `calaca/src/Services/ConfigService.php`
  - [ ] Add namespace `CalacaCMS\Services`
  - [ ] Constructor accepts target directory path

- [ ] **4.2** Extract `loadExistingConfig()` method
  - [ ] Move from InitCommand
  - [ ] Make instance method `loadExistingConfig(): array`
  - [ ] Keep file_exists and include logic
  - [ ] Return empty array on failure

- [ ] **4.3** Extract `collectStorageConfig()` method
  - [ ] Move from InitCommand
  - [ ] Make instance method `collectStorageConfig(string $driver, array $existing): array`
  - [ ] Keep flatfile, sqlite, mysql, pgsql logic
  - [ ] Use PromptService for interactive collection

- [ ] **4.4** Create config templates
  - [ ] Create `calaca/resources/templates/config/config.php.template`
  - [ ] Create `calaca/resources/templates/config/entrypoint.php.template`
  - [ ] Replace heredoc strings with file templates
  - [ ] Use {{variable}} placeholders for substitution

- [ ] **4.5** Extract `generateConfig()` method
  - [ ] Move from InitCommand
  - [ ] Make instance method `generateConfigFile(array $config): void`
  - [ ] Load config.php.template
  - [ ] Replace placeholders with actual values
  - [ ] Write to config/config.php

- [ ] **4.6** Extract `generateEntryPoint()` method
  - [ ] Move from InitCommand
  - [ ] Make instance method `generateEntryPoint(string $autoloadPath): void`
  - [ ] Load entrypoint.php.template
  - [ ] Replace autoload path placeholder
  - [ ] Write to index.php

- [ ] **4.7** Extract `buildStorageConfigCode()` method
  - [ ] Move from InitCommand
  - [ ] Make instance method `buildStorageConfig(array $config): string`
  - [ ] Keep flatfile, sqlite, mysql/pgsql formatting
  - [ ] Return PHP array code as string

- [ ] **4.8** Add validation
  - [ ] Validate config array structure
  - [ ] Check required fields (site name, URL, driver)
  - [ ] Validate storage config per driver
  - [ ] Throw exceptions on invalid data

- [ ] **4.9** Create `calaca/tests/Services/ConfigServiceTest.php`
  - [ ] Test config file generation
  - [ ] Test entry point generation
  - [ ] Test storage config building for each driver
  - [ ] Test existing config loading
  - [ ] Test validation logic

**Deliverable:** ConfigService.php (~200 lines), config templates, ConfigServiceTest.php

---

## Phase 5: Create UserSetupService (User & Database)

**Goal:** Extract user creation and migrations into dedicated service
**Time:** 1 hour | **Complexity:** MEDIUM | **Risk:** MEDIUM

### Tasks

- [ ] **5.1** Create `calaca/src/Services/UserSetupService.php`
  - [ ] Add namespace `CalacaCMS\Services`
  - [ ] Constructor inject dependencies:
    ```php
    public function __construct(
        private UsersRepository $usersRepository,
        private MigrationRunner $migrationRunner
    ) {}
    ```

- [ ] **5.2** Extract `runMigrations()` method
  - [ ] Move from InitCommand
  - [ ] Make instance method `runMigrations(array $storageConfig): void`
  - [ ] Create storage from StorageFactory
  - [ ] Run migrations with MigrationRunner
  - [ ] Return success/failure status

- [ ] **5.3** Extract `createAdminUser()` method
  - [ ] Move from InitCommand
  - [ ] Make instance method `createAdminUser(array $storageConfig, array $admin): void`
  - [ ] Create storage and repository
  - [ ] Check if user exists (by email)
  - [ ] Create user with hashed password
  - [ ] Return success/failure status

- [ ] **5.4** Add error handling
  - [ ] Wrap storage creation in try-catch
  - [ ] Catch database connection errors
  - [ ] Catch migration errors
  - [ ] Catch user creation errors
  - [ ] Return detailed error messages

- [ ] **5.5** Add transaction support
  - [ ] Wrap user creation in transaction (if driver supports)
  - [ ] Rollback on error
  - [ ] Log rollback events

- [ ] **5.6** Create `calaca/tests/Services/UserSetupServiceTest.php`
  - [ ] Mock UsersRepository
  - [ ] Mock MigrationRunner
  - [ ] Test user creation (success and failure)
  - [ ] Test migration running
  - [ ] Test user already exists scenario
  - [ ] Test error handling and rollback

**Deliverable:** UserSetupService.php (~80 lines), UserSetupServiceTest.php

---

## Phase 6: Create InstallationService (Orchestration)

**Goal:** Create service layer for installation workflow
**Time:** 2 hours | **Complexity:** HIGH | **Risk:** MEDIUM

### Tasks

- [ ] **6.1** Create value objects
  - [ ] Create `calaca/src/CLI/ValueObjects/InstallationRequest.php`
    - [ ] Properties: siteName, siteUrl, storageDriver, storageConfig, adminName, adminEmail, adminPassword
    - [ ] Validation methods
    - [ ] Static factory method fromArray()
  - [ ] Create `calaca/src/CLI/ValueObjects/InstallationResult.php`
    - [ ] Properties: success, errors, warnings, installedPaths
    - [ ] Methods: addError(), addWarning(), isSuccess()

- [ ] **6.2** Create `calaca/src/Services/InstallationService.php`
  - [ ] Add namespace `CalacaCMS\Services`
  - [ ] Constructor inject all dependencies:
    ```php
    public function __construct(
        private FileInstallerService $fileInstaller,
        private ConfigService $configService,
        private UserSetupService $userSetup,
        private OutputService $output
    ) {}
    ```

- [ ] **6.3** Implement `install()` method
  - [ ] Method signature: `install(InstallationRequest $request): InstallationResult`
  - [ ] Create InstallationResult instance
  - [ ] Wrap in try-catch for error handling

- [ ] **6.4** Add installation steps
  - [ ] Step 1: Create directory structure
    - [ ] Call FileInstallerService::createDirectoryStructure()
    - [ ] Add errors to result if failed
  - [ ] Step 2: Install files
    - [ ] Call installTemplates(), installPublicAssets(), installContentTypes()
    - [ ] Add errors to result if failed
  - [ ] Step 3: Generate config
    - [ ] Call ConfigService::generateConfigFile()
    - [ ] Call generateEntryPoint()
    - [ ] Call createGitignore()
    - [ ] Add errors to result if failed
  - [ ] Step 4: Database setup
    - [ ] If driver != 'flatfile', call runMigrations()
    - [ ] Call createAdminUser()
    - [ ] Add errors to result if failed

- [ ] **6.5** Add rollback support
  - [ ] Track created files/directories during installation
  - [ ] On critical error, rollback changes
  - [ ] Delete created files
  - [ ] Remove created directories
  - [ ] Log rollback actions

- [ ] **6.6** Add validation
  - [ ] Validate InstallationRequest before installation
  - [ ] Check required fields
  - [ ] Validate storage config
  - [ ] Validate email format
  - [ ] Validate password strength

- [ ] **6.7** Add progress reporting
  - [ ] Call OutputService methods for each step
  - [ ] Print step headers
  - [ ] Print success/error messages
  - [ ] Print summary at end

- [ ] **6.8** Create `calaca/tests/Services/InstallationServiceTest.php`
  - [ ] Mock all dependencies
  - [ ] Test successful installation flow
  - [ ] Test failure at each step
  - [ ] Test rollback functionality
  - [ ] Test validation
  - [ ] Test progress reporting

**Deliverable:** InstallationService.php (~150 lines), value objects, InstallationServiceTest.php

---

## Phase 7: Refactor InitCommand (CLI Adapter)

**Goal:** Convert InitCommand to thin CLI adapter
**Time:** 1 hour | **Complexity:** MEDIUM | **Risk:** LOW

### Tasks

- [ ] **7.1** Convert InitCommand to instance-based
  - [ ] Remove `static` keyword from class declaration
  - [ ] Convert static properties to instance properties
  - [ ] Convert static methods to instance methods

- [ ] **7.2** Add constructor with DI
  - [ ] Accept dependencies via constructor:
    ```php
    public function __construct(
        private InstallationService $installationService,
        private PromptService $promptService,
        private OutputService $outputService
    ) {}
    ```

- [ ] **7.3** Refactor `execute()` method
  - [ ] Keep public static for CLI compatibility: `public static function execute(array $args = []): void`
  - [ ] Create instance inside execute()
  - [ ] Resolve dependencies from container
  - [ ] Call instance method
  - [ ] Add deprecation notice if called statically

- [ ] **7.4** Create instance method `run()`
  - [ ] Move installation logic from execute() to run()
  - [ ] Use PromptService for all prompts
  - [ ] Use OutputService for all output
  - [ ] Build InstallationRequest from user input
  - [ ] Call InstallationService::install()
  - [ ] Display result summary

- [ ] **7.5** Update prompts to use PromptService
  - [ ] Replace prompt() with $this->prompt->ask()
  - [ ] Replace promptPassword() with $this->prompt->askPassword()
  - [ ] Add confirmation for reinstallation using confirm()

- [ ] **7.6** Update output to use OutputService
  - [ ] Replace all echo statements with OutputService calls
  - [ ] Use printHeader(), printStep(), success(), error()

- [ ] **7.7** Remove extracted methods
  - [ ] Delete all methods moved to services
  - [ ] Delete helper methods now in services
  - [ ] Keep only execute() and run()

- [ ] **7.8** Register InitCommand in DI container
  - [ ] Add to Application::registerServices()
  - [ ] Inject all dependencies
  - [ ] Register as shared service

- [ ] **7.9** Update bin/calaca
  - [ ] Resolve InitCommand from container
  - [ ] Call execute() method
  - [ ] Handle exceptions

- [ ] **7.10** Create integration test
  - [ ] Test full installation flow
  - [ ] Use temporary directory
  - [ ] Verify all files created
  - [ ] Verify config generated
  - [ ] Cleanup after test

**Deliverable:** Refactored InitCommand.php (~150 lines), integration test

---

## Phase 8: Testing & Documentation

**Goal:** Comprehensive test coverage and documentation
**Time:** 2 hours | **Complexity:** MEDIUM | **Risk:** LOW

### Tasks

- [ ] **8.1** Complete unit tests
  - [ ] Review all test files from previous phases
  - [ ] Ensure 80%+ code coverage
  - [ ] Add edge case tests
  - [ ] Add error path tests

- [ ] **8.2** Create integration test suite
  - [ ] Create `calaca/tests/CLI/InitCommandIntegrationTest.php`
  - [ ] Test complete installation flow
  - [ ] Test with all storage drivers (flatfile, sqlite, mysql, pgsql)
  - [ ] Test reinstallation scenario
  - [ ] Test error scenarios (invalid input, file permissions)

- [ ] **8.3** Create manual testing checklist
  - [ ] Test flatfile installation
  - [ ] Test sqlite installation
  - [ ] Test mysql installation
  - [ ] Test pgsql installation
  - [ ] Test reinstallation
  - [ ] Test with empty defaults
  - [ ] Test with special characters in input

- [ ] **8.4** Update documentation
  - [ ] Update README with installation instructions
  - [ ] Update InitCommand PHPDoc comments
  - [ ] Add service architecture diagram to docs
  - [ ] Document new services in API docs

- [ ] **8.5** Update refactor-candidates-status.md
  - [ ] Mark Task #4 as completed
  - [ ] Update metrics (lines, complexity)
  - [ ] Add success stories
  - [ ] Document lessons learned

- [ ] **8.6** Run full test suite
  - [ ] Run PHPUnit tests
  - [ ] Check code coverage
  - [ ] Fix any failing tests
  - [ ] Verify no regressions

- [ ] **8.7** Final verification
  - [ ] Test installation manually
  - [ ] Verify CLI interface unchanged
  - [ ] Check PHPStan compatibility
  - [ ] Verify no performance regression

**Deliverable:** Complete test suite, updated documentation, status report

---

## Summary

**Total Tasks:** 103
**Estimated Time:** 9.5 hours
**Complexity:** MEDIUM
**Risk:** LOW

**Completion Criteria:**
- ✅ InitCommand reduced from 484 to ~150 lines
- ✅ 6 new service classes created
- ✅ 80%+ test coverage achieved
- ✅ All tests passing
- ✅ Documentation updated
- ✅ No breaking changes to CLI interface

**Next Steps:**
1. Begin Phase 1 (OutputService)
2. Complete phases sequentially
3. Test after each phase
4. Commit after each completed phase
