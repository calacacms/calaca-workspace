# PRD: FormBuilder Refactoring

**File:** `calaca/src/Services/FormBuilder.php`
**Current Size:** 467 lines
**Target Size:** ~200 lines (FormBuilder) + separate renderer classes
**Priority:** HIGH (Score 3.1)
**Complexity:** Medium

## Problem Statement

The `FormBuilder` class has mixed responsibilities and significant code duplication:

### Current Issues

1. **HTML String Duplication**
   - Each field type (text, textarea, number, date, boolean, select, file) has its own method
   - Common patterns repeated 8×: label HTML, required stars, help text, CSS classes
   - Hardcoded Tailwind classes throughout (not themeable)

2. **Too Many Parameters**
   - `buildTextField()`: 7 parameters
   - `buildNumberField()`: 7 parameters
   - `buildTextareaField()`: 7 parameters
   - High cognitive load, error-prone

3. **Mixed Responsibilities**
   - Form orchestration (build())
   - Field routing (buildField())
   - HTML rendering (buildTextField, buildTextareaField, etc.)
   - Error handling (wrapField())
   - Label formatting (formatLabel())

4. **Hard to Test**
   - Can't test individual field rendering in isolation
   - Must instantiate full FormBuilder with ContentType
   - HTML strings embedded in methods

5. **Hard to Extend**
   - Adding new field type requires:
     - New method in FormBuilder (7+ parameters)
     - Update match expression
     - Duplicate label/help/error logic

## Proposed Architecture

### Service Layer Structure

```
FormBuilder (facade - ~200 lines)
├── FormRenderer (orchestrates field rendering)
├── FieldRendererInterface (interface)
│   ├── TextFieldRenderer
│   ├── TextareaFieldRenderer
│   ├── NumberFieldRenderer
│   ├── DateFieldRenderer
│   ├── BooleanFieldRenderer
│   ├── SelectFieldRenderer
│   ├── FileFieldRenderer
│   └── DefaultFieldRenderer
├── HtmlBuilder (utility - builds attributes)
└── FormTheme (config - CSS classes, structure)
```

### New Classes

1. **FieldRendererInterface** - Contract for all field renderers
   ```php
   interface FieldRendererInterface
   {
       public function render(FieldDefinition $field, mixed $value, array $errors): string;
       public function canRender(string $fieldType): bool;
   }
   ```

2. **HtmlBuilder** - Utility for building HTML attributes
   ```php
   class HtmlBuilder
   {
       public function attributes(array $attrs): string;
       public function classes(array $classes): string;
       public function escape(string $value): string;
   }
   ```

3. **FormTheme** - Configurable theme system
   ```php
   class FormTheme
   {
       public function getFieldWrapperClasses(): string;
       public function getLabelClasses(): string;
       public function getInputClasses(string $fieldType): string;
       public function getErrorClasses(): string;
   }
   ```

4. **AbstractFieldRenderer** - Base class with common logic
   ```php
   abstract class AbstractFieldRenderer implements FieldRendererInterface
   {
       protected function renderLabel(string $name, string $label, bool $required): string;
       protected function renderHelpText(string $help): string;
       protected function renderErrors(array $errors): string;
       protected function wrapField(string $html, array $errors): string;
   }
   ```

5. **Concrete Renderers** - One per field type
   ```php
   class TextFieldRenderer extends AbstractFieldRenderer
   {
       public function render(FieldDefinition $field, mixed $value, array $errors): string;
   }
   ```

6. **FormRenderer** - Orchestrates field rendering
   ```php
   class FormRenderer
   {
       public function __construct(
           private FormTheme $theme,
           private HtmlBuilder $html
       ) {
           $this->registerRenderers();
       }

       public function renderField(FieldDefinition $field, mixed $value, array $errors): string;
       private function registerRenderers(): void;
   }
   ```

7. **FormBuilder** (refactored) - Thin facade
   ```php
   class FormBuilder
   {
       public function __construct(
           private ContentType $contentType,
           private FormRenderer $renderer
       ) {}

       public function build(array $values = [], array $errors = []): string;
   }
   ```

## Implementation Phases

