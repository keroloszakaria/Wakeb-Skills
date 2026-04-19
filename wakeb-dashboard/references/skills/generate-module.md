# Skill: generate-module

Scaffolds a complete module for both frontend and backend, fully aligned with
the project's architecture. Uses existing factories, composables, and base
classes — never raw/manual implementations.

## When to Use

- User says "create a module for X"
- User says "scaffold X" or "add X feature"
- User provides a Figma design for a full CRUD entity

## Input

| Field       | Type   | Description                                                      |
| ----------- | ------ | ---------------------------------------------------------------- |
| `name`      | string | Module name in kebab-case (e.g. `user-groups`)                   |
| `fields`    | array  | List of { name, type, required, translatable, enum }             |
| `features`  | array  | Optional: `['soft-delete', 'toggle-active', 'export', 'import']` |
| `relations` | array  | Optional: `[{ type, model, foreign_key }]`                       |

## Execution

### Step 1 — Derive names

From input `name: "user-groups"`:

| Derived           | Value                 |
| ----------------- | --------------------- |
| FE module folder  | `user-groups`         |
| FE store name     | `userGroups`          |
| BE module folder  | `UserGroup`           |
| BE model          | `UserGroup`           |
| BE table          | `user_groups`         |
| BE controller     | `UserGroupController` |
| BE request        | `UserGroupRequest`    |
| BE resource       | `UserGroupResource`   |
| Route prefix      | `user-groups`         |
| Permission prefix | `user-group`          |
| i18n key          | `user_groups`         |

### Step 2 — Generate Backend files

Reference: [module-scaffold.md](../module-scaffold.md) for full templates.

**Migration** — columns from `fields`, always include:

- `$table->id()`
- `$table->boolean('is_active')->default(1)` (if toggle feature)
- `$table->softDeletes()` (if soft-delete feature)
- `$table->timestamps()`

**Model** — extends `BaseModel`, adds:

- `$fillable` from fields
- `$translatable` for translatable fields
- Relevant traits: `SoftDeletes`, `HasTranslations`, `LogsActivityOptions`, `CreatedByObserver`

**Controller** — extends `BaseController`, uses:

- `HasDeleteMethods` + `HasToggleActiveMethods` traits from config
- Pipeline filters for `index()`
- `wrapPaginate()` helper for paginated responses
- `successResponse()` / `failResponse()` for all returns

**Request** — extends `BaseFormRequest`:

- Uses `TranslatableRequired` / `TranslatableNullable` for translatable fields
- Uses `StrongPassword` for password fields
- Uses `UniqueCheck` for unique fields

**Resource** — extends `JsonResource`:

- Resolves translations via `resolveTrans()`
- Resolves photos via `resolvePhoto()`

**Filters** — Create filters for searchable/filterable fields:

- Reuse existing filters from [codebase-index.md](codebase-index.md) when possible
- Only create new filters for fields not covered

**Routes** — in `routes/api.php`:

```php
Route::middleware('auth:sanctum')->prefix('admin')->group(function () {
    Route::apiResource('user-groups', UserGroupController::class);
    // Additional routes for toggle, batch delete, etc.
});
```

**Seeder** — permission seeder:

```php
$permissions = ['view-user-group', 'create-user-group', 'update-user-group', 'delete-user-group'];
```

### Step 3 — Generate Frontend files

Reference: [module-scaffold.md](../module-scaffold.md) for full templates.

**config.js** — uses `createModuleConfig()`:

```js
export default createModuleConfig({
  module: "userGroups",
  permission: "user-group",
  apiEndpoint: "/admin/user-groups",
  deletable: true,
  toggleable: true,
  headers: [
    /* column config */
  ],
  enums: {
    /* if any */
  },
  models: {
    /* if any relations */
  },
});
```

**router/index.js** — route with permission guard:

```js
export default [
  {
    path: "/user-groups",
    name: "userGroups",
    component: () => import("../views/index.vue"),
    meta: { permission: "view-user-group" },
  },
];
```

**stores/{name}.js** — uses `useCrudFactory()`:

```js
import { useCrudFactory } from "@/factories/BaseCrudFactory";
import config from "../config";

export const useUserGroupsStore = useCrudFactory(config);
```

**schema/index.js** — uses FieldUtils:

```js
import { createTextField, createSelectField } from "@/utils/FieldUtils";

export default function (isView) {
  return [
    createTextField({ key: "name", required: true, translatable: true }),
    // ...more fields from input
  ];
}
```

**views/index.vue** — uses `useLookupPage()`:

```vue
<script setup>
import { useLookupPage } from "@/composables/useLookupPage";
import config from "../config";

const IndexPage = useThemedComponent("IndexPage");
const {
  /* destructured props */
} = await useLookupPage({
  /* config */
});
</script>
```

**locales/ar.json + en.json** — translations for all field labels and module title

### Step 4 — Validate with enforce-structure

Run [enforce-structure.md](enforce-structure.md) checklist on all generated files.

## Output

Complete set of files for both FE and BE, listed in creation order with full
content. Each file uses the project's base classes, factories, and patterns.

## Rules

1. NEVER write raw CRUD logic — always use `useCrudFactory` (FE) and `BaseController` (BE)
2. NEVER write raw form inputs — always use FieldUtils creators
3. NEVER write pagination logic — always use `useLookupPage` (FE) and `wrapPaginate` (BE)
4. NEVER create filters that already exist — check [codebase-index.md](codebase-index.md)
5. Every translatable field uses `resolveTrans()` in Resource and `TranslatableRequired` in Request
6. All API responses use `successResponse()` / `failResponse()`
7. Permissions follow `{action}-{model}` pattern: view, create, update, delete
