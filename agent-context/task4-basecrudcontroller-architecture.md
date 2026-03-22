# Task #4: BaseCrudController Architecture

**Status**: PENDING - Implementation not started
**Priority**: HIGH (Final refactor task)
**Estimated Time**: 20-24 hours (5 phases)
**Architect**: team-lead
**Implementer**: TBD
**Reviewer**: reviewer

---

## Executive Summary

Create a `BaseCrudController` abstract class that extracts common CRUD operations from `ArticleController` and `PageController`, eliminating ~450 lines of duplication (80% overlap) while maintaining flexibility for content-type-specific requirements.

**Pattern**: Template Method Pattern
**Reduction Target**: ArticleController 314→150 lines (52%), PageController 250→120 lines (52%)

---

## Current Architecture Analysis

### Problem: Massive Code Duplication

**ArticleController.php** (314 lines):
- `index()`: Pagination and filtering (lines 42-71)
- `create()`: Show form (lines 76-107)
- `edit()`: Show edit form (lines 112-138)
- `store()`: Handle form submission (lines 143-180)
- `update()`: Handle update (lines 185-230)
- `delete()`: Delete item (lines 235-250)

**PageController.php** (250 lines):
- Identical CRUD methods with 80% same logic
- Differences: Field names (article vs page), template paths, specific services

**Duplication breakdown**:
```
index()      95% identical (pagination, filtering, CSRF token)
create()     90% identical (form structure, CSRF, default values)
edit()       85% identical (entity lookup, 404 handling, form display)
store()      80% identical (CSRF validate, form validate, file upload, create, redirect)
update()     75% identical (entity lookup, CSRF, validate, update, publish action)
delete()     90% identical (entity lookup, delete, redirect)
```

**Total duplicated lines**: ~450 lines across both controllers

### Violated Principles

