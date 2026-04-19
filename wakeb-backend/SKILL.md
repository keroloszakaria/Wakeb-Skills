---
name: wakeb-backend
description: >
  Generates production-ready Laravel API code using the Wakeb Backend Starter
  conventions. Handles module scaffolding, CRUD controllers with Pipeline
  filtering, form request validation, API resources, Eloquent models with
  traits, policy-based authorization, and Arabic-first i18n support. Use this
  skill whenever creating new API modules, controllers, models, form requests,
  resources, filters, policies, seeders, migrations, or doing any backend
  development task inside a Wakeb/Starter-Backend Laravel project — even if
  the user doesn't explicitly mention "Wakeb" or "starter".
---

# Wakeb Backend Skill

Laravel 12 + Sanctum + Spatie Permission + nwidart Modules API starter.
Arabic-first, English supported. Admin-only dashboard backend.

### When to Use

- Creating a new API module or CRUD endpoint
- Building controllers, models, requests, or resources
- Adding query filters or search functionality
- Setting up authorization policies and permissions
- Creating database migrations, seeders, or factories
- Working with translatable (multi-language) fields
- Adding notifications (email, SMS, realtime, in-app)
- Configuring auth, OTP, or LDAP settings
- Debugging API errors or validation issues
- Any code generation inside `Starter-Backend` or similar Wakeb Laravel projects

## Execution Flow

Every code generation request follows these 7 steps in order:

1. **Analyze request** — Classify what the user needs (module, endpoint, model, fix)
2. **Scan codebase** — Consult [codebase-index.md](references/codebase-index.md) to know what already exists
3. **Resolve reusable parts** — Find existing controllers, filters, traits, and services that match
4. **Generate structure** — Place files in correct locations (see [enforce-structure](../wakeb-dashboard/references/skills/enforce-structure.md))
5. **Generate code** — Write code using project base classes, traits, and patterns
6. **Validate output** — Run full validation checklist (see [validate-output](../wakeb-dashboard/references/skills/validate-output.md))
7. **Return result** — Only return code that passes all checks

## Skills Taxonomy

The core skills are shared across FE and BE. Backend-specific awareness:

| Category           | Skill                                                                          | Backend Focus                                                           |
| ------------------ | ------------------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| Module Scaffolding | [generate-module](../wakeb-dashboard/references/skills/generate-module.md)     | Controller, Model, Request, Resource, Migration, Routes, Filter, Seeder |
| API Integration    | [connect-api](../wakeb-dashboard/references/skills/connect-api.md)             | Endpoint validation, response format, request rules                     |
| Validation & QA    | [validate-output](../wakeb-dashboard/references/skills/validate-output.md)     | Base class usage, Pipeline, Gate, response helpers                      |
| Structure          | [enforce-structure](../wakeb-dashboard/references/skills/enforce-structure.md) | File placement, naming, module completeness                             |
| Reuse              | [reuse-composable](../wakeb-dashboard/references/skills/reuse-composable.md)   | Existing filters, traits, services                                      |

**Codebase Awareness:** [references/codebase-index.md](references/codebase-index.md) — full inventory of every controller, model, filter, trait, service, and rule. Consult BEFORE generating code.

## Architecture

