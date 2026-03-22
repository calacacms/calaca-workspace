# Refactoring PRD: InitCommand.php

> **Candidate:** calaca/src/CLI/InitCommand.php
> **Priority:** HIGH (Score: 3.3)
> **Status:** ⬜ Not Started
> **Generated:** 2026-03-19

## Executive Summary

InitCommand.php (484 lines) is the interactive installer for CalacaCMS. It suffers from static method anti-pattern, multiple responsibilities, and procedural complexity that makes testing difficult and violates separation of concerns.

**Goal:** Refactor into service-oriented architecture with proper dependency injection, separating concerns into focused, testable services.

---

## Current State Analysis

### File Metrics
- **Lines:** 484
- **Methods:** 21 (all static)
- **Responsibilities:** 7 (prompting, file ops, config gen, migrations, user creation, orchestration, output)
- **Test Coverage:** None (static methods prevent mocking)

### Technical Debt
1. **Static Anti-Pattern (Debt: 5/5)**: All methods are static, no DI support
2. **Multiple Responsibilities (Debt: 4/5)**: 7 distinct concerns in one class
3. **Procedural Complexity (Debt: 3/5)**: Long execute() method with nested logic
4. **Hardcoded Strings (Debt: 2/5)**: Templates embedded as heredoc strings
5. **Poor Testability (Debt: 5/5)**: Static methods prevent unit testing

### Architectural Violations
1. **Separation of Concerns**: User prompts, file operations, and business logic mixed
2. **Dependency Injection**: Hard-coded dependencies (StorageFactory, UsersRepository)
3. **Single Responsibility Principle**: One class handles entire installation workflow
4. **Open/Closed Principle**: Adding new installation steps requires modifying execute()

### Risk Assessment
- **Risk Level:** MEDIUM (3/5)
- **Impact:** HIGH - Installation command is critical for new projects
- **Breaking Changes:** LOW - CLI command interface remains same
- **Test Gaps:** No existing tests to break

---

## Proposed Architecture

### Service Layer Design

```
InitCommand (Orchestrator)
    ├── InstallationService (Business Logic)
    │   ├── ConfigService (Config Generation)
    │   ├── FileInstallerService (File Operations)
    │   └── UserSetupService (Admin User Creation)
    ├── PromptService (User Interaction)
    └── OutputService (CLI Formatting)
```

### Responsibility Breakdown

| Service | Responsibility | Dependencies |
|---------|---------------|--------------|
| **InstallationService** | Orchestrate installation workflow | FileInstallerService, ConfigService, UserSetupService, PromptService |
| **ConfigService** | Generate config.php files | None |
| **FileInstallerService** | Directory creation, file copying | None |
| **UserSetupService** | Create admin user, run migrations | UsersRepository, MigrationRunner |
| **PromptService** | Interactive CLI prompts | None |
| **OutputService** | CLI formatting (colors, symbols) | None |

---

## Refactoring Plan

### Phase 1: Create OutputService (Utility Layer)
**Goal:** Extract CLI formatting logic

**Actions:**
1. Create `calaca/src/CLI/OutputService.php`
2. Extract methods:
   - `printHeader()` → `OutputService::printHeader()`
   - `printStep()` → `OutputService::printStep()`
   - `ok()` → `OutputService::success()`
   - Add `error()`, `warning()`, `info()` methods
3. Make instance-based (not static)
4. Color codes as constants

**Deliverables:**
- OutputService.php (~80 lines)
- Unit tests for output formatting

**Complexity:** LOW | **Risk:** LOW | **Time:** ~30 min

---

### Phase 2: Create PromptService (Interaction Layer)
**Goal:** Extract user interaction logic

**Actions:**
1. Create `calaca/src/CLI/PromptService.php`
2. Extract methods:
   - `prompt()` → `PromptService::ask()`
   - `promptPassword()` → `PromptService::askPassword()`
   - Add `confirm()`, `select()` methods
3. Make instance-based
4. Support default values and validation

**Deliverables:**
- PromptService.php (~100 lines)
- Unit tests for prompt handling

**Complexity:** LOW | **Risk:** LOW | **Time:** ~45 min

---

### Phase 3: Create FileInstallerService (File Operations)
**Goal:** Extract filesystem operations

