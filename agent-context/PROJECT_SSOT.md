# CalacaCMS - Project Single Source of Truth (SSOT)

**Document Version:** 2.2  
**Last Updated:** 2026-03-22  
**Project:** CalacaCMS - A site-first PHP CMS

---

## 1. Project Overview

CalacaCMS is a **site-first PHP CMS** that adapts to your website, not the other way around. Built for developers who want flexibility without complexity.

### Core Philosophy
- **Site-first**: Start with your HTML/CSS, then add CMS capabilities
- **Flat-file first**: No database required, content lives in simple files
- **Progressive enhancement**: Start simple, add features when needed

---

## 2. Technology Stack

### Backend
- **PHP**: >=8.3
- **Template Engine**: Twig ^3.0
- **Dependency Injection**: League Container ^4.0
- **Routing**: Symfony Routing ^7.0
- **HTTP**: Symfony HttpFoundation ^7.0

### Frontend (Admin)
- **Build Tool**: Vite ^8.0.0
- **CSS Framework**: Tailwind CSS ^4.2.1
- **Language**: TypeScript ^5.0.0
- **JS Framework**: Alpine.js
- **Icons**: Lucide

### Development Tools
- **Testing**: PHPUnit ^11.5
- **Static Analysis**: PHPStan level 5
- **Code Style**: PHP-CS-Fixer ^3.94
- **Coding Standard**: Slevomat Coding Standard ^8.0

### Package Manager
- **PHP**: Composer
- **JavaScript**: Bun (not npm)

---

## 3. Project Structure

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
├── themes/                         # See themes/README.md in cms/
│   ├── admin/                      # Admin (control panel) theme
│   │   ├── assets/ts/              # TypeScript (Vite entry: admin-entry.ts; no barrel index.ts)
│   │   ├── assets/css/             # CSS with Tailwind
│   │   └── templates/              # Twig templates
│   └── site/                       # Public front-end themes
│       └── default/                # Bundled default site theme
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
│   ├── stubs/themes/               # Reference theme copies (admin, site)
│   ├── tests/                      # PHPUnit tests (281 tests)
│   ├── bin/calaca                  # CLI entry point
│   └── composer.json               # Engine package (calacacms/core)
├── vendor/                         # Composer (calacacms/core → symlink)
├── composer.json                   # Site-level Composer (path repo)
├── package.json                    # Frontend scripts + deps (Bun)
├── .env                            # Environment config (gitignored)
├── .env.example                    # Environment template
├── marketing/                      # Landing page (Astro static site)
├── comingsoon/                     # Coming soon page (static HTML)
├── agent-context/                  # SSOT + knowledge base (not the Starlight site)
│   ├── PROJECT_SSOT.md             # ★ This document
│   └── 01-knowledge-base/          # Domain knowledge per topic
├── AGENTS.md                       # AI agent instructions
└── index.php                       # Deprecated shim → web/index.php
```

---

## 4. Architecture Patterns

### Design Patterns
- **Repository Pattern**: Data access abstraction
- **Dependency Injection**: League Container
- **Factory Pattern**: StorageFactory, RepositoryGenerator
- **Strategy Pattern**: Storage drivers (FlatFile, SQLite, MySQL, PostgreSQL)

### Key Principles
1. **SSOT**: This document is the authoritative source
2. **Fail fast**: Surface errors early
3. **User-friendly errors**: Clear, actionable messages
4. **Bun-first**: Always use Bun, never npm

---

## 5. Development Standards

### Code Style
- Strict TypeScript (for frontend)
- PHP 8.3+ features (named arguments, union types, readonly properties)
- PSR-12 coding standard via PHP-CS-Fixer
- No barrel files, use explicit imports

### Error Handling
- Fail fast with clear messages
- Never expose internals or stack traces to users
- Log with context, never log secrets or PII
- Validate config at startup

### Testing
- PHPUnit for unit tests
- Target: 80%+ coverage
- Keep tests close to code they cover

---

## 6. Git Workflow

### Branching Model: Gitflow

**Permanent Branches:**
| Branch | Purpose | Direct Push? |
|--------|---------|--------------|
| `main` | Production code, stable | No - via PR |
| `develop` | Integration branch | Yes |

**Temporary Branches:**
| Prefix | From | Merges To | When |
|--------|------|-----------|------|
| `feature/` | `develop` | `develop` | New functionality |
| `bugfix/` | `develop` | `develop` | Non-urgent bugfix |
| `release/` | `develop` | `main` + `develop` | Prepare release |
| `hotfix/` | `main` | `main` + `develop` | Urgent production fix |

### Daily Workflow
```bash
# Feature branch
git flow feature start description-of-feature
# ... work ...
git flow feature finish description-of-feature
git push origin develop