```
app/
├── Http/
│   ├── Controllers/API/         # BaseController + feature controllers
│   │   ├── Auth/                # Login, Logout, OTP, ResetPassword
│   │   ├── DataEntry/           # Simple CRUD (Country, etc.)
│   │   ├── Global/              # Settings, Export, Report, Notifications
│   │   ├── Profile/             # User profile management
│   │   └── User/                # User, Role, Permission management
│   ├── Requests/                # BaseFormRequest + feature requests
│   └── Resources/               # JsonResource transformers
├── Models/                      # BaseModel + Eloquent models
├── Policies/                    # Gate authorization policies
├── Filters/                     # Pipeline query filters
├── Enum/                        # PHP backed enums with EnumMethods
├── Scopes/                      # Eloquent query scopes
├── Services/                    # Business logic services
├── Trait/Global/                # HasDeleteMethods, HasToggleActive, etc.
├── Rules/                       # Custom validation rules
├── Helpers/App.php              # Global helper functions
├── Events/                      # Broadcast events
└── Jobs/                        # Queued jobs (email, SMS)
Modules/
├── {Name}/
│   ├── app/Http/Controllers/Api/Admin/  # Module controllers
│   ├── app/Http/Requests/               # Module form requests
│   ├── app/Http/Resources/Admin/        # Module API resources
│   ├── app/Models/                      # Module models
│   ├── app/Observers/                   # Model observers
│   ├── routes/api.php                   # Module routes
│   ├── database/migrations/             # Module migrations
│   ├── database/seeders/                # Module seeders
│   ├── Providers/{Name}ServiceProvider  # Module registration
│   └── config/                          # Module config
config/project.php                       # Project-wide settings
routes/api.php                           # Main API routes
```

## Decision Protocol

| User says...                       | Workflow                      | Priority |
| ---------------------------------- | ----------------------------- | -------- |
| "Create module X" / "new API"      | Module scaffold (all 8 files) | CRITICAL |
| "Add endpoint" / "new CRUD"        | Controller + routes           | CRITICAL |
| "Add field" / "change validation"  | Request + migration           | HIGH     |
| "Add filter" / "search by"         | Pipeline filter class         | HIGH     |
| "Add permission" / "authorization" | Policy + model permissions    | MEDIUM   |
| "Fix bug" / "error" / "500"        | Debugging (see patterns.md)   | MEDIUM   |
| Any code output                    | Run quality checklist first   | ALWAYS   |

## Module Scaffold Workflow

Generate ALL 8 files for a new module. See [references/module-scaffold.md](references/module-scaffold.md) for complete templates.

**Why all 8?** The starter uses `nwidart/laravel-modules` which auto-discovers
modules via service providers. Missing files break the module or leave gaps.

**Files to generate:**

1. `Model` — extends `BaseModel`, uses traits
2. `Controller` — extends `BaseController`, uses Pipeline + traits
3. `FormRequest` — extends `BaseFormRequest`, validation rules
4. `Resource` — extends `JsonResource`, response transformation
5. `Migration` — database schema with soft deletes
6. `Routes` — `apiResource` + delete/restore/toggle routes
7. `Filter` — Pipeline-compatible query filter
8. `Seeder` — initial data (optional)

**Key rules:**

- Controllers extend `BaseController` — it sets the auth guard automatically
- Use `HasDeleteMethods` + `HasToggleActiveMethods` traits for delete/restore/toggle
- Use `Pipeline` for composable query filtering — never chain `where()` manually
  for search/filter/sort logic
- Use `Gate::authorize()` for authorization — it checks policies first, then
  falls back to Spatie permissions
- Wrap multi-step operations in `DB::transaction()`
- Return responses with `successResponse()` / `failResponse()` helpers

**Quick controller pattern:**

```php
class ProductController extends BaseController implements HasMiddleware
{
    use HasDeleteMethods, HasToggleActiveMethods;

    public function __construct()
    {
        parent::__construct();
        $this->model = Product::class;
    }

    public static function middleware(): array
    {
        return [
            new Middleware(PermissionMiddleware::using('create-product'), only: ['store']),
            new Middleware(PermissionMiddleware::using('update-product'), only: ['update']),
        ];
    }

    public function index(PageRequest $request): JsonResponse
    {
        $query = app(Pipeline::class)
            ->send(Product::query()->with(['creator']))
            ->through([JsonNameFilter::class, ActiveFilter::class, TrashedFilter::class, OrderByFilter::class])
            ->thenReturn();

        return successResponse(wrapPaginate($query, ProductResource::class));
    }

    public function store(ProductRequest $request): JsonResponse
    {
        Gate::authorize('create', Product::class);
        $product = Product::create($request->validated());
        return successResponse(new ProductResource($product->refresh()), __('api.created_success'));
    }

    public function show(Product $product): JsonResponse
    {
        Gate::authorize('view', $product);
        return successResponse(new ProductResource($product));
    }

    public function update(ProductRequest $request, Product $product): JsonResponse
    {
        Gate::authorize('update', $product);
        $product->update($request->validated());
        return successResponse(new ProductResource($product->refresh()), __('api.updated_success'));
    }
}
```

