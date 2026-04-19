# Module Scaffold Templates

Complete templates for creating a new module in the Wakeb Backend Starter.

## Table of Contents

1. [Model](#model)
2. [Controller](#controller)
3. [FormRequest](#formrequest)
4. [Resource](#resource)
5. [Migration](#migration)
6. [Routes](#routes)
7. [Filter](#filter)
8. [Policy](#policy)
9. [Seeder](#seeder)
10. [Module Routes (nwidart)](#module-routes-nwidart)

---

## Model

```php
<?php

namespace App\Models;
// For modules: namespace Modules\{Name}\app\Models;

use App\Trait\Global\CreatedByObserver;
use App\Trait\Global\LogsActivityOptions;
use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\SoftDeletes;
use Spatie\Activitylog\LogOptions;
use Spatie\Translatable\HasTranslations;

class {Model} extends BaseModel
{
    use HasTranslations, SoftDeletes, CreatedByObserver, LogsActivityOptions;

    public array $translatable = ['name']; // Add translatable JSON fields
    public bool $inPermission = true;
    public array $basicOperations = ['create', 'update', 'delete'];
    public array $specialOperations = ['view-all', 'view-own', 'force-delete', 'restore', 'toggle-active'];

    protected $fillable = [
        'name',
        'is_active',
        'created_by',
    ];

    protected $casts = [
        'is_active' => 'boolean',
    ];

    /*
    |--------------------------------------------------------------------------
    | Activity logs
    |--------------------------------------------------------------------------
    */
    public function getActivitylogOptions(): LogOptions
    {
        return LogOptions::defaults()->logOnlyDirty()->logOnly($this->fillable);
    }

    /*
    |--------------------------------------------------------------------------
    | Relations
    |--------------------------------------------------------------------------
    */
    public function creator(): BelongsTo
    {
        return $this->belongsTo(User::class, 'created_by');
    }

    /*
    |--------------------------------------------------------------------------
    | Scopes
    |--------------------------------------------------------------------------
    */
    public function scopeActive($query)
    {
        return $query->where('is_active', 1);
    }
}
```

### Translatable Model Notes

- `$translatable` fields are stored as JSON: `{"ar": "اسم", "en": "name"}`
- Access current locale: `$model->name` → returns string in current locale
- Access all translations: `$model->getTranslations('name')` → `{"ar": "...", "en": "..."}`
- Set translation: `$model->setTranslation('name', 'ar', 'اسم')`

### File Upload Model Notes

For models with file uploads, use Media manager:

```php
// Mutator for upload
public function setImageAttribute($value): void
{
    $path = Media::replace($this->image ?? null)->upload($value, 'products');
    $this->attributes['image'] = $path;
}

// Accessor for URL
public function image(): Attribute
{
    return Attribute::make(get: fn($value) => Media::url($value));
}
```

---

## Controller

```php
<?php

namespace App\Http\Controllers\API\DataEntry;
// For modules: namespace Modules\{Name}\app\Http\Controllers\Api\Admin;

use App\Filters\Global\ActiveFilter;
use App\Filters\Global\JsonNameFilter;
use App\Filters\Global\OrderByFilter;
use App\Filters\Global\TrashedFilter;
use App\Http\Controllers\API\BaseController;
use App\Http\Requests\DataEntry\{Model}Request;
use App\Http\Requests\Global\Other\PageRequest;
use App\Http\Resources\DataEntry\{Model}Resource;
use App\Models\{Model};
use App\Trait\Global\HasDeleteMethods;
use App\Trait\Global\HasToggleActiveMethods;
use Illuminate\Http\JsonResponse;
use Illuminate\Pipeline\Pipeline;
use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Gate;
use Spatie\Permission\Middleware\PermissionMiddleware;

class {Model}Controller extends BaseController implements HasMiddleware
{
    use HasDeleteMethods, HasToggleActiveMethods;

    public function __construct()
    {
        parent::__construct();
        $this->model = {Model}::class;
        // Optional: lifecycle callbacks
        // $this->beforeDelete('force', fn({Model} $m) => Media::delete($m->image));
    }

    public static function middleware(): array
    {
        return [
            new Middleware(PermissionMiddleware::using('create-{model}'), only: ['store']),
            new Middleware(PermissionMiddleware::using('update-{model}'), only: ['update']),
        ];
    }

    public function index(PageRequest $request): JsonResponse
    {
        $query = app(Pipeline::class)
            ->send({Model}::query()->with(['creator']))
            ->through([JsonNameFilter::class, ActiveFilter::class, TrashedFilter::class, OrderByFilter::class])
            ->thenReturn();

        return successResponse(wrapPaginate($query, {Model}Resource::class));
    }

    public function store({Model}Request $request): JsonResponse
    {
        Gate::authorize('create', {Model}::class);

        $item = {Model}::create($request->validated());

        return successResponse(new {Model}Resource($item->refresh()), __('api.created_success'));
    }

    public function show({Model} ${model}): JsonResponse
    {
        Gate::authorize('view', ${model});

        return successResponse(new {Model}Resource(${model}));
    }

    public function update({Model}Request $request, {Model} ${model}): JsonResponse
    {
        Gate::authorize('update', ${model});

        ${model}->update($request->validated());

        return successResponse(new {Model}Resource(${model}->refresh()), __('api.updated_success'));
    }
}
```

### Controller with Relations (complex)

For controllers that sync relations on store/update:

```php
public function store({Model}Request $request): JsonResponse
{
    Gate::authorize('create', {Model}::class);

    return DB::transaction(function () use ($request) {
        $item = {Model}::create($request->validated());
        $this->syncRelations($item, $request);

        return successResponse(
            new {Model}Resource($item->refresh()->load('relations')),
            __('api.created_success')
        );
    });
}

private function syncRelations({Model} $item, {Model}Request $request): void
{
    when($request->filled('tags'), fn() => $item->tags()->sync($request->tags));
    when($request->filled('categories'), fn() => $item->categories()->sync($request->categories));
}
```

---

## FormRequest

```php
<?php

namespace App\Http\Requests\DataEntry;
// For modules: namespace Modules\{Name}\app\Http\Requests;

use App\Http\Requests\BaseFormRequest;
use App\Rules\TranslatableRequired;
use App\Rules\UniqueCheck;
use Illuminate\Validation\Rule;
use Illuminate\Validation\Rules\File;

class {Model}Request extends BaseFormRequest
{
    public function rules(): array
    {
        $id = $this->route('{model}')?->getKey();

        return [
            // Translatable field
            'name' => [
                'required',
                'array',
                new UniqueCheck({Model}::class, {Model}Resource::class, $id),
                new TranslatableRequired('{table}', ['string', 'max:191'], '{model}'),
            ],

            // Simple fields
            'code'   => ['required', Rule::unique('{table}', 'code')->withoutTrashed()->ignore($id)],
            'price'  => ['required', 'numeric', 'min:0'],
            'status' => ['sometimes', 'boolean'],

            // File upload
            'image'  => ['sometimes', 'nullable', File::image()->max(20048)],

            // Relation IDs
            'category_id' => ['required', 'numeric', Rule::exists('categories', 'id')->withoutTrashed()],

            // Array of IDs
            'tags'   => ['sometimes', 'nullable', 'array'],
            'tags.*' => ['required', 'numeric', Rule::exists('tags', 'id')],
        ];
    }
}
```

### Validation Rules Cheat Sheet

| Rule                                                                | When to use                                   |
| ------------------------------------------------------------------- | --------------------------------------------- |
| `new TranslatableRequired('table', ['string', 'max:191'], 'model')` | Multi-language JSON field (name, description) |
| `new UniqueCheck(Model::class, Resource::class, $id)`               | Unique check on JSON/translatable field       |
| `Rule::unique('table', 'col')->withoutTrashed()->ignore($id)`       | Unique on simple column                       |
| `Rule::exists('table', 'id')->withoutTrashed()`                     | Foreign key exists and not soft-deleted       |
| `File::image()->max(20048)`                                         | Image upload (20MB max)                       |
| `new StrongPassword`                                                | Strong password enforcement                   |
| `new ValidLength($codeId, Country::class, 'phone_length')`          | Phone length per country                      |

---

## Resource

```php
<?php

namespace App\Http\Resources\DataEntry;
// For modules: namespace Modules\{Name}\app\Http\Resources\Admin;

use App\Http\Resources\Global\Other\BasicUserResource;
use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class {Model}Resource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'         => $this->id,

            // Translatable: current locale + all translations
            'translation_name' => $this->name,
            'name'       => $this->getTranslations('name'),

            // Simple fields
            'code'       => $this->code,
            'price'      => $this->price,
            'image'      => $this->image,
            'is_active'  => $this->is_active,

            // Relations (only if loaded)
            'creator'    => $this->whenLoaded('creator', fn() => new BasicUserResource($this->creator), ['id' => $this->created_by]),
            'category'   => $this->whenLoaded('category', fn() => new BasicResource($this->category)),

            // Timestamps
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

### Resource Patterns

- **Translatable field**: `'translation_name' => $this->name` (current locale) + `'name' => $this->getTranslations('name')` (all)
- **Conditional relation**: `$this->whenLoaded('relation', fn() => new Resource($this->relation), fallback)`
- **Enum display**: `'display_status' => StatusEnum::resolve($this->status)`
- **Nested phone**: `'phone' => ['phone' => $this->phone, 'phone_code' => ..., 'phone_code_id' => ...]`

---

## Migration

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('{table}', function (Blueprint $table) {
            $table->id();

            // Translatable fields use mediumText (JSON)
            $table->mediumText('name');

            // Simple fields
            $table->string('code')->unique();
            $table->decimal('price', 10, 2)->default(0);
            $table->string('image')->nullable();
            $table->boolean('is_active')->default(true);

            // Foreign keys
            $table->foreignId('category_id')->nullable()->constrained('categories')->nullOnDelete();
            $table->foreignId('created_by')->nullable()->constrained('users')->nullOnDelete();

            $table->timestamps();
            $table->softDeletes();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('{table}');
    }
};
```

### Migration Conventions

- Translatable columns → `mediumText` (stores JSON)
- Boolean defaults → `->default(true)` or `->default(false)`
- Foreign keys → `->constrained()->nullOnDelete()` (never cascade delete)
- Always include `timestamps()` + `softDeletes()`
- Always include `created_by` FK to users

---

## Routes

### In `routes/api.php` (for app-level modules)

```php
Route::middleware(['auth:sanctum'])->group(function () {
    Route::prefix('{models}')->group(function () {
        Route::delete('force-delete', [{Model}Controller::class, 'forceDelete']);
        Route::delete('delete', [{Model}Controller::class, 'destroy']);
        Route::post('restore', [{Model}Controller::class, 'restore']);
        Route::put('toggle-active', [{Model}Controller::class, 'toggleActive']);
        Route::apiResource('/', {Model}Controller::class)
            ->parameters(['' => '{model}'])
            ->except(['destroy']);
    });
});
```

### Route naming pattern

| HTTP Method | URI                         | Action       | Notes                |
| ----------- | --------------------------- | ------------ | -------------------- |
| GET         | /api/{models}               | index        | Paginated list       |
| POST        | /api/{models}               | store        | Create               |
| GET         | /api/{models}/{id}          | show         | Single item          |
| PUT/PATCH   | /api/{models}/{id}          | update       | Update               |
| DELETE      | /api/{models}/delete        | destroy      | Soft delete (bulk)   |
| POST        | /api/{models}/restore       | restore      | Restore (bulk)       |
| DELETE      | /api/{models}/force-delete  | forceDelete  | Permanent (bulk)     |
| PUT         | /api/{models}/toggle-active | toggleActive | Toggle status (bulk) |

---

## Filter

```php
<?php

namespace App\Filters\{Feature};

use Closure;

class {Model}Filter
{
    public function handle($request, Closure $next)
    {
        $query = $next($request);

        $query->when(
            request()->has('search') && !empty(request('search')),
            function ($query) {
                $query->where(function ($q) {
                    $q->where('code', 'like', '%' . request('search') . '%')
                      ->orWhere('email', 'like', '%' . request('search') . '%');
                });
            }
        );

        return $query;
    }
}
```

### Available Global Filters

| Filter           | Query param                         | What it does                   |
| ---------------- | ----------------------------------- | ------------------------------ |
| `JsonNameFilter` | `?search=...`                       | Search in JSON `name` field    |
| `ActiveFilter`   | `?is_active=true/false`             | Filter by `is_active` column   |
| `TrashedFilter`  | `?is_trashed=true`                  | Show only soft-deleted records |
| `OrderByFilter`  | `?sort_column=x&sort_direction=asc` | Sort by column                 |
| `DateFilter`     | `?date_from=...&date_to=...`        | Date range filter              |

---

## Policy

```php
<?php

namespace App\Policies\{Feature};

use App\Models\{Model};
use Illuminate\Auth\Access\HandlesAuthorization;
use Illuminate\Foundation\Auth\User as Authenticatable;

class {Model}Policy
{
    use HandlesAuthorization;

    public function view(Authenticatable $user, ?{Model} $model = null): bool
    {
        return $this->canAny($user, $model, ['view-all-{model}', 'view-own-{model}']);
    }

    public function create(Authenticatable $user): bool
    {
        return $user->can('create-{model}');
    }

    public function update(Authenticatable $user, {Model} $model): bool
    {
        return $user->can('update-{model}') && $this->ownsOrAll($user, $model);
    }

    public function delete(Authenticatable $user, ?{Model} $model = null): bool
    {
        return $user->can('delete-{model}') && $this->ownsOrAll($user, $model);
    }

    public function restore(Authenticatable $user): bool
    {
        return $user->can('restore-{model}');
    }

    public function forceDelete(Authenticatable $user, {Model} $model): bool
    {
        return $user->can('force-delete-{model}') && $this->ownsOrAll($user, $model);
    }

    public function toggleActive(Authenticatable $user, {Model} $model): bool
    {
        return $user->can('toggle-active-{model}') && $this->ownsOrAll($user, $model);
    }

    protected function ownsOrAll(Authenticatable $user, ?{Model} $model): bool
    {
        if ($user->can('view-all-{model}')) return true;
        return $model?->created_by === $user->id;
    }

    protected function canAny(Authenticatable $user, ?{Model} $model, array $permissions): bool
    {
        foreach ($permissions as $perm) {
            if ($user->can($perm)) {
                if (str_contains($perm, 'view-own')) {
                    return $model?->created_by === $user->id;
                }
                return true;
            }
        }
        return false;
    }
}
```

---

## Seeder

```php
<?php

namespace Database\Seeders;

use App\Models\{Model};
use Illuminate\Database\Seeder;

class {Model}Seeder extends Seeder
{
    public function run(): void
    {
        {Model}::truncate();

        $items = [
            ['name' => ['ar' => 'عنصر 1', 'en' => 'Item 1'], 'code' => 'ITEM-001', 'is_active' => true],
            ['name' => ['ar' => 'عنصر 2', 'en' => 'Item 2'], 'code' => 'ITEM-002', 'is_active' => true],
        ];

        foreach ($items as $item) {
            {Model}::create($item);
        }
    }
}
```

---

## Module Routes (nwidart)

For modules under `Modules/`, the routes file is at `Modules/{Name}/routes/api.php`:

```php
<?php

use Illuminate\Support\Facades\Route;
use Modules\{Name}\app\Http\Controllers\Api\Admin\{Model}Controller;

Route::middleware(['auth:sanctum'])->group(function () {
    Route::apiResource('{models}', {Model}Controller::class);
    Route::delete('{models}/delete-all', [{Model}Controller::class, 'destroyAll']);
    Route::post('{models}/{id}/restore', [{Model}Controller::class, 'restore']);
    Route::delete('{models}/{id}/force-delete', [{Model}Controller::class, 'forceDelete']);
    Route::post('{models}/{model}/change-status', [{Model}Controller::class, 'changeStatus']);
});
```
