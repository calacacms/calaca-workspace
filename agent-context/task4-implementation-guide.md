# Task #4: BaseCrudController - Implementation Guide

**Quick Reference for Implementer**

---

## Phase 1: Create BaseCrudController

### Step 1: Create the file

```bash
touch calaca/src/Controllers/Admin/BaseCrudController.php
```

### Step 2: BaseCrudController structure

```php
<?php

declare(strict_types=1);

namespace CalacaCMS\Controllers\Admin;

use Symfony\Component\HttpFoundation\Response;

/**
 * Base CRUD Controller using Template Method Pattern
 *
 * Provides standard CRUD operations for admin controllers.
 * Subclasses define WHAT they manage via abstract methods.
 * Subclasses customize behavior via optional hook methods.
 */
abstract class BaseCrudController extends BaseAdminController
{
    // ============================================================
    // CONFIGURATION - Subclasses MUST implement these
    // ============================================================

    /**
     * Get the repository instance for this entity type
     */
    abstract protected function getRepository(): object;

    /**
     * Get the content type identifier
     * @return string e.g., 'article', 'page'
     */
    abstract protected function getContentType(): string;

    /**
     * Get the base URL for routes
     * @return string e.g., '/admin/articles', '/admin/pages'
     */
    abstract protected function getRoutePrefix(): string;

    /**
     * Get the singular entity name
     * @return string e.g., 'article', 'page'
     */
    abstract protected function getSingularName(): string;

    /**
     * Get the plural entity name
     * @return string e.g., 'articles', 'pages'
     */
    abstract protected function getPluralName(): string;

    /**
     * Get the template namespace
     * @return string e.g., '@admin/pages/articles', '@admin/pages/pages'
     */
    abstract protected function getTemplateNamespace(): string;

    // ============================================================
    // HOOKS - Subclasses MAY override these
    // ============================================================

    /**
     * Get additional data for index page
     * @return array Additional template variables
     */
    protected function getIndexData(): array
    {
        return [];
    }

    /**
     * Get default form data structure for create form
     * @return array Default entity structure
     */
    protected function getFormData(): array
    {
        return [];
    }

    /**
     * Get validation rules
     * @param string $context 'create' or 'update'
     * @return array Validation rules
     */
    protected function getValidationRules(string $context): array
    {
        return [];
    }

    /**
     * Prepare data before store (post-validation)
     * @param array $data Raw request data
     * @return array Prepared data
     */
    protected function prepareStoreData(array $data): array
    {
        return $data;
    }

    /**
     * Prepare data before update (post-validation)
     * @param mixed $entity Existing entity
     * @param array $data Raw request data
     * @return array Prepared data
     */
    protected function prepareUpdateData($entity, array $data): array
    {
        return $data;
    }

    /**
     * Before store hook (pre-validation)
     * @param array $data Raw request data
     * @return Response|null Return response to short-circuit
     */
    protected function beforeStore(array $data): ?Response
    {
        return null;
    }

    /**
     * After store hook
     * @param mixed $entity Created entity
     * @param array $data Raw request data
     */
    protected function afterStore($entity, array $data): void
    {
        // No-op by default
    }

    /**
     * Before update hook (pre-validation)
     * @param mixed $entity Existing entity
     * @param array $data Raw request data
     * @return Response|null Return response to short-circuit
     */
    protected function beforeUpdate($entity, array $data): ?Response
    {
        return null;
    }

    /**
     * After update hook
     * @param mixed $entity Updated entity
     * @param array $data Raw request data
     */
    protected function afterUpdate($entity, array $data): void
    {
        // No-op by default
    }

    /**
     * Before delete hook
     * @param mixed $entity Entity to delete
     * @return Response|null Return response to short-circuit
     */
    protected function beforeDelete($entity): ?Response
    {
        return null;
    }

    /**
     * After delete hook
     * @param mixed $entity Deleted entity
     */
    protected function afterDelete($entity): void
    {
        // No-op by default
    }

    // ============================================================
    // CRUD METHODS - Final, enforced workflow
    // ============================================================

    /**
     * List all entities
     */
    final public function index(): Response
    {
        $this->requireAuth();

        $page = (int) $this->request->query->get('page', 1);
        $perPage = 20;
        $status = $this->request->query->get('status', '');
        $search = $this->request->query->get('search', '');

        $repository = $this->getRepository();

        if ($search) {
            $entities = $repository->search($search);
        } elseif ($status) {
            // Try status-specific method if exists
            if (method_exists($repository, 'findByStatus')) {
                $entities = $repository->findByStatus($status);
            } elseif ($status === 'published' && method_exists($repository, 'findPublished')) {
                $entities = $repository->findPublished();
            } else {
                $entities = $repository->findAll();
            }
        } else {
            $entities = $repository->paginate($page, $perPage);
        }

        $data = array_merge([
            $this->getPluralName() => $entities,
            'current_page' => $page,
            'total_pages' => $search || $status ? 1 : ceil($repository->countAll() / $perPage),
            'status' => $status,
            'search' => $search,
            'csrf_token' => $this->generateCsrfToken(),
        ], $this->getIndexData());

        return $this->html($this->render($this->getTemplateNamespace() . '/index.twig', $data));
    }

    /**
     * Show create form
     */
    final public function create(): Response
    {
        $this->requireAuth();

        $contentType = $this->contentTypeManager->getContentType($this->getContentType());
        $formHtml = '';
        if ($contentType) {
            $builder = new \CalacaCMS\Services\FormBuilder($contentType);
            $formHtml = $builder->build();
        }

        $data = array_merge([
            $this->getSingularName() => $this->getFormData(),
            'csrf_token' => $this->generateCsrfToken(),
            'form_html' => $formHtml,
        ], $this->getIndexData());

        return $this->html($this->render($this->getTemplateNamespace() . '/edit.twig', $data));
    }

    /**
     * Show edit form
     */
    final public function edit(int $id): Response
    {
        $this->requireAuth();

        $repository = $this->getRepository();
        $entity = $repository->find($id);

        if (!$entity) {
            $this->setFlash('error', ucfirst($this->getSingularName()) . ' not found.');
            return $this->redirect($this->getRoutePrefix());
        }

        $contentType = $this->contentTypeManager->getContentType($this->getContentType());
        $formHtml = '';
        if ($contentType) {
            $builder = new \CalacaCMS\Services\FormBuilder($contentType);
            $formHtml = $builder->build(is_array($entity) ? $entity : []);
        }

        $data = array_merge([
            $this->getSingularName() => $entity,
            'csrf_token' => $this->generateCsrfToken(),
            'form_html' => $formHtml,
        ], $this->getIndexData());

        return $this->html($this->render($this->getTemplateNamespace() . '/edit.twig', $data));
    }

    /**
     * Store new entity
     */
    final public function store(): Response
    {
        $this->requireAuth();

        if (!$this->validateWriteRequest($this->request->request->get('_token'))) {
            return $this->redirectWithMessage(
                $this->getRoutePrefix() . '/create',
                'error',
                'Invalid security token. Please try again.'
            );
        }

        $data = $this->request->request->all();

        // Before store hook (pre-validation)
        $hookResponse = $this->beforeStore($data);
        if ($hookResponse !== null) {
            return $hookResponse;
        }

        // Validate
        $errors = $this->validate($this->getValidationRules('create'), $data);
        if (!empty($errors)) {
            $this->setFlash('error', 'Please fix the validation errors.');
            return $this->redirect($this->getRoutePrefix() . '/create');
        }

        // Prepare data
        $preparedData = $this->prepareStoreData($data);

        // Create
        $repository = $this->getRepository();
        $entity = $repository->create($preparedData);

        // After store hook
        $this->afterStore($entity, $data);

        $this->setFlash('success', ucfirst($this->getSingularName()) . ' created successfully.');
        return $this->redirect($this->getRoutePrefix());
    }

    /**
     * Update existing entity
     */
    final public function update(int $id): Response
    {
        $this->requireAuth();

        $repository = $this->getRepository();
        $entity = $repository->find($id);

        if (!$entity) {
            $this->setFlash('error', ucfirst($this->getSingularName()) . ' not found.');
            return $this->redirect($this->getRoutePrefix());
        }

        if (!$this->validateWriteRequest($this->request->request->get('_token'))) {
            return $this->redirectWithMessage(
                $this->getRoutePrefix() . "/{$id}/edit",
                'error',
                'Invalid security token. Please try again.'
            );
        }

        $data = $this->request->request->all();
        $action = $data['action'] ?? 'update';

        // Before update hook (pre-validation)
        $hookResponse = $this->beforeUpdate($entity, $data);
        if ($hookResponse !== null) {
            return $hookResponse;
        }

        // Validate
        $errors = $this->validate($this->getValidationRules('update'), $data);
        if (!empty($errors)) {
            $this->setFlash('error', 'Please fix the validation errors.');
            return $this->redirect($this->getRoutePrefix() . "/{$id}/edit");
        }

        // Prepare data
        $preparedData = $this->prepareUpdateData($entity, $data);

        // Update
        $repository->update($id, $preparedData);

        // After update hook
        $this->afterUpdate($entity, $data);

        $actionPast = $action === 'publish' ? 'published' : 'updated';
        $this->setFlash('success', ucfirst($this->getSingularName()) . " {$actionPast} successfully.");
        return $this->redirect($this->getRoutePrefix());
    }

    /**
     * Delete entity
     */
    final public function delete(int $id): Response
    {
        $this->requireAuth();

        $repository = $this->getRepository();
        $entity = $repository->find($id);

        if (!$entity) {
            $this->setFlash('error', ucfirst($this->getSingularName()) . ' not found.');
            return $this->redirect($this->getRoutePrefix());
        }

        // Before delete hook
        $hookResponse = $this->beforeDelete($entity);
        if ($hookResponse !== null) {
            return $hookResponse;
        }

        // Delete
        $repository->delete($id);

        // After delete hook
        $this->afterDelete($entity);

        $this->setFlash('success', ucfirst($this->getSingularName()) . ' deleted successfully.');
        return $this->redirect($this->getRoutePrefix());
    }
}
```

