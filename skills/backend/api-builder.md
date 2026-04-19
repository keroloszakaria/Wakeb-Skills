# Skill: API Builder

Builds REST API endpoints using the Wakeb Starter BaseController, Pipeline,
and resource patterns. Handles standard CRUD and custom endpoints.

## When to Use

- When creating API endpoints for a module
- When adding custom actions beyond CRUD
- When the frontend needs a new endpoint
- When designing API contracts

## Standard CRUD (Automatic via BaseController)

BaseController provides these endpoints automatically:

| Method | Route                | Action  | Description   |
| ------ | -------------------- | ------- | ------------- |
| GET    | /api/{resource}      | index   | List + filter |
| POST   | /api/{resource}      | store   | Create        |
| GET    | /api/{resource}/{id} | show    | Single item   |
| PUT    | /api/{resource}/{id} | update  | Update item   |
| DELETE | /api/{resource}/{id} | destroy | Soft delete   |

The controller constructor configures everything:

```php
public function __construct()
{
    $this->model = new Employee();
    $this->resource = EmployeeResource::class;
    $this->request = EmployeeRequest::class;
    $this->pipelines = [
        FilterEmployee::class,
        SortEmployee::class,
    ];
}
```

## Custom Endpoints

### Adding Actions to Controller

```php
class EmployeeController extends BaseController
{
    public function __construct() { /* ... */ }

    // Custom action: POST /api/employees/{id}/activate
    public function activate($id)
    {
        $employee = Employee::findOrFail($id);
        $employee->update(['status' => 'active']);
        return new EmployeeResource($employee);
    }

    // Custom action: GET /api/employees/export
    public function export()
    {
        $employees = Employee::active()->get();
        // export logic
        return response()->download($filePath);
    }

    // Override index for custom behavior
    public function index()
    {
        $query = Employee::with(['department', 'manager']);
        $data = $this->applyPipelines($query)->paginate(request('per_page', 15));
        return EmployeeResource::collection($data);
    }
}
```

### Custom Routes

```php
Route::middleware(['auth:sanctum'])->group(function () {
    Route::apiResource('employees', EmployeeController::class);

    // Custom routes — define BEFORE apiResource if they conflict
    Route::post('employees/{id}/activate', [EmployeeController::class, 'activate']);
    Route::get('employees/export', [EmployeeController::class, 'export']);
});
```

## Pipeline Patterns

### Filter Pipeline

```php
class FilterEmployee extends BasePipeline
{
    protected function apply($query)
    {
        return $query
            ->when(request('search'), function ($q, $search) {
                $q->where(function ($q) use ($search) {
                    $q->where('name', 'like', "%{$search}%")
                      ->orWhere('email', 'like', "%{$search}%");
                });
            })
            ->when(request('status'), function ($q, $status) {
                $q->where('status', $status);
            })
            ->when(request('department_id'), function ($q, $deptId) {
                $q->where('department_id', $deptId);
            })
            ->when(request('date_from'), function ($q, $date) {
                $q->whereDate('created_at', '>=', $date);
            })
            ->when(request('date_to'), function ($q, $date) {
                $q->whereDate('created_at', '<=', $date);
            });
    }
}
```

### Sort Pipeline

```php
class SortEmployee extends BasePipeline
{
    protected array $sortable = ['name', 'email', 'created_at', 'status'];

    protected function apply($query)
    {
        $sortBy = request('sort_by', 'created_at');
        $sortOrder = request('sort_order', 'desc');

        if (in_array($sortBy, $this->sortable)) {
            return $query->orderBy($sortBy, $sortOrder);
        }

        return $query->latest();
    }
}
```

## Request Validation

```php
class EmployeeRequest extends BaseRequest
{
    public function rules(): array
    {
        $rules = [
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:employees,email',
            'department_id' => 'required|exists:departments,id',
            'status' => 'in:active,inactive,suspended',
        ];

        // On update, ignore current record for unique check
        if ($this->isMethod('PUT')) {
            $rules['email'] = 'required|email|unique:employees,email,' . $this->route('employee');
        }

        return $rules;
    }
}
```

## Resource Response

```php
class EmployeeResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'status' => $this->status,
            'department' => new DepartmentResource($this->whenLoaded('department')),
            'permissions' => $this->whenLoaded('permissions', function () {
                return $this->permissions->pluck('name');
            }),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

## API Response Format

```json
// Success (single)
{
  "data": { "id": 1, "name": "..." }
}

// Success (list)
{
  "data": [{ "id": 1 }, { "id": 2 }],
  "meta": {
    "current_page": 1,
    "last_page": 5,
    "per_page": 15,
    "total": 72
  }
}

// Validation Error (422)
{
  "message": "The given data was invalid.",
  "errors": {
    "email": ["The email has already been taken."]
  }
}
```

## Rules

```
□ Use BaseController for standard CRUD — do not rewrite index/store/update/destroy
□ Custom endpoints in controller methods + explicit routes
□ Pipeline classes for filtering/sorting — never filter in controller
□ Request validation in FormRequest classes — never validate in controller
□ Resource classes for response shaping — never return raw models
□ Auth middleware: auth:sanctum on all routes
□ Permission middleware: ->can('permission-name') where needed
□ Eager load relationships in Resource with whenLoaded()
□ Paginate list endpoints — never return all records
```
