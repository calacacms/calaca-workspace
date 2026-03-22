# Refactoring Task List: themes/admin/assets/ts/index.ts

**Generated from:** docs/prd-refactor-index-ts.md
**Date:** 2026-03-19
**File:** themes/admin/assets/ts/index.ts
**Estimated Time:** 1.5 - 2 hours

---

## Task Overview

Refactor `themes/admin/assets/ts/index.ts` to eliminate duplicated theme management code, decouple CSS imports, and separate concerns. Target is to reduce from 185 lines to ~50 lines while maintaining exact functionality.

---

## Phase 1: Remove Duplicated Theme Code

### Task 1.1: Audit theme duplication
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Identify all theme-related code in index.ts that exists in theme.ts

**Subtasks:**
- [ ] Compare `index.ts` lines 21-55 with `utils/theme.ts`
- [ ] List all duplicated functions:
  - [ ] getTheme()
  - [ ] applyTheme()
  - [ ] setTheme()
  - [ ] isDarkMode()
  - [ ] themeData()
- [ ] Verify `themeData` function signatures match
- [ ] Document any differences found

**Acceptance Criteria:**
- Complete list of duplicated functions
- Comparison document with line numbers
- Verification that behavior is identical

---

### Task 1.2: Update imports
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Add imports from theme.ts to replace duplicated code

**Subtasks:**
- [ ] Add import: `import { themeData, themeManager } from '../utils/theme'`
- [ ] Verify import path is correct
- [ ] Check TypeScript compilation with new imports

**Acceptance Criteria:**
- Imports added without errors
- TypeScript compiles successfully

---

### Task 1.3: Remove duplicate theme functions
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Delete duplicated theme management code from index.ts

**Subtasks:**
- [ ] Remove lines 21-32 (getTheme, applyTheme functions)
- [ ] Remove lines 34-36 (setTheme function)
- [ ] Remove lines 38-41 (isDarkMode function)
- [ ] Remove lines 43-95 (themeData function - entire block)
- [ ] Verify no orphaned references to removed functions

**Acceptance Criteria:**
- All duplicate theme code removed
- No references to deleted functions
- File reduced by ~40 lines

---

### Task 1.4: Update theme data usage
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Replace local themeData with imported version

**Subtasks:**
- [ ] Update line 149: `window.themeData = themeData;` (use imported)
- [ ] Update line 160: Replace `applyTheme(getTheme())` with `themeManager.init()`
- [ ] Update line 27: Change local storage key usage (if any)

**Acceptance Criteria:**
- All themeData calls use imported version
- themeManager.init() replaces manual initialization
- No breaking changes to global API

---

## Phase 2: Decouple CSS

### Task 2.1: Remove CSS import
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Remove direct CSS import from index.ts

**Subtasks:**
- [ ] Remove line 6: `import '../css/admin.css'`
- [ ] Search for any other CSS imports
- [ ] Document removal in commit message

**Acceptance Criteria:**
- CSS import line removed
- No other CSS imports found

---

### Task 2.2: Verify CSS loading
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Ensure admin.css still loads without direct import

**Subtasks:**
- [ ] Check Vite config (themes/admin/vite.config.ts or package.json)
- [ ] Verify admin.css is included in build entry points
- [ ] Test build: `bun run build:admin`
- [ ] Inspect output: `web/assets/admin/*.css` exists
- [ ] If not included, add to Vite config

**Acceptance Criteria:**
- admin.css is in build output
- Vite config includes CSS correctly
- Build completes without errors

---

### Task 2.3: Document CSS responsibility
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Add comment documenting CSS import location

**Subtasks:**
- [ ] Add comment at top of index.ts:
  ```typescript
  /**
   * CalacaCMS Admin JavaScript Entry Point
   * Main bundle - IIFE format for direct browser execution
   *
   * Note: CSS is loaded via Vite entry point, not here.
   * See: themes/admin/vite.config.ts or package.json for CSS imports
   */
  ```
