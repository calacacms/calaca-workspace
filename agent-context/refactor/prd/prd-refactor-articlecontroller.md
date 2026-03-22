# PRD: ArticleController Refactoring

**File:** `calaca/src/Controllers/Admin/ArticleController.php`
**Current Size:** 368 lines
**Target Size:** ~200 lines
**Priority:** MEDIUM (Score 2.6)
**Pattern:** Same as MediaController refactoring (Task #2)

## Problem Statement

ArticleController has the same code duplication issues as MediaController:

### Current Issues

1. **CSRF Validation Duplication**
   ```php
   // Line 123 (store method)
   if (!$this->validateCsrfToken($this->request->request->get('_token'))) { ... }

   // Line 189 (update method)
   if (!$this->validateCsrfToken($this->request->request->get('_token'))) { ... }
   ```
   Same validation logic duplicated in 2 methods.

2. **Data Preparation Duplication**
   - Store method (lines 142-157): 15 lines of data preparation
   - Update method (lines 208-221): 14 lines of nearly identical logic
   - Both do: sanitization, slug generation, tag parsing, date handling

3. **Image Upload Duplication**
   ```php
   // Lines 160-165 (store)
   if (isset($_FILES['featured_image']) && $_FILES['featured_image']['error'] === UPLOAD_ERR_OK) {
       $uploadedFile = $this->handleImageUpload($_FILES['featured_image']);
       if ($uploadedFile) { $articleData['image'] = $uploadedFile; }
   }

   // Lines 229-234 (update)
   if (isset($_FILES['featured_image']) && $_FILES['featured_image']['error'] === UPLOAD_ERR_OK) {
       $uploadedFile = $this->handleImageUpload($_FILES['featured_image']);
       if ($uploadedFile) { $updateData['image'] = $uploadedFile; }
   }
   ```
   Same upload logic duplicated.

4. **Helper Methods in Controller**
   - `getCategories()` - Hardcoded data (lines 268-276)
   - `parseTags()` - Tag parsing logic (lines 281-285)
   - `generateSlug()` - Slug generation (lines 290-299)
   - `handleImageUpload()` - Image upload (lines 304-333)

5. **No Reuse of Existing Services**
   - MediaController created `FileUploadService` - not reused here
   - `handleImageUpload()` duplicates functionality

## Proposed Architecture

### Service Layer Structure

```
ArticleController (facade - ~200 lines)
├── ArticleDataPreparer (service)
│   ├── prepareForCreate()
│   ├── prepareForUpdate()
│   ├── sanitizeContent()
│   ├── parseTags()
│   └── generateSlug()
├── SlugGenerator (utility - reusable)
│   └── generate()
├── TagParser (utility - reusable)
│   └── parse()
├── FileUploadService (reuse from MediaController)
│   └── upload()
└── CategoryRepository (new)
    ├── findAll()
    └── find()
```

### New/Reused Classes

1. **ArticleDataPreparer** (new)
   ```php
   class ArticleDataPreparer
   {
       public function __construct(
           private HtmlSanitizer $sanitizer,
           private SlugGenerator $slugGenerator,
           private TagParser $tagParser
       ) {}

       public function prepareForCreate(array $data, ?array $uploadedFile): array;
       public function prepareForUpdate(array $data, ?array $uploadedFile): array;
   }
   ```

2. **SlugGenerator** (new, reusable)
   ```php
   class SlugGenerator
   {
       public function generate(string $text, string $delimiter = '-'): string;
   }
   ```

3. **TagParser** (new, reusable)
   ```php
   class TagParser
   {
       public function parse(string $tagsString): array;
       public function toString(array $tags): string;
   }
   ```

4. **FileUploadService** (reuse from Task #2)
   - Already exists in `calaca/src/Services/FileUploadService.php`
   - Handles validation, file type checking, size limits
   - Returns upload path or error

5. **CategoryRepository** (new)
   ```php
   interface CategoryRepositoryInterface
   {
       public function findAll(): array;
       public function find(int $id): ?array;
   }

   class ArrayCategoryRepository implements CategoryRepositoryInterface
   {
       // Temporary implementation with hardcoded data
   }
   ```

## Implementation Phases

### Phase 1: Create Utility Services (3 tasks)
- [ ] Create `SlugGenerator` utility class
- [ ] Create `TagParser` utility class
- [ ] Unit test both utilities

**Tests:** Test slug generation with various inputs, test tag parsing

### Phase 2: Create ArticleDataPreparer (4 tasks)
- [ ] Create `ArticleDataPreparer` service class
- [ ] Implement `prepareForCreate()` method
- [ ] Implement `prepareForUpdate()` method
- [ ] Unit test data preparation

**Tests:** Test with valid/invalid data, test file attachment

### Phase 3: Integrate FileUploadService (2 tasks)
- [ ] Inject `FileUploadService` into ArticleController
- [ ] Update `store()` and `update()` to use service
- [ ] Remove `handleImageUpload()` method

**Tests:** Test file upload with valid/invalid files

### Phase 4: Create CategoryRepository (3 tasks)
- [ ] Create `CategoryRepositoryInterface`
- [ ] Create `ArrayCategoryRepository` (temporary)
- [ ] Inject into ArticleController
- [ ] Update `getCategories()` to use repository

**Tests:** Test category retrieval

### Phase 5: Extract CSRF Validation (2 tasks)
- [ ] Add `validateWriteRequest()` to BaseAdminController
- [ ] Update `store()` and `update()` to use it
- [ ] Remove duplicate CSRF validation

**Tests:** Test CSRF validation passes/fails

### Phase 6: Clean Up Controller (3 tasks)
- [ ] Inject `ArticleDataPreparer` into controller
- [ ] Update `store()` to use ArticleDataPreparer
- [ ] Update `update()` to use ArticleDataPreparer
- [ ] Remove all helper methods (parseTags, generateSlug, handleImageUpload)

**Tests:** Full integration test of create/update flow

### Phase 7: API Methods (2 tasks)
- [ ] Keep API methods as-is (already clean)
- [ ] Ensure they use new services if needed

### Phase 8: Documentation & Cleanup (3 tasks)
- [ ] Document new architecture
- [ ] Update class docblocks
- [ ] Remove obsolete comments

## Success Criteria

- [ ] ArticleController reduced to ~200 lines
- [ ] 4 new service classes created
- [ ] FileUploadService reused from MediaController
- [ ] CSRF validation centralized (removed duplication)
- [ ] Data preparation logic extracted
- [ ] Helper methods removed from controller
- [ ] All functionality preserved
- [ ] No regressions

## Migration Notes

**No Breaking Changes:**
- Public controller methods unchanged
- Routes unchanged
- API unchanged
- Behavior unchanged

**Service Reuse:**
- FileUploadService already exists from MediaController refactoring
- SlugGenerator and TagParser can be used by other controllers (Page, etc.)

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Image upload regression | Use tested FileUploadService from Task #2 |
| Slug generation changes | Preserve exact logic from original |
| Tag parsing edge cases | Comprehensive unit tests |
| Category migration | Interface allows easy swap to real repository later |

## Estimated Complexity

- **New Classes:** 4 services
- **Reused Classes:** 1 (FileUploadService)
- **Lines of Code:** ~300 total (vs 368 current)
- **Development Time:** 2-3 hours
- **Risk Level:** Low (same pattern as MediaController)

## References

- Original ArticleController: 368 lines
- MediaController refactoring: Task #2 (completed)
- FileUploadService: Already exists from Task #2
- Pattern: Same as MediaController refactoring

## Comparison with MediaController

| Aspect | MediaController | ArticleController |
|--------|----------------|-------------------|
| Original Lines | 510 → 417 | 368 → ~200 |
| Services Created | FolderService, MediaFormatter | ArticleDataPreparer, SlugGenerator, TagParser |
| Services Reused | FileUploadService (new) | FileUploadService (existing) |
| Complexity | HIGH | MEDIUM |
| Pattern | Established | Follow established pattern |
