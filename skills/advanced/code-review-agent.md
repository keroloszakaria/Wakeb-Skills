# Skill: Code Review Agent

Performs automated code review on generated or existing code, checking for
pattern compliance, security, performance, and Wakeb Starter conventions.

## When to Use

- After generating any code (automatic review pass)
- When the user asks for a code review
- Before finalizing a feature implementation

## Review Checklist

### Structure Compliance

```
FE Module:
□ Config file with headers + form schema
□ IndexView uses useLookupPage
□ AddEditView uses useCrudFactory
□ Router with lazy-loaded routes
□ Locale files (ar.json + en.json)

BE Module:
□ Controller extends BaseController
□ Request extends BaseRequest
□ Resource with toArray()
□ Model with $fillable, $casts, SoftDeletes
□ Pipeline classes for filter + sort
□ Routes with auth:sanctum
□ Migration with up() + down()
```

### Code Quality

```
□ No console.log left in production code
□ No TODO/FIXME without issue reference
□ No commented-out code blocks
□ No unused imports or variables
□ No hardcoded strings — use $t() for FE, __() for BE
□ No hardcoded colors — use design tokens
□ No ml-/mr- — use ms-/me-
□ No left/right CSS — use start/end
□ Functions < 30 lines
□ Components < 300 lines
□ Files have single responsibility
```

### Security (see security-scanner skill for details)

```
□ Auth middleware on all routes
□ Input validated in FormRequest
□ No raw SQL with user input
□ No hardcoded secrets
□ File uploads validated (type + size)
□ Sensitive fields in $hidden
```

### Performance (see performance-optimizer skill for details)

```
□ Routes lazy-loaded
□ Lists paginated
□ Computed for derived state
□ No deep watchers on large objects
□ Heavy components async-loaded
```

### Accessibility (see accessibility-ux-validator skill for details)

```
□ Images have alt text
□ Forms have labels
□ Keyboard navigation works
□ Color not sole indicator
□ Loading/empty states present
```

## Review Output Format

```
## Code Review Summary

### ✅ Passed
- Structure follows module conventions
- Auth middleware present on all routes
- Design tokens used consistently

### ⚠️ Warnings
- [file.vue:L25] Missing loading state for async operation
- [Controller.php:L40] Consider extracting complex query to Pipeline

### ❌ Issues
- [form.vue:L15] Hardcoded string "Save" — use $t('actions.save')
- [model.php:L8] Password in $fillable — security risk

### 💡 Suggestions
- Consider adding empty state message for table
- Filter pipeline could support date range
```

## Rules

```
□ Review ALL generated files, not just the main ones
□ Flag issues by severity: ❌ must fix, ⚠️ should fix, 💡 nice to have
□ Reference specific file + line number
□ Provide the fix, not just the problem
□ Don't flag stylistic preferences — only convention violations
□ Auto-fix ❌ issues when possible
```