### Step 3: Verify syntax

```bash
php -l calaca/src/Controllers/Admin/BaseCrudController.php
```

Expected output: `No syntax errors detected in BaseCrudController.php`

---

## Phase 2: Refactor ArticleController

### Step 1: Change extends clause

```php
// BEFORE:
class ArticleController extends BaseAdminController

// AFTER:
class ArticleController extends BaseCrudController
```

### Step 2: Implement abstract methods

```php
protected function getRepository(): object
{
    return $this->articleRepository;
}

protected function getContentType(): string
{
    return 'article';
}

protected function getRoutePrefix(): string
{
    return '/admin/articles';
}

protected function getSingularName(): string
{
    return 'article';
}

protected function getPluralName(): string
{
    return 'articles';
}

protected function getTemplateNamespace(): string
{
    return '@admin/pages/articles';
}
```

### Step 3: Override hooks

```php
protected function getIndexData(): array
{
    return [
        'categories' => $this->categoryRepository->findAll(),
    ];
}

protected function getFormData(): array
{
    return [
        'id' => null,
        'title' => '',
        'slug' => '',
        'content' => '',
        'excerpt' => '',
        'status' => 'draft',
        'featured' => false,
        'category_id' => '',
        'tags' => [],
        'meta_title' => '',
        'meta_description' => '',
    ];
}

protected function getValidationRules(string $context): array
{
    $rules = [
        'title' => ['required' => true, 'max' => 200],
        'content' => ['required' => true],
    ];

    if ($context === 'create') {
        $rules['slug'] = ['slug' => true];
    }

    return $rules;
}

protected function prepareStoreData(array $data): array
{
    // Handle featured image upload
    $uploadedFile = null;
    if (isset($_FILES['featured_image']) && $_FILES['featured_image']['error'] === UPLOAD_ERR_OK) {
        $uploadedFile = $this->fileUploadService->upload($_FILES['featured_image'], 'featured_image');
    }

    return $this->articlePreparer->prepareForCreate($data, $uploadedFile);
}

protected function prepareUpdateData($entity, array $data): array
{
    // Handle featured image upload
    $uploadedFile = null;
    if (isset($_FILES['featured_image']) && $_FILES['featured_image']['error'] === UPLOAD_ERR_OK) {
        $uploadedFile = $this->fileUploadService->upload($_FILES['featured_image'], 'featured_image');
    }

    $action = $data['action'] ?? 'update';
    return $this->articlePreparer->prepareForUpdate($entity, $data, $uploadedFile, $action);
}
```