**Actions:**
1. Create `calaca/src/Services/FileInstallerService.php`
2. Extract methods:
   - `createDirectories()` → `FileInstallerService::createDirectoryStructure()`
   - `copyTemplates()` → `FileInstallerService::installTemplates()`
   - `copyPublicAssets()` → `FileInstallerService::installPublicAssets()`
   - `copyContentTypes()` → `FileInstallerService::installContentTypes()`
   - `recursiveCopy()` → `FileInstallerService::recursiveCopy()`
   - `createGitignore()` → `FileInstallerService::createGitignore()`
3. Accept paths via constructor
4. Return success/failure status

**Deliverables:**
- FileInstallerService.php (~150 lines)
- Unit tests for file operations

**Complexity:** MEDIUM | **Risk:** LOW | **Time:** ~1 hour

---

### Phase 4: Create ConfigService (Configuration Generation)
**Goal:** Extract config generation logic

**Actions:**
1. Create `calaca/src/Services/ConfigService.php`
2. Extract methods:
   - `generateConfig()` → `ConfigService::generateConfigFile()`
   - `generateEntryPoint()` → `ConfigService::generateEntryPoint()`
   - `buildStorageConfigCode()` → `ConfigService::buildStorageConfig()`
   - `collectStorageConfig()` → `ConfigService::collectStorageConfig()`
   - `loadExistingConfig()` → `ConfigService::loadExistingConfig()`
3. Use templates instead of heredoc strings
4. Support multiple config formats (PHP, maybe YAML later)

**Deliverables:**
- ConfigService.php (~200 lines)
- Config templates in `resources/templates/config/`
- Unit tests for config generation

**Complexity:** MEDIUM | **Risk:** MEDIUM | **Time:** ~1.5 hours

---

### Phase 5: Create UserSetupService (User & Database)
**Goal:** Extract user creation and migrations

**Actions:**
1. Create `calaca/src/Services/UserSetupService.php`
2. Extract methods:
   - `createAdminUser()` → `UserSetupService::createAdminUser()`
   - `runMigrations()` → `UserSetupService::runMigrations()`
3. Inject dependencies via constructor:
   ```php
   public function __construct(
       private UsersRepository $usersRepository,
       private MigrationRunner $migrationRunner
   ) {}
   ```
4. Add error handling and rollback support

**Deliverables:**
- UserSetupService.php (~80 lines)
- Unit tests with mocked repositories

**Complexity:** MEDIUM | **Risk:** MEDIUM | **Time:** ~1 hour

---

### Phase 6: Create InstallationService (Orchestration)
**Goal:** Create service layer for installation workflow

**Actions:**
1. Create `calaca/src/Services/InstallationService.php`
2. Implement orchestration method:
   ```php
   public function install(InstallationRequest $request): InstallationResult
   {
       // 1. Validate request
       // 2. Create directories
       // 3. Install files
       // 4. Generate config
       // 5. Run migrations
       // 6. Create admin user
       // 7. Return result
   }
   ```
3. Use value objects:
   - `InstallationRequest` (site info, storage config, admin credentials)
   - `InstallationResult` (success, errors, warnings)
4. Throw exceptions for failures

**Deliverables:**
- InstallationService.php (~150 lines)
- InstallationRequest.php (value object)
- InstallationResult.php (value object)
- Integration tests

**Complexity:** HIGH | **Risk:** MEDIUM | **Time:** ~2 hours

---

### Phase 7: Refactor InitCommand (CLI Adapter)
**Goal:** Convert InitCommand to thin CLI adapter

**Actions:**
1. Make InitCommand instance-based (not static)
2. Inject dependencies via constructor:
   ```php
   public function __construct(
       private InstallationService $installationService,
       private PromptService $promptService,
       private OutputService $outputService
   ) {}
   ```
3. Convert execute() to orchestrate prompts and call InstallationService
4. Keep `execute()` as public interface for CLI compatibility
5. Add command class to DI container

**Deliverables:**
- Refactored InitCommand.php (~150 lines, down from 484)
- Integration tests
- Updated bin/calaca to use DI container

**Complexity:** MEDIUM | **Risk:** LOW | **Time:** ~1 hour