- [ ] Verify comment is accurate

**Acceptance Criteria:**
- Documentation comment added
- Comment accurately reflects CSS loading

---

## Phase 3: Separate Concerns (Optional - Enhances quality)

### Task 3.1: Extract sidebar component
**Status:** ⬜ Not Started
**Estimated:** 20 minutes
**Description:** Move Alpine.js sidebar component to dedicated file

**Subtasks:**
- [ ] Create file: `themes/admin/assets/ts/alpine/sidebar-registry.ts`
- [ ] Extract lines 101-142 to new file
- [ ] Export register function: `export function registerSidebarComponent()`
- [ ] Import and call in index.ts: `registerSidebarComponent()`
- [ ] Delete sidebar code from index.ts

**Acceptance Criteria:**
- Sidebar component in separate file
- Function exported cleanly
- index.ts imports and calls it
- Sidebar functionality unchanged

---

### Task 3.2: Extract global object initialization
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Move CalacaCMSAdmin global to separate module

**Subtasks:**
- [ ] Create file: `themes/admin/assets/ts/utils/global-init.ts`
- [ ] Extract lines 152-157 to new file
- [ ] Export function: `export function initializeGlobalObject()`
- [ ] Import and call in index.ts
- [ ] Delete from index.ts

**Acceptance Criteria:**
- Global object in separate module
- Clean function signature
- index.ts calls it appropriately
- Global object accessible as `window.CalacaCMSAdmin`

---

### Task 3.3: Simplify index.ts to orchestration
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Final cleanup of index.ts to thin orchestration layer

**Subtasks:**
- [ ] Verify all concerns extracted
- [ ] Check file is ~50 lines
- [ ] Ensure only orchestration remains:
  - Imports
  - Alpine.js setup
  - Component registration calls
  - Editor initialization
- [ ] Add JSDoc comment explaining orchestration role

**Acceptance Criteria:**
- File is minimal orchestration layer
- All logic extracted to modules
- Clear JSDoc documentation
- File size ~50 lines

---

## Phase 4: Verification & Testing

### Task 4.1: TypeScript compilation check
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Verify no TypeScript errors

**Subtasks:**
- [ ] Run: `bun run build:admin`
- [ ] Check for TypeScript errors
- [ ] Fix any type errors found
- [ ] Verify build outputs generated

**Acceptance Criteria:**
- Zero TypeScript errors
- Build completes successfully
- Output files generated

---

### Task 4.2: Bundle size verification
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Confirm bundle size reduction

**Subtasks:**
- [ ] Build current version (before refactor): Record size
- [ ] Build refactored version: Record size
- [ ] Compare sizes: Expect ~1KB+ reduction
- [ ] Verify no unexpected size increases

**Acceptance Criteria:**
- Bundle size reduced by ≥1KB
- No significant size increases
- Build output is similar in structure

---

### Task 4.3: Manual browser testing
**Status:** ⬜ Not Started
**Estimated:** 20 minutes
**Description:** Test all functionality in browser

**Subtasks:**
- [ ] Start dev server: `bun run dev:admin`
- [ ] Open admin panel: http://localhost:5173/admin
- [ ] Test theme toggle:
  - [ ] Click toggle, verify dark mode
  - [ ] Click again, verify light mode
  - [ ] Reload page, verify persistence
- [ ] Test sidebar:
  - [ ] Toggle open/closed
  - [ ] Reload, verify state persists
- [ ] Check CSS loads:
  - [ ] Inspect network tab for admin.css
  - [ ] Verify styles render correctly
- [ ] Check console:
  - [ ] Zero errors
  - [ ] Alpine.js initializes correctly

**Acceptance Criteria:**
- Theme switching works perfectly
- Sidebar functions correctly
- CSS loads and styles apply
- Zero console errors
- All functionality preserved

---

### Task 4.4: Regression testing
**Status:** ⬜ Not Started
**Estimated:** 15 minutes
**Description:** Test all admin pages for regressions

