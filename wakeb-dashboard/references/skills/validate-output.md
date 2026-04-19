# Skill: validate-output

Final validation gate. Checks all generated code for structure, component
usage, naming, API correctness, and project conventions. MUST run before
returning any code to the user.

## When to Use

- After generating any code (always — this is mandatory)
- When the user reports issues with generated code
- When reviewing existing code for compliance

## Checks

### 1. Structure Validation

```
□ Files are in correct directories (see enforce-structure)
□ Module has all required files (config, router, store, view, schema, locales)
□ File names follow naming conventions
□ No files outside the project's folder structure
```

### 2. Component Usage

```
□ Every UI element uses a project component (no raw HTML equivalents)
□ No duplicate components — if Badge.vue exists, no custom status tags
□ FieldUtils used for all form fields — no raw <input> or <v-text-field>
□ useLookupPage used for index pages — no manual table/pagination
□ useCrudFactory used for stores — no manual API calls
□ useThemedComponent used for theme-specific components
```

### 3. Naming Conventions

```
□ Vue files: PascalCase (UserCard.vue)
□ Composables: use{Name} prefix (useUserGroups.js)
□ Store files: camelCase (userGroups.js)
□ Locale keys: snake_case (user_name)
□ PHP classes: PascalCase (UserGroupController)
□ DB tables: snake_case plural (user_groups)
□ Routes: kebab-case plural (user-groups)
□ Permissions: kebab-case {action}-{model} (create-user-group)
```

### 4. API Correctness

```
□ Frontend apiEndpoint matches backend route prefix
□ Form schema fields match backend validation rules
□ Response handling uses standard project pattern
□ No raw fetch/axios — uses project API layer
□ Auth handled by interceptor — no manual token passing
□ Pagination uses wrapPaginate (BE) and useLookupPage (FE)
```

### 5. Code Conventions

```
□ <script setup> — no Options API
□ Composition API — ref/computed, not data/computed properties
□ All user-facing text in $t() or t() — no hardcoded strings
□ RTL-safe: start/end, ms/me, ps/pe — no left/right
□ Colors from design tokens — no hardcoded hex values
□ No inline styles — Tailwind or design token classes only
□ No console.log — removed before output
□ No TODO/FIXME — code must be complete
```

### 6. Backend Conventions

```
□ Controller extends BaseController
□ Request extends BaseFormRequest
□ Model extends BaseModel
□ Translatable fields use resolveTrans() in Resource
□ Translatable fields use TranslatableRequired/Nullable in Request
□ Pipeline used for filtering — no manual where() chains
□ Responses use successResponse() / failResponse()
□ Policies registered and used for authorization
□ Activity logging configured with LogsActivityOptions
```

### 7. Anti-Pattern Detection

| Anti-Pattern                         | Correct Pattern                       | Severity |
| ------------------------------------ | ------------------------------------- | -------- |
| `<button>` in template               | `<Button>` component                  | Critical |
| `<table>` in template                | `Table/index.vue` via `useLookupPage` | Critical |
| `<input>` in form                    | FieldUtils creator                    | Critical |
| `axios.get()` / `fetch()`            | Store method via `useCrudFactory`     | Critical |
| `color: #hex`                        | `var(--v-theme-*)`                    | High     |
| `margin-left` / `padding-right`      | `ms-*` / `pe-*`                       | High     |
| Hardcoded Arabic/English text        | `$t('key')`                           | High     |
| `export default { data() }`          | `<script setup>` + `ref()`            | High     |
| Manual pagination logic              | `useLookupPage`                       | High     |
| `Auth::user()` without guard         | `$this->user` from BaseController     | Medium   |
| Raw `DB::table()` queries            | Eloquent Model methods                | Medium   |
| Manual `where()` chain in controller | Pipeline filters                      | Medium   |
| String permission checks             | Policy methods                        | Medium   |

## Output

```
✅ PASS — Structure (7/7)
✅ PASS — Component Usage (5/5)
⚠️ WARN — Naming (4/5) — store file should be camelCase
❌ FAIL — API (3/4) — endpoint mismatch: config has '/users' but route is '/admin/users'
✅ PASS — Code Conventions (8/8)
✅ PASS — Backend Conventions (6/6)
✅ PASS — Anti-Patterns (0 detected)

Issues to fix:
1. [WARN] Rename stores/UserGroups.js → stores/userGroups.js
2. [FAIL] Change apiEndpoint in config.js from '/users' to '/admin/users'
```

## Rules

1. This skill MUST run on every code generation output
2. Any Critical anti-pattern is a hard block — fix before returning code
3. High severity issues should be fixed — warn if there's a valid reason to skip
4. Medium severity issues are recommendations — fix when reasonable
5. If validation fails, fix the code and re-validate — never return failing code
