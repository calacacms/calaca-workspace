# FormBuilder Refactoring Documentation

## Overview

The `FormBuilder` service has been refactored from a 467-line monolithic class into a modular, extensible renderer architecture. This document explains the new architecture and how to use it.

## Architecture

### Before Refactoring
```
FormBuilder (467 lines)
├── build() - orchestrates field rendering
├── buildField() - routes by field type
├── buildTextField() - 7 parameters, 40 lines
├── buildTextareaField() - 7 parameters, 40 lines
├── buildNumberField() - 7 parameters, 50 lines
├── buildDateField() - 7 parameters, 45 lines
├── buildBooleanField() - 6 parameters, 35 lines
├── buildSelectField() - 8 parameters, 55 lines
├── buildFileField() - 6 parameters, 45 lines
├── buildDefaultField() - 3 parameters, 20 lines
├── wrapField() - error wrapping
└── formatLabel() - label formatting
```

### After Refactoring
```
FormBuilder (93 lines) ← Thin facade
├── FormRenderer (orchestrator)
│   ├── TextFieldRenderer
│   ├── TextareaFieldRenderer
│   ├── NumberFieldRenderer
│   ├── DateFieldRenderer
│   ├── BooleanFieldRenderer
│   ├── SelectFieldRenderer
│   ├── FileFieldRenderer
│   └── DefaultFieldRenderer
├── HtmlBuilder (utility)
├── FormTheme (config)
└── AbstractFieldRenderer (base)
```

## Core Classes

### FormBuilder (Facade)

**Location:** `calaca/src/Services/FormBuilder.php`
**Lines:** 93 (80% reduction from 467)
**Purpose:** Thin facade that creates form HTML from content types

**Usage:**
```php
use CalacaCMS\Services\FormBuilder;
use CalacaCMS\Models\ContentType;

// Create FormBuilder with ContentType
$formBuilder = new FormBuilder($contentType);

// Generate form HTML
$html = $formBuilder->build(
    values: ['title' => 'My Article', 'content' => '...'],
    errors: ['title' => ['Title is required']]
);

echo $html;
```

**Public Methods:**
- `build(array $values, array $errors): string` - Generate form HTML
- `getContentType(): ContentType` - Get content type
- `getActionUrl(string $baseAction): string` - Get form action URL
- `getRenderer(): FormRenderer` - Get form renderer

### FormRenderer (Orchestrator)

**Location:** `calaca/src/Services/FormBuilder/FormRenderer.php`
**Purpose:** Routes field rendering to appropriate field renderer

**Usage:**
```php
use CalacaCMS\Services\FormBuilder\FormRenderer;
use CalacaCMS\Services\FormBuilder\FormTheme;
use CalacaCMS\Services\FormBuilder\HtmlBuilder;

$theme = new FormTheme();
$html = new HtmlBuilder();
$renderer = new FormRenderer($theme, $html, $contentType);

// Render a single field
$fieldHtml = $renderer->renderField($fieldDefinition, $value, $errors);
```

**Methods:**
- `renderField(FieldDefinition $field, mixed $value, array $errors): string`
- `registerRenderer(FieldRendererInterface $renderer): void`
- `setContentType(ContentType $contentType): void`
- `getTheme(): FormTheme`
- `getHtml(): HtmlBuilder`

### HtmlBuilder (Utility)

**Location:** `calaca/src/Services/FormBuilder/HtmlBuilder.php`
**Purpose:** Build HTML attributes and escape content

**Usage:**
```php
use CalacaCMS\Services\FormBuilder\HtmlBuilder;

$html = new HtmlBuilder();

// Build attributes string
$attrs = $html->attributes([
    'type' => 'text',
    'class' => 'form-control',
    'required' => true,
    'placeholder' => 'Enter text',
]);
// Result: type="text" class="form-control" required placeholder="Enter text"

// Build CSS classes
$classes = $html->classes(['form-control', 'is-invalid']);
// Result: "form-control is-invalid"

// Escape HTML
$safe = $html->escape('<script>alert("XSS")</script>');
// Result: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;
```

**Methods:**
- `attributes(array $attrs): string` - Build HTML attribute string
- `classes(array $classes): string` - Build CSS class string
- `escape(string $value): string` - Escape HTML special characters
- `element(string $tag, string $content, array $attrs): string` - Build HTML element
- `voidElement(string $tag, array $attrs): string` - Build self-closing element

### FormTheme (Configuration)

**Location:** `calaca/src/Services/FormBuilder/FormTheme.php`
**Purpose:** Define CSS classes for form styling

