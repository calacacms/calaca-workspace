# Refactoring Task List: calaca/src/Controllers/Admin/MediaController.php

**Generated from:** docs/prd-refactor-mediacontroller.md
**Date:** 2026-03-19
**File:** calaca/src/Controllers/Admin/MediaController.php
**Estimated Time:** 3 - 3.5 hours

---

## Task Overview

Refactor `calaca/src/Controllers/Admin/MediaController.php` to eliminate God Controller anti-pattern, integrate existing MediaRepository, extract services, and reduce from 510 lines to ~150 lines while maintaining exact functionality.

---

## Phase 1: Integrate MediaRepository

### Task 1.1: Analyze MediaRepository interface
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Review MediaRepository to understand available methods

**Subtasks:**
- [ ] Review MediaRepository->find() method signature
- [ ] Review MediaRepository->findAll() method with filters
- [ ] Review MediaRepository->delete() method
- [ ] Review MediaRepository->search() method
- [ ] Document method return types and expected data structures
- [ ] Map controller methods to repository methods

**Acceptance Criteria:**
- Complete mapping of controller methods to repository methods
- Documentation of repository interface
- Understanding of data structures used

---

### Task 1.2: Update controller constructor
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Inject MediaRepository and remove unused dependencies

**Subtasks:**
- [ ] Add `use CalacaCMS\Repositories\MediaRepository;` import
- [ ] Remove `use CalacaCMS\Repositories\ArticleRepository;` (unused)
- [ ] Remove `use CalacaCMS\Repositories\PageRepository;` (unused)
- [ ] Add `MediaRepository $mediaRepository` parameter to constructor
- [ ] Remove `ArticleRepository $articleRepository` parameter
- [ ] Remove `PageRepository $pageRepository` parameter
- [ ] Update parent::__construct() call (remove unused parameters)
- [ ] Add `$this->mediaRepository = $mediaRepository;` assignment

**Acceptance Criteria:**
- MediaRepository injected properly
- Unused dependencies removed
- Constructor signature updated
- TypeScript (if applicable) or PHPStan passes

---

### Task 1.3: Replace getMediaFileById placeholder
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Remove placeholder method and use MediaRepository->find()

**Subtasks:**
- [ ] Delete private getMediaFileById() method (lines 434-439)
- [ ] Find all usages of getMediaFileById() in controller
- [ ] Replace with `$this->mediaRepository->find($id)`
- [ ] Update delete() method to use MediaRepository
- [ ] Update info() method to use MediaRepository
- [ ] Update download() method to use MediaRepository
- [ ] Add null checks after repository calls

**Acceptance Criteria:**
- getMediaFileById() method deleted
- All usages replaced with MediaRepository->find()
- Null checks added after repository calls
- No compilation errors

---

### Task 1.4: Update upload() to use MediaRepository
**Status:** ⬜ Not Started
**Estimated:** 20 minutes
**Description:** Store uploaded files in MediaRepository for tracking

**Subtasks:**
- [ ] After successful upload, call MediaRepository->create()
- [ ] Map upload result to repository create() structure:
  ```php
  [
      'type' => $fileType, // 'image' or 'document'
      'filename' => $result['filename'],
      'path' => $result['path'],
      'size' => $file['size'],
      'mime_type' => $file['type'],
  ]
  ```
- [ ] Store repository returned data (id, created_at, etc.)
- [ ] Update JSON response to include repository data
- [ ] Handle create() errors gracefully

**Acceptance Criteria:**
- Uploads create MediaRepository records
- Response includes file metadata from repository
- Error handling for repository failures
- No data loss in uploads

---

### Task 1.5: Update delete() to use MediaRepository
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Use MediaRepository->delete() instead of FileUploadService->deleteFile()

**Subtasks:**
- [ ] Keep getMediaFileById() call (now uses repository)
- [ ] Replace `$this->fileUploadService->deleteFile($file['path'])`
- [ ] With `$this->mediaRepository->delete($id)`
- [ ] Update success message to reference repository delete
- [ ] Update error handling for repository failures

**Acceptance Criteria:**
- Uses MediaRepository->delete() method
- FileUploadService->deleteFile() removed from delete()
- Error handling updated for repository
- Success messages reference repository operations

---

### Task 1.6: Update bulkDelete() to use MediaRepository
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Delete multiple files using MediaRepository

