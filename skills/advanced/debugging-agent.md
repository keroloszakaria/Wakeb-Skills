# Skill: Debugging Agent

Systematic approach to diagnosing and fixing bugs in Wakeb Starter projects.
Follows a structured flow to avoid guessing.

## When to Use

- When the user reports a bug or unexpected behavior
- When generated code doesn't work as expected
- When there are console errors or API failures

## Debugging Flow

```
1. Reproduce → Understand the exact symptom
2. Isolate   → Narrow down to FE or BE
3. Trace     → Follow the data flow
4. Identify  → Find the root cause
5. Fix       → Apply minimal targeted fix
6. Verify    → Confirm the fix resolves the issue
```

## Step 1: Classify the Problem

```
Symptom                          → Start Here
───────────────────────────────  ──────────────────
Page blank / white screen        → Check console for JS errors
Data not showing in table        → Check Network tab → API response
Form not submitting              → Check validation → Network request
404 on API call                  → Check route definition + endpoint name
422 validation error             → Check request rules vs sent data
401 unauthorized                 → Check auth token + Sanctum config
403 forbidden                    → Check permissions + middleware
500 server error                 → Check Laravel logs (storage/logs/)
Layout broken                    → Check RTL classes + design tokens
Styles wrong                     → Check token names + scoped CSS
```

## Step 2: Frontend Debugging

### Console Errors

```
"Cannot read properties of undefined"
  → Check if API returned expected data structure
  → Check if store is populated before component renders
  → Check optional chaining: item?.name vs item.name

"[Vue warn]: Property X was accessed during render but is not defined"
  → Check if variable is declared in setup/data
  → Check if composable return value is destructured correctly

"Failed to resolve component"
  → Check component import and registration
  → Check if component name matches (case-sensitive)
```

### API Call Issues

```
Check in this order:
1. useCrudFactory endpoint matches BE route name?
2. Request payload matches BE validation rules?
3. Response structure matches FE expectations?
4. Auth token being sent? (check jervis-connect interceptors)
5. CORS blocking? (check browser console for CORS errors)
```

### State Issues

```
Check in this order:
1. Store action dispatched? (add console.log in action)
2. Store state updated? (Vue DevTools → Pinia)
3. Component reactive to store? (computed vs direct access)
4. Cache stale? (clear cache in useCrudFactory)
```

## Step 3: Backend Debugging

### Reading Laravel Logs

```
Location: storage/logs/laravel.log
Key info: Exception class, message, file:line, stack trace

Common errors:
- QueryException → SQL issue (check migration + model)
- ModelNotFoundException → findOrFail with wrong ID
- AuthorizationException → missing permission
- ValidationException → request rules mismatch
```

### API Response Debugging

```php
// Temporary debug in controller (REMOVE after debugging)
return response()->json([
    'debug' => [
        'request' => $request->all(),
        'query' => $query->toSql(),
        'bindings' => $query->getBindings(),
    ]
]);
```

### Database Issues

```
□ Migration ran? php artisan migrate:status
□ Column exists? Check migration matches model $fillable
□ Foreign key valid? Check referenced table has the ID
□ Soft deleted? Check if record is soft-deleted (deleted_at not null)
□ Cast correct? Check model $casts matches column type
```

## Common Wakeb-Specific Issues

```
Issue: useLookupPage returns empty data
Fix: Check that config.module matches API endpoint name

Issue: GenericForm fields not showing
Fix: Check schema array structure — each field needs key, type, label

Issue: Table headers show key instead of label
Fix: Check headers config — title should use $t() with locale key

Issue: Design tokens not applying
Fix: Check token name matches exactly (copy from tokens file)
     Use rgba(var(--v-theme-{name}), 1) format

Issue: RTL layout broken
Fix: Replace ml-/mr- with ms-/me-, left/right with start/end

Issue: 422 errors not showing on form fields
Fix: Check handleErrors() is called, field keys match BE validation keys
```

## Rules

```
□ Never guess — always trace the data flow
□ Check the simplest explanation first (typo, wrong name, missing import)
□ Read error messages carefully — they usually point to the issue
□ One fix at a time — don't change multiple things simultaneously
□ Remove all debug code after fixing
□ If fix requires a pattern change, update the relevant skill file
```
