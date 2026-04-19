# Validation Reference

## BaseFormRequest

All requests extend `BaseFormRequest` which provides:

```php
abstract class BaseFormRequest extends FormRequest
{
    // Converts empty strings/arrays to null before validation
    protected function prepareForValidation(): void
    {
        $this->replace(collect($this->all())
            ->map(fn($value) => resolveEmptyToNull($value))
            ->toArray());
    }

    // Normalizes boolean inputs (0, 1, 'true', 'false', true, false)
    protected function booleanInput(string $key, bool $default = false): bool;

    // Returns JSON error (422) instead of redirect
    protected function failedValidation(Validator $validator)
    {
        $errors = (new ValidationException($validator))->errors();
        $firstMessage = collect($errors)->flatten()->first();
        throw new HttpResponseException(response()->json([
            'message' => $firstMessage,
            'errors' => $errors,
        ], 422));
    }
}
```

## Custom Validation Rules

### TranslatableRequired

Validates translatable JSON fields for each language:

```php
'name' => [
    'required',
    'array',
    new TranslatableRequired('table_name', ['string', 'max:191'], 'model_name'),
]
```

- First param: table name for unique checks
- Second param: rules to apply to each language value
- Third param: model name for error messages

### UniqueCheck

Custom unique validation with soft-delete and JSON field awareness:

```php
'name' => [
    'required',
    'array',
    new UniqueCheck(
        Model::class,              // Model to check against
        ModelResource::class,      // Resource class (returned in error for existing record)
        $this->route('model')?->id // Ignore ID for updates
    ),
]
```

- Throws `ModelAlreadyExistsException` with the existing resource in the response
- Handles JSON/translatable fields
- Respects soft deletes

### StrongPassword

Enforces password complexity:

```php
'password' => ['required', 'confirmed', 'min:8', new StrongPassword],
```

Checks: uppercase, lowercase, digit, special char, no repeats (aaa), no sequences (abc), no dictionary words.

Only applied when `config('project.auth.strong_password')` is true.

### ValidLength

Validates phone number length based on country:

```php
'phone' => ['nullable', 'regex:/^[0-9]+$/', new ValidLength($this->input('phone_code_id'), Country::class, 'phone_length')],
```

### CheckSamePassword

Ensures new password differs from current:

```php
'password' => [new CheckSamePassword],
```

### TotalFileSize

Validates total size of multiple file uploads:

```php
'attachments' => [new TotalFileSize(maxMB: 50)],
```

## Common Validation Patterns

### Update with unique ignore

```php
$id = $this->route('product')?->getKey();

'email' => ['required', 'email', Rule::unique('users', 'email')->withoutTrashed()->ignore($id)],
'code'  => ['required', Rule::unique('products', 'code')->withoutTrashed()->ignore($id)],
```

### Foreign key exists (not soft-deleted)

```php
'category_id' => ['required', 'numeric', Rule::exists('categories', 'id')->withoutTrashed()],
```

### File upload

```php
'image'  => ['sometimes', 'nullable', File::image()->max(20048)],  // 20MB
'avatar' => ['sometimes', 'nullable', File::image()->max(20048)],
'file'   => ['sometimes', 'nullable', File::types(['pdf', 'docx'])->max(20048)],
```

### Enum validation

```php
use Illuminate\Validation\Rules\Enum;
use App\Enum\User\UserGenderEnum;

'gender' => ['required', new Enum(UserGenderEnum::class)],
```

### Array of relation IDs

```php
'roles'   => ['required', 'array', 'min:1'],
'roles.*' => ['required', 'numeric', Rule::exists('roles', 'id')->whereNot('name', 'root')],

'permissions'   => ['sometimes', 'nullable', 'array', 'distinct', 'min:1'],
'permissions.*' => ['required_with:permissions', Rule::exists('permissions', 'id')],
```

### Conditional rules

```php
'steps'          => ['required_if:has_steps,true', 'array'],
'steps.*.name'   => ['required_if:has_steps,true', 'array'],
'steps.*.fields' => ['required_if:has_steps,true', 'array'],
```
