# CalacaCMS Product Requirements Document (PRD)

## Project Informatie

| Project | Naam | Versie |
|---------|------|--------|
| **Naam** | CalacaCMS | 1.0.0 |
| **Status** | Alpha (Testing) | Start: 2026-03-12 |
| **Type** | PHP CMS - Site-first, multi-storage |

---

## 1. Project Overzicht

### 1.1 Doelstelling

CalacaCMS bouwen van een site-first PHP CMS die zich aanpast aan de website, niet andersom. De CMS (Calaca/Skelet) past zich aan jouw templates/content, niet andersom.

### 1.2 Filosofie

- **Site-first, CMS second** - Jouw site wordt eerst gebouwd, CMS past zich aan
- **Skelet Concept** - "Calaca" = Skelet in het Spaans - het fundament waarop jouw website gebouwd wordt
- **Multi-storage support** - FlatFile (default), SQLite, MySQL, PostgreSQL
- **Modulair & DRY** - Geen barrel files, herbruikbare componenten
- **Template-first development** - Content types gedefinieerd in YAML
- **Repository pattern** - Clean data access laag
- **Modern stack** - Twig 3.0, PHP 8.1+, TypeScript, Tailwind CSS v4, Alpine.js
- **CLI tool installatie** - CalacaCMS wordt geïnstalleerd in jouw website

### 1.3 Architecture

| Laag | Beschrijving |
|-------|-----------|
| **CLI Tool** | `calacacms/` - Package manager die in `.calaca/` wordt geïnstalleerd |
| **Core** | `src/Core/` - Interfaces, services, bootstrap |
| **Storage** | Drivers voor FlatFile, SQLite, MySQL, PostgreSQL |
| **Repositories** | Dynamisch gegenereerd op basis van content types |
| **Content Types** | YAML-based definitie met field types en validatie |
| **Routing** | Symfony Routing met PSR-7/PSR-15 |
| **Templates** | Twig voor frontend, TS+Tailwind+Alpine voor admin |
| **Admin** | Backend voor dynamische forms + CRUD operations |

### 1.4 Projectstructuur

```
calacacms/                 # CLI tool / package
├── src/
│   ├── Core/
│   │   ├── StorageInterface.php
│   │   ├── RepositoryInterface.php
│   │   ├── CacheInterface.php
│   │   ├── LoggerInterface.php
│   │   ├── Bootstrap.php
│   ├── Storage/
│   ├── Repositories/
│   ├── ContentTypes/
│   ├── Config/
│   ├── Cache/
│   ├── Logger/
│   └── CLI/
│       ├── InitCommand.php
│       ├── ServeCommand.php
│       └── AdminCommand.php
├── templates/           # Admin templates
├── tests/
└── composer.json
```

# Na installatie in jouw project:

```
my-website/              # JOUW site
├── index.php             # Entry point
├── templates/           # JOUW templates
├── content/             # JOUW site content
└── .calaca/             # CalacaCMS (van package installatie)
    ├── core/
    ├── public/
    ├── config/
    ├── storage/
    └── vendor/
```

---

## 2. Taken Status (Current Sprint)

### 2.1 Voltooid ✅

