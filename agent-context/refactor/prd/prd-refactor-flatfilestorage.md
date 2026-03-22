# PRD: FlatFileStorage Refactoring

**File:** `calaca/src/Storage/FlatFileStorage.php`
**Current Size:** 352 lines
**Target Size:** ~200 lines (FlatFileStorage) + separate service classes
**Priority:** HIGH (Score 3.2)
**Complexity:** Medium

## Problem Statement

The `FlatFileStorage` class implements a complete database engine with mixed responsibilities:

### Current Issues

1. **Query Logic Duplication**
   - WHERE clause matching repeated in `select()`, `update()`, `delete()`
   - ORDER BY sorting only in `select()` but could be useful elsewhere
   - No reusable query building components

2. **Transaction Management Mixed with Data Access**
   - Transaction state (`$inTransaction`, `$transactionData`) managed alongside CRUD
   - Cache invalidation logic scattered throughout methods
   - Hard to test transaction logic independently

3. **File System Operations Scattered**
   - `getTableDir()`, `getRecordPath()`, `writeRecord()`, `syncTableToDisk()`
   - Migration logic (`migrateLegacyTable()`) mixed with CRUD
   - No abstraction for file I/O

4. **Cache Management Coupled with Storage**
   - Cache array directly manipulated in multiple methods
   - No clear cache invalidation strategy
   - Hard to add caching layers (e.g., Redis, Memcached)

5. **Migration Logic Tightly Coupled**
   - Legacy format detection in `loadTable()`
   - Migration triggered automatically on load
   - Can't test migration independently
   - Can't run migrations in batch

## Proposed Architecture

### Service Layer Structure

```
FlatFileStorage (facade - ~200 lines)
├── QueryProcessor (query logic)
│   ├── where()
│   ├── orderBy()
│   ├── limit()
│   └── offset()
├── TransactionManager (transaction state)
│   ├── begin()
│   ├── commit()
│   ├── rollback()
│   └── isInTransaction()
├── CacheManager (in-memory caching)
│   ├── get()
│   ├── set()
│   ├── has()
│   ├── clear()
│   └── invalidate()
├── FileStorageAdapter (file I/O)
│   ├── readTable()
│   ├── writeTable()
│   ├── readRecord()
│   ├── writeRecord()
│   ├── deleteRecord()
│   └── tableExists()
├── MigrationService (legacy migration)
│   ├── detectLegacyFormat()
│   ├── migrateTable()
│   └── removeLegacyFile()
└── TableNameSanitizer (utility)
    └── sanitize()
```

### New Classes

1. **QueryProcessor** - Query logic operations
   ```php
   class QueryProcessor
   {
       public function where(array $data, array $conditions): array;
       public function orderBy(array $data, array $sort): array;
       public function limit(array $data, int $limit): array;
       public function offset(array $data, int $offset): array;
   }
   ```

2. **TransactionManager** - Transaction state management
   ```php
   class TransactionManager
   {
       public function begin(): void;
       public function commit(): void;
       public function rollback(): void;
       public function isInTransaction(): bool;
       public function getSnapshot(): array;
   }
   ```

3. **CacheManager** - Cache operations
   ```php
   class CacheManager
   {
       public function get(string $key): ?array;
       public function set(string $key, array $data): void;
       public function has(string $key): bool;
       public function clear(): void;
       public function clearTable(string $table): void;
   }
   ```

4. **FileStorageAdapter** - File I/O abstraction
   ```php
   class FileStorageAdapter
   {
       public function readTable(string $tableDir): array;
       public function writeTable(string $tableDir, array $data): void;
       public function readRecord(string $filePath): ?array;
       public function writeRecord(string $filePath, array $record): void;
       public function deleteRecord(string $filePath): bool;
       public function tableExists(string $tableDir): bool;
   }
   ```

5. **MigrationService** - Legacy format migration
   ```php
   class MigrationService
   {
       public function detectLegacyFormat(string $table): bool;
       public function migrateTable(string $table, string $legacyFile): void;
       public function removeLegacyFile(string $legacyFile): void;
   }
   ```

6. **TableNameSanitizer** - Utility for sanitizing table names
   ```php
   class TableNameSanitizer
   {
       public function sanitize(string $table): string;
   }
   ```

