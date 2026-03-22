# Refactoring PRD: calaca/src/Controllers/Admin/MediaController.php

**Document Version:** 1.0
**Date:** 2026-03-19
**File:** calaca/src/Controllers/Admin/MediaController.php
**Current Lines:** 510
**Priority:** CRITICAL

---

## 1. Problem Statement

The `MediaController.php` file currently has multiple critical issues:

1. **God Controller Anti-pattern** (CRITICAL):
   - 510 lines with too many responsibilities
   - Handles: upload, delete, bulk delete, info, download, create folder, API endpoints, plus helper methods
   - Single file doing HTTP handling, business logic, data access, formatting

2. **Placeholder Implementation** (CRITICAL):
   - `getMediaFileById()` returns `null` (line 438)
   - Comment says "This will be implemented with a media repository"
   - **MediaRepository already exists** with full `find()` method - not being used!
   - Controller bypasses proper data layer

3. **Duplicate Code** (HIGH):
   - CSRF validation repeated 4 times (lines 77, 147, 181, 272)
   - Path traversal checks duplicated in createFolder
   - Storage info calculation should be in service layer

4. **Mixed Concerns** (HIGH):
   - Business logic (file type detection) mixed with HTTP handling
   - Formatting logic (formatBytes) in controller instead of utility
   - Folder creation logic in controller instead of service

5. **Tight Coupling** (MEDIUM):
   - Direct file operations instead of using existing services
   - FileUploadService exists but not fully utilized
   - MediaRepository exists but completely ignored

---

## 2. Impact Assessment

### Technical Debt Score: 4/5
- God Controller: Too many responsibilities in one class
- Placeholder: Critical functionality returns null
- Duplication: CSRF validation, path checks repeated
- Coupling: Bypasses existing service layer

### Impact Score: 5/5
- User-facing: Media management is core admin functionality
- Dependencies: File upload affects articles/pages
- Data risk: File operations without proper repository layer
- Security risk: Placeholder means no actual file tracking

### Risk Score: 4/5
- Critical path: Media operations are data-critical
- Test coverage: Existing MediaRepository has tests (MediaRepositoryTest.php, 363 lines)
- Data integrity: No repository usage means no audit trail
- Rollback: Medium risk - changes are localized to media operations

### Complexity Score: 3/5
- Code is straightforward but large
- Clear separation points identified
- Existing services can be leveraged

**Overall Priority: CRITICAL (Score: 4.1/5)**

---

## 3. Refactoring Goals

### Primary Goals
1. **Integrate MediaRepository**: Use existing repository instead of placeholder
2. **Extract Folder Operations**: Create FolderService for folder management
3. **Remove CSRF Duplication**: Create middleware or helper method
4. **Extract Formatting**: Move formatting utilities to dedicated class
5. **Thin Controller**: Keep only HTTP request/response handling

### Secondary Goals
1. **Improve Testability**: Make controller testable by removing business logic
2. **Add Audit Trail**: Use MediaRepository for data tracking
3. **Enhance Security**: Consistent CSRF validation across all methods
4. **Reduce Coupling**: Controller should not know about file operations directly

---

## 4. Current Architecture

```
MediaController (510 lines)
├── Constructor (lines 22-30)
│   └── Injects: FileUploadService, ArticleRepository, PageRepository
├── Public Methods (7 methods, 248 lines)
│   ├── index() - Media library view
│   ├── upload() - Handle file upload
│   ├── delete() - Delete single file
│   ├── bulkDelete() - Delete multiple files
│   ├── info() - Get file info
│   ├── download() - Download file
│   └── createFolder() - Create new folder
├── API Methods (2 methods, 36 lines)
│   ├── apiMedia() - JSON media list
│   └── apiFolders() - JSON folder list
└── Private Methods (7 methods, 196 lines)
    ├── getMediaFiles() - Filter and enrich files
    ├── getFolders() - List upload folders
    ├── getBreadcrumbs() - Build navigation
    ├── getStorageInfo() - Calculate storage usage
    ├── getMediaFileById() - PLACEHOLDER (returns null)
    ├── getFileType() - Detect file type from MIME
    └── formatBytes() - Human-readable file size
```