**Subtasks:**
- [ ] Keep file retrieval logic (getMediaFileById via repository)
- [ ] Replace FileUploadService->deleteFile() with MediaRepository->delete()
- [ ] Update error tracking to use repository feedback
- [ ] Maintain count tracking for deleted files
- [ ] Ensure transaction consistency (all or nothing)

**Acceptance Criteria:**
- Bulk delete uses MediaRepository->delete()
- No direct file operations
- Proper error handling
- Transactional behavior (all succeed or all fail)

---

### Task 1.7: Update info() to use MediaRepository
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Use MediaRepository->find() for file info endpoint

**Subtasks:**
- [ ] Replace getMediaFileById() call with MediaRepository->find()
- [ ] Update 404 error to match repository null result
- [ ] Return repository data instead of file data
- [ ] Include all repository fields in response

**Acceptance Criteria:**
- Uses MediaRepository->find()
- Returns repository data structure
- Proper 404 handling
- All fields included

---

### Task 1.8: Update apiMedia() to use MediaRepository
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** API endpoint uses MediaRepository methods

**Subtasks:**
- [ ] Remove getMediaFiles() private method call
- [ ] Replace with MediaRepository->findAll()
- [ ] Apply type filter via repository (if supported)
- [ ] Apply search via repository (if supported)
- [ ] Keep URL enrichment logic (add url field to results)
- [ ] Add thumbnail_url logic (if applicable)

**Acceptance Criteria:**
- Uses MediaRepository->findAll() or search()
- No private getMediaFiles() method
- Filters applied via repository
- URL enrichment preserved

---

### Task 1.9: Update apiFolders() to prepare for FolderService
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Prepare API endpoint for FolderService extraction

**Subtasks:**
- [ ] Note that getFolders() will be moved to FolderService
- [ ] Plan to inject FolderService in constructor
- [ ] Update method to call `$this->folderService->listFolders()`
- [ ] Ensure JSON response format preserved

**Acceptance Criteria:**
- API endpoint prepared for FolderService
- Response format unchanged
- No breaking changes to API

---

## Phase 2: Create FolderService

### Task 2.1: Create FolderService file
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Create new service file structure

**Subtasks:**
- [ ] Create file: `calaca/src/Services/FolderService.php`
- [ ] Add namespace: `namespace CalacaCMS\Services;`
- [ ] Add class declaration: `class FolderService`
- [ ] Add constructor with TemplateService or project root
- [ ] Add proper PHPDoc comments
- [ ] Follow FileUploadService style guidelines

**Acceptance Criteria:**
- FolderService.php created
- Proper namespace and class structure
- PHPDoc comments added
- Follows coding standards

---

### Task 2.2: Implement createFolder() method
**Status:** ⬜ Not Started
**Estimated:** 20 minutes
**Description:** Move folder creation logic from controller to service

**Subtasks:**
- [ ] Copy lines 267-325 from MediaController (58 lines)
- [ ] Extract into public createFolder() method
- [ ] Add parameters: string $name, string $parent = ''
- [ ] Return array with success/error structure:
  ```php
  ['success' => bool, 'message' => string, 'path' => string]
  ```
- [ ] Update path resolution to use service's project root
- [ ] Keep all validation logic (path traversal, etc.)
- [ ] Add proper error handling

**Acceptance Criteria:**
- createFolder() method implemented
- All validation logic preserved
- Returns proper data structure
- Error handling complete

---

### Task 2.3: Implement listFolders() method
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Move folder listing from controller to service

**Subtasks:**
- [ ] Copy lines 362-383 from MediaController (22 lines)
- [ ] Extract into public listFolders() method
- [ ] Add parameter: string $parent = ''
- [ ] Update path resolution to use service's project root
- [ ] Keep DirectoryIterator logic
- [ ] Return array of folder structures

**Acceptance Criteria:**
- listFolders() method implemented
- Directory listing logic preserved
- Path resolution updated
- Returns array of folders

---

### Task 2.4: Implement getFolderInfo() method (if needed)
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Add method for folder metadata

**Subtasks:**
- [ ] Determine if folder info is needed (count, size, etc.)
- [ ] Implement getFolderInfo(string $path) method
- [ ] Calculate folder statistics
- [ ] Return formatted folder data
- [ ] Add proper error handling

