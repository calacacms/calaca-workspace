# Admin UI Components

## Overview

CalacaCMS admin UI components are built with TypeScript and Tailwind CSS v4, using semantic color classes and content-aware themes.

## Component Philosophy

- **Basecoat UI style**: Modular, reusable components
- **Semantic colors**: `bg-primary`, `text-foreground` instead of hardcoded values
- **TypeScript**: Strict typing for all components
- **No barrel files**: Explicit imports only
- **Tailwind CSS v4**: CSS-first configuration

## Core Components

### Layout Components

#### Card

```typescript
// types/CardProps.ts
export interface CardProps {
  title?: string;
  description?: string;
  children: HTMLElement;
  footer?: HTMLElement;
}
```

```html
<!-- Template -->
<div class="bg-base-100 text-base-content rounded-lg border border-border shadow">
  <div class="p-6">
    <h3 class="text-lg font-semibold leading-none tracking-tight">Card Title</h3>
    <p class="text-base-content/80 text-sm mt-1">Card description</p>
  </div>
  <div class="p-6 pt-0">
    <!-- Content -->
  </div>
  <div class="p-6 pt-0 flex items-center">
    <!-- Footer -->
  </div>
</div>
```

#### Sidebar Navigation

```html
<aside class="bg-base-200 text-base-content w-64 h-screen fixed left-0 top-0 border-r border-border">
  <nav class="space-y-1 p-4">
    <a href="/admin/dashboard" class="sidebar-item active">
      <span class="icon">📊</span>
      Dashboard
    </a>
    <a href="/admin/content" class="sidebar-item">
      <span class="icon">📝</span>
      Content
    </a>
    <a href="/admin/media" class="sidebar-item">
      <span class="icon">🖼️</span>
      Media
    </a>
    <a href="/admin/users" class="sidebar-item">
      <span class="icon">👥</span>
      Users
    </a>
    <a href="/admin/settings" class="sidebar-item">
      <span class="icon">⚙️</span>
      Settings
    </a>
  </nav>
</aside>
```

```css
.sidebar-item {
  @apply flex items-center gap-3 px-3 py-2 rounded-md text-base-content
         hover:bg-base-300
         transition-colors;
}

.sidebar-item.active {
  @apply bg-primary text-primary-content;
}
```

### Form Components

#### Input

```html
<div class="space-y-2">
  <label class="text-sm font-medium leading-none">
    Label
  </label>
  <input
    type="text"
    class="flex h-9 w-full rounded-md border border-border bg-base-200 px-3 py-1
           text-base-content shadow-sm transition-colors
           placeholder:text-base-content/60
           focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-primary-focus"
    placeholder="Enter value..."
  />
  <p class="text-xs text-base-content/60">Helper text</p>
</div>
```

#### Textarea

```html
<textarea
  class="flex min-h-[80px] w-full rounded-md border border-border bg-base-200 px-3 py-2
         text-base-content shadow-sm transition-colors
         placeholder:text-base-content/60
         focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-primary-focus"
  placeholder="Enter longer text..."
></textarea>
```

#### Select

```html
<select
  class="flex h-9 w-full rounded-md border border-border bg-base-200 px-3 py-1
         text-base-content shadow-sm transition-colors
         focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-primary-focus"
>
  <option value="">Select option...</option>
  <option value="1">Option 1</option>
  <option value="2">Option 2</option>
</select>
```

### Button Components

```typescript
// types/ButtonProps.ts
export interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'destructive' | 'outline' | 'ghost';
  size?: 'sm' | 'default' | 'lg';
  children: string | HTMLElement;
  onClick?: () => void;
  type?: 'button' | 'submit' | 'reset';
  disabled?: boolean;
}
```

```html
<!-- Primary Button -->
<button class="btn-primary h-9 px-4 py-2">
  Save Changes
</button>

<!-- Secondary Button -->
<button class="btn-secondary h-9 px-4 py-2">
  Cancel
</button>

<!-- Destructive Button -->
<button class="btn-destructive h-9 px-4 py-2">
  Delete
</button>

<!-- Outline Button -->
<button class="btn-outline h-9 px-4 py-2">
  Preview
</button>
```

