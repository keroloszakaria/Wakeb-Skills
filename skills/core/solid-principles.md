# Skill: SOLID Principles

Applies SOLID principles adapted to the Wakeb Starter architecture.
Maps each principle to concrete patterns used in the project.

## When to Use

- When designing new modules, services, or components
- When refactoring existing code
- When code review reveals design issues
- When a class or component is growing too large

## S — Single Responsibility

**Each file does ONE thing.**

### Front-End

| File                | Responsibility         | NOT its job                   |
| ------------------- | ---------------------- | ----------------------------- |
| `config.js`         | Module metadata        | Business logic, API calls     |
| `stores/{name}.js`  | API state management   | UI logic, validation          |
| `schema/index.js`   | Form field definitions | API calls, business rules     |
| `views/IndexView`   | Page composition       | Direct API calls, store logic |
| `components/common` | Reusable UI rendering  | Business logic, API calls     |
| Composables         | Shared stateful logic  | UI rendering, routing         |

**Violation:** A view component that imports axios and makes API calls directly.
**Fix:** Use `useCrudFactory()` in a store, then `useLookupPage()` in the view.

### Back-End

| File          | Responsibility          | NOT its job                    |
| ------------- | ----------------------- | ------------------------------ |
| Controller    | HTTP request handling   | Business logic, validation     |
| FormRequest   | Input validation        | Database queries, auth checks  |
| Model         | Data structure + scopes | HTTP responses, validation     |
| Resource      | Response transformation | Business logic, queries        |
| Policy        | Authorization rules     | Data fetching, response format |
| Service       | Business logic          | HTTP handling, validation      |
| Filter (Pipe) | Single query condition  | Multiple unrelated conditions  |

**Violation:** A controller with 200+ lines doing validation, queries, and transformation.
**Fix:** Validation → FormRequest, queries → Pipeline, transformation → Resource.

## O — Open/Closed

**Extend behavior without modifying existing code.**

### Front-End

```
✅ useCrudFactory hooks — extend store behavior without touching the factory:
   useCrudFactory('endpoint', {
     hooks: {
       beforeCreate: (data) => { /* transform */ return data },
       afterGetAll: (response) => { /* post-process */ }
     }
   })

✅ Module config — add new modules without modifying the router or sidebar:
   Auto-discovered from src/modules/*/router/index.js

✅ Theme system — add new themes without modifying components:
   useThemedComponent('IndexPage') resolves to the active theme

❌ Modifying BaseCrudFactory to add module-specific behavior
❌ Modifying GenericForm to handle one module's special field
```

### Back-End

```
✅ Pipeline filters — add new filters without modifying the controller:
   ->through([...existingFilters, NewCustomFilter::class])

✅ Model traits — add behavior without modifying BaseModel:
   class Product extends BaseModel { use HasTranslations, SoftDeletes; }

✅ Observer pattern — react to model events without modifying the model:
   class ProductObserver { public function created(Product $p) { ... } }

❌ Modifying BaseController for one controller's special case
❌ Adding if/else branches in shared code for module-specific logic
```

## L — Liskov Substitution

**Subclasses must work wherever the parent is expected.**

```
FE:
  ✅ Every module's IndexView works with IndexPage theme wrapper
  ✅ Every module's config works with createModuleConfig()
  ✅ Every module's store works with useLookupPage()
  ❌ A module that breaks useLookupPage by returning different data shapes

BE:
  ✅ Every controller works with the route system (extends BaseController)
  ✅ Every request works with validation (extends BaseFormRequest)
  ✅ Every model works with Pipeline filters (extends BaseModel)
  ❌ A controller that returns a different response shape than successResponse()
```

## I — Interface Segregation

**Don't force dependence on unused interfaces.**

```
FE:
  ✅ FieldUtils — import only the creators you need:
     import { createTextField, createSelectField } from '@/utils/FieldUtils'
  ✅ Composables — each does one thing:
     useAlert for alerts, useStorage for localStorage, useIsDark for theme
  ❌ A single useEverything() composable that exports 50 functions

BE:
  ✅ Traits — use only what you need:
     use HasDeleteMethods;          // only if module has delete
     use HasToggleActiveMethods;    // only if module has toggle
  ✅ Filters — each filter class handles one query condition
  ❌ A single filter class that handles search, sort, and status in one file
```

## D — Dependency Inversion

**Depend on abstractions, not concrete implementations.**

```
FE:
  ✅ httpRequest() from services/api.js — abstracts HTTP client
     Components don't know about axios or fetch
  ✅ useThemedComponent('IndexPage') — abstracts theme resolution
     Views don't know which theme is active
  ✅ useCrudFactory('endpoint') — abstracts CRUD operations
     Views don't know about API URLs or HTTP methods
  ❌ import axios from 'axios' in a component
  ❌ import AwareIndexPage from '@/themes/aware/IndexPage.vue'

BE:
  ✅ Pipeline pattern — controller doesn't know filter implementations
  ✅ Gate::authorize() — controller doesn't know policy details
  ✅ successResponse() — controller doesn't know response serialization
  ❌ Direct DB::select() queries in controllers
  ❌ Hard-coding specific permission names in controller logic
```