**Acceptance Criteria:**
- getFolderInfo() method implemented (if needed)
- Folder statistics calculated
- Proper error handling

---

### Task 2.5: Test FolderService
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Verify FolderService works correctly

**Subtasks:**
- [ ] Create folder in test directory
- [ ] Call createFolder() with valid data
- [ ] Call listFolders() to verify listing
- [ ] Test with invalid parent folder
- [ ] Test with empty folder name
- [ ] Test path traversal attempts
- [ ] Verify all validations work
- [ ] Clean up test directory

**Acceptance Criteria:**
- createFolder() works with valid data
- listFolders() returns correct data
- Validations prevent invalid operations
- No security vulnerabilities in path handling

---

### Task 2.6: Inject FolderService into controller
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Add FolderService to MediaController dependencies

**Subtasks:**
- [ ] Add `use CalacaCMS\Services\FolderService;` import
- [ ] Add `FolderService $folderService` parameter to constructor
- [ ] Add `$this->folderService = $folderService;` assignment
- [ ] Update constructor if already modified in Task 1.2

**Acceptance Criteria:**
- FolderService injected properly
- Controller has access to folder operations
- Constructor signature correct

---

### Task 2.7: Update createFolder() controller method
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Use FolderService instead of inline logic

**Subtasks:**
- [ ] Remove inline folder creation logic (lines 267-325)
- [ ] Call `$this->folderService->createFolder()`
- [ ] Pass request parameters: name, parent
- [ ] Handle service result
- [ ] Return appropriate JSON response
- [ ] Keep CSRF validation (will be centralized later)

**Acceptance Criteria:**
- Inline folder logic removed
- Uses FolderService->createFolder()
- Response format preserved
- No logic duplication

---

### Task 2.8: Update apiFolders() controller method
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Use FolderService->listFolders()

**Subtasks:**
- [ ] Remove inline getFolders() private method
- [ ] Call `$this->folderService->listFolders()`
- [ ] Pass parent parameter if needed
- [ ] Return service result as JSON
- [ ] Keep auth validation

**Acceptance Criteria:**
- Inline folder listing removed
- Uses FolderService->listFolders()
- Response format preserved
- Auth validation maintained

---

## Phase 3: Create MediaFormatter

### Task 3.1: Create MediaFormatter file
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Create new utility class for formatting

**Subtasks:**
- [ ] Create file: `calaca/src/Services/MediaFormatter.php`
- [ ] Add namespace: `namespace CalacaCMS\Services;`
- [ ] Add class declaration: `class MediaFormatter`
- [ ] Add proper PHPDoc comments
- [ ] Follow project coding standards

**Acceptance Criteria:**
- MediaFormatter.php created
- Proper namespace and class structure
- PHPDoc comments added
- Follows coding standards

---

### Task 3.2: Implement formatBytes() method
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Move formatBytes from controller to formatter

**Subtasks:**
- [ ] Copy lines 460-469 from MediaController
- [ ] Extract into public formatBytes() method
- [ ] Add parameter: int $bytes
- [ ] Return string with unit
- [ ] Keep exact logic (no behavior change)

**Acceptance Criteria:**
- formatBytes() method implemented
- Logic identical to original
- Public method accessible to controller

---

### Task 3.3: Implement buildBreadcrumbs() method
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Move breadcrumbs logic from controller to formatter

**Subtasks:**
- [ ] Copy lines 388-403 from MediaController
- [ ] Extract into public buildBreadcrumbs() method
- [ ] Add parameter: string $path
- [ ] Return array of breadcrumb items
- [ ] Keep exact logic (no behavior change)

**Acceptance Criteria:**
- buildBreadcrumbs() method implemented
- Logic identical to original
- Returns same breadcrumb structure

---

### Task 3.4: Implement formatFileDate() method (optional)
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Add utility for date formatting

**Subtasks:**
- [ ] Add formatFileDate(int $timestamp) method
- [ ] Format date for display
- [ ] Return formatted string
- [ ] Use in controller if needed

**Acceptance Criteria:**
- formatFileDate() method implemented
- Date formatting logic correct
- Optional (only if needed)

---

### Task 3.5: Inject MediaFormatter into controller
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Add MediaFormatter to MediaController