**Usage:**
```php
use CalacaCMS\Services\FormBuilder\FormTheme;

// Use default theme
$theme = new FormTheme();

// Get CSS classes for different elements
$wrapperClasses = $theme->getFieldWrapperClasses();
$labelClasses = $theme->getLabelClasses();
$inputClasses = $theme->getInputClasses('text');
$errorClasses = $theme->getErrorClasses();
```

**Extending FormTheme:**
```php
use CalacaCMS\Services\FormBuilder\FormTheme;

class CustomFormTheme extends FormTheme
{
    public function getInputClasses(string $fieldType): string
    {
        // Use Bootstrap classes instead of Tailwind
        return match($fieldType) {
            'text', 'textarea', 'number', 'date' => 'form-control',
            'checkbox' => 'form-check-input',
            default => 'form-control',
        };
    }

    public function getRequiredStarClasses(): string
    {
        return 'text-danger'; // Bootstrap danger color
    }
}

// Use custom theme
$theme = new CustomFormTheme();
$renderer = new FormRenderer($theme, new HtmlBuilder(), $contentType);
```

### Field Renderers

**Location:** `calaca/src/Services/FormBuilder/Renderers/`
**Purpose:** Render specific field types

All renderers implement `FieldRendererInterface`:
```php
interface FieldRendererInterface
{
    public function render(FieldDefinition $field, mixed $value, array $errors): string;
    public function canRender(string $fieldType): bool;
}
```

**Available Renderers:**
- `TextFieldRenderer` - Standard text inputs
- `TextareaFieldRenderer` - Multi-line text areas
- `NumberFieldRenderer` - Number inputs with min/max/step
- `DateFieldRenderer` - Date inputs with min/max dates
- `BooleanFieldRenderer` - Checkbox inputs
- `SelectFieldRenderer` - Dropdown selects (single/multiple)
- `FileFieldRenderer` - File uploads
- `DefaultFieldRenderer` - Fallback for unknown types

### AbstractFieldRenderer (Base)

**Location:** `calaca/src/Services/FormBuilder/AbstractFieldRenderer.php`
**Purpose:** Base class with common rendering logic

**Usage:**
```php
use CalacaCMS\Services\FormBuilder\AbstractFieldRenderer;
use CalacaCMS\Services\FormBuilder\FormTheme;
use CalacaCMS\Services\FormBuilder\HtmlBuilder;

class CustomFieldRenderer extends AbstractFieldRenderer
{
    public function canRender(string $fieldType): bool
    {
        return $fieldType === 'custom';
    }

    protected function renderField(FieldDefinition $field, mixed $value, array $errors): string
    {
        // Use inherited helper methods
        $labelHtml = $this->renderLabel($name, $label, $required);
        $helpHtml = $this->renderHelpText($help);
        $errorHtml = $this->renderErrors($errors);

        // Your custom field HTML
        return "<div>$labelHtml<div>...</div>$helpHtml$errorHtml</div>";
    }
}
```

**Helper Methods:**
- `renderLabel(string $name, string $label, bool $required): string`
- `renderHelpText(string $help): string`
- `renderErrors(array $errors): string`
- `wrapField(string $name, string $html, array $errors): string`
- `formatLabel(string $fieldName): string`
- `getValue(mixed $value, string $default): string`

## Creating Custom Field Types

### Step 1: Create Renderer Class

```php
<?php
// calaca/src/Services/FormBuilder/Renderers/EmailFieldRenderer.php

declare(strict_types=1);

namespace CalacaCMS\Services\FormBuilder\Renderers;

use CalacaCMS\Models\FieldDefinition;
use CalacaCMS\Services\FormBuilder\AbstractFieldRenderer;

class EmailFieldRenderer extends AbstractFieldRenderer
{
    public function canRender(string $fieldType): bool
    {
        return $fieldType === 'email';
    }

    protected function renderField(FieldDefinition $field, mixed $value, array $errors): string
    {
        $name = $field->getName();
        $label = $field->getLabel() ?? $this->formatLabel($name);
        $isRequired = $field->isRequired();
        $placeholder = $field->getPlaceholder() ?? '';
        $help = $field->getHelp() ?? '';

        $inputValue = $this->getValue($value, '');
        $requiredAttr = $isRequired ? ['required' => true] : [];
        $placeholderAttr = $placeholder ? ['placeholder' => $placeholder] : [];

        $attributes = array_merge([
            'type' => 'email',
            'id' => 'field_' . $name,
            'name' => $name,
            'value' => $inputValue,
            'class' => $this->theme->getInputClasses('text'),
        ], $requiredAttr, $placeholderAttr);

        $labelHtml = $this->renderLabel($name, $label, $isRequired);
        $inputHtml = $this->html->voidElement('input', $attributes);
        $helpHtml = $this->renderHelpText($help);

        return sprintf(
            '<div class="%s">%s%s%s</div>',
            $this->theme->getFieldWrapperClasses(),
            $labelHtml,
            $inputHtml,
            $helpHtml
        );
    }
}
```