### Step 4: Remove CRUD methods

Delete these methods (now inherited):
- `index()`
- `create()`
- `edit()`
- `store()`
- `update()`
- `delete()`

### Step 5: Keep API methods

Keep these methods (not part of CRUD pattern):
- `apiArticles()`
- `apiCategories()`

### Step 6: Verify syntax

```bash
php -l calaca/src/Controllers/Admin/ArticleController.php
```

---

## Phase 3: Refactor PageController

Follow same pattern as ArticleController, but with these differences:

### Abstract methods

```php
protected function getRepository(): object
{
    return $this->pageRepository;
}

protected function getContentType(): string
{
    return 'page';
}

protected function getRoutePrefix(): string
{
    return '/admin/pages';
}

protected function getSingularName(): string
{
    return 'page';
}

protected function getPluralName(): string
{
    return 'pages';
}

protected function getTemplateNamespace(): string
{
    return '@admin/pages/pages';
}
```

### Hooks with homepage logic

```php
protected function beforeStore(array $data): ?Response
{
    // If setting as homepage, unset homepage flag on other pages
    if (isset($data['homepage']) && $data['homepage']) {
        $this->pageService->unsetHomepage();
    }

    return null; // Continue normal flow
}

protected function beforeUpdate($entity, array $data): ?Response
{
    // If setting as homepage, unset homepage flag on other pages
    if (isset($data['homepage']) && $data['homepage']) {
        $this->pageService->unsetHomepage();
    }

    return null; // Continue normal flow
}

protected function getIndexData(): array
{
    return [
        'page_types' => $this->pageService->getPageTypes(),
    ];
}

protected function getFormData(): array
{
    return [
        'id' => null,
        'title' => '',
        'slug' => '',
        'subtitle' => '',
        'intro' => '',
        'content' => '',
        'status' => 'draft',
        'template' => 'default',
        'type' => 'default',
        'homepage' => false,
        'parent_id' => null,
        'sort_order' => 0,
        'meta_title' => '',
        'meta_description' => '',
    ];
}

protected function prepareStoreData(array $data): array
{
    // Handle OG image upload
    $uploadedFile = null;
    if (isset($_FILES['og_image']) && $_FILES['og_image']['error'] === UPLOAD_ERR_OK) {
        $uploadedFile = $this->fileUploadService->upload($_FILES['og_image'], 'og_image');
    }

    return $this->pagePreparer->prepareForCreate($data, $uploadedFile);
}

protected function prepareUpdateData($entity, array $data): array
{
    // Handle OG image upload
    $uploadedFile = null;
    if (isset($_FILES['og_image']) && $_FILES['og_image']['error'] === UPLOAD_ERR_OK) {
        $uploadedFile = $this->fileUploadService->upload($_FILES['og_image'], 'og_image');
    }

    $action = $data['action'] ?? 'update';
    return $this->pagePreparer->prepareForUpdate($entity, $data, $uploadedFile, $action);
}
```