**Subtasks:**
- [ ] Add `use CalacaCMS\Services\MediaFormatter;` import
- [ ] Add `MediaFormatter $mediaFormatter` parameter to constructor
- [ ] Add `$this->mediaFormatter = $mediaFormatter;` assignment
- [ ] Update constructor if already modified

**Acceptance Criteria:**
- MediaFormatter injected properly
- Controller has access to formatting methods
- Constructor signature correct

---

### Task 3.6: Update index() to use MediaFormatter
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Use formatter for storage info and breadcrumbs

**Subtasks:**
- [ ] Remove inline getStorageInfo() method (lines 408-429)
- [ ] Call `$this->mediaFormatter->formatBytes()` if needed
- [ ] Call `$this->mediaFormatter->buildBreadcrumbs()`
- [ ] Update data array to use formatted values
- [ ] Keep response format identical

**Acceptance Criteria:**
- getStorageInfo() removed
- Uses MediaFormatter methods
- Response format preserved
- No visual changes

---

## Phase 4: Centralize CSRF Validation

### Task 4.1: Create validateWriteRequest() method
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Extract CSRF validation to single method

**Subtasks:**
- [ ] Add private validateWriteRequest() method to controller
- [ ] Return type: ?Response (null if valid, error Response if invalid)
- [ ] Call `$this->validateCsrfToken($this->request->request->get('_token'))`
- [ ] Return error response if invalid
- [ ] Return null if valid
- [ ] Add PHPDoc comment explaining purpose

**Acceptance Criteria:**
- validateWriteRequest() method created
- Returns proper data type
- Calls existing validateCsrfToken
- Documented with PHPDoc

---

### Task 4.2: Update upload() to use centralized validation
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Replace CSRF validation in upload()

**Subtasks:**
- [ ] Remove lines 76-79 (duplicate CSRF validation)
- [ ] Add: `if (($error = $this->validateWriteRequest()) !== null) { return $error; }`
- [ ] Keep rest of method unchanged
- [ ] Verify flow is correct

**Acceptance Criteria:**
- CSRF validation block removed
- Uses validateWriteRequest()
- Method flow preserved
- No logic errors

---

### Task 4.3: Update delete() to use centralized validation
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Replace CSRF validation in delete()

**Subtasks:**
- [ ] Remove lines 146-149 (duplicate CSRF validation)
- [ ] Add: `if (($error = $this->validateWriteRequest()) !== null) { return $error; }`
- [ ] Keep rest of method unchanged
- [ ] Verify flow is correct

**Acceptance Criteria:**
- CSRF validation block removed
- Uses validateWriteRequest()
- Method flow preserved
- No logic errors

---

### Task 4.4: Update bulkDelete() to use centralized validation
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Replace CSRF validation in bulkDelete()

**Subtasks:**
- [ ] Remove lines 180-183 (duplicate CSRF validation)
- [ ] Add: `if (($error = $this->validateWriteRequest()) !== null) { return $error; }`
- [ ] Keep rest of method unchanged
- [ ] Verify flow is correct

**Acceptance Criteria:**
- CSRF validation block removed
- Uses validateWriteRequest()
- Method flow preserved
- No logic errors

---

### Task 4.5: Update createFolder() to use centralized validation
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Replace CSRF validation in createFolder()

**Subtasks:**
- [ ] Remove lines 271-274 (duplicate CSRF validation)
- [ ] Add: `if (($error = $this->validateWriteRequest()) !== null) { return $error; }`
- [ ] Keep rest of method unchanged
- [ ] Verify flow is correct

**Acceptance Criteria:**
- CSRF validation block removed
- Uses validateWriteRequest()
- Method flow preserved
- No logic errors

---

## Phase 5: Clean Up Controller

### Task 5.1: Remove getMediaFiles() private method
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Remove unused private helper method

**Subtasks:**
- [ ] Verify method is no longer used
- [ ] Delete lines 330-357 (getMediaFiles)
- [ ] Check for any remaining references
- [ ] Confirm no compilation errors

**Acceptance Criteria:**
- getMediaFiles() deleted
- No remaining references
- No compilation errors

---

### Task 5.2: Remove getFolders() private method
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Remove unused private helper method

**Subtasks:**
- [ ] Verify method is no longer used (moved to FolderService)
- [ ] Delete lines 362-383 (getFolders)
- [ ] Check for any remaining references
- [ ] Confirm no compilation errors

