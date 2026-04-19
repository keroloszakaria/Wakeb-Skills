# Patterns & Conventions Reference

## Table of Contents

1. [Authentication Flow](#authentication-flow)
2. [Authorization Pattern](#authorization-pattern)
3. [Controller Traits](#controller-traits)
4. [Helper Functions](#helper-functions-appphp)
5. [Notification System](#notification-system)
6. [Settings Service](#settings-service)
7. [Enums](#enums)
8. [config/project.php](#configprojectphp)
9. [Export & Report](#export--report)
10. [Debugging Checklist](#debugging-checklist)
11. [Code Style Summary](#code-style-summary)

---

## Authentication Flow

### Token-based (Sanctum)

```php
// LoginService handles authentication
class LoginService
{
    public function login(array $credentials): array
    {
        $user = $this->attempt($credentials);
        $token = $user->createToken('admin_api_token')->plainTextToken;

        return [
            'user'  => new UserResource($user),
            'token' => $token,
        ];
    }
}
```

### LDAP Authentication (optional)

Enabled via `config('project.auth.ldap_mode')`:

```php
if (config('project.auth.ldap_mode')) {
    $ldapUser = LdapService::authenticate($credentials);
    $user = User::updateOrCreate(['email' => $ldapUser->email], [...]);
}
```

### OTP Flow

Two-step: 1) Login → returns `otp_token`, 2) Verify OTP → returns `api_token`.

Enabled per-user or globally via `config('project.auth.otp_enabled')`.

## Authorization Pattern

### Gate + Policy + Spatie

```php
// In controller:
Gate::authorize('create', Product::class);
Gate::authorize('update', $product);
Gate::authorize('view', $product);
Gate::authorize('delete', $product);
```

Gate checks flow:

1. Policy method (if registered in `AuthServiceProvider`)
2. Spatie permission fallback (e.g., `create-product`)

### Middleware-based permission

```php
public static function middleware(): array
{
    return [
        new Middleware(PermissionMiddleware::using('create-product'), only: ['store']),
        new Middleware(PermissionMiddleware::using('update-product'), only: ['update']),
    ];
}
```

**When to use which:**

- `Gate::authorize()` — for actions inside controller methods (show, store, update)
- `PermissionMiddleware` — for route-level protection (blocks request before reaching method)

## Controller Traits

### HasDeleteMethods

Provides `destroy()`, `forceDelete()`, `restore()` methods:

```php
trait HasDeleteMethods
{
    /**
     * Soft delete (bulk).
     * DELETE /models/delete  Body: { "ids": [1, 2, 3] }
     */
    public function destroy(Request $request): JsonResponse
    {
        $items = $this->model::whereIn('id', $request->ids)->get();
        foreach ($items as $item) {
            Gate::authorize('delete', $item);
            $item->delete();
        }
        return successResponse(null, __('api.deleted_success'));
    }

    /**
     * Force delete (bulk).
     * DELETE /models/force-delete  Body: { "ids": [1, 2, 3] }
     */
    public function forceDelete(Request $request): JsonResponse
    {
        $items = $this->model::withTrashed()->whereIn('id', $request->ids)->get();
        foreach ($items as $item) {
            Gate::authorize('forceDelete', $item);
            $this->runBeforeDelete('force', $item);
            $item->forceDelete();
        }
        return successResponse(null, __('api.force_deleted_success'));
    }

    /**
     * Restore soft-deleted (bulk).
     * POST /models/restore  Body: { "ids": [1, 2, 3] }
     */
    public function restore(Request $request): JsonResponse
    {
        $items = $this->model::withTrashed()->whereIn('id', $request->ids)->get();
        foreach ($items as $item) {
            Gate::authorize('restore', $item);
            $item->restore();
        }
        return successResponse(null, __('api.restored_success'));
    }
}
```

### HasToggleActiveMethods

```php
trait HasToggleActiveMethods
{
    /**
     * Toggle is_active (bulk).
     * PUT /models/toggle-active  Body: { "ids": [1, 2, 3] }
     */
    public function toggleActive(Request $request): JsonResponse
    {
        $items = $this->model::whereIn('id', $request->ids)->get();
        foreach ($items as $item) {
            Gate::authorize('toggleActive', $item);
            $item->update(['is_active' => !$item->is_active]);
        }
        return successResponse(null, __('api.toggle_active_success'));
    }
}
```

### Lifecycle hooks in traits

```php
public function __construct()
{
    parent::__construct();
    $this->model = Product::class;

    // Run cleanup before force-deleting
    $this->beforeDelete('force', function (Product $product) {
        Media::delete($product->getRawOriginal('image'));
        $product->items()->forceDelete();
    });
}
```

## Helper Functions (App.php)

```php
// Response helpers
successResponse($data = null, $msg = '', $code = 200): JsonResponse
failResponse($msg = '', $data = [], $code = 400): JsonResponse

// Pagination helper — paginates query and wraps in resource
wrapPaginate($query, $resourceClass, $perPage = null): array
// Returns: ['data' => ResourceCollection, 'meta' => pagination_meta]

// Setting helpers
getSetting('key', $default = null)         // Cached settings
getSettingMedia('key')                     // Settings with file URLs

// Auth helpers
getAuthUser()                              // Current authenticated user
getAuthGuard()                             // Current guard name

// Media helpers
resolveEmptyToNull($value)                 // Convert empty to null
```

## Notification System

### Sending Notifications

```php
// From model with ApplyNotification trait:
$model->sendNotification(
    [
        'user_ids' => [$userId1, $userId2],
        'title'    => 'New submission',
        'body'     => 'A new form was submitted',
        'data'     => ['form_id' => $form->id],
    ],
    ['email', 'realtime', 'notify', 'sms']
);
```

### Available Channels

| Channel    | Mechanism                                |
| ---------- | ---------------------------------------- |
| `email`    | Dispatches `SendEmailJob` (queued)       |
| `sms`      | Dispatches `SendSMSJob` (queued)         |
| `realtime` | Broadcasts via Laravel Echo / Reverb     |
| `notify`   | Stores in `notifications` table (in-app) |

### Notification Events (Reverb)

```php
class PusherEvent implements ShouldBroadcast
{
    public function broadcastOn(): Channel
    {
        return new PrivateChannel('notification.' . $this->userId);
    }
}
```

## Settings Service

Centralized settings stored in DB, cached:

```php
// Get a setting value
$value = getSetting('site_name', 'Default');

// Get media setting (returns full URL)
$logo = getSettingMedia('site_logo');

// Settings service (used internally)
SettingService::get('key');
SettingService::set('key', 'value');
```

## Enums

Use PHP backed enums with `EnumMethods` trait:

```php
<?php

namespace App\Enum\User;

use App\Trait\Global\EnumMethods;

enum UserGenderEnum: string
{
    use EnumMethods;

    case MALE = 'male';
    case FEMALE = 'female';

    // EnumMethods provides:
    // ::values()      → ['male', 'female']
    // ::resolve($val) → 'Male' (human-readable)
    // ::all()         → [['value' => 'male', 'label' => 'Male'], ...]
}
```

### Validation with enums:

```php
'gender' => ['required', new Enum(UserGenderEnum::class)],
```

## config/project.php

Central project configuration:

```php
return [
    'auth' => [
        'strong_password' => env('STRONG_PASSWORD', false),
        'otp_enabled'     => env('OTP_ENABLED', false),
        'ldap_mode'       => env('LDAP_MODE', false),
        'max_login_attempts' => 5,
    ],
    'mail' => [
        'is_active' => env('MAIL_IS_ACTIVE', false),
    ],
    'sms' => [
        'is_active' => env('SMS_IS_ACTIVE', false),
    ],
];
```

## Export & Report

Using `hasanhawary/export-builder` and `hasanhawary/report-builder`:

```php
// Export controller pattern
class ExportController extends BaseController
{
    public function export(ExportRequest $request): JsonResponse
    {
        $query = Pipeline::send(Model::query())
            ->through([...filters...])
            ->thenReturn();

        $file = ExportBuilder::make($query, ModelResource::class)
            ->columns($request->columns)
            ->export($request->format); // xlsx, csv, pdf

        return successResponse(['url' => $file]);
    }
}
```

## Debugging Checklist

When debugging 500 errors or unexpected behavior:

1. Check `storage/logs/laravel.log` for stack trace
2. Verify model `$fillable` includes all fields being mass-assigned
3. Verify request `rules()` matches the expected payload
4. Check `Gate::authorize()` — does the user have the required permission?
5. For translatable errors: ensure `$translatable` array matches JSON columns
6. For Pipeline errors: verify filter classes exist and are `use`d correctly
7. For 422 errors: `BaseFormRequest` converts empty → null, check nullable rules
8. For auth errors: check Sanctum token, `auth:sanctum` middleware on route

## Code Style Summary

| Aspect      | Convention                                  |
| ----------- | ------------------------------------------- |
| Namespace   | PSR-4, grouped by domain                    |
| Indentation | 4 spaces (PSR-12)                           |
| Types       | PHP 8.3+ types everywhere                   |
| Returns     | Always `JsonResponse` on controllers        |
| Comments    | Section headers with Laravel-style dividers |
| Imports     | Fully qualified, alphabetical, grouped      |
| Variables   | camelCase for variables, snake_case for DB  |
| Constants   | UPPER_SNAKE in enums and config             |
