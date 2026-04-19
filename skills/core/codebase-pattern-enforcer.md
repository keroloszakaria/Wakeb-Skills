# Skill: Codebase Pattern Enforcer

Scans generated code against the project's established patterns and rejects
anything that deviates. This skill is the quality gate — no code leaves
without passing it.

## When to Use

- After ANY code generation (mandatory — always the last step)
- When reviewing existing code for compliance
- When the user asks "is this code correct?"

## Pattern Rules

### Universal Rules (FE + BE)

```
□ No hardcoded strings in UI — use $t() / __() translations
□ No hardcoded colors — use design tokens / theme variables
□ No absolute file paths — use @ aliases or relative paths
□ No console.log / dd() in production code
□ No commented-out code blocks
□ No TODO without a ticket reference
```

### Front-End Pattern Rules

```
STRUCTURE:
□ <script setup> only — no Options API, no <script> without setup
□ Composition API refs, computed, watch — no this.$data, this.$watch
□ Single-file components — no renderless or JSX components

COMPONENTS:
□ Every UI element maps to a project component (see resolve-component)
□ No v-btn, v-chip, v-dialog, v-data-table-server, v-text-field, v-select
□ No raw <button>, <table>, <input>, <select>, <video>
□ Button.vue for buttons, Badge.vue for chips, Modal.vue for dialogs
□ Table/index.vue for tables, GenericForm for forms
□ SvgIcon for icons — no <svg>, <img> for icons

STYLING:
□ Colors from rgba(var(--v-theme-*), 1) — no hex, no rgb(), no hsl()
□ Spacing from Tailwind utilities (p-4, gap-2, etc.)
□ Layout from v-row/v-col — no raw flexbox for grid layouts
□ RTL: start/end, ms-/me-, ps-/pe- — no left/right, ml-/mr-, pl-/pr-

DATA:
□ Stores use useCrudFactory() — no manual axios calls
□ CRUD pages use useLookupPage() — no manual table wiring
□ Form schemas use FieldUtils creators — no manual field objects
□ Module config uses createModuleConfig() — no manual config objects

FORMAT:
□ No semicolons (Prettier)
□ Single quotes (Prettier)
□ 2-space indentation
□ Trailing comma: none
```

### Back-End Pattern Rules

```
STRUCTURE:
□ Controllers extend BaseController
□ Requests extend BaseFormRequest
□ Models extend BaseModel
□ Resources extend JsonResource

RESPONSES:
□ successResponse() / failResponse() — no response()->json()
□ wrapPaginate() for paginated results
□ __('api.key') for messages — no hardcoded strings

AUTHORIZATION:
□ Gate::authorize() — no $user->can() in controllers
□ PermissionMiddleware on store/update — not inline checks
□ Policy classes for complex authorization

QUERIES:
□ Pipeline for filtering — no chained where() for search/filter
□ Eager loading with() — no N+1 queries
□ $request->validated() — no $request->all()

DATA:
□ DB::transaction() for multi-model operations
□ SoftDeletes on all models
□ created_by tracked via CreatedByObserver
□ Translatable fields use HasTranslations + TranslatableRequired
```

## Violation Handling

When a violation is found:

1. **Identify** — Point to the exact line and rule
2. **Explain** — Why this pattern exists (not just "it's wrong")
3. **Fix** — Provide the correct code
4. **Prevent** — Note the pattern so it doesn't recur

Example:

```
❌ Line 15: <v-btn color="primary">Save</v-btn>
   Rule: No direct Vuetify components
   Why: Button.vue handles theming, sizes, RTL, and loading states
   Fix: <Button title="$t('save')" type="primary" />
```

## Severity Levels

| Level    | Action          | Examples                                     |
| -------- | --------------- | -------------------------------------------- |
| CRITICAL | Block output    | Wrong base class, raw HTML, hardcoded colors |
| HIGH     | Fix before send | Missing i18n, wrong spacing direction        |
| MEDIUM   | Warn + suggest  | Missing cache config, no breadcrumbs         |
| LOW      | Note for later  | Could use a composable, verbose code         |
