# Refactoring PRD: themes/admin/assets/ts/index.ts

**Document Version:** 1.0
**Date:** 2026-03-19
**File:** themes/admin/assets/ts/index.ts
**Current Lines:** 185
**Priority:** CRITICAL

---

## 1. Problem Statement

The `index.ts` file currently has three major issues:

1. **Duplicated Theme Management** (CRITICAL - SSOT Violation):
   - Theme management logic exists in both `index.ts` and `utils/theme.ts`
   - Functions `getTheme()`, `applyTheme()`, `setTheme()`, `isDarkMode()`, `themeData()` are duplicated
   - This violates the Single Source of Truth principle stated in PROJECT_SSOT.md

2. **Tight Coupling** (HIGH):
   - Direct CSS import: `import '../css/admin.css'`
   - Couples the bundle to a specific stylesheet
   - Makes testing difficult and reduces flexibility

3. **Mixed Concerns** (MEDIUM):
   - File handles theme management, Alpine.js initialization, and global exposure
   - Difficult to test individual concerns in isolation

---

## 2. Impact Assessment

### Technical Debt Score: 5/5
- Duplication: 40 lines of duplicate theme logic
- Coupling: Direct CSS import breaks modularity
- Maintainability: Changes to theme logic require updates in 2 places

### Impact Score: 4/5
- User-facing: Theme switching is core admin UI functionality
- Dependencies: 7 files import from index.ts (via Alpine.js)
- Regression risk: LOW - removing duplication, not changing behavior

### Risk Score: 3/5
- Critical path: Theme switching is important but not data-critical
- Test coverage: 0% (TypeScript admin has no tests)
- Rollback: Easy - changes are isolated to one file

### Complexity Score: 2/5
- Code is straightforward (no complex algorithms)
- No external dependencies beyond Alpine.js
- Clear separation of duplicated code possible

**Overall Priority: CRITICAL (Score: 3.8/5)**

---

## 3. Refactoring Goals

### Primary Goals
1. **Eliminate Duplication**: Remove all theme management code from `index.ts`
2. **Decouple CSS**: Remove direct CSS import, allow stylesheet flexibility
3. **Separate Concerns**: Split into focused modules
4. **Maintain Behavior**: Zero functional changes - preserve exact current behavior

### Secondary Goals
1. **Improve Testability**: Make individual concerns testable
2. **Reduce Bundle Size**: Remove duplicate code from final bundle
3. **Enhance Maintainability**: Clearer file with single responsibility

---

## 4. Current Architecture

```
index.ts (185 lines)
├── Direct CSS import (tight coupling)
├── Duplicated theme management (40 lines)
│   ├── getTheme()
│   ├── applyTheme()
│   ├── setTheme()
│   ├── isDarkMode()
│   └── themeData()
├── Alpine.js sidebar component (35 lines)
├── Global CalacaCMSAdmin object (15 lines)
└── Initialization logic (30 lines)
```

**Issues:**
- Theme functions duplicate `utils/theme.ts`
- CSS import couples bundle to specific file
- Mixed concerns in single file

---

## 5. Target Architecture

```
index.ts (after refactor - ~50 lines)
├── Import from theme.ts (SSOT)
├── Alpine.js sidebar component
├── Global CalacaCMSAdmin object
└── Initialization orchestration
    ├── Initialize theme from theme.ts
    ├── Register Alpine components
    └── Initialize editor
```

**Benefits:**
- No duplication
- Loosely coupled (no CSS import)
- Single responsibility: orchestration only
- Theme logic lives in one place

---

## 6. Implementation Plan

### Phase 1: Remove Duplicated Theme Code
- [ ] Remove all theme management functions from `index.ts`
- [ ] Import theme functions from `utils/theme.ts`
- [ ] Replace direct calls with imports
- [ ] Update `themeData()` usage to use `themeData()` from theme.ts

### Phase 2: Decouple CSS
- [ ] Remove `import '../css/admin.css'` line
- [ ] Document CSS import responsibility (move to Vite config or separate entry)
- [ ] Verify admin.css is still included in build

### Phase 3: Separate Concerns (Optional)
- [ ] Extract sidebar component to `alpine/sidebar.ts`
- [ ] Extract global object initialization to separate module
- [ ] Keep `index.ts` as thin orchestration layer

### Phase 4: Verification
- [ ] Test theme switching in browser
- [ ] Verify sidebar functionality
- [ ] Check bundle size reduction
- [ ] Confirm no console errors
- [ ] Validate admin.css is still loaded

---

## 7. Detailed Changes

