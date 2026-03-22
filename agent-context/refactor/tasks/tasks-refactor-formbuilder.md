# Task List: FormBuilder Refactoring

**File:** `calaca/src/Services/FormBuilder.php`
**PRD:** `docs/prd-refactor-formbuilder.md`
**Total Tasks:** 45 (7 phases)

## Phase 1: Create Infrastructure (New Classes) - 5 tasks

- [ ] 1.1 Create `FieldRendererInterface` with `render()` and `canRender()` methods
- [ ] 1.2 Create `HtmlBuilder` utility class with `attributes()`, `classes()`, `escape()` methods
- [ ] 1.3 Create `FormTheme` config class with CSS class getters
- [ ] 1.4 Create `AbstractFieldRenderer` base class with common rendering methods
- [ ] 1.5 Create `FormRenderer` orchestrator class with renderer registry

**Validation:** Unit tests for HtmlBuilder and FormTheme

## Phase 2: Create Concrete Renderers - 8 tasks

- [ ] 2.1 Create `TextFieldRenderer` extending AbstractFieldRenderer
- [ ] 2.2 Create `TextareaFieldRenderer` extending AbstractFieldRenderer
- [ ] 2.3 Create `NumberFieldRenderer` extending AbstractFieldRenderer
- [ ] 2.4 Create `DateFieldRenderer` extending AbstractFieldRenderer
- [ ] 2.5 Create `BooleanFieldRenderer` extending AbstractFieldRenderer
- [ ] 2.6 Create `SelectFieldRenderer` extending AbstractFieldRenderer
- [ ] 2.7 Create `FileFieldRenderer` extending AbstractFieldRenderer
- [ ] 2.8 Create `DefaultFieldRenderer` extending AbstractFieldRenderer

**Validation:** Unit test each renderer's output matches original

## Phase 3: Integrate FormRenderer - 3 tasks

- [ ] 3.1 Register all 8 renderers in FormRenderer constructor
- [ ] 3.2 Implement `renderField()` method with type-based routing
- [ ] 3.3 Add error handling and field not found fallback

**Validation:** Integration test FormRenderer routes to correct renderer

## Phase 4: Refactor FormBuilder - 5 tasks

- [ ] 4.1 Inject FormRenderer into FormBuilder constructor
- [ ] 4.2 Update `build()` method to use FormRenderer->renderField()
- [ ] 4.3 Remove all 8 build*Field() private methods
- [ ] 4.4 Keep `wrapField()` as proxy or move to FormRenderer
- [ ] 4.5 Keep `formatLabel()` and `getActionUrl()` methods

**Validation:** FormBuilder builds form with same HTML output

## Phase 5: Extract Common Patterns - 4 tasks

- [ ] 5.1 Consolidate label rendering in AbstractFieldRenderer->renderLabel()
- [ ] 5.2 Consolidate help text rendering in AbstractFieldRenderer->renderHelpText()
- [ ] 5.3 Consolidate error rendering in AbstractFieldRenderer->renderErrors()
- [ ] 5.4 Consolidate field wrapper rendering in AbstractFieldRenderer->wrapField()

**Validation:** Verify HTML output unchanged for all field types

## Phase 6: Documentation & Cleanup - 5 tasks

- [ ] 6.1 Document renderer architecture in class docblocks
- [ ] 6.2 Add example for adding custom field type in documentation
- [ ] 6.3 Update FormBuilder class docblock with new architecture
- [ ] 6.4 Remove obsolete comments and TODOs
- [ ] 6.5 Create refactoring documentation file

**Validation:** Documentation is clear and comprehensive

## Phase 7: Testing & Verification - 10 tasks

- [ ] 7.1 Test TextFieldRenderer renders text input correctly
- [ ] 7.2 Test TextareaFieldRenderer renders textarea correctly
- [ ] 7.3 Test NumberFieldRenderer with min/max/step attributes
- [ ] 7.4 Test DateFieldRenderer with min_date/max_date attributes
- [ ] 7.5 Test BooleanFieldRenderer checkbox checked state
- [ ] 7.6 Test SelectFieldRenderer with single and multiple options
- [ ] 7.7 Test FileFieldRenderer with current file display
- [ ] 7.8 Test error message display for all field types
- [ ] 7.9 Test with real ContentType from database
- [ ] 7.10 Verify CSS classes match original (no visual changes)

**Validation:** All tests pass, no regressions

## Final Tasks - 5 tasks

- [ ] 8.1 Run PHP syntax validation on all new files
- [ ] 8.2 Run existing tests to ensure no regressions
- [ ] 8.3 Create visual comparison (before/after screenshots)
- [ ] 8.4 Update refactor-candidates-status.md
- [ ] 8.5 Commit all changes with descriptive message

## Success Metrics

- FormBuilder reduced from 467 to ~200 lines (57% reduction)
- 8 renderer classes created (one per field type)
- 3 utility classes created (HtmlBuilder, FormTheme, FormRenderer)
- No HTML string duplication
- CSS classes themeable
- Zero functional regressions

## Notes

- Each phase should be committed separately
- Tests should be run after each phase
- Keep exact CSS classes to avoid visual changes
- Document any deviations from original behavior