### Phase 1: Create Infrastructure (New Classes)
- [ ] Create `FieldRendererInterface`
- [ ] Create `HtmlBuilder` utility class
- [ ] Create `FormTheme` config class
- [ ] Create `AbstractFieldRenderer` base class
- [ ] Create `FormRenderer` orchestrator

**Tests:** Unit tests for HtmlBuilder and FormTheme

### Phase 2: Create Concrete Renderers
- [ ] Create `TextFieldRenderer`
- [ ] Create `TextareaFieldRenderer`
- [ ] Create `NumberFieldRenderer`
- [ ] Create `DateFieldRenderer`
- [ ] Create `BooleanFieldRenderer`
- [ ] Create `SelectFieldRenderer`
- [ ] Create `FileFieldRenderer`
- [ ] Create `DefaultFieldRenderer`

**Tests:** Unit test each renderer's output

### Phase 3: Integrate FormRenderer
- [ ] Register all renderers in FormRenderer
- [ ] Implement renderField() with type routing
- [ ] Add error handling

**Tests:** Integration test FormRenderer + all renderers

### Phase 4: Refactor FormBuilder
- [ ] Inject FormRenderer into FormBuilder
- [ ] Update build() to use FormRenderer
- [ ] Remove all build*Field() methods
- [ ] Keep wrapField() as proxy or move to FormRenderer
- [ ] Keep formatLabel() and getActionUrl()

**Tests:** Verify FormBuilder produces same HTML as before

### Phase 5: Extract Common Patterns
- [ ] Consolidate label rendering in AbstractFieldRenderer
- [ ] Consolidate help text rendering
- [ ] Consolidate error message rendering
- [ ] Consolidate field wrapper rendering

**Tests:** Verify visual output unchanged

### Phase 6: Documentation & Cleanup
- [ ] Document renderer architecture
- [ ] Add examples for adding custom field types
- [ ] Update FormBuilder docblock
- [ ] Remove obsolete comments

### Phase 7: Testing & Verification
- [ ] Render each field type
- [ ] Verify error handling
- [ ] Test with real ContentType
- [ ] Verify CSS classes match original
- [ ] Check accessibility (ARIA attributes)

## Success Criteria

- [ ] FormBuilder reduced to ~200 lines
- [ ] 8 new renderer classes (one per field type)
- [ ] No HTML string duplication
- [ ] CSS classes themeable via FormTheme
- [ ] Adding new field type requires only:
  - Create new renderer class
  - Register in FormRenderer
- [ ] All existing functionality preserved
- [ ] No regressions in form rendering

## Migration Notes

**No Breaking Changes:**
- FormBuilder API unchanged (build() method signature)
- Existing code continues to work
- HTML output identical (except whitespace)

**Extension Points:**
- Custom renderers can be registered
- Theme can be overridden
- Field types can be added without modifying FormBuilder

## Open Questions

1. **Theme Configuration**
   - Should FormTheme use config file or allow runtime overrides?
   - Decision: Support both - config file with runtime override capability

2. **Error Display**
   - Should error display be inline (below field) or grouped (top of form)?
   - Decision: Keep current inline behavior, but make it configurable in FormTheme

3. **Field Registration**
   - Should renderers be auto-discovered or manually registered?
   - Decision: Manual registration in FormRenderer constructor for explicit control

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| CSS class changes break styling | Keep exact same classes, use FormTheme defaults |
| Parameter order bugs in renderers | Use strict types and PHPStan to validate |
| Performance degradation | Benchmarks - should be same or faster (less string concat) |
| Too many small classes | Group related renderers in namespace, document clearly |

## Estimated Complexity

- **New Classes:** 10 (interface + 8 renderers + 3 utilities)
- **Lines of Code:** ~800 total (vs 467 current)
- **Development Time:** 4-5 hours
- **Risk Level:** Medium (well-contained, isolated changes)

## References

- Original FormBuilder: 467 lines
- Each build*Field method: ~30-50 lines
- Field types supported: 8 (text, textarea, number, date, boolean, select, file, default)
- CSS framework: Tailwind CSS (DaisyUI theme classes)