Full templates: [references/module-scaffold.md](references/module-scaffold.md)

## Request Validation

Extend `BaseFormRequest` which auto-converts empty values to null and returns
JSON errors (422) instead of redirects.

```php
class ProductRequest extends BaseFormRequest
{
    public function rules(): array
    {
        $id = $this->route('product')?->getKey();

        return [
            'name'   => ['required', 'array', new TranslatableRequired('products', ['string', 'max:191'], 'product')],
            'price'  => ['required', 'numeric', 'min:0'],
            'status' => ['sometimes', 'boolean'],
            'image'  => ['sometimes', 'nullable', File::image()->max(20048)],
        ];
    }
}
```

Available custom rules: `StrongPassword`, `UniqueCheck`, `ValidLength`,
`TranslatableRequired`, `TranslatableNullable`, `CheckSamePassword`, `TotalFileSize`

Full validation reference: [references/validation.md](references/validation.md)

## API Response Format

All responses follow a consistent structure. Use helper functions:

```php
// Success
successResponse($data, $msg, $code = 200)
// → { "status": true, "code": 200, "message": "...", "data": {...} }

// Failure
failResponse($msg, $data = [], $code = 400)
// → { "status": false, "code": 400, "message": "...", "data": [] }

// Pagination
successResponse(wrapPaginate($query, ResourceClass::class))
// → { "status": true, "data": { "data": [...], "meta": { "current_page", "last_page", ... } } }
```

## Pipeline Filtering

Compose query logic with reusable filter classes instead of chaining `where()`:

```php
$query = app(Pipeline::class)
    ->send(Model::query()->with(['relations']))
    ->through([
        JsonNameFilter::class,    // Search in JSON 'name' field
        ActiveFilter::class,      // ?is_active=true/false
        TrashedFilter::class,     // ?is_trashed=true
        OrderByFilter::class,     // ?sort_column=name&sort_direction=asc
    ])
    ->thenReturn();
```

To create a custom filter, see [references/filters.md](references/filters.md).

## Model Conventions

```php
class Product extends BaseModel
{
    use HasTranslations, SoftDeletes, CreatedByObserver, LogsActivityOptions;

    public array $translatable = ['name', 'description'];
    public bool $inPermission = true;
    public array $specialOperations = ['force-delete', 'restore', 'toggle-active'];

    protected $fillable = ['name', 'description', 'price', 'image', 'is_active', 'created_by'];

    protected $casts = [
        'is_active' => 'boolean',
        'price'     => 'decimal:2',
    ];
}
```

Key model traits and their purpose:

- `HasTranslations` — Multi-language JSON fields (Spatie)
- `SoftDeletes` — Soft delete support
- `CreatedByObserver` — Auto-sets `created_by` from auth user
- `LogsActivityOptions` — Audit trail (Spatie ActivityLog)
- `ApplyNotification` — `sendNotification()` method for email/sms/realtime

Full model reference: [references/models.md](references/models.md)

## Routing Convention

```php
// In routes/api.php (or module routes/api.php)
Route::middleware(['auth:sanctum'])->group(function () {
    Route::prefix('products')->group(function () {
        Route::delete('force-delete', [ProductController::class, 'forceDelete']);
        Route::delete('delete', [ProductController::class, 'destroy']);
        Route::post('restore', [ProductController::class, 'restore']);
        Route::put('toggle-active', [ProductController::class, 'toggleActive']);
        Route::apiResource('/', ProductController::class)
            ->parameters(['' => 'product'])
            ->except(['destroy']);
    });
});
```