### Edit method needs special handling

The `edit()` method in BaseCrudController uses `$this->getIndexData()` for additional data. But PageController needs different data for edit vs index:

```php
protected function getIndexData(): array
{
    // This is called by index() and create()
    return [
        'page_types' => $this->pageService->getPageTypes(),
    ];
}

// But edit() needs additional data. We need to modify BaseCrudController.edit()
// to call a separate getEditData() hook if it exists.

// For now, we can override the entire edit() method:
final public function edit(int $id): Response
{
    $this->requireAuth();

    $repository = $this->getRepository();
    $entity = $repository->find($id);

    if (!$entity) {
        $this->setFlash('error', ucfirst($this->getSingularName()) . ' not found.');
        return $this->redirect($this->getRoutePrefix());
    }

    $contentType = $this->contentTypeManager->getContentType($this->getContentType());
    $formHtml = '';
    if ($contentType) {
        $builder = new \CalacaCMS\Services\FormBuilder($contentType);
        $formHtml = $builder->build(is_array($entity) ? $entity : []);
    }

    $data = array_merge([
        $this->getSingularName() => $entity,
        'csrf_token' => $this->generateCsrfToken(),
        'form_html' => $formHtml,
    ], $this->getEditData($entity));

    return $this->html($this->render($this->getTemplateNamespace() . '/edit.twig', $data));
}

// And add the getEditData() hook to BaseCrudController:
protected function getEditData($entity): array
{
    return [];
}
```

