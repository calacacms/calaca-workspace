# CalacaCMS System Architecture

> **Status**: ✅ **Production Ready** - All migration phases (A-D) complete
> **Last Updated**: 2026-03-19
> **Based on**: Current implementation with Bootstrap.php, Symfony Routing, and modern PHP patterns

## Overview

CalacaCMS is a lightweight, site-first PHP CMS built with a layered architecture emphasizing modularity, flexibility, and simplicity. The current implementation represents a **significant improvement** over the original architectural proposal, with production-ready features including:

- **Bootstrap-based service container** with League Container + Symfony Routing
- **Repository Pattern** for clean data access abstraction
- **Admin API** with 8 endpoints for headless operations
- **Multi-theme system** with scaffold-based ownership model
- **Environment-based configuration** via .env support
- **Rate limiting** and security middleware

## Migration History

The current architecture is the result of a **four-phase migration** (A-D) that transformed the original monolithic structure into a modern, production-ready CMS:

### Phase A: Document Root Separation ✅
- Moved entry point to `web/index.php`
- Separated public assets from private code
- Improved security with proper document root

### Phase B: Site Directories to Root ✅
- Moved `content/`, `config/`, `themes/`, `storage/` to project root
- Clear separation between engine and site code
- Site builder owns all customization directories

### Phase C: Engine as Composer Package ✅
- Extracted engine to `vendor/calacacms/core` (symlink to `calaca/`)
- Clean dependency management via Composer
- Engine becomes read-only package

### Phase D: New Features ✅
- Added `site/` directory with YAML blueprints
- Implemented theme scaffold pattern with CLI commands
- Added `.env` support for environment configuration