# Release
git flow release start 1.2.0
git flow release finish 1.2.0
git push origin main
git push origin develop
git push origin --tags
```

### Commit Messages
**Conventional Commits**: `type(scope): subject`

| Type | When |
|------|------|
| `feat` | New functionality |
| `fix` | Bugfix |
| `refactor` | Restructure code without behavior change |
| `perf` | Performance improvement |
| `chore` | Build, tooling, dependencies |
| `docs` | Documentation only |
| `test` | Add or modify tests |
| `ci` | CI/CD configuration |
| `style` | Formatting (no logic) |

---

## 7. Key Commands

### PHP (run from `calaca/` directory)
```bash
composer test                       # Run PHPUnit (281 tests)
composer lint                       # Check code style (dry-run)
composer fix                        # Auto-fix code style
composer analyse                    # Run PHPStan
```

### Frontend (run from project root)
```bash
bun install                         # Install dependencies
bun run start                       # Full dev: Vite + PHP + reverse proxy (single URL, default :8080 — roadmap R4)
bun run dev                         # Vite dev (admin + default) only
bun run dev:admin                   # Admin theme Vite (127.0.0.1:5173, base /__vite/admin/)
bun run dev:default                 # Default theme Vite (127.0.0.1:5174, base /__vite/default/)
bun run dev:proxy                   # Proxy only (forwards /__vite/* to Vite, rest to PHP)
bun run build                       # Build all themes → web/assets/
bun run build:admin                 # Build admin only
bun run serve                       # PHP on 127.0.0.1:8000 (web/ docroot; use with proxy for one browser URL)
```

**Bundle analysis (production build):** each `vite build` writes an interactive treemap via `rollup-plugin-visualizer`. Open in a browser after building:

- `themes/admin/bundle-stats.html` — admin JS/CSS dependency tree and sizes (gzip/brotli estimates)
- `themes/site/default/bundle-stats.html` — default site theme bundle

These HTML files live next to each theme (not under `web/assets/`) and are listed in `.gitignore`.

### CLI
```bash
./calaca/bin/calaca cache:clear        # Clear cache
./calaca/bin/calaca cache:warm         # Warm cache
./calaca/bin/calaca content-type:list  # List content types
./calaca/bin/calaca theme:list         # List available theme stubs
./calaca/bin/calaca theme:publish admin # Publish admin theme from stubs
./calaca/bin/calaca theme:publish site  # Publish public site themes (includes default/)
./calaca/bin/calaca theme:diff admin   # Show diff between theme and stubs
```

---

## 8. Content Types

Content types are defined as YAML blueprints in `site/blueprints/pages/*.yml`.
PHP overrides can be placed in `calaca/types/*.php`.

Supported field types:
- `text` - Single-line text
- `textarea` - Multi-line text
- `richtext` - WYSIWYG editor
- `number` - Numeric values
- `date` - Date picker
- `boolean` - Checkbox/toggle
- `select` - Dropdown selection
- `file` - File upload

---

## 9. Storage Drivers

| Driver | Status | Use Case |
|--------|--------|----------|
| FlatFile | ✅ Ready | Small sites, no DB needed |
| SQLite | ✅ Ready | Medium sites, embedded DB |
| MySQL | ✅ Ready | Large sites, production |
| PostgreSQL | ✅ Ready | Enterprise, complex needs |

---

## 10. Security Checklist

- [ ] No secrets/API keys in code
- [ ] Input validation on all user input
- [ ] Output escaping (Twig auto-escaping)
- [ ] CSRF protection on forms
- [ ] Secure file upload handling
- [ ] SQL injection prevention (prepared statements)
- [ ] XSS prevention
- [ ] Security headers configured

---

## 11. Related Documents

| Document | Purpose |
|----------|---------|
| `PRD.md` | Product Requirements Document |
| `REVIEW_FINDINGS.md` | Code review findings |
| `technical/cache-logger.md` | Cache & logger docs |
| `technical/storage-drivers.md` | Storage driver docs |
| `technical/testing.md` | Testing documentation |
| `technical/security-headers.md` | Security configuration |
| `01-knowledge-base/` | Domain knowledge per topic |

---

**Important:** Always read this document first before taking any actions on the project. This is the Single Source of Truth.
