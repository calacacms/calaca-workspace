# 🎨 Twig Best Practices voor CMS

## Overzicht
Moderne Twig template development patterns voor CMS systemen.

## 🎯 Template Structure

### 1. Base Layout
```twig
{# layouts/base.html.twig #}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}CMS{% endblock %}</title>
    <link href="https://cdn.tailwindcss.com" rel="stylesheet">
    {% block stylesheets %}{% endblock %}
</head>
<body class="bg-gray-50">
    {% block header %}
        <header class="bg-white shadow-sm">
            <nav class="container mx-auto px-4 py-4">
                {% block navigation %}{% endblock %}
            </nav>
        </header>
    {% endblock %}

    <main class="container mx-auto px-4 py-8">
        {% block content %}{% endblock %}
    </main>

    {% block footer %}
        <footer class="bg-gray-800 text-white py-8 mt-12">
            <div class="container mx-auto px-4 text-center">
                <p>&copy; {{ "now"|date("Y") }} CMS. All rights reserved.</p>
            </div>
        </footer>
    {% endblock %}

    {% block javascripts %}{% endblock %}
</body>
</html>
```

### 2. Template Inheritance
```twig
{# pages/content.html.twig #}
{% extends 'layouts/base.html.twig' %}

{% block title %}{{ content.title }} - CMS{% endblock %}

{% block content %}
    <article class="bg-white rounded-lg shadow-lg p-6">
        <header class="mb-6">
            <h1 class="text-3xl font-bold text-gray-800 mb-2">
                {{ content.title|e }}
            </h1>
            <div class="text-sm text-gray-600">
                <time datetime="{{ content.createdAt|date('c') }}">
                    {{ content.createdAt|date('F j, Y') }}
                </time>
                {% if content.author %}
                    by {{ content.author.name|e }}
                {% endif %}
            </div>
        </header>

        <div class="prose max-w-none">
            {{ content.content|raw }}
        </div>

        {% if content.categories %}
            <div class="mt-6 pt-6 border-t border-gray-200">
                <h3 class="text-sm font-medium text-gray-500 mb-2">Categories</h3>
                <div class="flex flex-wrap gap-2">
                    {% for category in content.categories %}
                        <span class="px-3 py-1 bg-blue-100 text-blue-800 rounded-full text-sm">
                            {{ category.name|e }}
                        </span>
                    {% endfor %}
                </div>
            </div>
        {% endif %}
    </article>
{% endblock %}
```

## 🧩 Component Patterns

### 1. Reusable Components
```twig
{# components/button.html.twig #}
{% set variant = variant|default('primary') %}
{% set size = size|default('md') %}

{% set classes = [
    'inline-flex items-center justify-center font-medium rounded-lg transition-colors',
    variant == 'primary' ? 'bg-blue-600 text-white hover:bg-blue-700' : '',
    variant == 'secondary' ? 'bg-gray-200 text-gray-900 hover:bg-gray-300' : '',
    variant == 'danger' ? 'bg-red-600 text-white hover:bg-red-700' : '',
    size == 'sm' ? 'px-3 py-1.5 text-sm' : '',
    size == 'md' ? 'px-4 py-2' : '',
    size == 'lg' ? 'px-6 py-3 text-lg' : ''
]|filter|join(' ') %}

<button class="{{ classes }}" {{ attributes|default({})|raw }}>
    {% if icon %}
        <span class="mr-2">{{ icon|raw }}</span>
    {% endif %}
    {{ text|e }}
</button>
```

### 2. Card Component
```twig
{# components/card.html.twig #}
<div class="bg-white rounded-lg shadow-lg overflow-hidden {{ class|default('') }}">
    {% if header %}
        <div class="px-6 py-4 border-b border-gray-200">
            <h3 class="text-lg font-semibold text-gray-800">{{ header|e }}</h3>
        </div>
    {% endif %}

    <div class="p-6">
        {{ content|raw }}
    </div>

    {% if footer %}
        <div class="px-6 py-4 bg-gray-50 border-t border-gray-200">
            {{ footer|raw }}
        </div>
    {% endif %}
</div>
```

## 🔧 Macros

### 1. Form Macros
```twig
{# macros/forms.html.twig #}
{% macro input(name, value, type, placeholder, required) %}
    <div class="mb-4">
        <label for="{{ name }}" class="block text-sm font-medium text-gray-700 mb-1">
            {{ name|title|e }}
            {% if required %}<span class="text-red-500">*</span>{% endif %}
        </label>
        <input
            type="{{ type|default('text') }}"
            id="{{ name }}"
            name="{{ name }}"
            value="{{ value|e }}"
            placeholder="{{ placeholder|e }}"
            {% if required %}required{% endif %}
            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
        >
    </div>
{% endmacro %}

{% macro textarea(name, value, placeholder, required) %}
    <div class="mb-4">
        <label for="{{ name }}" class="block text-sm font-medium text-gray-700 mb-1">
            {{ name|title|e }}
            {% if required %}<span class="text-red-500">*</span>{% endif %}
        </label>
        <textarea
            id="{{ name }}"
            name="{{ name }}"
            placeholder="{{ placeholder|e }}"
            {% if required %}required{% endif %}
            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            rows="4"
        >{{ value|e }}</textarea>
    </div>
{% endmacro %}
```