```css
/* Base button styles */
.btn {
  @apply inline-flex items-center justify-center rounded-md text-sm font-medium
         transition-colors focus-visible:outline-none focus-visible:ring-1
         focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50;
}

.btn-primary {
  @apply btn bg-primary text-primary-content shadow
         hover:bg-primary/90;
}

.btn-secondary {
  @apply btn bg-secondary text-secondary-content shadow-sm
         hover:bg-secondary/80;
}

.btn-destructive {
  @apply btn bg-error text-error-content shadow-sm
         hover:bg-error/90;
}

.btn-outline {
  @apply btn border border-border bg-base-100 shadow-sm
         hover:bg-accent hover:text-accent-content;
}

.btn-ghost {
  @apply btn hover:bg-base-200 hover:text-base-content;
}
```

### Data Display Components

#### Table

```html
<div class="rounded-md border border-border">
  <table class="w-full text-sm">
    <thead class="bg-base-200">
      <tr class="border-b border-border">
        <th class="h-10 px-4 text-left font-medium text-base-content/80">Title</th>
        <th class="h-10 px-4 text-left font-medium text-base-content/80">Status</th>
        <th class="h-10 px-4 text-left font-medium text-base-content/80">Date</th>
        <th class="h-10 px-4 text-right font-medium text-base-content/80">Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr class="border-b border-border bg-base-100">
        <td class="p-4 font-medium">Page Title</td>
        <td class="p-4">
          <span class="inline-flex items-center rounded-full bg-success px-2 py-1 text-xs text-success-content">
            Published
          </span>
        </td>
        <td class="p-4 text-base-content/60">2024-01-15</td>
        <td class="p-4 text-right">
          <button class="btn-ghost h-8 px-2">Edit</button>
        </td>
      </tr>
    </tbody>
  </table>
</div>
```

#### Badge

```html
<!-- Status badges -->
<span class="inline-flex items-center rounded-full border px-2.5 py-0.5 text-xs font-semibold transition-colors
             bg-success text-success-foreground border-transparent">
  Published
</span>

<span class="inline-flex items-center rounded-full border px-2.5 py-0.5 text-xs font-semibold transition-colors
             bg-warning text-warning-foreground border-transparent">
  Draft
</span>

<span class="inline-flex items-center rounded-full border px-2.5 py-0.5 text-xs font-semibold transition-colors
             bg-destructive text-destructive-foreground border-transparent">
  Error
</span>
```

### Feedback Components

#### Alert

```html
<!-- Success alert -->
<div class="relative w-full rounded-lg border border-success bg-success p-4 text-success-content">
  <div class="flex gap-3">
    <svg class="h-5 w-5">...</svg>
    <div class="flex-1">
      <h5 class="font-medium">Success</h5>
      <p class="text-sm opacity-90">Content saved successfully.</p>
    </div>
  </div>
</div>

<!-- Error alert -->
<div class="relative w-full rounded-lg border border-error bg-error p-4 text-error-content">
  <div class="flex gap-3">
    <svg class="h-5 w-5">...</svg>
    <div class="flex-1">
      <h5 class="font-medium">Error</h5>
      <p class="text-sm opacity-90">Failed to save content.</p>
    </div>
  </div>
</div>
```

#### Toast Notification

```typescript
// lib/toast.ts
export interface ToastOptions {
  title: string;
  description?: string;
  variant?: 'default' | 'success' | 'warning' | 'destructive';
  duration?: number;
}

export function toast(options: ToastOptions): void {
  // Implementation
}
```

## Tailkit Components

CalacaCMS integrates with Tailkit, a comprehensive library of 640+ pre-built Tailwind CSS UI components available in HTML, React, Vue.js, and Alpine.js formats.

### Using Tailkit Components

CalacaCMS uses the official Tailkit MCP server to access 640+ pre-built UI components. The MCP server provides programmatic access to Tailkit's component library.

For detailed setup and usage instructions, see: https://tailkit.com/docs/mcp-server/introduction

```bash
# Browse component catalog
mcp8_browse_catalog --level="packages"

# Search for specific components  
mcp8_search_components --query="dashboard sidebar"

# Get component code (supports html, react, vue, alpine)
mcp8_get_component_code --identifier="a-c-alerts-01" --tech="html"

# Get latest components
mcp8_get_latest_components --limit=10

# Get component suggestions for specific page types
mcp8_get_component_suggestions --page_type="dashboard"
```

### Available Component Categories