### Step 2: Register Renderer

```php
use CalacaCMS\Services\FormBuilder\FormRenderer;
use CalacaCMS\Services\FormBuilder\Renderers\EmailFieldRenderer;

$renderer = new FormRenderer($theme, $html, $contentType);
$renderer->registerRenderer(new EmailFieldRenderer($theme, $html));
```

### Step 3: Define Field Type

```yaml
# calaca/types/article.yaml
fields:
  email:
    type: email
    label: Email Address
    required: true
    help: We'll contact you at this address
```

## Testing

### Unit Testing a Renderer

```php
<?php

use PHPUnit\Framework\TestCase;
use CalacaCMS\Services\FormBuilder\Renderers\TextFieldRenderer;
use CalacaCMS\Services\FormBuilder\FormTheme;
use CalacaCMS\Services\FormBuilder\HtmlBuilder;
use CalacaCMS\Models\FieldDefinition;

class TextFieldRendererTest extends TestCase
{
    private TextFieldRenderer $renderer;
    private FormTheme $theme;
    private HtmlBuilder $html;

    protected function setUp(): void
    {
        $this->theme = new FormTheme();
        $this->html = new HtmlBuilder();
        $this->renderer = new TextFieldRenderer($this->theme, $this->html);
    }

    public function testCanRenderTextField(): void
    {
        $this->assertTrue($this->renderer->canRender('text'));
        $this->assertFalse($this->renderer->canRender('textarea'));
    }

    public function testRenderTextField(): void
    {
        $field = new FieldDefinition('title', 'text', [
            'label' => 'Title',
            'required' => true,
            'placeholder' => 'Enter title',
        ]);

        $html = $this->renderer->render($field, 'My Title', []);

        $this->assertStringContainsString('Title', $html);
        $this->assertStringContainsString('My Title', $html);
        $this->assertStringContainsString('required', $html);
        $this->assertStringContainsString('placeholder="Enter title"', $html);
    }
}
```

## Migration Guide

### Before Refactoring

```php
// Old FormBuilder - all logic in one class
$formBuilder = new FormBuilder($contentType);
$html = $formBuilder->build($values, $errors);
```

### After Refactoring

```php
// New FormBuilder - same API, different internals
$formBuilder = new FormBuilder($contentType);
$html = $formBuilder->build($values, $errors);

// OR inject custom renderer
$theme = new CustomFormTheme();
$html = new HtmlBuilder();
$renderer = new FormRenderer($theme, $html, $contentType);
$formBuilder = new FormBuilder($contentType, $renderer);

$html = $formBuilder->build($values, $errors);
```

**No code changes required** - the public API is identical!

## Refactoring Metrics

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| FormBuilder Lines | 467 | 93 | 80% reduction |
| Classes | 1 | 14 | Modular architecture |
| Methods | 12 | 4 | Focused responsibilities |
| Parameters per method | 3-8 | 1-2 | Simpler signatures |
| HTML duplication | 8× | 0× | DRY principle |
| Extensibility | Low | High | Add types without modifying core |

## Benefits

1. **Separation of Concerns**: Each renderer handles one field type
2. **Open/Closed Principle**: Add new field types without modifying existing code
3. **Testability**: Each renderer can be unit tested independently
4. **Themeability**: CSS classes centralized in FormTheme
5. **Maintainability**: Changes to one field type don't affect others
6. **Extensibility**: Easy to add custom field types
7. **Type Safety**: Strict types throughout

## Related Files

- `calaca/src/Services/FormBuilder.php` - Main facade (93 lines)
- `calaca/src/Services/FormBuilder/FormRenderer.php` - Orchestrator
- `calaca/src/Services/FormBuilder/HtmlBuilder.php` - HTML utility
- `calaca/src/Services/FormBuilder/FormTheme.php` - CSS config
- `calaca/src/Services/FormBuilder/AbstractFieldRenderer.php` - Base class
- `calaca/src/Services/FormBuilder/FieldRendererInterface.php` - Interface
- `calaca/src/Services/FormBuilder/Renderers/*` - Field renderers (8 classes)
