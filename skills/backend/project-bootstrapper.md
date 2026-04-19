# Skill: Project Bootstrapper

Scaffolds new Laravel modules with the correct 8-file structure using
nwidart/laravel-modules conventions and Wakeb Starter patterns.

## When to Use

- When the user asks to create a new module/feature
- When starting a new CRUD resource
- When setting up backend for a frontend module

## Module Structure

Every module follows this exact structure:

```
Modules/{ModuleName}/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   └── {ModuleName}Controller.php
│   │   ├── Requests/
│   │   │   └── {ModuleName}Request.php
│   │   └── Resources/
│   │       └── {ModuleName}Resource.php
│   ├── Models/
│   │   └── {ModuleName}.php
│   ├── Pipelines/
│   │   ├── Filter{ModuleName}.php
│   │   └── Sort{ModuleName}.php
│   └── Providers/
│       └── {ModuleName}ServiceProvider.php
├── database/
│   ├── factories/
│   ├── migrations/
│   │   └── YYYY_MM_DD_create_{table_name}_table.php
│   └── seeders/
│       └── {ModuleName}Seeder.php
├── routes/
│   └── api.php
├── config/
├── resources/
└── module.json
```

## Scaffolding Command

```bash
php artisan module:make {ModuleName}
```

Then generate the 8 core files:

### 1. Model

```php
<?php

namespace Modules\{ModuleName}\app\Models;

use App\Traits\HasBasicFilter;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class {ModuleName} extends Model
{
    use HasFactory, SoftDeletes, HasBasicFilter;

    protected $fillable = [
        // columns
    ];

    protected $casts = [
        // type casts
    ];

    // Relationships
}
```

### 2. Controller (extends BaseController)

```php
<?php

namespace Modules\{ModuleName}\app\Http\Controllers;

use App\Http\Controllers\BaseController;
use Modules\{ModuleName}\app\Models\{ModuleName};
use Modules\{ModuleName}\app\Http\Requests\{ModuleName}Request;
use Modules\{ModuleName}\app\Http\Resources\{ModuleName}Resource;
use Modules\{ModuleName}\app\Pipelines\Filter{ModuleName};
use Modules\{ModuleName}\app\Pipelines\Sort{ModuleName};

class {ModuleName}Controller extends BaseController
{
    public function __construct()
    {
        $this->model = new {ModuleName}();
        $this->resource = {ModuleName}Resource::class;
        $this->request = {ModuleName}Request::class;
        $this->pipelines = [
            Filter{ModuleName}::class,
            Sort{ModuleName}::class,
        ];
    }
}
```

### 3. Request (Form Validation)

```php
<?php

namespace Modules\{ModuleName}\app\Http\Requests;

use App\Http\Requests\BaseRequest;

class {ModuleName}Request extends BaseRequest
{
    public function rules(): array
    {
        return [
            'name' => 'required|string|max:255',
            // validation rules
        ];
    }
}
```

### 4. Resource (API Response)

```php
<?php

namespace Modules\{ModuleName}\app\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class {ModuleName}Resource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            // map fields
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

### 5. Filter Pipeline

```php
<?php

namespace Modules\{ModuleName}\app\Pipelines;

use App\Pipelines\BasePipeline;

class Filter{ModuleName} extends BasePipeline
{
    protected function apply($query)
    {
        return $query->when(request('search'), function ($q, $search) {
            $q->where('name', 'like', "%{$search}%");
        });
    }
}
```

### 6. Sort Pipeline

```php
<?php

namespace Modules\{ModuleName}\app\Pipelines;

use App\Pipelines\BasePipeline;

class Sort{ModuleName} extends BasePipeline
{
    protected function apply($query)
    {
        return $query->orderBy(
            request('sort_by', 'created_at'),
            request('sort_order', 'desc')
        );
    }
}
```

### 7. Migration

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('{table_name}', function (Blueprint $table) {
            $table->id();
            // columns
            $table->timestamps();
            $table->softDeletes();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('{table_name}');
    }
};
```

### 8. Routes

```php
<?php

use Illuminate\Support\Facades\Route;
use Modules\{ModuleName}\app\Http\Controllers\{ModuleName}Controller;

Route::middleware(['auth:sanctum'])->group(function () {
    Route::apiResource('{route-name}', {ModuleName}Controller::class);
});
```

## Rules

```
□ Module name: PascalCase singular (e.g., Employee, FlightPlan)
□ Table name: snake_case plural (e.g., employees, flight_plans)
□ Route name: kebab-case plural (e.g., employees, flight-plans)
□ Controller extends BaseController — provides index/show/store/update/destroy
□ Request extends BaseRequest — provides common validation patterns
□ Model uses SoftDeletes + HasBasicFilter traits
□ Pipeline classes for filtering and sorting
□ Routes use auth:sanctum middleware
□ Resource maps model fields to API response
```