- **Application UI**: Dashboards, forms, layouts, navigation
- **Marketing**: Hero sections, pricing tables, testimonials
- **E-commerce**: Product cards, shopping carts, checkout flows

### Integration Example

```html
<!-- Tailkit alert component (adapted for our semantic colors) -->
<div class="bg-success border border-success/20 text-success-foreground rounded-lg p-4">
  <div class="flex">
    <div class="flex-shrink-0">
      <svg class="h-5 w-5" fill="currentColor" viewBox="0 0 20 20">
        <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
      </svg>
    </div>
    <div class="ml-3">
      <p class="text-sm font-medium">Success</p>
      <p class="mt-1 text-sm opacity-90">Operation completed successfully.</p>
    </div>
  </div>
</div>
```

### Benefits

1. **Rapid Development**: Pre-built components accelerate UI development
2. **Consistent Design**: All components follow unified design patterns
3. **Multiple Formats**: HTML, React, Vue.js, and Alpine.js support
4. **Semantic Integration**: Components can be adapted to use our semantic color system
5. **Regular Updates**: New components added regularly

### Best Practices with Tailkit

- Always adapt Tailkit components to use semantic colors (`bg-primary`, `text-foreground`)
- Remove any hardcoded colors from Tailkit components
- Use Tailkit as a starting point, then customize for CalacaCMS needs
- Leverage the MCP server for efficient component discovery and retrieval

## Form Patterns

### Content Editing Form

```html
<form class="space-y-6">
  <!-- Title field -->
  <div class="space-y-2">
    <label class="text-sm font-medium">Title</label>
    <input type="text" class="flex h-9 w-full rounded-md border border-border bg-base-200 px-3 py-1 text-base-content shadow-sm transition-colors placeholder:text-base-content/60 focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-primary-focus" required />
  </div>

  <!-- Content field -->
  <div class="space-y-2">
    <label class="text-sm font-medium">Content</label>
    <textarea class="flex min-h-[80px] w-full rounded-md border border-border bg-base-200 px-3 py-2 text-base-content shadow-sm transition-colors placeholder:text-base-content/60 focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-primary-focus" rows="10"></textarea>
  </div>

  <!-- Status field -->
  <div class="space-y-2">
    <label class="text-sm font-medium">Status</label>
    <select class="flex h-9 w-full rounded-md border border-border bg-base-200 px-3 py-1 text-base-content shadow-sm transition-colors focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-primary-focus">
      <option value="draft">Draft</option>
      <option value="published">Published</option>
    </select>
  </div>

  <!-- Actions -->
  <div class="flex items-center gap-4">
    <button type="submit" class="btn-primary h-9 px-4 py-2">Save</button>
    <button type="button" class="btn-outline h-9 px-4 py-2">Preview</button>
    <button type="button" class="btn-ghost h-9 px-4 py-2">Cancel</button>
  </div>
</form>
```

### Settings Form

```html
<form class="space-y-8">
  <!-- Section -->
  <div class="space-y-4">
    <h3 class="text-lg font-medium">General Settings</h3>
    <p class="text-base-content/60 text-sm">Configure basic site settings</p>
    
    <div class="grid gap-4">
      <div class="space-y-2">
        <label class="text-sm font-medium">Site Name</label>
        <input type="text" class="flex h-9 w-full rounded-md border border-border bg-base-200 px-3 py-1 text-base-content shadow-sm transition-colors placeholder:text-base-content/60 focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-primary-focus" />
      </div>
      <div class="space-y-2">
        <label class="text-sm font-medium">Site Description</label>
        <textarea class="flex min-h-[80px] w-full rounded-md border border-border bg-base-200 px-3 py-2 text-base-content shadow-sm transition-colors placeholder:text-base-content/60 focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-primary-focus" rows="3"></textarea>
      </div>
    </div>
  </div>

  <!-- Section -->
  <div class="space-y-4">
    <h3 class="text-lg font-medium">Appearance</h3>
    <p class="text-base-content/60 text-sm">Customize the look and feel</p>
    
    <div class="flex items-center space-x-2">
      <input type="checkbox" id="dark-mode" class="h-4 w-4 rounded border border-border bg-base-200 text-primary focus:ring-primary-focus" />
      <label for="dark-mode" class="text-sm font-medium">Enable dark mode by default</label>
    </div>
  </div>
</form>
```

