# Skill: connect-api

Binds frontend components with backend API endpoints correctly, using the
project's existing HTTP layer, store factories, and response handling patterns.

## When to Use

- When connecting a frontend page to a backend endpoint
- When configuring a module's store for API calls
- When adding a new API call to an existing module
- When the user says "connect this to the API" or "wire up the backend"

## Input

| Field             | Type   | Description                                              |
| ----------------- | ------ | -------------------------------------------------------- |
| `endpoint`        | string | Backend route (e.g. `/admin/user-groups`)                |
| `module`          | string | Frontend module name                                     |
| `operations`      | array  | `['index', 'store', 'show', 'update', 'destroy']`        |
| `extra_endpoints` | array  | Optional: `[{ method, path, name }]` for non-CRUD routes |

## Execution

### Step 1 — Verify backend endpoint exists

Check [wakeb-backend codebase-index](../../wakeb-backend/references/codebase-index.md):

- Controller exists with required methods
- Routes are registered in `routes/api.php`
- Request validation matches frontend form schema
- Resource transforms match frontend display needs

### Step 2 — Configure frontend store

The project uses `useCrudFactory()` which auto-generates these API methods:

| Factory Method         | HTTP   | Endpoint                |
| ---------------------- | ------ | ----------------------- |
| `getItems()`           | GET    | `/{prefix}`             |
| `getItem(id)`          | GET    | `/{prefix}/{id}`        |
| `createItem(data)`     | POST   | `/{prefix}`             |
| `updateItem(id, data)` | PUT    | `/{prefix}/{id}`        |
| `deleteItem(id)`       | DELETE | `/{prefix}/{id}`        |
| `toggleActive(id)`     | PATCH  | `/{prefix}/{id}/toggle` |
| `batchDelete(ids)`     | POST   | `/{prefix}/batch`       |
| `exportItems(params)`  | GET    | `/{prefix}/export`      |

**All of these are provided by `useCrudFactory` — do NOT rewrite them.**

For custom endpoints not covered by the factory:

```js
// In the store file, add after useCrudFactory
import { useApi } from "@/composables/useApi";

export const useMyStore = useCrudFactory(config);

// Extend with custom actions
export function useMyCustomActions() {
  const api = useApi();

  async function approveItem(id) {
    return api.post(`/admin/my-module/${id}/approve`);
  }

  return { approveItem };
}
```

### Step 3 — Map API response to frontend

**Backend response format** (from `successResponse`):

```json
{
  "status": true,
  "message": "...",
  "data": {
    /* Resource output */
  }
}
```

**Paginated response** (from `wrapPaginate`):

```json
{
  "status": true,
  "data": [
    /* Resource array */
  ],
  "meta": { "current_page": 1, "last_page": 5, "per_page": 15, "total": 72 }
}
```

`useLookupPage` handles pagination response parsing automatically.

### Step 4 — Map form schema to request validation

Ensure frontend FieldUtils schema matches backend FormRequest rules:

| FieldUtils           | FormRequest Rule       | Notes                 |
| -------------------- | ---------------------- | --------------------- | --------- | --------------- |
| `required: true`     | `'required'`           | Must match            |
| `translatable: true` | `TranslatableRequired` | Both sides must agree |
| `type: 'email'`      | `'email'`              | Same validation       |
| `maxLength: 255`     | `'max:255'`            | Same limit            |
| `type: 'number'`     | `'numeric'`            | Type match            |
| `type: 'select'`     | `'exists:table,id'`    | FK validation         |
| `type: 'image'`      | `'image                | mimes:...             | max:...'` | File validation |

### Step 5 — Configure API endpoint in config.js

```js
export default createModuleConfig({
  module: "userGroups",
  apiEndpoint: "/admin/user-groups", // ← must match BE route prefix
  // ...
});
```

### Step 6 — Error handling

The project handles errors at the HTTP layer level:

- 401 → auto redirect to login
- 403 → permission denied toast
- 422 → validation errors mapped to form fields
- 500 → generic error toast

**Do NOT add custom error handling** unless the endpoint has special error cases.

## Output

Configured store, config, and (if needed) custom API functions that correctly
connect to the backend endpoints using the project's HTTP layer.

## Rules

1. NEVER use raw `fetch()` or `axios` — always use the project's API layer
2. NEVER rewrite CRUD methods that `useCrudFactory` already provides
3. API endpoint in config MUST match the backend route prefix exactly
4. Frontend form fields MUST match backend validation rules
5. Response format is standardized — do not add custom response parsing
6. Auth token is handled automatically by the HTTP interceptor — never pass it manually
7. File uploads use `ChunkFileController` for large files, standard multipart for small