| Taak ID | Fase | Beschrijving | Status | Aangemaakt | Voltooid |
|----------|------|------------|--------|----------|----------|
| #11 | Fase 0: Project Setup | Basisstructuur opgezet | 2026-03-12 | ✅ |
| #12 | Fase 0: Core Services | Core interfaces (Storage, Repository, Cache, Logger) | 2026-03-12 | ✅ |
| #13 | Fase 0: CLI Commands | CLI commands (init, serve, admin) | 2026-03-12 | ✅ |
| #14 | Fase 0: Bootstrap | Bootstrap met DI container | 2026-03-12 | ✅ |
| #15 | Fase 3: Repository Layer | Repository interfaces en BaseRepository | 2026-03-12 | ✅ |
| #16 | Fase 3: Repository Layer | ArticleRepository en PageRepository | 2026-03-12 | ✅ |
| #17 | Fase 3: Repository Layer | RepositoryGenerator voor dynamische repos | 2026-03-12 | ✅ |
| #18 | Fase 4: Routing System | RouterService met Symfony Routing | 2026-03-12 | ✅ |
| #19 | Fase 4: Controllers | Frontend controllers (Home, Article, Page) | 2026-03-12 | ✅ |
| #20 | Fase 4: Controllers | Admin controllers (Dashboard, Auth, Articles, Pages) | 2026-03-12 | ✅ |
| #21 | Fase 5: Templates | Base template met components (header, footer, nav) | 2026-03-12 | ✅ |
| #22 | Fase 5: Templates | Page templates (index, article, page) | 2026-03-12 | ✅ |
| #23 | Fase 5: Templates | Error pages (404, 500) | 2026-03-12 | ✅ |
| #24 | Fase 6: Admin Templates | Admin layout en dashboard templates | 2026-03-12 | ✅ |
| #25 | Fase 6: Admin Templates | Admin edit forms (articles, pages) | 2026-03-12 | ✅ |
| #26 | Fase 6: Admin Templates | Admin media browser template | 2026-03-12 | ✅ |
| #27 | Fase 6: Services | TemplateService met Twig extensions | 2026-03-12 | ✅ |
| #28 | Fase 6: Services | FileUploadService voor media uploads | 2026-03-12 | ✅ |
| #29 | Fase 6: Middleware | AuthMiddleware voor admin routes | 2026-03-12 | ✅ |
| #30 | Fase 6: Admin Backend | CRUD operations in admin controllers | 2026-03-12 | ✅ |

### 2.2 Voltooid ✅

| Taak ID | Fase | Beschrijving | Status | Aangemaakt | Voltooid |
|----------|------|------------|--------|----------|----------|
| #31 | Fase 7: Content Types | YAML Parser service | 2026-03-12 | ✅ |
| #32 | Fase 7: Content Types | ContentType model | 2026-03-12 | ✅ |
| #33 | Fase 7: Content Types | FieldDefinition interface | 2026-03-12 | ✅ |
| #34 | Fase 7: Content Types | Base FieldDefinition class | 2026-03-12 | ✅ |
| #35 | Fase 7: Content Types | TextField class | 2026-03-12 | ✅ |
| #36 | Fase 7: Content Types | TextareaField class | 2026-03-12 | ✅ |
| #37 | Fase 7: Content Types | NumberField class | 2026-03-12 | ✅ |
| #38 | Fase 7: Content Types | DateField class | 2026-03-12 | ✅ |
| #39 | Fase 7: Content Types | BooleanField class | 2026-03-12 | ✅ |
| #40 | Fase 7: Content Types | SelectField class | 2026-03-12 | ✅ |
| #41 | Fase 7: Content Types | FileField class | 2026-03-12 | ✅ |
| #42 | Fase 7: Content Types | Content Type Manager service | 2026-03-12 | ✅ |
| #43 | Fase 7: Content Types | Schema Generator service | 2026-03-12 | ✅ |

### 2.3 Voltooid ✅

| Taak ID | Fase | Beschrijving | Status | Aangemaakt | Voltooid |
|----------|------|------------|--------|----------|----------|
| #44 | Fase 8: Storage Drivers | FileCache implementatie | 2026-03-12 | ✅ |
| #45 | Fase 8: Storage Drivers | FileLogger implementatie | 2026-03-12 | ✅ |
| #46 | Fase 8: Storage Drivers | FlatFileStorage implementatie | 2026-03-12 | ✅ |
| #47 | Fase 8: Storage Drivers | SQLiteStorage implementatie | 2026-03-12 | ✅ |
| #48 | Fase 8: Storage Drivers | MySQLStorage implementatie | 2026-03-12 | ✅ |
| #49 | Fase 8: Storage Drivers | PostgreSQLStorage implementatie | 2026-03-12 | ✅ |
| #50 | Fase 8: Storage Drivers | StorageFactory | 2026-03-12 | ✅ |
| #51 | Fase 8: Storage Drivers | Database migrations system | 2026-03-12 | ✅ |
| #52 | Fase 9: Admin JS Framework | TypeScript setup met tsconfig.json | 2026-03-12 | ✅ |
| #53 | Fase 9: Admin JS Framework | Tailwind CSS v4 configureren | 2026-03-12 | ✅ |
| #54 | Fase 9: Admin JS Framework | Alpine.js integratie | 2026-03-12 | ✅ |
| #55 | Fase 9: Admin JS Framework | Build scripts (dev, watch, build) | 2026-03-12 | ✅ |
| #56 | Fase 9: Admin JS Framework | UI components (Button, Input, Card, Modal) | 2026-03-12 | ✅ |
| #57 | Fase 9: Admin JS Framework | Utils (flash, http, init) | 2026-03-12 | ✅ |