---

### Phase 8: Testing & Documentation
**Goal:** Comprehensive test coverage and documentation

**Actions:**
1. Write unit tests for each service:
   - OutputServiceTest
   - PromptServiceTest
   - FileInstallerServiceTest
   - ConfigServiceTest
   - UserSetupServiceTest
   - InstallationServiceTest
2. Write integration test for full installation flow
3. Update InitCommand documentation
4. Add examples to README

**Deliverables:**
- 8 test files
- Integration test suite
- Updated documentation

**Complexity:** MEDIUM | **Risk:** LOW | **Time:** ~2 hours

---

## Success Criteria

### Code Quality
- ✅ InitCommand reduced from 484 to ~150 lines (69% reduction)
- ✅ All services instance-based with DI
- ✅ Each service has single, clear responsibility
- ✅ All methods testable with mocks
- ✅ No static methods (except CLI entry point)

### Test Coverage
- ✅ Unit tests for all services (80%+ coverage)
- ✅ Integration test for full installation
- ✅ Mock tests for file operations and prompts

### Functionality
- ✅ CLI interface unchanged (backward compatible)
- ✅ All installation steps working
- ✅ Error handling improved
- ✅ Rollback on installation failure

### Performance
- ✅ No performance regression
- ✅ Installation time unchanged (< 2 seconds)

---

## Risk Mitigation

### Risk 1: Breaking CLI Interface
**Mitigation:** Keep `execute()` method signature unchanged, add deprecation notices for old usage

### Risk 2: File Operation Failures
**Mitigation:** Add comprehensive error handling, test on various filesystems, support dry-run mode

### Risk 3: Dependency Hell
**Mitigation:** Use League Container for service resolution, make services loosely coupled

### Risk 4: Testing Complexity
**Mitigation:** Start with unit tests for low-risk services (Output, Prompt), work up to integration tests

---

## Estimated Effort

| Phase | Time | Complexity | Risk |
|-------|------|------------|------|
| Phase 1: OutputService | 30 min | LOW | LOW |
| Phase 2: PromptService | 45 min | LOW | LOW |
| Phase 3: FileInstallerService | 1 hour | MEDIUM | LOW |
| Phase 4: ConfigService | 1.5 hours | MEDIUM | MEDIUM |
| Phase 5: UserSetupService | 1 hour | MEDIUM | MEDIUM |
| Phase 6: InstallationService | 2 hours | HIGH | MEDIUM |
| Phase 7: InitCommand Refactor | 1 hour | MEDIUM | LOW |
| Phase 8: Testing & Docs | 2 hours | MEDIUM | LOW |
| **Total** | **9.5 hours** | **MEDIUM** | **LOW** |

---

## Rollback Plan

If refactoring fails:
1. Git revert to pre-refactor commit
2. Keep newly created services as "experimental" branch
3. Document lessons learned
4. Consider incremental approach (extract one service at a time)

---

## Open Questions

1. Should config templates be stored as PHP files or use a template engine (Twig)?
   - **Recommendation:** Start with PHP files, simpler and no dependencies

2. Should InstallationService support async operations for large file copies?
   - **Recommendation:** No, keep synchronous for simplicity, add later if needed

3. Should we support non-interactive mode (config file-based installation)?
   - **Recommendation:** Yes, add in Phase 6 as InstallationRequest can be loaded from file

4. How to handle existing installations (detect, update, reinstall)?
   - **Recommendation:** Add detection logic to InstallationService, support "update" mode

---

## Next Steps

1. **Review and approve this PRD**
2. Generate detailed task list from this PRD
3. Start with Phase 1 (OutputService) - lowest risk, quick win
4. Complete phases in order 1-8
5. Test thoroughly before committing

---

## References

- Original file: `calaca/src/CLI/InitCommand.php` (484 lines)
- Similar refactors completed:
  - Task #1: themes/admin/assets/ts/index.ts (✅)
  - Task #2: MediaController.php (✅)
  - Task #3: Bootstrap.php (✅)
- Project architecture: `agent-context/PROJECT_SSOT.md` (workspace root; from `cms/`: `../agent-context/PROJECT_SSOT.md`)
- Refactor status: `docs/refactor-candidates-status.md`
