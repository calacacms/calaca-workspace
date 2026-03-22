# Content-Aware Theme System

## Overview

A smart theme system that automatically adapts colors and styles based on the selected theme, without manual `dark:` classes.

## Key Features

### Semantic Color System
- **Content-Aware Classes**: Use `bg-primary` instead of `bg-indigo-500 dark:bg-indigo-600`
- **Automatic Theme Switching**: Colors adapt automatically to the theme
- **CSS Variables**: Dynamic colors via CSS custom properties
- **No Dark Classes**: No manual dark mode classes needed

## Architecture

### CSS Variables System

```css
:root {
  /* Base Colors - Light Theme */
  --color-primary: 59 130 246; /* RGB values */
  --color-primary-foreground: 255 255 255;
  --color-secondary: 241 245 249;
  --color-secondary-foreground: 15 23 42;
  --color-accent: 59 130 246;
  --color-accent-foreground: 255 255 255;
  --color-background: 255 255 255;
  --color-foreground: 15 23 42;
  --color-muted: 248 250 252;
  --color-muted-foreground: 100 116 139;
  --color-border: 226 232 240;
  --color-input: 255 255 255;
  --color-ring: 59 130 246;
  --color-destructive: 239 68 68;
  --color-destructive-foreground: 255 255 255;
  --color-success: 34 197 94;
  --color-success-foreground: 255 255 255;
  --color-warning: 245 158 11;
  --color-warning-foreground: 255 255 255;
}

/* Dark Theme */
[data-theme="dark"] {
  --color-primary: 99 102 241;
  --color-primary-foreground: 255 255 255;
  --color-secondary: 30 41 59;
  --color-secondary-foreground: 248 250 252;
  --color-accent: 99 102 241;
  --color-accent-foreground: 255 255 255;
  --color-background: 15 23 42;
  --color-foreground: 248 250 252;
  --color-muted: 30 41 59;
  --color-muted-foreground: 148 163 184;
  --color-border: 51 65 85;
  --color-input: 30 41 59;
  --color-ring: 99 102 241;
  --color-destructive: 239 68 68;
  --color-destructive-foreground: 255 255 255;
  --color-success: 34 197 94;
  --color-success-foreground: 255 255 255;
  --color-warning: 245 158 11;
  --color-warning-foreground: 255 255 255;
}
```

### Semantic Color Classes

```css
/* Primary Colors */
.bg-primary { background-color: rgb(var(--color-primary)); }
.text-primary { color: rgb(var(--color-primary)); }
.bg-primary-foreground { background-color: rgb(var(--color-primary-foreground)); }
.text-primary-foreground { color: rgb(var(--color-primary-foreground)); }

/* Secondary Colors */
.bg-secondary { background-color: rgb(var(--color-secondary)); }
.text-secondary { color: rgb(var(--color-secondary)); }
.bg-secondary-foreground { background-color: rgb(var(--color-secondary-foreground)); }
.text-secondary-foreground { color: rgb(var(--color-secondary-foreground)); }

/* Accent Colors */
.bg-accent { background-color: rgb(var(--color-accent)); }
.text-accent { color: rgb(var(--color-accent)); }
.bg-accent-foreground { background-color: rgb(var(--color-accent-foreground)); }
.text-accent-foreground { color: rgb(var(--color-accent-foreground)); }

/* Base Colors */
.bg-background { background-color: rgb(var(--color-background)); }
.text-foreground { color: rgb(var(--color-foreground)); }
.bg-muted { background-color: rgb(var(--color-muted)); }
.text-muted { color: rgb(var(--color-muted-foreground)); }

/* Border & Input */
.border-border { border-color: rgb(var(--color-border)); }
.bg-input { background-color: rgb(var(--color-input)); }
.ring-ring { --tw-ring-color: rgb(var(--color-ring)); }

/* Status Colors */
.bg-destructive { background-color: rgb(var(--color-destructive)); }
.text-destructive { color: rgb(var(--color-destructive)); }
.bg-success { background-color: rgb(var(--color-success)); }
.text-success { color: rgb(var(--color-success)); }
.bg-warning { background-color: rgb(var(--color-warning)); }
.text-warning { color: rgb(var(--color-warning)); }
```

### Component Classes

```css
/* Card Components */
.card {
  @apply bg-card text-card rounded-lg shadow border-border;
}

.card-header {
  @apply p-6 pb-0;
}

.card-content {
  @apply p-6 pt-0;
}

.card-footer {
  @apply p-6 pt-0 flex items-center;
}

/* Sidebar Components */
.sidebar {
  @apply bg-sidebar text-sidebar-foreground;
}

.sidebar-item {
  @apply flex items-center gap-3 px-3 py-2 rounded-md
         text-sidebar-foreground
         hover:bg-sidebar-accent hover:text-sidebar-accent-foreground
         transition-colors;
}

.sidebar-item.active {
  @apply bg-sidebar-accent text-sidebar-accent-foreground;
}

/* Button Components */
.btn-primary {
  @apply bg-primary text-primary-foreground hover:opacity-90 transition-opacity;
}

.btn-secondary {
  @apply bg-secondary text-secondary-foreground hover:opacity-90 transition-opacity;
}

.btn-accent {
  @apply bg-accent text-accent-foreground hover:opacity-90 transition-opacity;
}

.btn-destructive {
  @apply bg-destructive text-destructive-foreground hover:opacity-90 transition-opacity;
}

/* Form Components */
.input {
  @apply bg-input text-foreground border-border rounded-md
         focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring;
}

/* Notification Components */
.notification-success {
  @apply bg-success text-success-foreground;
}

.notification-error {
  @apply bg-destructive text-destructive-foreground;
}

.notification-warning {
  @apply bg-warning text-warning-foreground;
}
```