Pattern: `apiResource` for standard CRUD + explicit routes for soft-delete
operations and toggle-active. The `except(['destroy'])` is because delete uses
the trait's `destroy()` which accepts bulk IDs via `DELETE /products/delete`.

## Code Conventions

- Models extend `BaseModel`, controllers extend `BaseController` — these base
  classes wire up auth guard, response format, and common behavior; skipping
  them means reimplementing that setup in every class
- All requests extend `BaseFormRequest` — it auto-converts empty strings to
  null and returns JSON validation errors (422) instead of HTML redirects,
  which would break the SPA frontend
- Responses use `successResponse()` / `failResponse()` — the frontend parser
  expects `{ status, code, message, data }`; `response()->json()` produces a
  different shape that breaks the dashboard
- Authorization via `Gate::authorize()` — checks policy first, then falls back
  to Spatie permissions, giving a unified authorization layer
- Translatable fields stored as JSON, validated with `TranslatableRequired` —
  ensures both Arabic and English values exist
- File uploads via `Media::upload()` / `Media::replace()` (hasanhawary/media-manager)
- Permission names: `{action}-{model}` (e.g., `create-product`, `view-all-product`)
- Enums use backed string enums with `EnumMethods` trait
- Notifications via `$model->sendNotification([...], ['email', 'realtime', 'notify'])`

Full conventions: [references/patterns.md](references/patterns.md)

## Anti-Patterns

These patterns cause real problems — broken API responses, security holes,
inconsistent query behavior, or code that looks alien next to the rest of the project:

| Do NOT                                              | Do instead                                     | Why it matters                                       |
| --------------------------------------------------- | ---------------------------------------------- | ---------------------------------------------------- |
| Return `response()->json()` directly                | `successResponse()` / `failResponse()`         | Frontend expects `{ status, code, message, data }`   |
| Chain `where()` for search/filter/sort              | Pipeline with filter classes                   | Pipeline filters are reusable; chained wheres aren't |
| Check permissions with `$user->can()` in controller | `Gate::authorize()` (policy + Spatie fallback) | Skips the policy layer, inconsistent auth behavior   |
| Manual soft-delete logic                            | `HasDeleteMethods` trait                       | Trait handles bulk delete, restore, and force-delete |
| Manual toggle-active logic                          | `HasToggleActiveMethods` trait                 | Trait handles bulk toggle with lifecycle hooks       |
| Hardcode strings in responses                       | `__('api.key')` translation helpers            | Breaks i18n and makes message changes harder         |
| Skip `DB::transaction()` for multi-model ops        | Wrap in transaction                            | Partial writes corrupt data on failure               |
| Use `$request->all()` in create/update              | `$request->validated()` only                   | Allows mass-assignment of unvalidated fields         |
| Extend `Controller` directly                        | Extend `BaseController`                        | Loses auth guard setup and shared behavior           |
| Skip `BaseFormRequest`                              | Extend it                                      | Loses null conversion and JSON error responses       |

## Quality Checklist

Run BEFORE outputting any code:

```
Base checks:
- [ ] Controller extends BaseController
- [ ] Request extends BaseFormRequest
- [ ] Model extends BaseModel
- [ ] Response uses successResponse() / failResponse()
- [ ] Authorization via Gate::authorize()
- [ ] Query filtering via Pipeline pattern
- [ ] Strings use __('api.key') translations
- [ ] $request->validated() (never $request->all())
```

**For modules, also check:**

```
- [ ] ServiceProvider registered in module.json
- [ ] Routes under auth:sanctum middleware
- [ ] Delete/restore/toggle routes + traits
- [ ] Permission middleware on store/update
- [ ] Migration with soft deletes + timestamps
- [ ] Seeder registered in DatabaseSeeder
```

**For models, also check:**

```
- [ ] $inPermission = true (if needs permissions)
- [ ] $specialOperations array defined
- [ ] Translatable fields in $translatable array
- [ ] SoftDeletes trait used
- [ ] CreatedByObserver trait used
- [ ] created_by in $fillable
```