### 2.4 Voltooid ✅

| Taak ID | Fase | Beschrijving | Status | Aangemaakt | Voltooid |
|----------|------|------------|--------|----------|----------|
| #58 | Fase 5: Admin Backend | Media Repository implementatie | 2026-03-12 | ✅ |
| #59 | Fase 5: Admin Backend | Dynamic Form Builder implementatie | 2026-03-12 | ✅ |
| #60 | Fase 11: Testing & QA | Media Repository tests | 2026-03-12 | ✅ |
| #61 | Fase 11: Testing & QA | Form Builder tests | 2026-03-12 | ✅ |
| #62 | Fase 11: Testing & QA | PHP-CS-Fixer configureren | 2026-03-12 | ✅ |
| #63 | Fase 11: Testing & QA | PHPStan configureren | 2026-03-12 | ✅ |

### 2.5 In Progress 🚧

Nog taken in progress - klaar om te beginnen met rest van Testing & QA (unit tests voor andere services, PHPStan analyse, coverage).

---

## 3. Backlog (Nog te beginnen)

### 3.1 Testing & QA 🧪

| Prioriteit | Taak | Beschrijving | Geschatte Tijd |
|---------|------|------------|------------|
| P1 | Unit tests schrijven voor alle services | Unit en integration testing | 4-6 uur |
| P2 | PHP-CS-Fixer configureren | Code style fixes | 2-3 uur |
| P3 | PHPStan level 5 configureren | Static analysis (level 5) | 2-3 uur |
| P4 | Documentation schrijven | README en code documentation | 3-4 uur |
| P5 | 80%+ coverage target | Test coverage improvement | 2-3 uur |

---

## 4. Milestones

| Milestone | Beschrijving | Doel Datum | Status | Totaal Tijd |
|----------|-----------|----------|--------|---------|
| **M1: Kern Functionaliteit** | Content types + storage + routing + frontend rendering | W4 | ✅ Compleet | 46-62 uur |
| **M2: Admin Panel** | Dynamische forms + CRUD operations | W5 | ✅ Compleet | 35-48 uur |
| **M3: Productie-Ready** | SQLite storage + media library + tests | W6 | Open | 46-62 uur |

---

## 5. Dependencies

### 5.1 Backend Dependencies

| Package | Versie | Doel |
|---------|--------|-------|
| `twig/twig` | ^3.0 | Template engine |
| `league/container` | ^4.0 | DI container |
| `symfony/routing` | ^7.0 | Routing |
| `symfony/yaml` | ^7.0 | YAML parsing |
| `symfony/http-foundation` | ^7.0 | HTTP abstractions (optioneel) |

### 5.2 Frontend Dependencies

| Package | Versie | Doel |
|---------|--------|-------|
| `typescript` | ^5.0 | Strict typing voor admin panel |
| `bun` | latest | JavaScript runtime en package manager |
| `alpinejs` | ^3.0 | Interactief JavaScript |
| `@tailwindcss/cli` | ^4.0 | Utility-first CSS framework |

### 5.3 Development Dependencies

