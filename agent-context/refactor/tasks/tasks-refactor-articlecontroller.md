# Task List: ArticleController Refactoring

**File:** `calaca/src/Controllers/Admin/ArticleController.php`
**PRD:** `docs/prd-refactor-articlecontroller.md`
**Total Tasks:** 25 (8 phases)
**Pattern:** Same as MediaController (Task #2)

## Phase 1: Create Utility Services - 3 tasks

- [ ] 1.1 Create `SlugGenerator` utility class with `generate()` method
- [ ] 1.2 Create `TagParser` utility class with `parse()` and `toString()` methods
- [ ] 1.3 Unit test SlugGenerator and TagParser

**Validation:** All tests pass for utilities

## Phase 2: Create ArticleDataPreparer - 4 tasks

- [ ] 2.1 Create `ArticleDataPreparer` service class
- [ ] 2.2 Implement `prepareForCreate()` with full data preparation
- [ ] 2.3 Implement `prepareForUpdate()` with selective data preparation
- [ ] 2.4 Unit test data preparation with various inputs

**Validation:** Article data correctly prepared for create/update

## Phase 3: Integrate FileUploadService - 2 tasks

- [ ] 3.1 Inject `FileUploadService` into ArticleController constructor
- [ ] 3.2 Update `store()` and `update()` to use FileUploadService
- [ ] 3.3 Remove `handleImageUpload()` method

**Validation:** File upload works via FileUploadService

## Phase 4: Create CategoryRepository - 3 tasks

- [ ] 4.1 Create `CategoryRepositoryInterface`
- [ ] 4.2 Create `ArrayCategoryRepository` with hardcoded data
- [ ] 4.3 Inject into ArticleController and update `getCategories()`

**Validation:** Categories retrieved from repository

## Phase 5: Extract CSRF Validation - 2 tasks

- [ ] 5.1 Verify `validateWriteRequest()` exists in BaseAdminController
- [ ] 5.2 Update `store()` and `update()` to use `validateWriteRequest()`
- [ ] 5.3 Remove duplicate CSRF validation blocks

**Validation:** CSRF validation centralized

## Phase 6: Clean Up Controller - 4 tasks

- [ ] 6.1 Inject `ArticleDataPreparer` into ArticleController
- [ ] 6.2 Update `store()` to use ArticleDataPreparer->prepareForCreate()
- [ ] 6.3 Update `update()` to use ArticleDataPreparer->prepareForUpdate()
- [ ] 6.4 Remove helper methods: parseTags(), generateSlug(), handleImageUpload()

**Validation:** Controller is thin HTTP layer

## Phase 7: API Methods - 2 tasks

- [ ] 7.1 Verify `apiArticles()` still works
- [ ] 7.2 Verify `apiCategories()` still works

**Validation:** API endpoints functional

## Phase 8: Documentation & Cleanup - 3 tasks

- [ ] 8.1 Document refactoring in docs/refactor-articlecontroller.md
- [ ] 8.2 Update class docblocks
- [ ] 8.3 Remove obsolete comments

**Validation:** Documentation complete

## Final Tasks - 5 tasks

- [ ] 9.1 Run PHP syntax validation
- [ ] 9.2 Test full article creation flow
- [ ] 9.3 Test article update flow
- [ ] 9.4 Test article deletion
- [ ] 9.5 Update refactor-candidates-status.md

## Success Metrics

- ArticleController: 368 → ~200 lines (46% reduction)
- New services: 4 (SlugGenerator, TagParser, ArticleDataPreparer, CategoryRepository)
- Reused services: 1 (FileUploadService from Task #2)
- Helper methods removed: 4
- CSRF validation: 2× → 1× centralized
- Data preparation: 29 lines → service call

## Notes

- Follows exact same pattern as MediaController refactoring
- Reuses FileUploadService (already tested)
- Utilities (SlugGenerator, TagParser) are reusable for other controllers
- Each phase should be committed separately
