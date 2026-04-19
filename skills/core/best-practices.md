# Skill: Best Practices

Universal best practices enforced across all Wakeb code. These are not
preferences — they prevent real bugs, security holes, and maintenance debt.

## When to Use

- During all code generation (automatic)
- When reviewing pull requests
- When debugging issues that stem from bad practices

## Security Best Practices

```
AUTHENTICATION:
□ Use Sanctum for SPA auth — no manual JWT handling
□ Token expiration configured in config/sanctum.php
□ Logout invalidates all tokens — not just current
□ Password reset uses signed URLs with expiry

AUTHORIZATION:
□ Gate::authorize() for every write operation
□ Policy classes for resource-level checks
□ Never trust client-side role checks alone
□ Permission middleware on route level

INPUT VALIDATION:
□ $request->validated() — never $request->all()
□ BaseFormRequest converts empty strings to null
□ File uploads validated: type, size, dimensions
□ SQL injection prevented via Eloquent / query builder (no raw queries)

OUTPUT:
□ API resources transform data — never return raw models
□ Sensitive fields excluded: password, remember_token, etc.
□ Error responses use failResponse() — never expose internals
□ Stack traces hidden in production (APP_DEBUG=false)

FRONTEND:
□ No v-html with user input (XSS risk)
□ Sanitize before rendering any user-generated content
□ CSRF handled by Sanctum cookie auth
□ API tokens stored in httpOnly cookies — not localStorage
```

## Performance Best Practices

```
FRONTEND:
□ Lazy load routes: () => import('../views/Page.vue')
□ Async components: defineAsyncComponent for heavy components
□ Debounce search inputs: 300ms minimum
□ Virtual scrolling for lists > 100 items
□ Image lazy loading with loading="lazy"
□ Cache API responses via useCrudFactory({ cache: { enabled: true } })

BACKEND:
□ Eager load relations: ->with(['relation']) — prevent N+1
□ Paginate all list endpoints — never return unbounded results
□ Cache expensive queries: Cache::remember()
□ Queue heavy operations: email, notifications, exports
□ Use database indexes on frequently queried columns
□ Chunk large dataset processing: Model::chunk(1000, fn)
```

## Code Quality Best Practices

```
NAMING:
□ Variables describe what they hold, not their type
□ Functions describe what they do, not how
□ Boolean variables start with is/has/can/should
□ Constants in UPPER_SNAKE_CASE

ERROR HANDLING:
□ Catch specific exceptions — never catch(\Exception $e) without logging
□ User-facing errors use translated messages
□ System errors logged with context (user ID, request data)
□ API errors return consistent shape via failResponse()

TESTING:
□ Feature tests for API endpoints
□ Unit tests for services and utilities
□ Test both happy path and error cases
□ Test authorization (forbidden for wrong role)
```

## DRY (Don't Repeat Yourself)

```
IF you're writing the same code twice:
  FE → Extract to a composable (use{Name})
  FE → Extract to a utility function in utils/
  BE → Extract to a Service class
  BE → Extract to a Trait
  BE → Extract to a Pipeline Filter (for queries)

IF you're creating the same UI pattern twice:
  FE → Check if a component already exists in common/
  FE → If not, create one in common/ — not in the module
```
