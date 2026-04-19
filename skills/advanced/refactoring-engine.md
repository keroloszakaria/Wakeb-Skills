# Skill: Refactoring Engine

Identifies and applies safe refactoring patterns to improve code quality
without changing behavior. Follows Wakeb Starter conventions.

## When to Use

- When the user asks to refactor or clean up code
- When code review identifies pattern violations
- When duplicated code is detected
- When a component/module grows too large

## Refactoring Catalog

### Extract Composable (FE)

```
BEFORE: Logic duplicated across components
AFTER:  Shared composable in src/composables/

Trigger: Same logic in 2+ components

// Extract to composable
export function useFeatureName() {
  const state = ref(initialValue)
  const action = () => { /* logic */ }
  return { state, action }
}
```

### Extract Pipeline (BE)

```
BEFORE: Complex query logic in controller
AFTER:  Pipeline class in app/Pipelines/

Trigger: Controller method > 20 lines of query building

// Move filtering to pipeline
class FilterByX extends BasePipeline {
  protected function apply($query) {
    return $query->when(request('x'), fn($q, $v) => $q->where('x', $v));
  }
}
```

### Replace Magic Strings

```
BEFORE: Scattered string literals
  if (status === 'active') ...
  if (status === 'inactive') ...

AFTER: Constants or enums
  // FE: config/constants.ts
  export const STATUS = { ACTIVE: 'active', INACTIVE: 'inactive' }

  // BE: app/Enums/StatusEnum.php
  enum StatusEnum: string { case Active = 'active'; case Inactive = 'inactive'; }
```

### Simplify Conditional Logic

```
BEFORE:
  if (condition) {
    return true
  } else {
    return false
  }

AFTER:
  return condition

BEFORE:
  if (x) { doA() }
  if (x && y) { doB() }
  if (x && y && z) { doC() }

AFTER:
  if (!x) return
  doA()
  if (!y) return
  doB()
  if (!z) return
  doC()
```

### Split Large Components

```
Trigger: Component > 300 lines

Strategy:
1. Identify logical sections (header, filters, table, form)
2. Extract each section to a child component
3. Parent orchestrates with props/events
4. Shared state stays in composable or store
```

### Consolidate API Calls

```
BEFORE: Multiple direct axios/fetch calls
AFTER:  Single useCrudFactory with proper config

// Replace scattered API calls with:
const { getItems, createItem, updateItem, deleteItem } = useCrudFactory('endpoint')
```

## Safe Refactoring Rules

```
□ One refactoring type at a time — don't combine
□ Verify behavior is unchanged after each refactoring
□ Keep the same public API (props, events, exports)
□ Update imports in all consuming files
□ Run lint after refactoring to catch issues
□ Do NOT refactor code you're not asked to touch
□ Do NOT add features during refactoring
```

## When NOT to Refactor

```
□ Code is working and no one asked for refactoring
□ The "improvement" is purely stylistic
□ It would require changing 10+ files for marginal benefit
□ The code is about to be replaced
□ There's no clear benefit — just preference
```