**Acceptance Criteria:**
- getFolders() deleted
- No remaining references
- No compilation errors

---

### Task 5.3: Remove getBreadcrumbs() private method
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Remove unused private helper method

**Subtasks:**
- [ ] Verify method is no longer used (moved to MediaFormatter)
- [ ] Delete lines 388-403 (getBreadcrumbs)
- [ ] Check for any remaining references
- [ ] Confirm no compilation errors

**Acceptance Criteria:**
- getBreadcrumbs() deleted
- No remaining references
- No compilation errors

---

### Task 5.4: Remove getStorageInfo() private method
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Remove unused private helper method

**Subtasks:**
- [ ] Verify method is no longer used (moved to services)
- [ ] Delete lines 408-429 (getStorageInfo)
- [ ] Check for any remaining references
- [ ] Confirm no compilation errors

**Acceptance Criteria:**
- getStorageInfo() deleted
- No remaining references
- No compilation errors

---

### Task 5.5: Remove getFileType() private method
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Remove unused private helper method

**Subtasks:**
- [ ] Verify method is no longer used
- [ ] Delete lines 444-455 (getFileType)
- [ ] Check for any remaining references
- [ ] Confirm no compilation errors

**Acceptance Criteria:**
- getFileType() deleted
- No remaining references
- No compilation errors

---

### Task 5.6: Remove formatBytes() private method
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Remove unused private helper method

**Subtasks:**
- [ ] Verify method is no longer used (moved to MediaFormatter)
- [ ] Delete lines 460-469 (formatBytes)
- [ ] Check for any remaining references
- [ ] Confirm no compilation errors

**Acceptance Criteria:**
- formatBytes() deleted
- No remaining references
- No compilation errors

---

### Task 5.7: Remove FileUploadService dependency
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Remove unused service dependency

**Subtasks:**
- [ ] Remove `use CalacaCMS\Services\FileUploadService;` import
- [ ] Remove `$fileUploadService` parameter from constructor
- [ ] Remove `$this->fileUploadService = $fileUploadService;` assignment
- [ ] Search for any remaining FileUploadService usage
- [ ] Remove all references

**Acceptance Criteria:**
- FileUploadService import removed
- Constructor parameter removed
- All references removed
- No compilation errors

---

### Task 5.8: Update JSDoc comments
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Update class and method documentation

**Subtasks:**
- [ ] Update class PHPDoc comment
- [ ] Document new dependencies (MediaRepository, FolderService, MediaFormatter)
- [ ] Update method PHPDoc for all public methods
- [ ] Remove outdated comments
- [ ] Ensure all parameter types are correct

**Acceptance Criteria:**
- Class documentation updated
- All methods documented
- Outdated comments removed
- Types are accurate

---

### Task 5.9: Verify final line count
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Confirm controller meets size target

**Subtasks:**
- [ ] Count lines in refactored MediaController
- [ ] Verify target of ~150 lines achieved
- [ ] Calculate actual reduction percentage
- [ ] Document final metrics

**Acceptance Criteria:**
- Line count measured
- Target ~150 lines achieved (±20%)
- Reduction calculated
- Metrics documented

---

## Phase 6: Verification & Testing

### Task 6.1: PHPStan static analysis
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Verify no type errors

**Subtasks:**
- [ ] Run: `composer analyse calaca/src/Controllers/Admin/MediaController.php`
- [ ] Check for errors at level 5
- [ ] Fix any type errors found
- [ ] Verify no deprecation warnings

**Acceptance Criteria:**
- Zero PHPStan errors at level 5
- No type issues
- No deprecation warnings

---

### Task 6.2: PHPUnit - Run MediaRepositoryTest
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Verify MediaRepository still works

**Subtasks:**
- [ ] Run: `composer test tests/Unit/MediaRepositoryTest.php`
- [ ] Verify all tests pass
- [ ] Check for any new test failures
- [ ] Review test output for issues

**Acceptance Criteria:**
- All MediaRepository tests pass
- No new test failures
- Test output clean

---

### Task 6.3: Manual testing - Media library view
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Test index() method in browser

