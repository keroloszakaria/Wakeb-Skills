# Skill: Contract Sync (FE ↔ BE)

Ensures frontend API calls match backend endpoint contracts. Validates that
field names, types, and response structures are aligned between Vue and Laravel.

## When to Use

- After generating both FE and BE code for a feature
- When adding new fields to an existing feature
- When debugging "field not showing" or "undefined" errors
- When the user says data isn't appearing correctly

## Contract Validation Checklist

### 1. Endpoint Alignment

```
FE (jervis-connect / useCrudFactory):
  endpoint: 'employees'
  → GET    /api/employees
  → POST   /api/employees
  → PUT    /api/employees/{id}
  → DELETE /api/employees/{id}

BE (routes/api.php):
  Route::apiResource('employees', EmployeeController::class);
  → Must match the FE endpoint name exactly
```

### 2. Field Name Matching

```
BE Resource:                    FE Schema / Headers:
─────────────────────────────   ─────────────────────────
'id' => $this->id,             key: 'id'
'name' => $this->name,         key: 'name'
'email' => $this->email,       key: 'email'
'status' => $this->status,     key: 'status'
'department' => ...             key: 'department.name'
'created_at' => ...             key: 'created_at'

□ Every FE header key must exist in BE Resource toArray()
□ Nested objects: FE uses dot notation, BE uses nested Resource
□ Dates: BE returns ISO string, FE formats with formatDate()
```

### 3. Request Body Matching

```
FE Form Schema:                 BE Request Rules:
─────────────────────────────   ─────────────────────────
{ key: 'name', type: 'text' }  'name' => 'required|string|max:255'
{ key: 'email', type: 'text' } 'email' => 'required|email'
{ key: 'dept', type: 'select'} 'department_id' => 'required|exists:...'
                                ─────────────────────────
□ FE field key must match BE validation key
□ FE required fields must have 'required' in BE rules
□ FE select/lookup fields send ID, not the display value
```

### 4. Filter/Search Parameters

```
FE useLookupPage config:         BE Filter Pipeline:
─────────────────────────────    ─────────────────────────
search: true                     ->when(request('search'), ...)
filters: [                       ->when(request('status'), ...)
  { key: 'status', ... }         ->when(request('department_id'), ...)
  { key: 'department_id', ... }
]

□ FE filter keys must match BE pipeline request() keys
□ FE search sends 'search' param, BE filters on it
□ FE sort sends 'sort_by' + 'sort_order', BE applies them
```

### 5. Pagination

```
FE expects:                      BE returns:
─────────────────────────────    ─────────────────────────
data: []                         'data' => Resource::collection(...)
meta.current_page                meta from ->paginate()
meta.last_page                   auto-included by Laravel
meta.per_page                    per_page from request
meta.total                       auto-included by Laravel

□ BE must use ->paginate() not ->get()
□ FE useLookupPage handles pagination automatically
□ per_page default: 15 (configurable)
```

### 6. Error Response

```
BE 422 returns:                  FE handleErrors() expects:
─────────────────────────────    ─────────────────────────
{                                GenericForm reads errors object
  "message": "...",              and maps to field keys
  "errors": {
    "email": ["taken"]           → Shows under email field
  }
}

□ BE validation key names match FE form field keys
□ FE handleErrors(error) maps 422 errors to form fields
```

## Sync Procedure

When generating a full feature:

```
1. Define the data model (fields + types)
2. Generate BE: migration → model → resource → request → controller → routes
3. Generate FE: config → store hooks → pages → locale files
4. Run this contract check:
   - Resource fields ↔ header keys
   - Request rules ↔ form schema keys
   - Pipeline params ↔ filter config
   - Route names ↔ endpoint config
5. Flag any mismatches before the user tests
```

## Common Mismatches

```
❌ FE uses 'department' but BE returns 'department_id' (number)
   → BE should return nested: 'department' => new DepartmentResource(...)
   → FE header: key: 'department.name'

❌ FE sends 'dept_id' but BE validates 'department_id'
   → Field keys must match exactly

❌ FE expects array but BE returns paginated object
   → BE must use ->paginate() for list endpoints

❌ FE sends 'is_active' as string "true" but BE validates boolean
   → FE form type should be 'switch' (sends boolean)
   → Or BE rule: 'is_active' => 'required|boolean' with cast
```
