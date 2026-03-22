---
description: Spawn agent team for comprehensive CalacaCMS application review (security, performance, quality, testing)
---

# Agent Team - Application Review

Spawn an agent team to perform a comprehensive review of the CalacaCMS application.

**Workspace root:** this monorepo (`calacacms/`). CMS code lives under `cms/`.

```
Read agent-context/PROJECT_SSOT.md first — it is the Single Source of Truth for
this project and contains the full stack, folder structure, commands,
architecture, and code guidelines.

Create an agent team:

1. security-teammate
   Files:
   - cms/calaca/src/Controllers/ (all auth controllers)
   - cms/calaca/src/Middleware/ (auth/session middleware)
   - cms/calaca/src/Models/ (user models)
   - cms/calaca/src/Core/ (bootstrap, config loading)
   - cms/site/ (blueprints, site-level definitions)
   Task: Security review focusing on:
   - Authentication vulnerabilities (session handling, login brute force)
   - Input validation and sanitization (SQL injection, XSS prevention)
   - Secrets handling (hardcoded keys, env exposure)
   - CSRF protection implementation
   - File upload security
   - Output escaping in Twig templates
   - Password storage and hashing
   Read AGENTS.md (root → cms/AGENTS.md) for project conventions.

2. performance-teammate
   Files:
   - cms/calaca/src/Models/ (database models)
   - cms/calaca/src/Repositories/ (data repositories)
   - cms/calaca/src/Controllers/ (API controllers)
   - cms/content/ (JSON storage layer)
   Task: Performance review focusing on:
   - Database query efficiency (JSON file operations, N+1 queries)
   - Caching strategy (absence of cache implementation)
   - API response times and payload sizes
   - Frontend asset bundling and optimization
   - Memory usage patterns
   - File I/O bottlenecks in JSON storage
   Read AGENTS.md for project conventions.

3. quality-teammate
   Files:
   - cms/calaca/src/ (all PHP source files)
   - cms/themes/admin/ (admin theme)
   - docs/ (documentation)
   Task: Code quality review focusing on:
   - Code duplication and DRY violations
   - Naming conventions consistency (classes, methods, variables)
   - Error handling patterns (try/catch usage, error logging)
   - Maintainability and complexity metrics
   - Documentation completeness
   - Architectural pattern adherence (MVC)
   - SOLID principles violations
   Read AGENTS.md for project conventions.

4. testing-teammate
   Files:
   - cms/calaca/tests/ (existing test suite)
   - cms/calaca/src/Controllers/ (all controllers)
   - cms/calaca/src/Models/ (all models)
   - cms/calaca/src/Services/ (business logic)
   Task: Test coverage review focusing on:
   - Untested code paths in controllers
   - Missing edge cases in models
   - Service layer test gaps
   - Integration test requirements
   - E2E test scenarios for critical flows
   - Test infrastructure setup
   Read AGENTS.md for project conventions.

All teammates work independently and report findings back in a structured format:
- Critical issues (security risks, data loss potential)
- Warnings (performance impact, maintainability concerns)
- Suggestions (improvements, best practices)
- File paths with line numbers for each finding

Each teammate should verify their findings with specific code citations.
```

## Usage

Run this prompt in Claude Code lead session to spawn the full agent team.