## Layout Patterns

### Admin Page Layout

```html
<div class="min-h-screen bg-base-100">
  <!-- Sidebar -->
  <aside class="sidebar">
    <!-- Navigation -->
  </aside>

  <!-- Main content -->
  <main class="ml-64 p-8">
    <!-- Page header -->
    <div class="mb-8">
      <h1 class="text-2xl font-bold text-base-content">Page Title</h1>
      <p class="text-base-content/60">Page description</p>
    </div>

    <!-- Content -->
    <div class="space-y-6">
      <!-- Cards, forms, etc -->
    </div>
  </main>
</div>
```

### Dashboard Layout

```html
<div class="grid gap-6 md:grid-cols-2 lg:grid-cols-4">
  <!-- Stat cards -->
  <div class="bg-base-100 p-6 rounded-lg border border-border shadow">
    <p class="text-base-content/60 text-sm">Total Pages</p>
    <p class="text-2xl font-bold">24</p>
  </div>
  <div class="bg-base-100 p-6 rounded-lg border border-border shadow">
    <p class="text-base-content/60 text-sm">Published</p>
    <p class="text-2xl font-bold">18</p>
  </div>
  <!-- ... -->
</div>

<div class="grid gap-6 md:grid-cols-2">
  <!-- Recent activity -->
  <div class="bg-base-100 rounded-lg border border-border shadow">
    <div class="p-6 border-b border-border">
      <h3 class="text-lg font-semibold">Recent Activity</h3>
    </div>
    <div class="p-6">
      <!-- Activity list -->
    </div>
  </div>

  <!-- Quick actions -->
  <div class="bg-base-100 rounded-lg border border-border shadow">
    <div class="p-6 border-b border-border">
      <h3 class="text-lg font-semibold">Quick Actions</h3>
    </div>
    <div class="p-6">
      <!-- Action buttons -->
    </div>
  </div>
</div>
```

## TypeScript Types

```typescript
// types/ui.ts

export interface ComponentBase {
  className?: string;
  children?: React.ReactNode;
}

export interface ButtonProps extends ComponentBase {
  variant?: 'primary' | 'secondary' | 'destructive' | 'outline' | 'ghost';
  size?: 'sm' | 'default' | 'lg';
  onClick?: () => void;
  type?: 'button' | 'submit' | 'reset';
  disabled?: boolean;
  loading?: boolean;
}

export interface InputProps extends ComponentBase {
  type?: string;
  name?: string;
  value?: string;
  placeholder?: string;
  required?: boolean;
  disabled?: boolean;
  onChange?: (value: string) => void;
}

export interface CardProps extends ComponentBase {
  title?: string;
  description?: string;
  footer?: React.ReactNode;
}

export interface BadgeProps extends ComponentBase {
  variant?: 'default' | 'secondary' | 'destructive' | 'outline' | 'success' | 'warning';
}
```

## Best Practices

1. **Use semantic colors**: `bg-primary` not `bg-blue-500`
2. **Consistent spacing**: Use Tailwind's spacing scale
3. **Focus states**: Always include focus-visible styles
4. **Disabled states**: Use `disabled:opacity-50` and `disabled:pointer-events-none`
5. **Transitions**: Use `transition-colors` for interactive elements
6. **No hardcoded colors**: Everything through CSS variables

## File Structure

```
themes/admin/
├── assets/
│   ├── css/
│   │   ├── admin.css           # Main CSS file
│   │   ├── base/               # Base styles
│   │   ├── components/         # Component styles
│   │   └── themes/             # Theme definitions
│   ├── ts/                     # TypeScript source
│   │   ├── components/         # UI components
│   │   ├── utils/              # Utility functions
│   │   └── index.ts            # Entry point
│   └── templates/              # Twig templates
├── vite.config.js              # Vite configuration
├── tsconfig.json              # TypeScript configuration
└── theme.json                 # Theme metadata
```

## Built Output

```
web/assets/admin/
├── css/
│   └── admin.css              # Built CSS
└── js/
    └── admin.js               # Built JavaScript
```

## Resources

- [Semantic Colors](./semantic-colors.md)
- [Tailwind v4 Setup](./tailwind-v4-setup.md)
- [Content-Aware Themes](../05-content-aware-themes/content-aware-themes.md)
