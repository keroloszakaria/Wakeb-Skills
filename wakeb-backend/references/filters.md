# Pipeline Filters Reference

## How Filtering Works

The project uses Laravel's `Pipeline` to compose query logic from independent
filter classes. Each filter receives the query, applies its logic, and passes
it to the next filter.

```php
$query = app(Pipeline::class)
    ->send(Model::query()->with(['relations']))
    ->through([Filter1::class, Filter2::class, ...])
    ->thenReturn();
```

## Built-in Global Filters

### JsonNameFilter

Searches in JSON `name` field across all languages:

```php
// app/Filters/Global/JsonNameFilter.php
class JsonNameFilter
{
    public function handle($request, Closure $next)
    {
        $query = $next($request);
        $search = request('search');
        when($search, static fn() => QueryHelper::applyJsonSearch($query, 'name', $search));
        return $query;
    }
}
```

Query param: `?search=keyword`

### ActiveFilter

Filters by `is_active` column:

```php
class ActiveFilter
{
    public function handle($request, Closure $next)
    {
        $query = $next($request);
        $query->when(
            request()->has('is_active'),
            fn($q) => $q->where('is_active', (bool) request('is_active')),
        );
        return $query;
    }
}
```

Query param: `?is_active=true` or `?is_active=false`

### TrashedFilter

Shows only soft-deleted records:

```php
class TrashedFilter
{
    public function handle($request, Closure $next)
    {
        $query = $next($request);
        $query->when(request('is_trashed', false), fn($q) => $q->onlyTrashed());
        return $query;
    }
}
```

Query param: `?is_trashed=true`

### OrderByFilter

Sorts by any column with direction:

```php
class OrderByFilter
{
    public function handle($request, Closure $next)
    {
        $query = $next($request);
        $sortColumn = $this->resolveSortColumn($table, request('sort_column', 'id'));
        $sortDirection = $this->resolveSortDirection(request('sort_direction'));
        return $query->orderBy($sortColumn, $sortDirection);
    }
}
```

Query params: `?sort_column=name&sort_direction=asc`

Features:

- Handles JSON dot notation: `name.en` → `name->en`
- Falls back to `id` if column doesn't exist
- Validates direction is `asc` or `desc`

### DateFilter

Filters by date range:

Query params: `?date_from=2024-01-01&date_to=2024-12-31`

## Creating a Custom Filter

```php
<?php

namespace App\Filters\Product;

use Closure;

class ProductFilter
{
    public function handle($request, Closure $next)
    {
        $query = $next($request);

        // Search by multiple fields
        $query->when(
            request()->has('search') && !empty(request('search')),
            function ($query) {
                $search = request('search');
                $query->where(function ($q) use ($search) {
                    $q->where('code', 'like', "%{$search}%")
                      ->orWhere('sku', 'like', "%{$search}%");
                });
            }
        );

        // Filter by category
        $query->when(
            request()->has('category_id'),
            fn($q) => $q->where('category_id', request('category_id'))
        );

        // Filter by price range
        $query->when(request('min_price'), fn($q) => $q->where('price', '>=', request('min_price')));
        $query->when(request('max_price'), fn($q) => $q->where('price', '<=', request('max_price')));

        return $query;
    }
}
```

### Register in Controller

```php
public function index(PageRequest $request): JsonResponse
{
    $query = app(Pipeline::class)
        ->send(Product::query()->with(['creator', 'category']))
        ->through([
            ProductFilter::class,   // Custom filter first
            ActiveFilter::class,    // Then global filters
            TrashedFilter::class,
            OrderByFilter::class,
        ])
        ->thenReturn();

    return successResponse(wrapPaginate($query, ProductResource::class));
}
```

## Filter Rules

1. Always receive `$request` and `$next` — call `$next($request)` first
2. Use `$query->when()` — only apply filter if the query param is present
3. Wrap multi-field search in a `where(function($q) { ... })` to avoid OR leaks
4. Place custom filters before global filters in the `through()` array
5. Keep each filter focused on one concern