**Issues:**
- getMediaFileById() is placeholder - MediaRepository.find() exists
- CSRF validation duplicated 4 times
- Formatting logic belongs in utility class
- Folder logic should be in service layer
- Too many private helper methods in controller

---

## 5. Target Architecture

```
MediaController (after refactor - ~150 lines)
├── Constructor
│   └── Injects: MediaRepository, FolderService, MediaFormatter
├── Public Methods (same 7 methods)
│   ├── index() - Delegates to services
│   ├── upload() - Uses MediaRepository
│   ├── delete() - Uses MediaRepository
│   ├── bulkDelete() - Uses MediaRepository
│   ├── info() - Uses MediaRepository
│   ├── download() - Uses MediaRepository
│   └── createFolder() - Uses FolderService
├── API Methods (same 2 methods)
│   ├── apiMedia() - Delegates to MediaRepository
│   └── apiFolders() - Delegates to FolderService
└── Minimal Logic (no private helper methods)
```

**New Services:**
```
FolderService (new file)
├── createFolder(name, parent) - Create folder with validation
├── listFolders(parent) - List folders in directory
├── deleteFolder(path) - Delete folder
└── getFolderInfo(path) - Get folder metadata

MediaFormatter (new utility class)
├── formatBytes(bytes) - Human-readable file size
├── formatFileDate(timestamp) - Formatted date strings
└── buildBreadcrumbs(path) - Navigation breadcrumbs
```

**Benefits:**
- Uses existing MediaRepository (no placeholder)
- Controller is thin HTTP layer
- Business logic in service layer
- Formatting extracted to utilities
- CSRF validation centralized

---

## 6. Implementation Plan

### Phase 1: Integrate MediaRepository
- [ ] Inject MediaRepository into controller constructor
- [ ] Replace getMediaFileById() with MediaRepository->find()
- [ ] Update upload() to create MediaRepository records
- [ ] Update delete() to use MediaRepository->delete()
- [ ] Update bulkDelete() to use MediaRepository->deleteByType()
- [ ] Update info() to use MediaRepository->find()
- [ ] Update apiMedia() to use MediaRepository methods
- [ ] Remove private getMediaFiles(), use MediaRepository->findAll()

### Phase 2: Create FolderService
- [ ] Create calaca/src/Services/FolderService.php
- [ ] Implement createFolder() with path traversal protection
- [ ] Implement listFolders() for directory listing
- [ ] Implement deleteFolder() for folder deletion
- [ ] Move createFolder() logic from controller to service

### Phase 3: Create MediaFormatter
- [ ] Create calaca/src/Services/MediaFormatter.php
- [ ] Move formatBytes() to formatter class
- [ ] Move getBreadcrumbs() logic to formatter
- [ ] Add formatFileDate() method
- [ ] Inject formatter into controller

### Phase 4: Centralize CSRF Validation
- [ ] Create private validateCsrfForWrite() method
- [ ] Replace all 4 CSRF validations with single method call
- [ ] Add consistent error messages

### Phase 5: Clean Up Controller
- [ ] Remove all private helper methods
- [ ] Remove direct file operations
- [ ] Remove duplicate code
- [ ] Update JSDoc comments
- [ ] Ensure all methods use services

### Phase 6: Verification
- [ ] Test file upload with MediaRepository tracking
- [ ] Test folder creation/deletion
- [ ] Test media library view (uses new services)
- [ ] Test bulk delete operations
- [ ] Test download functionality
- [ ] Verify CSRF protection works
- [ ] Run existing MediaRepositoryTest.php

---

## 7. Detailed Changes

### Change 1: Inject MediaRepository
**Before:**
```php
public function __construct(
    FileUploadService $fileUploadService,
    ArticleRepository $articleRepository,
    PageRepository $pageRepository,
    ?Request $request = null
) {
    parent::__construct($articleRepository, $pageRepository, $request);
    $this->fileUploadService = $fileUploadService;
}
```

**After:**
```php
public function __construct(
    MediaRepository $mediaRepository,
    FolderService $folderService,
    MediaFormatter $mediaFormatter,
    ?Request $request = null
) {
    $this->mediaRepository = $mediaRepository;
    $this->folderService = $folderService;
    $this->mediaFormatter = $mediaFormatter;
    $this->request = $request ?? Request::createFromGlobals();
}
```