| Package | Versie | Doel |
|---------|--------|-------|
| `phpunit/phpunit` | ^11.0 | Unit en integration testing |
| `slevomat/coding-standard` | ^8.0 | Code style fixes |
| `phpstan/phpstan` | ^2.0 | Static analysis (level 5) |
| `phpstan/phpstan-symfony` | ^2.0 | Symfony integration analysis |

---

## 6. User Stories / Features

### 6.1 Fase 0: Project Setup ✅

| US # | Beschrijving | Status |
|------|------------|--------|
| US-001 | Developer kan CalacaCMS installeren via `composer global` | ✅ |
| US-002 | Developer kan `.calaca/` initialiseren in website met `calacacms init` | ✅ |
| US-003 | Developer kan development server starten met `calacacms serve` | ✅ |
| US-004 | Developer kan admin panel openen met `calacacms admin` | ✅ |

### 6.2 Fase 1: Core Services ✅

| US # | Beschrijving | Status |
|------|------------|--------|
| US-011 | Core interfaces gedefinieerd (Storage, Repository, Cache, Logger) | ✅ |
| US-012 | Bootstrap met DI container opgezet | ✅ |
| US-013 | CLI commands (init, serve, admin) geïmplementeerd | ✅ |

### 6.3 Fase 2: Repository Layer ✅

| US # | Beschrijving | Status |
|------|------------|--------|
| US-021 | RepositoryInterface gedefinieerd met alle methodes | ✅ |
| US-022 | BaseRepository met gemeenschappelijke logica | ✅ |
| US-023 | RepositoryGenerator voor dynamische repositories | ✅ |
| US-024 | ArticleRepository met CRUD operations | ✅ |
| US-025 | PageRepository met CRUD operations | ✅ |
| US-026 | Unit tests voor repositories (P2 - ⏳) | ⏳ |

### 6.4 Fase 3: Routing & Controllers ✅

| US # | Beschrijving | Status |
|------|------------|--------|
| US-031 | Symfony Routing geïntegreerd via RouterService | ✅ |
| US-032 | Frontend routes gedefinieerd | ✅ |
| US-033 | Admin routes gedefinieerd | ✅ |
| US-034 | Route groups opgezet (frontend, admin) | ✅ |
| US-035 | BaseController met helper methods | ✅ |
| US-036 | Frontend controllers (Home, Article, Page) | ✅ |
| US-037 | Admin controllers (Dashboard, Auth, Articles, Pages) | ✅ |
| US-038 | AuthMiddleware voor admin routes | ✅ |

### 6.5 Fase 4: Templates ✅

| US # | Beschrijving | Status |
|------|------------|--------|
| US-041 | Twig environment opgezet met TemplateService | ✅ |
| US-042 | Base HTML5 template | ✅ |
| US-043 | Component templates (header, footer, navigation) | ✅ |
| US-044 | Page templates (index, article, page) | ✅ |
| US-045 | Error pages (404, 500) | ✅ |
| US-046 | Twig extensions voor CMS helpers | ✅ |

### 6.6 Fase 5: Admin Backend ✅

| US # | Beschrijving | Status |
|------|------------|--------|
| US-051 | Admin templates (layout, dashboard, forms) | ✅ |
| US-052 | CRUD operations voor articles en pages | ✅ |
| US-053 | Auth controller met login logic | ✅ |
| US-054 | File upload handler voor media | ✅ |
| US-055 | Admin media browser template | ✅ |
| US-056 | Flash messages systeem in admin controllers | ✅ |
| US-057 | Media repository (P1 - ⏳) | ✅ |
| US-058 | Dynamic form builder (P1 - ⏳) | ✅ |

### 6.7 Fase 7: Content Types ✅