**Subtasks:**
- [ ] Start PHP server: `php -S localhost:8000 -t web`
- [ ] Navigate to http://localhost:8000/admin/media
- [ ] Verify page loads without errors
- [ ] Check media files display correctly
- [ ] Verify folders display correctly
- [ ] Check storage info shows
- [ ] Verify breadcrumbs render
- [ ] Test theme toggle works
- [ ] Inspect browser console for errors

**Acceptance Criteria:**
- Page loads successfully
- Media files listed
- Folders displayed
- Storage info shown
- Breadcrumbs render
- No console errors

---

### Task 6.4: Manual testing - File upload
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Test upload() method

**Subtasks:**
- [ ] Upload test image file
- [ ] Upload test document file
- [ ] Verify file appears in list
- [ ] Check MediaRepository record created
- [ ] Test file type validation
- [ ] Test file size limits
- [ ] Upload to subfolder
- [ ] Verify success message
- [ ] Test with invalid CSRF token

**Acceptance Criteria:**
- Uploads work correctly
- MediaRepository records created
- Validation works
- CSRF protection active
- Success messages display

---

### Task 6.5: Manual testing - File deletion
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Test delete() and bulkDelete() methods

**Subtasks:**
- [ ] Delete single file
- [ ] Verify file removed from list
- [ ] Check MediaRepository record deleted
- [ ] Test bulk delete multiple files
- [ ] Verify all files removed
- [ ] Test with invalid CSRF token
- [ ] Test with non-existent file ID
- [ ] Verify error messages
- [ ] Refresh page to confirm deletion

**Acceptance Criteria:**
- Single delete works
- Bulk delete works
- MediaRepository records deleted
- CSRF protection active
- Error handling correct

---

### Task 6.6: Manual testing - File info & download
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Test info() and download() methods

**Subtasks:**
- [ ] Test file info endpoint
- [ ] Verify MediaRepository data returned
- [ ] Test file download
- [ ] Verify correct content type
- [ ] Check filename in download dialog
- [ ] Test with non-existent file ID
- [ ] Verify 404 errors
- [ ] Test download without auth (if applicable)

**Acceptance Criteria:**
- File info works
- Download works
- 404 handling correct
- Content types correct
- Filenames accurate

---

### Task 6.7: Manual testing - Folder creation
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Test createFolder() method

**Subtasks:**
- [ ] Create folder in root
- [ ] Create subfolder
- [ ] Verify folder appears in list
- [ ] Test with empty name
- [ ] Test with invalid characters
- [ ] Test path traversal attempts
- [ ] Test duplicate folder creation
- [ ] Verify error messages
- [ ] Test with invalid CSRF token

**Acceptance Criteria:**
- Folder creation works
- Validations prevent invalid input
- Path traversal blocked
- Error messages clear
- CSRF protection active

---

### Task 6.8: Manual testing - API endpoints
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Test apiMedia() and apiFolders() methods

**Subtasks:**
- [ ] Test /admin/api/media endpoint
- [ ] Verify JSON response format
- [ ] Test with filters (type, search)
- [ ] Test /admin/api/folders endpoint
- [ ] Verify folder list in JSON
- [ ] Test with invalid auth (if applicable)
- [ ] Check response headers
- [ ] Verify CORS if applicable

**Acceptance Criteria:**
- API endpoints return correct JSON
- Filters work correctly
- Folder list accurate
- Auth protected
- Response headers correct

---

### Task 6.9: Regression testing
**Status:** ⬜ Not Started
**Estimated:** 20 minutes
**Description:** Test for unintended side effects

**Subtasks:**
- [ ] Navigate to all admin pages
- [ ] Verify no broken links
- [ ] Test article creation (uses uploads)
- [ ] Test page creation (uses uploads)
- [ ] Check no performance regressions
- [ ] Verify storage consistency
- [ ] Test concurrent operations
- [ ] Check for memory leaks
- [ ] Verify no orphaned files

**Acceptance Criteria:**
- All admin pages functional
- No broken functionality
- Storage consistent
- No performance issues

---

### Task 6.10: Final verification
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Final checks before completion

**Subtasks:**
- [ ] Verify final line count: ~150 lines
- [ ] Count removed lines: ~360 lines (expected)
- [ ] Verify zero placeholder implementations
- [ ] Check all methods use services
- [ ] Verify no duplicate code
- [ ] Run linter: `composer fix --dry-run calaca/src/Controllers/Admin/MediaController.php`
- [ ] Review git diff for unexpected changes
- [ ] Confirm all tasks completed