### Change 2: Replace Placeholder with Repository
**Before:**
```php
private function getMediaFileById(string $id): ?array
{
    // This will be implemented with a media repository
    // For now, return dummy data
    return null;
}
```

**After:**
```php
// In upload(), delete(), info(), download():
$file = $this->mediaRepository->find($id);
if (!$file) {
    return $this->jsonError('File not found.', 404);
}
```

### Change 3: Centralize CSRF Validation
**Before (repeated 4 times):**
```php
if (!$this->validateCsrfToken($this->request->request->get('_token'))) {
    return $this->jsonError('Invalid security token.');
}
```

**After:**
```php
// Add to controller:
private function validateWriteRequest(): ?Response
{
    if (!$this->validateCsrfToken($this->request->request->get('_token'))) {
        return $this->jsonError('Invalid security token.');
    }
    return null;
}

// In each write method:
if (($error = $this->validateWriteRequest()) !== null) {
    return $error;
}
```

### Change 4: Extract to FolderService
**Before (in controller, lines 267-325):**
```php
public function createFolder(): Response
{
    // ... 58 lines of folder creation logic ...
    $uploadsDir = realpath(TemplateService::getProjectRoot() . '/storage/uploads');
    // ... path validation ...
    mkdir($folderPath, 0755, true);
}
```

**After (new FolderService):**
```php
// calaca/src/Services/FolderService.php
public function createFolder(string $name, string $parent = ''): array
{
    // ... folder creation logic with validation ...
}

// In controller:
public function createFolder(): Response
{
    $this->requireAuth();

    if (($error = $this->validateWriteRequest()) !== null) {
        return $error;
    }

    $result = $this->folderService->createFolder(
        $this->request->request->get('name'),
        $this->request->request->get('parent', '')
    );

    return $result['success']
        ? $this->jsonSuccess(['message' => $result['message']])
        : $this->jsonError($result['error']);
}
```

### Change 5: Extract to MediaFormatter
**Before (formatBytes in controller):**
```php
private function formatBytes(int $bytes): string
{
    $units = ['B', 'KB', 'MB', 'GB'];
    // ... formatting logic ...
    return round($bytes, 2) . ' ' . $units[$pow];
}
```

**After:**
```php
// In controller:
$storageInfo = $this->folderService->getStorageInfo();
$data['storage_used'] = $storageInfo['used'];
$data['storage_available'] = $storageInfo['available'];
$data['storage_percent'] = $storageInfo['percent'];
```

---

## 8. Risk Mitigation

### Risk 1: Breaking Media Upload
- **Mitigation**: MediaRepository has comprehensive tests (MediaRepositoryTest.php)
- **Rollback**: Keep original MediaController in branch for 24 hours
- **Testing**: Manual testing of all media operations

### Risk 2: Data Loss from File Operations
- **Mitigation**: MediaRepository handles actual file deletion with proper checks
- **Backup**: Test in staging environment before production
- **Validation**: Run existing MediaRepositoryTest.php after refactor

### Risk 3: CSRF Protection Issues
- **Mitigation**: Keep same validation logic, just extract to method
- **Testing**: Test all write operations with invalid tokens
- **Review**: Verify BaseAdminController's validateCsrfToken is correct

---

## 9. Success Criteria

- [ ] MediaController reduced to ~150 lines (from 510)
- [ ] Zero placeholder implementations
- [ ] All methods use MediaRepository
- [ ] Folder operations use FolderService
- [ ] CSRF validation centralized
- [ ] Formatting logic extracted
- [ ] Zero direct file operations in controller
- [ ] Existing MediaRepositoryTest.php still passes
- [ ] All media operations functional
- [ ] Download functionality works
- [ ] Folder creation/deletion works
- [ ] Bulk delete operations work
- [ ] No regressions in media library view

---

## 10. Testing Strategy

### Unit Testing
- [ ] Run existing MediaRepositoryTest.php
- [ ] Verify all repository operations still work
- [ ] Check create() operation with controller integration
- [ ] Test delete() operation with controller integration