7. **FlatFileStorage** (refactored) - Facade
   ```php
   class FlatFileStorage implements StorageInterface
   {
       public function __construct(
           string $storagePath,
           ?QueryProcessor $queryProcessor = null,
           ?TransactionManager $transactionManager = null,
           ?CacheManager $cacheManager = null,
           ?FileStorageAdapter $storageAdapter = null,
           ?MigrationService $migrationService = null
       ) {}
   }
   ```

## Implementation Phases

### Phase 1: Create Utility Classes (2 tasks)
- [ ] Create `TableNameSanitizer` utility class
- [ ] Create `FileStorageAdapter` for file I/O operations

**Tests:** Unit tests for FileStorageAdapter with temp directory

### Phase 2: Extract Query Logic (3 tasks)
- [ ] Create `QueryProcessor` class
- [ ] Implement `where()`, `orderBy()`, `limit()`, `offset()`
- [ ] Update `select()` to use QueryProcessor

**Tests:** Unit tests for each query operation

### Phase 3: Extract Transaction Management (3 tasks)
- [ ] Create `TransactionManager` class
- [ ] Implement `begin()`, `commit()`, `rollback()`
- [ ] Update FlatFileStorage to use TransactionManager

**Tests:** Test transaction commit/rollback behavior

### Phase 4: Extract Cache Management (3 tasks)
- [ ] Create `CacheManager` class
- [ ] Implement cache operations
- [ ] Update FlatFileStorage to use CacheManager

**Tests:** Test cache hit/miss behavior

### Phase 5: Extract Migration Logic (3 tasks)
- [ ] Create `MigrationService` class
- [ ] Implement legacy format detection and migration
- [ ] Update FlatFileStorage to use MigrationService

**Tests:** Test migration with legacy file

### Phase 6: Refactor FlatFileStorage (4 tasks)
- [ ] Inject all services into constructor
- [ ] Update CRUD methods to use services
- [ ] Remove duplicated code
- [ ] Keep public API unchanged

**Tests:** Integration tests for all CRUD operations

### Phase 7: Documentation & Cleanup (3 tasks)
- [ ] Document new architecture
- [ ] Add usage examples
- [ ] Update class docblocks

### Phase 8: Testing & Verification (5 tasks)
- [ ] Test SELECT with WHERE/ORDER/LIMIT/OFFSET
- [ ] Test INSERT with auto-increment ID
- [ ] Test UPDATE with WHERE condition
- [ ] Test DELETE with WHERE condition
- [ ] Test transactions (commit/rollback)
- [ ] Test legacy migration
- [ ] Verify no data loss

## Success Criteria

- [ ] FlatFileStorage reduced to ~200 lines
- [ ] 6 new service classes created
- [ ] No query logic duplication
- [ ] Transaction logic testable independently
- [ ] Cache operations centralized
- [ ] File I/O abstracted
- [ ] Migration logic separable
- [ ] All existing functionality preserved
- [ ] No data loss or corruption

## Migration Notes

**No Breaking Changes:**
- FlatFileStorage implements StorageInterface (unchanged)
- All public methods unchanged (select, insert, update, delete, etc.)
- Existing code continues to work
- File format unchanged

**Extension Points:**
- Custom QueryProcessor for different query dialects
- Custom CacheManager for distributed caching (Redis)
- Custom FileStorageAdapter for different file formats
- Custom MigrationService for other legacy formats

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Data corruption during refactor | Comprehensive integration tests, backup/restore |
| Performance regression from abstraction | Benchmarks, optimize hot paths |
| Transaction edge cases | Extensive transaction testing |
| Cache coherency issues | Clear invalidation strategy |
| Legacy migration failures | Test migration with various legacy files |

## Estimated Complexity

- **New Classes:** 6 service classes
- **Lines of Code:** ~600 total (vs 352 current)
- **Development Time:** 3-4 hours
- **Risk Level:** Medium (critical data layer, changes need thorough testing)

## References

- Original FlatFileStorage: 352 lines
- Implements: StorageInterface (full CRUD + transactions)
- File format: JSON files, one per record
- Legacy format: Single JSON file per table
- Features: Auto-migration, caching, transactions
