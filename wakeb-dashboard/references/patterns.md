# Patterns, Debugging & Conventions

## Contents

- Code style & conventions
- i18n & RTL rules
- Routing & navigation
- API layer
- Theme system
- Performance
- Debugging & error handling
- Environment variables

---

## Code Style

**File naming:**

- Components: PascalCase (`Button.vue`, `AddEditPage.vue`)
- JS/TS: camelCase (`useAlert.js`, `config.js`)
- Composables: `use{Name}` prefix
- Modules: camelCase directories

**Component structure:** `<template>` first, then `<script setup>`. Composition API only.

**Prettier:**

```json
{
  "semi": false,
  "tabWidth": 2,
  "singleQuote": true,
  "printWidth": 100,
  "trailingComma": "none"
}
```

---

## i18n & RTL

**Templates:** `{{ $t('module.field') }}`, `:label="$t('fields.name')"`

**Script:** `import i18n from '@/utils/i18n'; const t = i18n.global.t`

**RTL rules:**

- Use `start`/`end` instead of `left`/`right`
- Use `gap` for element spacing
- Use `ms-*`/`me-*` (margin), `ps-*`/`pe-*` (padding) Tailwind classes
- Use `border-inline-start`, `inset-inline-start`

**Locale structure:**

```json
{
  "module_name": {
    "title": "...",
    "fields": { "name": "...", "email": "..." },
    "messages": { "created": "...", "deleted": "..." }
  }
}
```

---

## Routing

| Route Type    | Description                        |
| ------------- | ---------------------------------- |
| `'protected'` | Requires auth (default)            |
| `'auth'`      | Public only (login, register)      |
| `'public'`    | Accessible to all                  |
| `'both'`      | Based on `route.meta.requiresAuth` |

**Permissions:** `meta: { permissions: ['users.index'] }`

**Auto-loading:** Routes auto-imported from `src/modules/*/router/index.js`.

---

## API Layer

```js
import { httpRequest } from "@/services/api";

await httpRequest("/endpoint", { method: "GET", params: { page: 1 } });
await httpRequest("/endpoint", { method: "POST", data: payload });
await httpRequest("/endpoint/1", { method: "PUT", data: payload });
await httpRequest("/endpoint/1", { method: "DELETE" });

// File upload
await httpRequest("/upload", {
  method: "POST",
  data: formData,
  headers: { "Content-Type": "multipart/form-data" },
});
```

- Base URL: `import.meta.env.VITE_SERVER_URL`
- Auth: JWT auto-injected from cookies
- Interceptors: 401 (logout), 403, 422 (validation → schema), 404, 500

---

## Theme System

- Themes in `src/themes/{themeName}/`
- Each has: Layout, Sidebar, Navbar, Table, IndexPage, AddEditPage, Stepper
- Active theme: `.starterrc.json` → `theme.name`
- Resolve: `useThemedComponent('ComponentName')`
- Dark mode: `useIsDark()` composable
- Pipeline: `Figma → primitives.json → semantics.json → tokens.json → CSS Variables`

---

## Performance

1. Lazy load routes: `() => import('...')`
2. Async components: `defineAsyncComponent`
3. Cache API: `useCrudFactory` with `cache: { enabled: true }`
4. Tree-shake icons: import only needed
5. Virtual scrolling: Vuetify's virtual-scroller for large lists
6. Debounce search: 300ms+ on API inputs
7. Computed over methods for derived state

---

## Debugging & Error Handling

### Automatic error handling

`jervis-connect` interceptor handles:

- **401** → Clear auth, redirect to login
- **403** → Permission denied alert
- **422** → Validation errors applied to schema via `handleErrors()`
- **404** → Not found alert
- **500** → Server error alert

### Manual handling

```js
const result = await submitForm(data);
if (result?.errors) {
  // errors already applied to schema fields
  console.log(result.errors); // { field: ['message'] }
}
```

### Common issues

| Symptom                   | Cause                 | Fix                                                 |
| ------------------------- | --------------------- | --------------------------------------------------- |
| Form fields don't reset   | Schema not in `ref()` | Wrap in `ref([...])`                                |
| Table stale after CRUD    | Store cache           | `cache: { enabled: false }` or call `getItems()`    |
| Theme component not found | Wrong case            | `useThemedComponent('IndexPage')` not `'indexPage'` |
| Translations missing      | Key not in locale     | Add to both `ar.json` AND `en.json`                 |
| RTL broken                | Using left/right      | Replace with start/end, ms/me, ps/pe                |
| Route not showing         | Missing router export | Check `src/modules/{name}/router/index.js`          |
| Sidebar missing           | No sidebar config     | Add `sidebar: { order: N, icon: 'name' }`           |
| Permission denied         | Missing meta          | Add `permissions: ['module.action']` to route meta  |

---

## Environment Variables

| Variable          | Purpose         | Default                  |
| ----------------- | --------------- | ------------------------ |
| `VITE_SERVER_URL` | API base URL    | —                        |
| `VITE_CLIENT`     | Client branding | `"wakeb"`                |
| `VITE_FONT`       | Font family     | `"ibm-plex-sans-arabic"` |
| `VITE_LOCALE`     | Default locale  | `"ar"`                   |