## Usage

### Template Inheritance

```twig
<!-- base.twig -->
<!DOCTYPE html>
<html class="h-full bg-background text-foreground" lang="en" data-theme="{{ theme }}">
<head>
    <title>{% block title %}{{ page_title }}{% endblock %}</title>
    {% for css in theme.assets.css %}
        <link rel="stylesheet" href="{{ asset(css) }}">
    {% endfor %}
</head>
<body class="h-full">
    {% block content %}{% endblock %}
</body>
</html>
```

### Component Usage

```twig
<!-- Cards -->
<div class="card p-6">
    <div class="card-header">
        <h3 class="text-foreground">Card Title</h3>
    </div>
    <div class="card-content">
        <p class="text-muted">Card content</p>
    </div>
</div>

<!-- Buttons -->
<button class="btn-primary px-4 py-2 rounded-md">Primary Button</button>
<button class="btn-secondary px-4 py-2 rounded-md">Secondary Button</button>
<button class="btn-destructive px-4 py-2 rounded-md">Delete</button>

<!-- Forms -->
<input type="text" class="input w-full p-2" placeholder="Enter text">
<select class="input w-full p-2">
    <option>Select option</option>
</select>

<!-- Sidebar -->
<div class="sidebar">
    <a href="/dashboard" class="sidebar-item active">Dashboard</a>
    <a href="/content" class="sidebar-item">Content</a>
    <a href="/settings" class="sidebar-item">Settings</a>
</div>

<!-- Notifications -->
<div class="notification-success p-4 rounded-md">Success message</div>
<div class="notification-error p-4 rounded-md">Error message</div>
```

## Theme Switching

```php
// Switch theme
$themeManager->setTheme('dark');

// Toggle dark mode
$isDark = $themeManager->toggleDarkMode();

// Generate theme CSS
$cssVariables = $themeManager->generateThemeCss();

// Get theme preview
$preview = $themeManager->getThemePreview('minimal');
```

## Theme Configuration

### Theme YAML Structure

```yaml
name: Theme Name
version: 1.0.0
description: Theme description
author: Author Name
type: admin

config:
  colors:
    primary: '#3b82f6'
    secondary: '#f1f5f9'
    accent: '#3b82f6'
    background: '#ffffff'
    foreground: '#0f172a'
    muted: '#f8fafc'
    border: '#e2e8f0'
    input: '#ffffff'
    ring: '#3b82f6'
    destructive: '#ef4444'
    success: '#22c55e'
    warning: '#f59e0b'

  layout:
    container_padding: '2rem'
    section_gap: '2rem'
    card_padding: '1.5rem'

  typography:
    font_family: 'system-ui'
    font_size: '16px'
    line_height: '1.6'

  components:
    sidebar:
      background: 'indigo-600'
      border: true
      shadow: false
    cards:
      shadow: true
      border: true
      rounded: true
    buttons:
      rounded: true
      shadow: false
    forms:
      rounded: true
      border: true

templates:
  - base
  - dashboard
  - themes

assets:
  css:
    - ../../dist/style.css
  js:
    - ../../dist/admin.js

options:
  sidebar:
    collapsed: false
    show_icons: true
    show_labels: true
  dashboard:
    show_stats: true
    show_recent_activity: true
    show_quick_actions: true
    stats_layout: grid
  colors:
    dark_mode: false
    custom_primary: null
    custom_accent: null
```

## Development

### Creating New Themes

1. **Create Theme Directory**: `public/admin/themes/my-theme/`
2. **Add theme.yml**: Configure colors, layout, typography
3. **Create Templates**: Use semantic classes
4. **Test Theme**: Switch between themes to test
5. **Export Theme**: Package for distribution

### Best Practices

- **Use Semantic Classes**: `bg-primary` instead of `bg-indigo-500`
- **Avoid Dark Classes**: Let the system handle theme switching
- **Component Consistency**: Use consistent component classes
- **Color Harmony**: Ensure colors work well together
- **Accessibility**: Test with different color combinations

### Testing

```bash
# Test theme switching
php -r "require 'core/AdminThemeManager.php'; \$manager = new Core\AdminThemeManager(); \$manager->setTheme('dark');"

# Generate CSS variables
php -r "require 'core/AdminThemeManager.php'; \$manager = new Core\AdminThemeManager(); echo \$manager->generateThemeCss();"

# Toggle dark mode
php -r "require 'core/AdminThemeManager.php'; \$manager = new Core\AdminThemeManager(); \$manager->toggleDarkMode();"
```

## Benefits

### Developer Experience
- **No Dark Classes**: No manual `dark:` classes needed
- **Semantic Naming**: Clear, meaningful class names
- **Automatic Theming**: Themes adapt automatically
- **Consistent Design**: Uniform styling across components

### User Experience
- **Smooth Transitions**: Fluid theme switching
- **Consistent Colors**: Harmonious color palettes
- **Accessibility**: Automatic contrast adjustments
- **Performance**: No runtime theme switching overhead

### Maintenance
- **DRY Principle**: No duplication of dark mode classes
- **Easy Updates**: Theme changes in one place
- **Scalable**: Easy to add new themes
- **Future-Proof**: Ready for new theme features

## Resources

- [Admin Components](../03-admin-ui/admin-components.md)
- [Semantic Colors](../03-admin-ui/semantic-colors.md)
- [Tailwind v4 Setup](../03-admin-ui/tailwind-v4-setup.md)