**Subtasks:**
- [ ] Navigate to /admin
- [ ] Navigate to /admin/articles
- [ ] Navigate to /admin/articles/create
- [ ] Navigate to /admin/pages
- [ ] Navigate to /admin/pages/create
- [ ] Navigate to /admin/media
- [ ] Navigate to /admin/settings
- [ ] Navigate to /admin/users
- [ ] Check each page:
  - [ ] Styles load correctly
  - [ ] Theme toggle works
  - [ ] Sidebar works
  - [ ] No JavaScript errors

**Acceptance Criteria:**
- All admin pages functional
- No broken functionality
- Consistent behavior across pages

---

### Task 4.5: Final verification
**Status:** ⬜ Not Started
**Estimated:** 10 minutes
**Description:** Final checks before completion

**Subtasks:**
- [ ] Verify file size: ~50 lines (target)
- [ ] Count removed lines: ~135 lines (expected)
- [ ] Verify zero duplicate code
- [ ] Check all imports are necessary
- [ ] Verify no orphaned code
- [ ] Run linter: `bunx eslint themes/admin/assets/ts/index.ts` (if configured)

**Acceptance Criteria:**
- File is minimal and focused
- Zero code duplication
- All code has purpose
- Linter passes (if configured)

---

## Phase 5: Documentation & Cleanup

### Task 5.1: Update JSDoc
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Add/update file documentation

**Subtasks:**
- [ ] Add comprehensive JSDoc at top of file
- [ ] Document purpose: "Orchestration layer for admin JavaScript"
- [ ] Document dependencies: Alpine.js, theme manager, components
- [ ] Document initialization sequence

**Acceptance Criteria:**
- Clear, professional JSDoc
- Purpose and dependencies documented
- Initialization sequence explained

---

### Task 5.2: Clean up any temporary files
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Remove any temporary or backup files

**Subtasks:**
- [ ] Check for .backup, .orig, or temp files
- [ ] Remove any temporary files
- [ ] Verify git status shows only intended changes

**Acceptance Criteria:**
- No temporary files in directory
- Git status is clean (only intended changes)

---

### Task 5.3: Commit changes
**Status:** ⬜ Not Started
**Estimated:** 5 minutes
**Description:** Commit refactoring with clear message

**Subtasks:**
- [ ] Stage changes: `git add themes/admin/assets/ts/index.ts`
- [ ] Stage any new files: `git add themes/admin/assets/ts/alpine/*.ts`
- [ ] Commit with message:
  ```
  refactor(admin): remove duplicate theme code from index.ts

  - Remove 40 lines of duplicated theme management
  - Delegate theme logic to utils/theme.ts (SSOT)
  - Decouple CSS import, handle via Vite
  - Reduce file from 185 to ~50 lines

  Fixes architectural SSOT violation found in refactor analysis.

  Related: docs/prd-refactor-index-ts.md
  ```
- [ ] Verify commit created successfully

**Acceptance Criteria:**
- Changes committed with clear message
- Commit references PRD
- All changes included

---

## Summary

**Total Tasks:** 16 tasks across 5 phases
**Total Estimated Time:** 1.5 - 2 hours (without Phase 3), 2-3 hours (with Phase 3)

**Phases:**
1. ✅ Remove Duplicated Theme Code (40 min)
2. ✅ Decouple CSS (20 min)
3. ✅ Separate Concerns - Optional (45 min)
4. ✅ Verification & Testing (60 min)
5. ✅ Documentation & Cleanup (15 min)

**Completion Criteria:**
- All tasks marked as completed
- Manual testing passed
- Bundle size verified reduced
- Zero regressions found
- Documentation updated
- Changes committed

---

## Notes

- **Phase 3 is optional**: Can defer if time constrained; refactoring is valuable without it
- **Testing priority**: Manual browser testing is critical; no automated tests exist yet
- **Rollback ready**: Keep original file content handy for quick rollback if needed
- **Next steps**: After completion, consider adding TypeScript tests for theme management