### Integration Testing
- [ ] Test file upload (image and document)
- [ ] Test file deletion (single and bulk)
- [ ] Test folder creation with parent folder
- [ ] Test folder navigation (breadcrumbs)
- [ ] Test download functionality
- [ ] Test media library view rendering
- [ ] Test storage info calculation

### Security Testing
- [ ] Test CSRF protection on all write operations
- [ ] Test path traversal attempts in folder creation
- [ ] Test file type validation
- [ ] Test unauthorized access attempts

### Regression Testing
- [ ] Navigate to /admin/media
- [ ] Test all media CRUD operations
- [ ] Verify no broken functionality
- [ ] Check API endpoints (apiMedia, apiFolders)

---

## 11. Rollback Plan

If critical issues arise after deployment:

1. **Immediate**: Revert to original MediaController.php
2. **Within 1 hour**: Restore from git backup
3. **Investigation**: Review MediaRepository integration
4. **Fix**: Address specific breaking change
5. **Re-test**: Run MediaRepositoryTest.php before redeploy

---

## 12. Dependencies

### Internal Dependencies
- **MediaRepository**: Must integrate properly (exists, tested)
- **FileUploadService**: Already injected, will keep using
- **FolderService**: New service to create
- **MediaFormatter**: New utility class to create

### External Dependencies
- **Symfony HttpFoundation**: Request/Response handling
- **PHP File Functions**: Handled by MediaRepository and FileUploadService
- **Storage Drivers**: MediaRepository abstracts storage layer

### Build Tooling
- **Composer**: For dependency injection (already configured)
- **PHPUnit**: For running MediaRepositoryTest.php

---

## 13. Time Estimate

- **Phase 1** (Integrate MediaRepository): 45 minutes
- **Phase 2** (Create FolderService): 40 minutes
- **Phase 3** (Create MediaFormatter): 30 minutes
- **Phase 4** (Centralize CSRF): 20 minutes
- **Phase 5** (Clean Up Controller): 30 minutes
- **Phase 6** (Verification): 40 minutes

**Total**: 3 - 3.5 hours

---

## 14. Post-Refactor Metrics

### Before Refactor
- Lines: 510
- Placeholder implementations: 1 (getMediaFileById)
- CSRF validation: 4 duplicate blocks
- Private helper methods: 7
- Direct file operations: Yes (folder creation)
- Service layer usage: Minimal (only FileUploadService)

### After Refactor
- Lines: ~150 (70% reduction)
- Placeholder implementations: 0
- CSRF validation: 1 centralized method
- Private helper methods: 0
- Direct file operations: No (all via services)
- Service layer usage: Full (MediaRepository, FolderService, MediaFormatter)

### Technical Debt Reduction
- **Placeholder fixed**: 100% - using MediaRepository
- **Duplication removed**: CSRF validation
- **Coupling reduced**: Controller no longer does file operations
- **Maintainability score**: +4 (on 1-10 scale)
- **Test coverage**: MediaRepository already has tests

---

## 15. Migration Notes

### Data Migration
- **No migration needed**: MediaRepository uses same storage interface
- **Backward compatible**: Same storage structure maintained
- **Zero data loss**: Only refactoring code, not data

### API Changes
- **No breaking changes**: Same HTTP interface maintained
- **Response format**: Identical JSON structure
- **Request format**: Same parameters accepted

### Configuration
- **No config changes**: Uses existing dependency injection
- **No new dependencies**: Leverages existing services

---

## 16. Related Issues

This refactoring addresses multiple issues from the refactor analysis:

1. **God Controller**: Fixed by thinning controller
2. **Placeholder Implementation**: Fixed by using MediaRepository
3. **Mixed Concerns**: Fixed by extracting services
4. **Duplicate Code**: Fixed by centralizing CSRF validation
5. **Tight Coupling**: Fixed by using service layer

---

## 17. Future Improvements

After this refactoring, consider:

1. **Add Thumbnail Generation**: Enhance MediaRepository with image processing
2. **Add Media Search**: Use MediaRepository->search() method
3. **Add Media Categorization**: Extend MediaRepository with tags/categories
4. **Add Media Sharing**: Implement sharing URLs for media items
5. **Add Media Versioning**: Track file versions/edits
6. **Add Media Analytics**: Track most-used, recently-uploaded files
