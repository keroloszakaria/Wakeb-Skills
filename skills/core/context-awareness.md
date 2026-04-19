# Skill: Context Awareness Engine

Detects what kind of project, module, and file the user is working on — then
loads the correct conventions, components, and patterns automatically. This is
the foundation skill that all other skills depend on.

## When to Use

- At the START of every interaction (always — this is automatic)
- When the user opens a new file or switches context
- When the request is ambiguous about which module or layer

## Execution

### Step 1 — Detect Project Type

```
Check in order:
1. .starterrc.json          → Wakeb Starter (FE dashboard)
2. composer.json + Modules/  → Wakeb Backend (Laravel)
3. Both present              → Full-stack Wakeb project
4. Neither                   → Not a Wakeb project — ask the user
```

### Step 2 — Detect Current Module

```
From the active file path, extract:
  src/modules/{moduleName}/  → FE module name
  Modules/{ModuleName}/      → BE module name

If no module detected:
  - Check if working in components/common/ → shared component context
  - Check if working in themes/            → theme context
  - Check if working in utils/             → utility context
  - Otherwise                              → project-level context
```

### Step 3 — Load Context

```
FE Context:
  □ Load module config (config.js) to know headers, CRUD mode, enums, models
  □ Load module schema to know existing form fields
  □ Load module store to know endpoint and cache settings
  □ Load module locales to know existing translations

BE Context:
  □ Load controller to know existing endpoints and filters
  □ Load model to know fillable, casts, relations, traits
  □ Load form request to know validation rules
  □ Load routes to know URL patterns and middleware
```

### Step 4 — Set Conventions

Based on detected context, enforce:

| Context      | Convention Set                                              |
| ------------ | ----------------------------------------------------------- |
| FE Module    | script setup, FieldUtils, useLookupPage, design tokens, RTL |
| FE Component | script setup, props/emits patterns, design tokens           |
| FE Theme     | Token pipeline, responsive, dark mode support               |
| BE Module    | BaseController, Pipeline, Gate, successResponse             |
| BE Model     | BaseModel, traits, translatable, soft deletes               |
| BE Request   | BaseFormRequest, validated(), custom rules                  |

### Step 5 — Report Context

Before generating code, state what was detected:

```
📁 Project: Wakeb Dashboard (FE)
📦 Module: drones
📄 File: views/IndexView.vue
🎨 Theme: aware
🌐 Locale: ar (primary), en
```

## Output

The detected context is passed to ALL other skills as implicit input.
Never ask the user what project they're in — detect it.