**Result**: The current implementation is **superior** to the original architectural proposal, with better separation of concerns, security, and developer experience.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Web Layer                            │
├──────────────┬──────────────────┬───────────────────────────┤
│  Frontend    │     Admin        │         Admin API         │
│  web/index.php│   (TypeScript    │      /admin/api/*       │
│  + Symfony   │   + Tailwind v4) │                           │
└──────┬───────┴────────┬─────────┴───────────┬──────────────┘
       │                │                     │
├──────▼────────────────▼─────────────────────▼─────────────┤
│                    Bootstrap Layer                         │
│  ┌────────────┐  ┌──────────┐  ┌────────┐  ┌──────────┐   │
│  │ Bootstrap  │  │  Auth    │  │  CSRF  │  │  Cache   │   │
│  │   .php     │  │Middleware│  │Middleware│  │FileCache│   │
│  └──────┬─────┘  └────┬─────┘  └───┬────┘  └────┬─────┘   │
│         │             │            │            │          │
│  ┌──────▼─────────────▼────────────▼────────────▼───────┐  │
│  │               Service Container                      │  │
│  │              (League Container)                      │  │
│  └──────────────────────┬──────────────────────────────┘  │
│                         │                                  │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │           Content Type Manager                       │  │
│  │         (YAML + PHP blueprints)                     │  │
│  └──────────────────────┬──────────────────────────────┘  │
│                         │                                  │
│  ┌──────────────────────▼──────────────────────────────┐  │
│  │            Storage Abstraction                       │  │
│  │  ┌──────────┐ ┌──────────┐ ┌─────────┐ ┌──────────┐  │  │
│  │  │ FlatFile │ │  SQLite  │ │  MySQL  │ │PostgreSQL│  │  │
│  │  └──────────┘ └──────────┘ └─────────┘ └──────────┘  │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
       │
├──────▼──────────────────────────────────────────────────────┤
│                   Repository Layer                         │
│  ┌─────────┐ ┌──────────┐ ┌─────────┐ ┌─────────┐          │
│  │Article  │ │   Page   │ │  Users  │ │Settings │          │
│  │Repository│ │Repository│ │Repository│ │Service  │          │
│  └─────────┘ └──────────┘ └─────────┘ └─────────┘          │
└─────────────────────────────────────────────────────────────┘
       │
├──────▼──────────────────────────────────────────────────────┤
│                    Site Layer                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ site/ (single site, multi-site ready)               │   │
│  │ ├── blueprints/pages/    # YAML content definitions │   │
│  │ ├── collections/         # Custom collections       │   │
│  │ ├── controllers/         # Custom controllers        │   │
│  │ └── content/            # FlatFile content storage  │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Layer Details

### 1. Web Layer

Entry points for different interfaces:

- **`web/index.php`** — Frontend entry, Bootstrap-based routing with Symfony
- **`web/admin/`** — Modern admin panel (TypeScript + Tailwind CSS v4 + Bun + Vite)
- **`/admin/api/*`** — REST JSON API for admin operations (8 endpoints)

### 2. Bootstrap Layer

Central bootstrap and services:

| Component | Responsibility | Implementation |
|-----------|----------------|----------------|
| `Bootstrap.php` | Service container, bootstrapping, routing | League Container + Symfony |
| `AuthMiddleware` | User authentication, sessions | SessionService + RateLimiter |
| `CsrfMiddleware` | CSRF protection tokens | Token-based validation |
| `FileCache` | File-based caching | `/storage/cache` directory |
| `ContentTypeManager` | Content type definitions and CRUD | YAML blueprints + PHP overrides |
| `StorageFactory` | Storage driver instantiation | Factory pattern |

### 3. Repository Layer

Data access abstraction with repositories:

```php
interface RepositoryInterface {
    public function find(string $id): ?array;
    public function findBy(array $criteria): array;
    public function all(): array;
    public function create(array $data): string;
    public function update(string $id, array $data): bool;
    public function delete(string $id): bool;
}
```

**Repositories**:
- **ArticleRepository** — Article content management
- **PageRepository** — Page content management  
- **UsersRepository** — User management
- **SettingsService** — Site configuration management

### 4. Storage Abstraction

Multiple storage backends with unified interface:

```php
interface StorageInterface {
    public function find(string $type, string $id): ?array;
    public function findBy(string $type, array $criteria): array;
    public function all(string $type): array;
    public function create(string $type, array $data): string;
    public function update(string $type, string $id, array $data): bool;
    public function delete(string $type, string $id): bool;
}
```

**Drivers**:
- **FlatFile** — JSON files, ideal for development
- **SQLite** — Embedded database, good for small/medium sites
- **MySQL/MariaDB** — Production relational database
- **PostgreSQL** — Enterprise relational database

### 5. Site Layer

Single-site capable with multi-site ready structure:

```
site/
├── blueprints/pages/      # YAML content type definitions
│   ├── page.yml
│   └── article.yml
├── collections/           # Custom collections (future)
├── controllers/          # Custom controllers (future)
└── content/              # FlatFile storage root
    ├── pages/
    ├── articles/
    └── users/
```

**Content Types**: YAML blueprints in `site/blueprints/pages/*.yml`
**Storage**: JSON files in `content/` directory
**Future Ready**: Structure supports multi-site expansion

## Key Design Patterns

### Bootstrap Service Container

```php
// src/Core/Bootstrap.php
class Bootstrap {
    private static ?StorageInterface $storage = null;
    private static ?CacheInterface $cache = null;
    private static array $repositories = [];
    private static array $controllers = [];
    
    public static function initialize(): void {
        self::loadConfig();
        self::initializeStorage();
        self::initializeRepositories();
        self::initializeControllers();
    }
}

// Usage in controllers
$storage = Bootstrap::getStorage();
$repository = Bootstrap::getRepository('articles');
```

### Repository Pattern

```php
// src/Repositories/ArticleRepository.php
class ArticleRepository implements RepositoryInterface {
    public function __construct(private StorageInterface $storage) {}
    
    public function find(string $id): ?array {
        return $this->storage->find('articles', $id);
    }
    
    public function all(): array {
        return $this->storage->all('articles');
    }
}

// Usage in Bootstrap
self::$repositories['articles'] = new ArticleRepository(self::$storage);
```

### Content Type System

```php
// src/Services/ContentTypeManager.php
class ContentTypeManager {
    public function loadContentTypes(): void {
        // Load YAML blueprints first
        $blueprints = $this->loadYamlBlueprints('site/blueprints/pages/');
        
        // Load PHP overrides second
        $overrides = $this->loadPhpOverrides('calaca/types/');
        
        // Merge and register
        $this->registerContentTypes($blueprints, $overrides);
    }
}

// YAML blueprint example (site/blueprints/pages/page.yml)
name: Page
fields:
  title:
    type: text
    required: true
  content:
    type: richtext
    required: false
```

### Content Type Validation

```php
// src/Models/FieldDefinition.php
class FieldDefinition {
    public function __construct(
        private string $name,
        private string $type,
        private bool $required = false,
        private array $options = []
    ) {}
    
    public function validate(mixed $value): array {
        $errors = [];
        if ($this->required && empty($value)) {
            $errors[] = 'This field is required';
        }
        return $errors;
    }
}
```

## Data Flow

### Content Rendering Flow

```
1. Request → web/index.php
2. Bootstrap::initialize() + Symfony Routing
3. Controller → Repository → Storage
4. TemplateService → render Twig template
5. Response with HTML
```

### Admin Content Management Flow

```
1. Request → /admin/articles
2. AuthMiddleware + CSRF validation
3. AdminArticleController → ArticleRepository
4. CRUD operations via StorageInterface
5. Twig → render admin template with Alpine.js
```

### Admin API Request Flow

```
1. Request → /admin/api/articles
2. AuthMiddleware (session-based)
3. Controller method (apiArticles, apiCategories, etc.)
4. Repository operations
5. JSON response with proper headers
```

## Configuration

### Environment Configuration

```bash
# .env (environment-specific)
APP_DEBUG=true
STORAGE_DRIVER=flatfile
CONTENT_PATH=content
SITE_NAME="My Website"
APP_URL=http://localhost:8000

# Database (optional)
DB_HOST=localhost
DB_DATABASE=calacacms
DB_USERNAME=user
DB_PASSWORD=secret
```

### Site Configuration

```json
// config/settings.json
{
  "site_title": "My Website",
  "site_url": "https://example.com",
  "theme": "default",
  "admin_per_page": 20,
  "cache_enabled": true
}
```

### Content Type Definition

```yaml
# site/blueprints/pages/page.yml
name: Page
fields:
  title:
    type: text
    label: Title
    required: true
  slug:
    type: slug
    label: URL Slug
    required: true
    source: title
  content:
    type: richtext
    label: Content
    required: false
  meta_description:
    type: textarea
    label: Meta Description
    required: false
config:
  title_field: title
  list_fields: [title, slug, updated_at]
  sort_field: created_at
  sort_direction: desc
```

## Directory Structure

```
calacacms/                          # Workspace root (= site root)
├── web/                            # Document root (public)
│   ├── index.php                   # Entry point (loads .env + autoloader)
│   ├── .htaccess                   # Apache rewrite rules
│   └── assets/                     # Vite build output (gitignored)
│       ├── admin/                  # Admin theme CSS/JS
│       └── default/                # Default theme CSS/JS
├── site/                           # Site customizations
│   ├── blueprints/pages/           # YAML content type definitions
│   ├── collections/                # Custom collections
│   └── controllers/                # Custom controllers
├── content/                        # Flat-file content (JSON)
├── config/                         # Site configuration
│   └── settings.json               # Site settings
├── themes/                         # Themes (owned by site builder)
│   ├── admin/                      # Admin theme (Vite + Tailwind + TS)
│   │   ├── assets/ts/              # TypeScript source
│   │   ├── assets/css/             # CSS with Tailwind
│   │   └── templates/              # Twig templates
│   └── default/                    # Default frontend theme
├── storage/                        # Cache, logs, uploads (gitignored)
│   ├── cache/
│   ├── logs/
│   └── uploads/
├── calaca/                         # CMS engine (= vendor/calacacms/core)
│   ├── src/                        # PHP source (namespace: CalacaCMS\)
│   │   ├── CLI/                    # CLI commands (theme:publish, etc.)
│   │   ├── Cache/                  # FileCache
│   │   ├── ContentTypes/           # BlueprintReader + interfaces
│   │   ├── Controllers/            # MVC Controllers (Admin/ + Frontend)
│   │   ├── Core/                   # Bootstrap and core services
│   │   ├── Database/               # Database abstraction layer
│   │   ├── Logger/                 # FileLogger
│   │   ├── Middleware/             # HTTP middleware (auth, CSRF)
│   │   ├── Models/                 # Data models + FieldDefinition
│   │   ├── Repositories/           # Data access layer
│   │   ├── Routing/                # Router service
│   │   ├── Services/               # Business logic services
│   │   ├── Storage/                # Storage drivers
│   │   └── Views/                  # TemplateService (Twig wrapper)
│   ├── stubs/themes/               # Reference theme copies (admin, default)
│   ├── tests/                      # PHPUnit tests (281 tests)
│   ├── bin/calaca                  # CLI entry point
│   └── composer.json               # Engine package (calacacms/core)
├── vendor/                         # Composer (calacacms/core → symlink)
├── composer.json                   # Site-level Composer (path repo)
├── package.json                    # Frontend scripts + deps (Bun)
├── .env                            # Environment config (gitignored)
├── .env.example                    # Environment template
├── docs/                           # Documentation
│   ├── PROJECT_SSOT.md             # ★ Single Source of Truth
│   └── 01-knowledge-base/          # Domain knowledge per topic
├── AGENTS.md                       # AI agent instructions
└── index.php                       # Deprecated shim → web/index.php
```

## Security Considerations

1. **CSRF Protection**: All state-changing admin actions require CSRF tokens via middleware
2. **Input Validation**: All user input validated through ContentType system and FieldDefinition
3. **Output Escaping**: Twig auto-escaping for XSS prevention
4. **Authentication**: Session-based with RateLimiter for brute force protection
5. **File Uploads**: FileUploadService with type restrictions, size limits, stored outside web root
6. **Environment Variables**: Sensitive config via .env, never committed to git
7. **SQL Injection Prevention**: Repository pattern with prepared statements in storage drivers

## Performance Optimizations

1. **Caching**: FileCache for content and templates in `/storage/cache`
2. **Lazy Loading**: Content types loaded on demand via ContentTypeManager
3. **Storage Abstraction**: Efficient queries per driver (FlatFile, SQLite, MySQL, PostgreSQL)
4. **Minimal Dependencies**: Core has few external dependencies (Symfony Routing, League Container)
5. **Vite Build System**: Fast asset compilation and hot module replacement
6. **Repository Pattern**: Efficient data access with caching layer

## Extension Points

1. **Content Types**: Add new field types via FieldDefinition classes
2. **Storage Drivers**: Implement StorageInterface for new backends
3. **Controllers**: Extend BaseController or BaseAdminController for custom functionality
4. **Themes**: CSS variables + Twig templates for custom admin themes
5. **Repositories**: Extend RepositoryInterface for custom data access
6. **Middleware**: Add custom middleware for authentication, logging, etc.
7. **CLI Commands**: Add commands to `calaca/src/CLI/` directory

## Resources

- [PROJECT_SSOT.md](../../PROJECT_SSOT.md) — Single Source of Truth
- [AGENTS.md](../../AGENTS.md) — AI agent instructions
- [01-knowledge-base/](../) — Domain knowledge per topic
- [PHPStan Analysis](../../../calaca/tests/) — Testing documentation
- [CLI Commands](../../../calaca/bin/calaca) — Theme scaffolding commands