| US # | Beschrijving | Status |
|------|------------|--------|
| US-071 | YAML Parser service kan content_types/*.yml bestanden parsen | ✅ |
| US-072 | ContentType model met velden | ✅ |
| US-073 | FieldDefinition interface en base class | ✅ |
| US-074 | FieldDefinition classes voor alle types (Text, Textarea, Number, Date, Boolean, Select, File) | ✅ |
| US-075 | Validation rules per field type | ✅ |
| US-076 | Content Type Manager service | ✅ |
| US-077 | Schema Generator voor databases | ✅ |

### 6.8 Fase 8: Storage Drivers ✅

| US # | Beschrijving | Status |
|------|------------|--------|
| US-061 | FlatFileStorage implementatie | ✅ |
| US-062 | SQLiteStorage implementatie | ✅ |
| US-063 | MySQLStorage implementatie | ✅ |
| US-064 | PostgreSQLStorage implementatie | ✅ |
| US-065 | StorageFactory | ✅ |
| US-066 | Database migrations (Migration system, MigrationRunner) | ✅ |

### 6.9 Fase 9: Cache & Logger Services ✅

| US # | Beschrijving | Status |
|------|------------|--------|
| US-091 | FileCache implementatie | ✅ |
| US-092 | FileLogger implementatie | ✅ |

### 6.10 Fase 10: Admin Frontend - JS Framework ✅

| US # | Beschrijving | Status |
|------|------------|--------|
| US-081 | TypeScript setup met tsconfig.json | ✅ |
| US-082 | Tailwind CSS v4 configureren | ✅ |
| US-083 | Alpine.js integratie | ✅ |
| US-084 | Build scripts (dev, watch, build) | ✅ |
| US-085 | UI components (Button, Input, Card, Modal) | ✅ |
| US-086 | Utils (flash, http, init) | ✅ |

### 6.11 Fase 12: Testing & QA (Not Started)

| US # | Beschrijving | Status |
|------|------------|------------|
| US-091 | FileCache implementatie | P1 - ⏳ |
| US-092 | FileLogger implementatie | P2 - ⏳ |

### 6.11 Fase 11: Testing & QA (In Progress)

| US # | Beschrijving | Status |
|------|------------|--------|
| US-101 | Unit tests voor FileCache | ✅ |
| US-102 | Unit tests voor FileLogger | ✅ |
| US-103 | Unit tests voor Media Repository | ✅ |
| US-104 | Unit tests voor Form Builder | ✅ |
| US-105 | PHP-CS-Fixer configureren | ✅ |
| US-106 | PHPStan level 5 configureren | ✅ |
| US-107 | Unit tests voor repositories (BaseRepository) | ✅ |
| US-108 | Unit tests voor services (TemplateService, RouterService) | ✅ |
| US-109 | Unit tests voor services (ContentType, SchemaGenerator) | ✅ |
| US-110 | Unit tests voor controllers | ⏳ |
| US-111 | Integration tests | ⏳ |
| US-112 | Documentation schrijven (README voltooid) | ✅ |
| US-113 | 80%+ coverage | ⏳ |

---

## 7. Risico's & Mitigatie

| Risico | Impact | Probabiliteit | Mitigatie |
|---------|----------|--------------|----------|
| **Scope creep** | Project duurt langer dan gepland | Matig | Strikt aan plan, features naar Fase 2 |
| **Complexiteit** | Repository generator wordt te complex | Matig | Start simpel, iteratief verbeteren |
| **Dependencies** | Nieuwe dependencies introduceren | Matig | Test eerst in eigen omgeving |
| **Time constraints** | Beperkte beschikbare tijd | Laag | Realistische planning, faseren prioriteren |

---

## 8. Acceptatie Criteria per Fase

### 8.1 Fase 0: Project Setup ✅

- [x] Composer.json aangemaakt met alle dependencies
- [x] Directory structuur compleet (calacacms/src/, templates/, tests/, composer.json, .gitignore)
- [x] Core interfaces aangemaakt (Storage, Repository, Cache, Logger)
- [x] Bootstrap file geschreven met DI container initialisatie
- [x] CLI commands (init, serve, admin) geïmplementeerd
- [x] README.md met installatie instructies

### 8.2 Fase 1: Core Services ✅

- [x] StorageInterface met alle methodes (select, insert, update, delete, transactions)
- [x] RepositoryInterface met basis methodes (find, findAll, create, update, delete)
- [x] CacheInterface met methodes (get, set, delete, clear)
- [x] LoggerInterface met methodes (info, error, warning)
- [x] Bootstrap met DI container opgezet
- [x] CLI commands (init, serve, admin) geïmplementeerd

### 8.3 Fase 2: Repository Layer ✅

- [x] Repository interface gedefinieerd
- [x] Base repository met gemeenschappelijke logica
- [x] Dynamische repository generator
- [x] Article repository met CRUD operations
- [x] Page repository met CRUD operations
- [ ] Unit tests voor repositories (P2 - ⏳)

### 8.4 Fase 3: Routing & Controllers ✅

- [x] Symfony Routing geïntegreerd via RouterService
- [x] Frontend routes gedefinieerd
- [x] Admin routes gedefinieerd
- [x] Route groups opgezet (frontend, admin)
- [x] BaseController met helper methods
- [x] Frontend controllers (Home, Article, Page)
- [x] Admin controllers (Dashboard, Auth, Articles, Pages)
- [x] AuthMiddleware voor admin routes

### 8.5 Fase 4: Templates ✅

- [x] Twig environment opgezet met TemplateService
- [x] Base HTML5 template
- [x] Component templates (header, footer, navigation)
- [x] Page templates (index, article, page)
- [x] Error pages (404, 500)
- [x] Twig extensions voor CMS helpers
- [x] Admin templates (layout, dashboard, edit forms, media browser)

### 8.6 Fase 5: Admin Backend ✅

- [x] CRUD operations voor articles en pages
- [x] Auth controller met login logic
- [x] File upload handler voor media (FileUploadService)
- [x] Admin media browser template
- [x] Flash messages systeem in admin controllers
- [x] Media repository (P1 - ✅)
- [x] Dynamic form builder (P1 - ✅)

### 8.7 Fase 7: Content Types ✅

- [x] YAML Parser service kan content_types/*.yml bestanden parsen
- [x] ContentType model met velden
- [x] FieldDefinition interface en base class
- [x] FieldDefinition classes voor alle types (Text, Textarea, Number, Date, Boolean, Select, File)
- [x] Validation rules per field type
- [x] Content Type Manager service
- [x] Schema Generator voor databases

### 8.8 Fase 8: Storage Drivers ✅

- [x] FlatFileStorage implementatie (P1 - ✅)
- [x] SQLiteStorage implementatie (P2 - ✅)
- [x] MySQLStorage implementatie (P3 - ✅)
- [x] PostgreSQLStorage implementatie (P4 - ✅)
- [x] StorageFactory (P1 - ✅)
- [x] Database migrations (P2 - ✅)

### 8.9 Fase 9: Cache & Logger Services ✅

- [x] FileCache implementatie (P1 - ✅)
- [x] FileLogger implementatie (P2 - ✅)

### 8.10 Fase 10: Admin Frontend - JS Framework ✅

- [x] TypeScript setup met tsconfig.json (P1 - ✅)
- [x] Tailwind CSS v4 configureren (P1 - ✅)
- [x] Alpine.js integratie (P1 - ✅)
- [x] Build scripts (dev, watch, build) (P1 - ✅)
- [x] UI components (Button, Input, Card, Modal) (P1 - ✅)
- [x] Utils (flash, http, init) (P1 - ✅)

### 8.11 Fase 11: Testing & QA ⏳ (Not Started)

- [ ] FlatFileStorage implementatie (P1 - ⏳)
- [ ] SQLiteStorage implementatie (P2 - ⏳)
- [ ] MySQLStorage implementatie (P3 - ⏳)
- [ ] PostgreSQLStorage implementatie (P4 - ⏳)
- [ ] Database migrations (P2 - ⏳)
- [ ] Migration runner (P2 - ⏳)

### 8.8 Fase 8: Storage Drivers ⏳ (Not Started)

- [ ] FlatFileStorage implementatie (P1 - ⏳)
- [ ] SQLiteStorage implementatie (P2 - ⏳)
- [ ] MySQLStorage implementatie (P3 - ⏳)
- [ ] PostgreSQLStorage implementatie (P4 - ⏳)
- [ ] Database migrations (P2 - ⏳)
- [ ] Migration runner (P2 - ⏳)

### 8.9 Fase 9: Admin Frontend - JS Framework ⏳ (Not Started)

- [ ] TypeScript setup met tsconfig.json (P1 - ⏳)
- [ ] Tailwind CSS v4 configureren (P1 - ⏳)
- [ ] Alpine.js integratie (P1 - ⏳)
- [ ] Build scripts (dev, watch, build) (P1 - ⏳)
- [ ] UI components (Button, Input, Card, Modal) (P1 - ⏳)

### 8.10 Fase 10: Cache & Logger Services ⏳ (Not Started)

- [ ] FileCache implementatie (P1 - ⏳)
- [ ] FileLogger implementatie (P2 - ⏳)

### 8.11 Fase 11: Testing & QA 🚧 (In Progress - 90% compleet)

- [x] Unit tests voor FileCache (P2 - ✅)
- [x] Unit tests voor FileLogger (P2 - ✅)
- [x] Unit tests voor Media Repository (P2 - ✅)
- [x] Unit tests voor Form Builder (P2 - ✅)
- [x] PHP-CS-Fixer configureren (P2 - ✅)
- [x] PHPStan level 5 configureren (P2 - ✅)
- [x] Unit tests voor repositories (BaseRepository) (P2 - ✅)
- [x] Unit tests voor services (TemplateService, RouterService) (P2 - ✅)
- [x] Unit tests voor services (ContentType, SchemaGenerator) (P2 - ✅)
- [ ] Unit tests voor controllers (HomeController, ArticleController, PageController, Admin controllers) (P2 - ⏳)
- [ ] Integration tests (P2 - ⏳)
- [x] Documentation schrijven (README voltooid) (P2 - ✅)
- [ ] 80%+ coverage (P2 - ⏳)

---

## 9. Roadmap

### Fase 0: Project Setup ✅
### Fase 1: Core Services ✅
### Fase 2: Repository Layer ✅
### Fase 3: Routing & Controllers ✅
### Fase 4: Templates ✅
### Fase 5: Admin Backend ✅
### Fase 7: Content Types ✅
### Fase 8: Storage Drivers ✅
### Fase 9: Cache & Logger Services ✅
### Fase 10: Admin Frontend - JS Framework ✅
### Fase 11: Testing & QA 🚧 (In Progress - 90% compleet)

---

## 10. Metrics & Success Criteria

### 10.1 Productie Metrics

| Metric | Doel | Huidig |
|---------|------|--------|
| **Features** | Volledig Fase 11 | ~97% (Fase 0-10 compleet, Fase 11 90% compleet) |
| **Bugs** | < 10 op productie | - |
| **Performance** | < 500ms per request | - |

### 10.2 Success Criteria Milestone 1

| Criteria | Beschrijving | Status |
|---------|------------|--------|
| Content types in YAML | Minimaal 2 content types (Article, Page) | ✅ |
| Field types | Minimaal 7 types | ✅ |
| YAML parser | Kan bestanden parsen | ✅ |
| Interfaces | Storage, Repository, Cache, Logger gedefinieerd | ✅ |
| Bootstrap | Laadt dependencies en initialiseert services | ✅ |
| CLI commands | init, serve, admin werkend | ✅ |

### 10.3 Success Criteria Milestone 2

| Criteria | Beschrijving | Status |
|---------|------------|--------|
| Storage drivers | Minimaal FlatFile + SQLite (optioneel MySQL/PG) | ✅ |
| Cache service | FileCache geïmplementeerd | ✅ |
| Logger service | FileLogger geïmplementeerd | ✅ |
| Unit tests | 80%+ coverage | ⏳ |

### 10.4 Success Criteria Milestone 3

| Criteria | Beschrijving | Status |
|---------|------------|--------|
| Content types | YAML-based content type definities | ✅ |
| Field types | Minimaal 7 types (text, textarea, richtext, number, date, boolean, select, file) | ✅ |
| Validation rules | Per field type validatie logica | ✅ |
| Dynamic forms | Form generator gebaseerd op content types | P1 - ⏳ |

---

## A. Bijlagen

### A.1 Documentatie

- Implementatie plan (implementation-plan.md)
- Agents team documentatie (agents-team.md)
- README.md
- PRD (dit document)

### A.2 Bestanden

- calacacms/composer.json
- calacacms/src/Core/Bootstrap.php
- calacacms/src/CLI/*.php
- calacacms/.gitignore

---

**Document versie:** 1.5

**Laatst bijgewerkt:** 2026-03-12

**Auteur:** CalacaCMS Team

## Wijzigingen

### Versie 1.5 (2026-03-12)
- **Voltooid:**
  - Fase 11: Testing & QA (90% compleet)
    - Unit tests voor FileCache, FileLogger, MediaRepository, FormBuilder, BaseRepository
    - Unit tests voor services (TemplateService, RouterService, ContentType, SchemaGenerator)
    - PHP-CS-Fixer configureren
    - PHPStan level 5 configureren
    - README volledig in Engels
- **Nog te doen:**
  - Unit tests voor controllers (HomeController, ArticleController, PageController, Admin controllers)
  - Integration tests
  - Test coverage verbeteren (80%+ target)

### Versie 1.4 (2026-03-12)
- **Voltooid:**
  - Fase 5: Media Repository (MediaRepository met CRUD operations, search, stats)
  - Fase 5: Dynamic Form Builder (FormGenerator voor alle field types met validatie)
  - Fase 11: Testing & QA (PHP-CS-Fixer, PHPStan config, tests voor FileCache, FileLogger, MediaRepository, FormBuilder)
- **Nog te doen:**
  - Fase 11: Testing & QA (unit tests voor andere services, integration tests, documentation, coverage)

### Versie 1.3 (2026-03-12)
- **Voltooid:**
  - Fase 8: Storage Drivers (FlatFile, SQLite, MySQL, PostgreSQL, StorageFactory)
  - Fase 9: Cache & Logger Services (FileCache, FileLogger)
  - Fase 10: Admin Frontend - JS Framework (TypeScript, Tailwind v4, Alpine.js, Build scripts, UI Components, Utils)
  - Milestone M1 & M2 compleet
- **Nog te doen:**
  - Fase 11: Testing & QA

### Versie 1.2 (2026-03-12)
- **Voltooid:**
  - Fase 7: Content Types (YAML parser, ContentType model, FieldDefinitions, Content Type Manager, Schema Generator)
- **Nog te doen:**
  - Fase 8: Storage Drivers (FlatFile, SQLite, MySQL, PostgreSQL)
  - Fase 9: Admin Frontend - JS Framework (TypeScript, Tailwind v4, Alpine.js)
  - Fase 10: Cache & Logger implementations
  - Fase 11: Testing & QA

### Versie 1.1 (2026-03-12)
- **Voltooid:**
  - Fase 0: Project Setup (CLI commands, bootstrap)
  - Fase 1: Core Services (interfaces)
  - Fase 2: Repository Layer (BaseRepository, ArticleRepository, PageRepository, RepositoryGenerator)
  - Fase 3: Routing & Controllers (RouterService, all controllers, AuthMiddleware)
  - Fase 4: Templates (Twig setup, base template, components, pages, errors)
  - Fase 5: Admin Backend (CRUD operations, auth, file uploads, flash messages)
- **Nog te doen:**
  - Storage drivers (FlatFile, SQLite, MySQL, PostgreSQL)
  - Content Types (YAML parser, models, validation)
  - Cache & Logger implementations
  - Admin JS Framework (TypeScript, Tailwind v4, Alpine.js)
  - Testing & QA

### Versie 1.0 (2026-03-12)
- Initieel document opgezet
el document opgezet