1. **DRY (Don't Repeat Yourself)**: Same CRUD logic repeated twice
2. **Single Responsibility Principle**: Controllers contain both HTTP orchestration AND business logic patterns
3. **Open/Closed Principle**: Adding new content types requires copying entire CRUD pattern
4. **Maintainability**: Bug fixes must be applied in 2+ places

---

## Target Architecture

### Template Method Pattern

BaseCrudController defines the CRUD workflow skeleton, with abstract methods for configuration and hook methods for customization.

```php
abstract class BaseCrudController extends BaseAdminController
{
    // Configuration: "What am I managing?"
    abstract protected function getRepository();
    abstract protected function getContentType(): string;
    abstract protected function getRoutePrefix(): string;
    abstract protected function getSingularName(): string;
    abstract protected function getPluralName(): string;
    abstract protected function getTemplateNamespace(): string;

    // Hooks: "How do I customize behavior?"
    protected function getIndexData(): array { return []; }
    protected function getFormData(): array { return []; }
    protected function getValidationRules(string $context): array { return []; }
    protected function prepareStoreData(array $data): array { return $data; }
    protected function prepareUpdateData($entity, array $data): array { return $data; }
    protected function beforeStore(array $data): ?Response { return null; }
    protected function afterStore($entity, array $data): void {}
    protected function beforeUpdate($entity, array $data): ?Response { return null; }
    protected function afterUpdate($entity, array $data): void {}
    protected function beforeDelete($entity): ?Response { return null; }
    protected function afterDelete($entity): void {}

    // Final CRUD methods: "Consistent workflow enforcement"
    final public function index(): Response;
    final public function create(): Response;
    final public function edit(int $id): Response;
    final public function store(): Response;
    final public function update(int $id): Response;
    final public function delete(int $id): Response;
}
```

### Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    HTTP REQUEST                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              FINAL CRUD METHOD (BaseCrudController)         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 1. Authentication check                             │    │
│  │ 2. CSRF validation (for write methods)              │    │
│  │ 3. Entity lookup (for edit/update/delete)           │    │
│  │ 4. Call hook: beforeXxx()                          │    │
│  │ 5. Get validation rules via getValidationRules()    │    │
│  │ 6. Validate request data                            │    │
│  │ 7. Call hook: prepareXxxData()                     │    │
│  │ 8. Call repository method                          │    │
│  │ 9. Call hook: afterXxx()                           │    │
│  │ 10. Set flash message + redirect                    │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│          CONCRETE CONTROLLER (Article/Page)                 │
│  - Override hooks for content-type-specific logic          │
│  - Example: beforeStore() for homepage flag (pages)        │
│  - Example: prepareStoreData() for file upload handling    │
└─────────────────────────────────────────────────────────────┘
```

### Inheritance Hierarchy

```
BaseAdminController (existing)
    │
    ├── MediaController (already refactored in Task #3)
    ├── UserController (different pattern - not in scope)
    │
    └── BaseCrudController (NEW)
            │
            ├── ArticleController (refactored)
            └── PageController (refactored)
```

---

## Implementation Phases

### Phase 1: Create BaseCrudController (6-8 hours)

**File**: `calaca/src/Controllers/Admin/BaseCrudController.php`

**Tasks**:
1. Extend BaseAdminController
2. Declare 6 abstract configuration methods
3. Implement 11 hook methods (default: no-op or return empty array)
4. Implement 6 final CRUD methods
5. Add comprehensive PHPDoc comments
6. Verify syntax: `php -l src/Controllers/Admin/BaseCrudController.php`

**Validation**: File compiles without errors, all abstract methods declared

---

### Phase 2: Refactor ArticleController (4-5 hours)

**File**: `calaca/src/Controllers/Admin/ArticleController.php`

**Tasks**:
1. Extend BaseCrudController instead of BaseAdminController
2. Implement 6 abstract configuration methods:
   ```php
   protected function getRepository() { return $this->articleRepository; }
   protected function getContentType(): string { return 'article'; }
   protected function getRoutePrefix(): string { return '/admin/articles'; }
   protected function getSingularName(): string { return 'article'; }
   protected function getPluralName(): string { return 'articles'; }
   protected function getTemplateNamespace(): string { return '@admin/pages/articles'; }
   ```
3. Override getIndexData(): Add categories list
4. Override getFormData(): Return default article structure
5. Override getValidationRules(): Return article-specific rules
6. Override prepareStoreData(): Handle featured image upload via ArticleDataPreparer
7. Override prepareUpdateData(): Handle featured image update via ArticleDataPreparer
8. Remove all CRUD methods (now inherited from BaseCrudController)
9. Keep API methods (apiArticles, apiCategories) - these are not CRUD

**Expected Result**: 314 → ~150 lines (52% reduction)

---

### Phase 3: Refactor PageController (3-4 hours)

**File**: `calaca/src/Controllers/Admin/PageController.php`

**Tasks**:
1. Extend BaseCrudController instead of BaseAdminController
2. Implement 6 abstract configuration methods (same pattern as ArticleController)
3. Override getIndexData(): Add page_types array
4. Override getFormData(): Add template/parent/page type selections
5. Override getValidationRules(): Return page-specific rules
6. Override beforeStore(): Call pageService->unsetHomepage() if homepage flag set
7. Override beforeUpdate(): Call pageService->unsetHomepage() if homepage flag set
8. Override prepareStoreData(): Handle OG image upload via PageDataPreparer
9. Override prepareUpdateData(): Handle OG image update via PageDataPreparer
10. Remove all CRUD methods (now inherited from BaseCrudController)

**Expected Result**: 250 → ~120 lines (52% reduction)

---

### Phase 4: Update ContainerProvider (1-2 hours)

**File**: `calaca/src/Core/Providers/ContainerProvider.php`

**Tasks**:
1. No DI container changes needed (BaseCrudController is abstract)
2. Verify ArticleController and PageController registrations remain unchanged
3. Ensure constructor dependencies still resolve correctly

**Validation**: All existing tests pass (281 tests)

---

### Phase 5: Testing & Verification (3-4 hours)

**Tasks**:
1. Run full test suite: `composer test`
2. Manual testing in browser:
   - Create article: Verify form displays, submission works
   - Edit article: Verify pre-populated form, update works
   - Delete article: Verify deletion works
   - Publish article: Verify publish action works
   - Repeat for pages
3. Check for regressions:
   - File uploads (featured image, OG image)
   - Homepage flag behavior (pages only)
   - Category selection (articles only)
   - Template/parent selection (pages only)
4. Verify all flash messages appear correctly
5. Verify CSRF protection still works
6. Verify authentication/authorization checks still work

**Success Criteria**:
- All 281 existing tests pass
- No behavioral changes detected in manual testing
- Code reduced by ~450 lines total
- Both controllers follow same pattern

---

## Risk Assessment

### High Risk Areas

1. **File upload handling** (ArticleController: featured image, PageController: OG image)
   - **Mitigation**: Keep exact same upload logic in prepareStoreData/prepareUpdateData hooks
   - **Test**: Create article/page with image, edit with new image, edit without image

2. **Homepage flag logic** (PageController only)
   - **Mitigation**: Move exact same logic to beforeStore/beforeUpdate hooks
   - **Test**: Create page as homepage, verify previous homepage unset

3. **Publish action** (update method with action=publish)
   - **Mitigation**: Keep action parameter handling in BaseCrudController.update()
   - **Test**: Update article with action=update, verify "Article updated successfully"
   - **Test**: Update article with action=publish, verify "Article published successfully"

### Medium Risk Areas

1. **Template namespace differences**
   - Articles: `@admin/pages/articles/`
   - Pages: `@admin/pages/pages/`
   - **Mitigation**: Ensure getTemplateNamespace() returns correct paths

2. **Repository method compatibility**
   - Both ArticleRepository and PageRepository inherit from same interface
   - **Mitigation**: Verify both have same method signatures (find, create, update, delete, paginate)

### Low Risk Areas

1. **CSRF token generation** (handled by BaseAdminController)
2. **Authentication checks** (handled by BaseAdminController)
3. **Flash messages** (handled by BaseAdminController)
4. **Redirects** (handled by BaseCrudController)

---

## Rollback Strategy

If issues are discovered during testing:

1. **Immediate rollback**: Revert Phase 3 and Phase 2 changes (restore original ArticleController and PageController)
2. **Investigation**: Determine if issue is in BaseCrudController implementation or controller refactoring
3. **Fix BaseCrudController**: Address the root cause in the base class
4. **Re-apply**: Re-run Phase 2 and Phase 3

**Git approach**:
```bash
# Before starting Phase 2
git checkout -b feature/task4-basecrudcontroller

# If rollback needed
git revert <commit-range>

# After verification
git merge feature/task4-basecrudcontroller into develop
```

---

## Success Metrics

### Quantitative Metrics

- **Code Reduction**: ArticleController 314→150 lines (52%), PageController 250→120 lines (52%)
- **Duplication Eliminated**: ~450 lines of duplicated CRUD logic
- **File Count**: +1 file (BaseCrudController.php)
- **Test Coverage**: Maintain 281 passing tests

### Qualitative Metrics

- **Maintainability**: CRUD changes now only need to be made in one place
- **Extensibility**: New content types can extend BaseCrudController with minimal code
- **Consistency**: All CRUD operations work identically
- **Clarity**: Controller intent is clearer (configuration vs. customization)

---

## Post-Implementation Tasks

1. **Documentation**: Update AGENTS.md with BaseCrudController pattern
2. **Examples**: Add example of extending BaseCrudController for new content type
3. **Consideration**: Evaluate if UserController could also benefit (Phase 6, optional)

---

## Architect's Notes

### Why Template Method Pattern?

- **Predictable workflow**: All CRUD operations follow same sequence (auth → validate → prepare → save → redirect)
- **Enforced consistency**: Final methods prevent accidental deviations from the pattern
- **Flexibility**: Hooks allow customization without duplicating the entire workflow
- **Testability**: Each hook can be tested independently

### Why Not Other Patterns?

- **Strategy Pattern**: Would require separate strategy classes, over-engineering for this use case
- **Composition over Inheritance**: Would break existing controller structure, more invasive changes
- **Traits**: Would allow partial implementation but doesn't enforce workflow consistency

### Key Design Decisions

1. **Final CRUD methods**: Prevent subclasses from breaking the workflow
2. **Optional hooks**: Default to no-op, only override when needed
3. **Abstract configuration**: Compile-time enforcement of required setup
4. **Service injection in constructors**: Keep existing DI pattern, no changes to ContainerProvider

---

## Appendix: Complete Hook Signatures

```php
/**
 * Get additional data for index page
 * @return array Additional template data
 */
protected function getIndexData(): array;

/**
 * Get default form data structure
 * @return array Default entity structure for create form
 */
protected function getFormData(): array;

/**
 * Get validation rules
 * @param string $context 'create' or 'update'
 * @return array Validation rules array
 */
protected function getValidationRules(string $context): array;

/**
 * Prepare data before store
 * @param array $data Raw request data
 * @return array Prepared data for repository->create()
 */
protected function prepareStoreData(array $data): array;

/**
 * Prepare data before update
 * @param mixed $entity Existing entity
 * @param array $data Raw request data
 * @return array Prepared data for repository->update()
 */
protected function prepareUpdateData($entity, array $data): array;

/**
 * Before store hook (pre-validation)
 * @param array $data Raw request data
 * @return Response|null Return response to short-circuit, null to continue
 */
protected function beforeStore(array $data): ?Response;

/**
 * After store hook
 * @param mixed $entity Created entity
 * @param array $data Raw request data
 */
protected function afterStore($entity, array $data): void;

/**
 * Before update hook (pre-validation)
 * @param mixed $entity Existing entity
 * @param array $data Raw request data
 * @return Response|null Return response to short-circuit, null to continue
 */
protected function beforeUpdate($entity, array $data): ?Response;

/**
 * After update hook
 * @param mixed $entity Updated entity
 * @param array $data Raw request data
 */
protected function afterUpdate($entity, array $data): void;

/**
 * Before delete hook
 * @param mixed $entity Entity to delete
 * @return Response|null Return response to short-circuit, null to continue
 */
protected function beforeDelete($entity): ?Response;

/**
 * After delete hook
 * @param mixed $entity Deleted entity
 */
protected function afterDelete($entity): void;
```

---

**Document Version**: 1.0
**Last Updated**: 2026-03-20
**Next Review**: After Phase 1 completion
