# Skill: Security Scanner

Validates generated backend code against OWASP Top 10 and Wakeb Starter
security patterns. Runs automatically after code generation.

## When to Use

- After generating any backend code (automatic check)
- When reviewing existing code for security issues
- When the user asks about security best practices
- When implementing authentication or authorization features

## Security Checklist

### Authentication (Sanctum)

```
□ All API routes behind auth:sanctum middleware
□ Token abilities defined for different access levels
□ Token expiration configured in config/sanctum.php
□ Login rate limiting applied (throttle:login middleware)
□ Password hashing via Hash::make() — NEVER store plain text
□ Logout revokes tokens: $request->user()->currentAccessToken()->delete()
```

### Authorization (Spatie Permission)

```
□ Permissions assigned to roles, not directly to users
□ Controller actions check permissions:
  - Middleware: ->middleware('permission:employees.view')
  - In controller: $this->authorize('employees.create')
  - In policy: Gate::authorize('update', $employee)

□ Resource-level authorization:
  - Users can only access their own data (unless admin)
  - Scope queries: Employee::where('company_id', auth()->user()->company_id)

□ Super-admin bypass: handled by Spatie — do not hardcode admin checks
```

### Input Validation

```
□ ALL input validated in FormRequest classes:
  EmployeeRequest extends BaseRequest with rules()

□ String inputs:
  'name' => 'required|string|max:255'        ✅
  'name' => 'required'                        ❌ (no type, no max)

□ Email inputs:
  'email' => 'required|email:rfc,dns'         ✅

□ Numeric inputs:
  'amount' => 'required|numeric|min:0|max:999999'  ✅

□ File uploads:
  'avatar' => 'nullable|image|max:2048|mimes:jpg,png,webp'  ✅

□ Foreign keys:
  'department_id' => 'required|exists:departments,id'  ✅

□ Enum values:
  'status' => 'required|in:active,inactive,suspended'  ✅

□ JSON/Array:
  'metadata' => 'nullable|array'
  'metadata.*.key' => 'required|string'       ✅
```

### SQL Injection Prevention

```
□ Use Eloquent ORM — NEVER raw SQL with user input:
  Employee::where('name', $request->name)                    ✅
  DB::select("SELECT * FROM employees WHERE name = '$name'") ❌

□ If raw query is needed, use parameter binding:
  DB::select('SELECT * FROM employees WHERE name = ?', [$name])  ✅

□ LIKE queries use parameter binding:
  ->where('name', 'like', '%' . $search . '%')               ✅
  ->whereRaw("name LIKE '%$search%'")                        ❌
```

### Mass Assignment Protection

```
□ Model $fillable explicitly lists allowed fields:
  protected $fillable = ['name', 'email', 'status'];         ✅

□ Sensitive fields NEVER in $fillable:
  - id, password (except in User model for registration)
  - role, is_admin, permissions
  - created_at, updated_at, deleted_at
```

### XSS Prevention

```
□ API responses use Resource classes (auto-escapes in JSON)
□ File upload content-type validation (mimes rule)
□ User-generated HTML sanitized if stored:
  use strip_tags() or HTMLPurifier for rich text
```

### CSRF / CORS

```
□ Sanctum handles CSRF for SPA mode
□ CORS configured in config/cors.php:
  - allowed_origins: specific domains, NOT '*' in production
  - allowed_methods: specific methods needed
  - supports_credentials: true for Sanctum SPA
```

### Rate Limiting

```
□ Login endpoint: throttle:5,1 (5 attempts per minute)
□ API endpoints: throttle:api (60 per minute default)
□ Sensitive actions: custom rate limiter
  RateLimiter::for('exports', function ($request) {
      return Limit::perMinute(5)->by($request->user()->id);
  });
```

### File Upload Security

```
□ Validate file type (mimes rule)
□ Validate file size (max rule in KB)
□ Store in non-public directory when possible
□ Generate random filenames — never use original filename
□ Scan for malware if processing uploads
  $path = $request->file('avatar')->store('avatars', 'private');  ✅
```

### Sensitive Data

```
□ Model $hidden for sensitive fields:
  protected $hidden = ['password', 'remember_token'];

□ Env variables for secrets — NEVER hardcode:
  env('API_KEY')                    ✅
  $apiKey = 'sk-abc123...'         ❌

□ Logs do not contain sensitive data:
  Log::info('User login', ['user_id' => $user->id]);      ✅
  Log::info('User login', ['password' => $password]);      ❌
```

## Severity Levels

```
🔴 CRITICAL (block generation):
  - Missing auth middleware
  - Raw SQL with user input
  - Hardcoded secrets
  - Missing input validation

🟡 WARNING (flag for review):
  - Missing rate limiting
  - Overly broad CORS
  - Missing file type validation
  - Mass assignment concerns

🟢 INFO (suggest improvement):
  - Missing $hidden on model
  - Broad permission checks
  - Missing logging for sensitive actions
```