### 2. Navigation Macros
```twig
{# macros/navigation.html.twig #}
{% macro breadcrumb(items) %}
    <nav class="flex" aria-label="Breadcrumb">
        <ol class="flex items-center space-x-2">
            {% for item in items %}
                <li class="flex items-center">
                    {% if not loop.last %}
                        <a href="{{ item.url }}" class="text-gray-500 hover:text-gray-700">
                            {{ item.name|e }}
                        </a>
                        <svg class="w-4 h-4 mx-2 text-gray-400" fill="currentColor" viewBox="0 0 20 20">
                            <path fill-rule="evenodd" d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z" clip-rule="evenodd"></path>
                        </svg>
                    {% else %}
                        <span class="text-gray-900 font-medium">{{ item.name|e }}</span>
                    {% endif %}
                </li>
            {% endfor %}
        </ol>
    </nav>
{% endmacro %}
```

## 🎨 Styling Patterns

### 1. Tailwind CSS Integration
```twig
{# components/content-list.html.twig #}
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
    {% for content in contents %}
        <article class="bg-white rounded-lg shadow-md hover:shadow-lg transition-shadow">
            {% if content.featuredImage %}
                <img
                    src="{{ content.featuredImage.url }}"
                    alt="{{ content.featuredImage.alt|e }}"
                    class="w-full h-48 object-cover rounded-t-lg"
                >
            {% endif %}

            <div class="p-6">
                <h3 class="text-xl font-semibold text-gray-800 mb-2">
                    <a href="{{ path('content_show', {slug: content.slug}) }}"
                       class="hover:text-blue-600 transition-colors">
                        {{ content.title|e }}
                    </a>
                </h3>

                <p class="text-gray-600 mb-4 line-clamp-3">
                    {{ content.excerpt|e }}
                </p>

                <div class="flex items-center justify-between">
                    <time class="text-sm text-gray-500">
                        {{ content.createdAt|date('M j, Y') }}
                    </time>
                    <a href="{{ path('content_show', {slug: content.slug}) }}"
                       class="text-blue-600 hover:text-blue-800 font-medium">
                        Read more →
                    </a>
                </div>
            </div>
        </article>
    {% endfor %}
</div>
```

### 2. Responsive Design
```twig
{# components/hero.html.twig #}
<section class="relative bg-gradient-to-r from-blue-600 to-purple-600 text-white">
    <div class="container mx-auto px-4 py-16 md:py-24">
        <div class="max-w-4xl mx-auto text-center">
            <h1 class="text-4xl md:text-6xl font-bold mb-6">
                {{ title|e }}
            </h1>
            {% if subtitle %}
                <p class="text-xl md:text-2xl mb-8 opacity-90">
                    {{ subtitle|e }}
                </p>
            {% endif %}
            {% if cta %}
                <a href="{{ cta.url }}"
                   class="inline-block bg-white text-blue-600 px-8 py-3 rounded-lg font-semibold hover:bg-gray-100 transition-colors">
                    {{ cta.text|e }}
                </a>
            {% endif %}
        </div>
    </div>
</section>
```

## 🔒 Security Patterns

### 1. Output Escaping
```twig
{# Always escape user input #}
<h1>{{ content.title|e }}</h1>

{# For HTML content, use raw filter carefully #}
<div class="content">
    {{ content.body|raw }}
</div>

{# Escape in attributes #}
<img src="{{ image.url|e }}" alt="{{ image.alt|e }}">

{# Escape in URLs #}
<a href="{{ path('content_show', {slug: content.slug|e}) }}">
    {{ content.title|e }}
</a>
```

### 2. CSRF Protection
```twig
{# forms/content.html.twig #}
<form method="post" action="{{ path('content_save') }}">
    <input type="hidden" name="_token" value="{{ csrf_token('content_save') }}">

    <div class="mb-4">
        <label for="title" class="block text-sm font-medium text-gray-700 mb-1">
            Title
        </label>
        <input
            type="text"
            id="title"
            name="title"
            value="{{ form.title|e }}"
            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            required
        >
    </div>

    <button type="submit" class="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700">
        Save Content
    </button>
</form>
```

## ⚡ Performance Patterns

### 1. Lazy Loading
```twig
{# components/image.html.twig #}
<img
    src="{{ image.url }}"
    alt="{{ image.alt|e }}"
    class="{{ class|default('') }}"
    loading="lazy"
    {% if image.width %}width="{{ image.width }}"{% endif %}
    {% if image.height %}height="{{ image.height }}"{% endif %}
>
```

### 2. Template Caching
```twig
{# Cache expensive operations #}
{% cache 'content_list' ~ contents|length %}
    <div class="content-list">
        {% for content in contents %}
            {# Expensive rendering logic #}
        {% endfor %}
    </div>
{% endcache %}
```

## 🧪 Testing Templates

### 1. Template Testing
```php
class TemplateTest extends TestCase
{
    public function testContentTemplate(): void
    {
        $twig = new Environment(new ArrayLoader([
            'content.html.twig' => file_get_contents('templates/content.html.twig')
        ]));

        $content = [
            'title' => 'Test Content',
            'body' => 'Test body content',
            'createdAt' => new DateTime()
        ];

        $html = $twig->render('content.html.twig', ['content' => $content]);

        $this->assertStringContainsString('Test Content', $html);
        $this->assertStringContainsString('Test body content', $html);
    }
}
```

## 📋 Best Practices Checklist

- [ ] Gebruik template inheritance
- [ ] Implementeer proper escaping
- [ ] Maak reusable components
- [ ] Gebruik macros voor complex logic
- [ ] Implementeer responsive design
- [ ] Cache expensive operations
- [ ] Valideer user input
- [ ] Gebruik semantic HTML
- [ ] Implementeer accessibility features
- [ ] Test templates regelmatig

## 🚀 Performance Tips

1. **Use template caching** voor expensive operations
2. **Implement lazy loading** voor images
3. **Minimize template complexity**
4. **Use includes** voor reusable parts
5. **Optimize database queries** in templates
6. **Use CDN** voor static assets
7. **Implement proper caching** strategies
