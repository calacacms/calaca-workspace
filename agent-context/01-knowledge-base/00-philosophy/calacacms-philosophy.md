# CalacaCMS Philosophy

## Overview

The philosophy behind CalacaCMS — a CMS that adapts to your website, not the other way around.

## Core Belief

Most CMSes force you as a creator into their system. You have to adapt your design and vision to fit within their frameworks.
**We turn that around: the CMS adapts to your website, not the other way around.**

A CMS should be a helper, not a dictator. It's there to make content manageable, without requiring concessions on design or structure.

## Core Principles

### 1. Site first, CMS second

You start with the website you want to build. HTML, CSS, JavaScript, Twig. Then you connect the CMS. The CMS respects your choices and only adds manageability.

### 2. Less is more

No abundance of options, menus, or settings you never use. Only the fields and functions your site needs.

### 3. Adaptable per site

Every site is different. The CMS is configured via simple files, so it exactly matches the project. No unnecessary bloat.

### 4. Freedom in design

The CMS imposes nothing. Your frontend templates are completely yours. The CMS only provides the data, you determine how it looks.

### 5. Direct publishing

Save = live. No build step, no waiting time. Content changes are immediately visible.

### 6. Simplicity in use

- **For developers:** simple to set up, no heavy dependencies
- **For editors:** a clear admin, without distraction

## Why This Is Different

### Traditional CMSes (WordPress, Drupal, Joomla)

- Large, cumbersome, too many features you don't use
- Your site must adapt to the CMS
- Overwhelming admin interface
- Performance overhead

### Headless CMSes

- Flexible, but often complex (GraphQL/APIs, build steps)
- Separation between content and presentation
- Often overkill for simple projects

### CalacaCMS

- Lightweight, modular, and focused on your project
- Offers the sweet spot: simple AND extensible
- Site-first approach
- Direct publishing

## How It Works

### 1. Build your site as you want it

```twig
<!-- Your own HTML/Twig templates -->
<article class="project-card">
    <h2>{{ project.title }}</h2>
    <p>{{ project.description }}</p>
    <img src="{{ project.image.url }}" alt="{{ project.image.alt }}">
</article>
```

### 2. Define content types in config

```php
// calaca/types/project.php
return new class implements ContentTypeDefinitionInterface {
    public static function define(): ContentType {
        return ContentType::create('project', 'Projects')
            ->field(TextField::make('title')->required())
            ->field(TextareaField::make('description'))
            ->field(FileField::make('image')->accept('image/*'))
            ->field(SelectField::make('category')->options(['web', 'design', 'branding']))
            ->field(BooleanField::make('featured'));
    }
};
```

### 3. Automatic admin UI generation

The CMS automatically generates the admin interface based on your configuration:

- Forms for content input
- Clear list views
- Media management
- User-friendly interface

### 4. Direct content management

```php
// In your templates
$projects = $contentTypeManager->getContentByType('project');
$pages = $contentTypeManager->getContentByType('page');
```

## Who Is This For?

### Designers and Developers

- Freedom in technology stack choices
- No limitations from CMS templates
- Full control over frontend
- Modern development workflow

### Agencies

- Quickly deliver flexible sites
- No complexity of existing CMSes
- Project-specific configuration
- Efficient development process

### Content Teams

- Clarity and simplicity
- No unnecessary features
- Intuitive interface
- Direct publishing

## Technical Stack

### Backend

- **PHP 8.x** - Modern PHP patterns
- **Twig** - Template engine
- **Multiple storage drivers** - FlatFile, SQLite, MySQL, PostgreSQL

### Admin Frontend

- **TypeScript** - Strict type checking
- **Bun** - Fast JavaScript runtime and package manager
- **Tailwind CSS v4** - Utility-first styling with semantic classes
- **Content-Aware Themes** - Automatic theme switching without `dark:` classes

### Key Technologies

- PHP fluent builder API for content type definitions (type-safe, full IDE autocomplete)
- PHPUnit for testing
- PHP-CS-Fixer and PHPStan for code quality

## Implementation Principles

### 1. Template-First Development

```twig
{# You start with the template #}
<div class="hero-section">
    <h1>{{ hero.title }}</h1>
    <p>{{ hero.subtitle }}</p>
    <img src="{{ hero.background_image.url }}" alt="{{ hero.background_image.alt }}">
</div>
```

### 2. Content Type Mapping

```php
// calaca/types/hero.php
return new class implements ContentTypeDefinitionInterface {
    public static function define(): ContentType {
        return ContentType::create('hero', 'Hero')
            ->field(TextField::make('title'))
            ->field(TextareaField::make('subtitle'))
            ->field(FileField::make('background_image'));
    }
};
```

### 3. Zero-Config Admin

- Automatic form generation
- Intuitive interface
- No unnecessary options
- Project-specific features

## Technical Benefits

### Performance

- No unnecessary features
- Optimized for your use case
- Direct publishing (no build step)
- Minimal overhead

### Flexibility

- Full control over frontend
- Modern development stack
- No vendor lock-in
- Easy to extend via modules

### Developer Experience

- Simple setup
- No heavy dependencies
- Modern PHP patterns with full type-safety
- IDE autocomplete for content type definitions
- TypeScript support in admin

## Best Practices

### Content Modeling

- Start with your design
- Define only needed fields
- Use semantic field types
- Keep it simple

### Template Development

- Use Twig for templating
- Implement responsive design
- Focus on performance
- Test on different devices

### Admin Experience

- Keep interface clean
- Use clear labels
- Implement validation
- Provide feedback on actions

## Getting Started

### 1. Setup your project

```bash
# Clone the repository
git clone <repository-url>
cd calacacms

# Install PHP dependencies
composer install

# Install admin frontend dependencies
cd public/admin
bun install
bun run build
```

### 2. Configure content types

```php
// calaca/types/my-content-type.php
return new class implements ContentTypeDefinitionInterface {
    public static function define(): ContentType {
        return ContentType::create('my-type', 'My Content Type')
            ->field(TextField::make('title')->required());
    }
};
```

### 3. Build your templates

```twig
{% extends 'base.twig' %}

{% block content %}
<div class="project-list">
    {% for project in projects %}
        <article class="project-card">
            <h2>{{ project.title }}</h2>
            <p>{{ project.description }}</p>
        </article>
    {% endfor %}
</div>
{% endblock %}
```

## Our Mission

We want to change the way people look at CMSes. Not an all-in-one monster that wants to be everything, but a flexible foundation on which you build exactly the site you envision.

**A CMS that doesn't get in your way, but moves with you.**

---

For detailed technical information, refer to the [Single Source of Truth (SSOT)](../docs/SSOT.md).