**Acceptance Criteria:**
- File is ~150 lines
- Zero placeholder implementations
- All code uses services
- Linter passes
- All tasks completed

---

## Phase 7: Documentation & Cleanup

### Task 7.1: Update CHANGELOG (if exists)
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Document refactoring in changelog

**Subtasks:**
- [ ] Check for CHANGELOG.md file
- [ ] Add entry for MediaController refactoring
- [ ] Document breaking changes (none expected)
- [ ] Document improvements
- [ ] Include reference to PRD

**Acceptance Criteria:**
- Changelog updated (if exists)
- Entry properly formatted
- PRD referenced

---

### Task 7.2: Clean up temporary files
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Remove any temporary files created

**Subtasks:**
- [ ] Check for backup files (*.bak, *.backup, *.old)
- [ ] Remove temporary files created during testing
- [ ] Verify git status shows only intended changes
- [ ] Clean up test artifacts

**Acceptance Criteria:**
- No temporary files remaining
- Git status clean
- Only intended changes staged

---

### Task 7.3: Commit changes
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Commit refactoring with clear message

**Subtasks:**
- [ ] Stage all changed files: `git add calaca/src/Controllers/Admin/MediaController.php calaca/src/Services/`
- [ ] Create commit message following conventional commits:
  ```
  refactor(admin): integrate MediaRepository and extract services

  - Replace placeholder getMediaFileById with MediaRepository->find()
  - Create FolderService for folder operations
  - Create MediaFormatter for formatting utilities
  - Centralize CSRF validation to single method
  - Reduce controller from 510 to ~150 lines (70%)
  - Remove all private helper methods (7 methods removed)
  - Eliminate God Controller anti-pattern

  Fixes critical architectural issues:
  - Placeholder implementation
  - Duplicate CSRF validation
  - Mixed concerns in controller
  - Tight coupling to file operations

  Related: docs/prd-refactor-mediacontroller.md

  Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
  ```
- [ ] Verify commit created successfully
- [ ] Check commit hash

**Acceptance Criteria:**
- All changes committed
- Clear commit message
- References PRD
- Co-author included

---

## Summary

**Total Tasks:** 50 tasks across 7 phases
**Total Estimated Time:** 3 - 3.5 hours

**Phases:**
1. ✅ Integrate MediaRepository (1.5 hours)
2. ✅ Create FolderService (1 hour)
3. ✅ Create MediaFormatter (1 hour)
4. ✅ Centralize CSRF Validation (30 minutes)
5. ✅ Clean Up Controller (1 hour)
6. ✅ Verification & Testing (2 hours)
7. ✅ Documentation & Cleanup (20 minutes)

**Completion Criteria:**
- All tasks marked as completed
- Manual testing passed
- MediaRepositoryTest.php still passes
- PHPStan analysis clean
- Zero regressions found
- Documentation updated
- Changes committed

---

## Notes

- **Critical Finding**: MediaRepository already exists (319 lines) with full CRUD but controller ignores it completely
- **Placeholder Issue**: getMediaFileById() returns null - critical functionality gap
- **Service Layer**: FileUploadService exists but controller bypasses it for some operations
- **Testing**: MediaRepositoryTest.php exists (363 lines) - leverage existing tests

**Dependencies Created:**
- FolderService (new)
- MediaFormatter (new)

**Dependencies Used:**
- MediaRepository (existing, now integrated)
- FileUploadService (removed, replaced by MediaRepository)

**Next Steps:**
- After completion, consider adding integration tests for MediaController
- Consider adding performance tests for large file operations
- Consider adding end-to-end tests for media workflow

---

## Risk Assessment

**Low Risk Areas:**
- Creating new services (well-defined scope)
- Extracting formatting logic (pure function, no side effects)

**Medium Risk Areas:**
- Integrating MediaRepository (changes data flow)
- Folder operations (filesystem operations)

**High Risk Areas:**
- File upload/deletion (data-critical operations)
- CSRF validation changes (security-sensitive)

**Mitigation in Place:**
- Existing MediaRepositoryTest.php validates data layer
- Manual testing plan for all operations
- Rollback plan ready if critical issues arise
- Incremental approach (complete one phase before next)