Then in PageController:

```php
protected function getEditData($entity): array
{
    return [
        'available_templates' => $this->pageService->getAvailableTemplates(),
        'available_parents' => $this->pageService->getAvailableParents($entity['id'] ?? 0),
        'page_types' => $this->pageService->getPageTypes(),
    ];
}
```

---

## Testing Checklist

### Phase 1 Testing

- [ ] BaseCrudController.php has no syntax errors
- [ ] All abstract methods declared
- [ ] All hook methods have default implementations
- [ ] All CRUD methods are marked as final

### Phase 2 Testing

- [ ] ArticleController extends BaseCrudController
- [ ] All 6 abstract methods implemented
- [ ] Hooks correctly handle file uploads
- [ ] Hooks correctly call ArticleDataPreparer
- [ ] apiArticles() and apiCategories() still work

### Phase 3 Testing

- [ ] PageController extends BaseCrudController
- [ ] All 6 abstract methods implemented
- [ ] beforeStore/beforeUpdate correctly call pageService->unsetHomepage()
- [ ] Hooks correctly call PageDataPreparer
- [ ] getEditData() provides template/parent data

### Integration Testing

- [ ] Create article → displays in list
- [ ] Edit article → updates correctly
- [ ] Delete article → removed from list
- [ ] Publish article → status changes to published
- [ ] Create page → displays in list
- [ ] Edit page → updates correctly
- [ ] Delete page → removed from list
- [ ] Set page as homepage → previous homepage unset
- [ ] Upload featured image → image saved and linked
- [ ] Upload OG image → image saved and linked

---

## Common Issues & Solutions

### Issue: "Call to undefined method getEditData()"

**Solution**: Add getEditData() hook to BaseCrudController and make edit() method call it.

### Issue: File uploads not working

**Solution**: Ensure prepareStoreData() and prepareUpdateData() check $_FILES for the upload field.

### Issue: Homepage flag not unsetting previous homepage

**Solution**: Ensure beforeStore() and beforeUpdate() in PageController call pageService->unsetHomepage() BEFORE the validation step.

### Issue: Repository method not found

**Solution**: Check that both ArticleRepository and PageRepository have the same method signatures (find, create, update, delete, paginate, search, countAll).

---

**Quick Reference Complete!**

Good luck with the implementation! 🚀