### Change 1: Remove Duplicate Theme Functions (Lines 21-55)
**Before:**
```typescript
function getTheme() { ... }
function applyTheme(theme) { ... }
function setTheme(theme) { ... }
function isDarkMode(theme) { ... }
function themeData() { ... }
```

**After:**
```typescript
import { themeData, themeManager } from '../utils/theme';

// Use themeData directly from theme.ts
```

### Change 2: Remove CSS Import (Line 6)
**Before:**
```typescript
import '../css/admin.css';
```

**After:**
```typescript
// CSS import removed - handled by Vite entry point
// Document in Vite config: add admin.css to entry array
```

### Change 3: Update Theme Data Usage (Line 149)
**Before:**
```typescript
window.themeData = themeData; // local function
```

**After:**
```typescript
import { themeData } from '../utils/theme';
window.themeData = themeData; // imported function
```

### Change 4: Update Initial Theme Application (Line 160)
**Before:**
```typescript
applyTheme(getTheme()); // local functions
```

**After:**
```typescript
import { themeManager } from '../utils/theme';
themeManager.init(); // use theme manager
```

---

## 8. Risk Mitigation

### Risk 1: Breaking Theme Functionality
- **Mitigation**: Test in all browsers (Chrome, Firefox, Safari)
- **Rollback**: Keep original code in separate branch for 24 hours

### Risk 2: CSS Not Loading
- **Mitigation**: Verify Vite build includes admin.css in output
- **Fallback**: Add CSS link in base template if needed

### Risk 3: Alpine.js Integration Breaking
- **Mitigation**: Test sidebar toggle functionality
- **Validation**: Check that Alpine components still register correctly

---

## 9. Success Criteria

- [ ] `index.ts` reduced to ~50 lines (from 185)
- [ ] Zero duplicate theme management code
- [ ] Theme switching works in light/dark/system modes
- [ ] Sidebar toggle functions correctly
- [ ] admin.css loads and applies styles
- [ ] No console errors in browser
- [ ] Bundle size reduced by at least 1KB
- [ ] Zero functional regressions

---

## 10. Testing Strategy

### Manual Testing Checklist
1. **Theme Switching**
   - [ ] Click theme toggle, verify dark mode
   - [ ] Click again, verify light mode
   - [ ] Reload page, verify preference persists
   - [ ] Change system theme, verify system mode follows

2. **Sidebar**
   - [ ] Toggle sidebar open/closed
   - [ ] Reload page, verify state persists
   - [ ] Check collapsed/expanded classes apply

3. **CSS Loading**
   - [ ] Inspect page, verify admin.css in network tab
   - [ ] Check styles render correctly (no unstyled content)

4. **Console**
   - [ ] Open DevTools, verify no errors
   - [ ] Check Alpine.js initializes properly

### Regression Testing
- [ ] Navigate to all admin pages
- [ ] Test forms (articles, pages, media)
- [ ] Verify editor loads correctly

---

## 11. Rollback Plan

If critical issues arise after deployment:

1. **Immediate**: Revert to original `index.ts` file
2. **Within 1 hour**: Restore from git backup
3. **Investigation**: Review browser console logs
4. **Fix**: Address specific breaking change
5. **Re-test**: Run full test suite before redeploy

---

## 12. Dependencies

### Internal Dependencies
- `utils/theme.ts` - Must remain stable during refactor
- `editor.ts` - Editor initialization must still work
- Alpine.js - Must initialize correctly after changes

### External Dependencies
- Alpine.js v3.x
- Browser localStorage API
- Browser matchMedia API

### Build Tooling
- Vite 8.0.0
- Verify CSS inclusion in build process

---

## 13. Time Estimate

- **Phase 1** (Remove duplication): 30 minutes
- **Phase 2** (Decouple CSS): 15 minutes
- **Phase 3** (Separate concerns): 45 minutes (optional)
- **Phase 4** (Verification): 30 minutes

**Total**: 1.5 - 2 hours (without Phase 3), 2-3 hours (with Phase 3)

---

## 14. Post-Refactor Metrics

### Before Refactor
- Lines: 185
- Duplicated code: 40 lines
- CSS coupling: Direct import
- Testability: 0 (no tests)

### After Refactor
- Lines: ~50 (73% reduction)
- Duplicated code: 0
- CSS coupling: None (decoupled)
- Testability: Improved (isolated concerns)

### Technical Debt Reduction
- **Duplication removed**: 40 lines
- **Coupling reduced**: 100%
- **Maintainability score**: +3 (on 1-10 scale)
