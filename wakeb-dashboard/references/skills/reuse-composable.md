# Skill: reuse-composable

Detects reusable logic in generated or existing code and extracts it into
composables or utilities that follow the project's patterns.

## When to Use

- When generating code that duplicates logic found in an existing composable
- When a code pattern appears in 2+ places
- When the user asks to refactor or extract shared logic
- Before creating any custom hook or utility function

## Input

| Field     | Type   | Description                          |
| --------- | ------ | ------------------------------------ |
| `code`    | string | The code snippet to analyze          |
| `context` | string | What the code does / where it's used |

## Execution

### Step 1 — Check existing composables

Before creating anything, search [codebase-index.md](codebase-index.md):

| If the code does...                            | Use existing                           |
| ---------------------------------------------- | -------------------------------------- |
| CRUD operations (list, create, update, delete) | `useCrudFactory` via `BaseCrudFactory` |
| Table with filters + pagination + modal        | `useLookupPage`                        |
| Locale/language switching                      | `useLocale`                            |
| Form schema definition                         | `FieldUtils` creators                  |
| Theme-aware component loading                  | `useThemedComponent`                   |
| Toast/notification display                     | `useToast`                             |
| Dark mode toggle                               | `useDarkMode`                          |
| File download                                  | `useDownload`                          |
| Screen breakpoint checks                       | `useScreen`                            |
| Route-based permissions                        | `usePermission`                        |
| RTL detection                                  | `useDirection`                         |
| Config access                                  | `useConfig`                            |
| Table column config                            | `TableFactory`                         |
| Form generation                                | `GenericFormFactory`                   |
| Enum label resolution                          | `useEnumUtils`                         |
| Status rendering                               | `useStatusRenderer`                    |
| Menu/nav construction                          | `useMenu`                              |
| Item actions (edit, delete, toggle)            | `useItemActions`                       |

### Step 2 — Identify extraction candidates

If the code is NOT covered by an existing composable, check:

1. **Is it used in 2+ modules?** → Extract to `src/composables/use{Name}.js`
2. **Is it a pure utility (no Vue reactivity)?** → Extract to `src/utils/{name}.js`
3. **Is it module-specific?** → Keep in module's `components/` or inline

### Step 3 — Extract following patterns

New composables must follow the project's conventions:

```js
// src/composables/useMyLogic.js
import { ref, computed } from 'vue'

export function useMyLogic(options = {}) {
  // Reactive state
  const state = ref(options.initial ?? null)

  // Computed
  const derived = computed(() => /* ... */)

  // Methods
  function doSomething() { /* ... */ }

  return { state, derived, doSomething }
}
```

**Rules for composable extraction:**

- Prefix with `use`
- Accept options object, not positional args
- Return an object with named properties (not an array)
- Use `ref`/`computed` from Vue, not `reactive` for top-level
- Keep composable focused on ONE concern

### Step 4 — Verify no duplication

After extraction, confirm:

- No existing composable does the same thing (re-check index)
- The new composable is imported and used correctly
- Original inline code is replaced with the composable call

## Output

Either:

- **"Reuse existing"** → import path + usage example
- **"Extract new"** → new composable file + usage in original code

## Rules

1. ALWAYS check existing composables before creating new ones
2. NEVER duplicate what `useLookupPage` already provides (pagination, search, modal state, etc.)
3. NEVER duplicate what `useCrudFactory` already provides (API calls, loading, error state)
4. New composables must be genuinely reusable — not single-use wrappers
5. Pure functions without Vue reactivity go in `utils/`, not `composables/`
