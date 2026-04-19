# Skill: enforce-structure

Validates that generated files follow the project's folder structure, naming
conventions, and file organization. Run this AFTER generating code, BEFORE
returning it to the user.

## When to Use

- After generating any new file
- After scaffolding a module
- When the user reports "file not found" or "module not loading" errors

## Validation Rules

### Frontend File Placement

| File Type         | Must Be In                                | Naming                                     |
| ----------------- | ----------------------------------------- | ------------------------------------------ |
| Module config     | `src/modules/{name}/config.js`            | camelCase module name                      |
| Module router     | `src/modules/{name}/router/index.js`      | Always `index.js`                          |
| Module store      | `src/modules/{name}/stores/{name}.js`     | camelCase, matches module                  |
| Module views      | `src/modules/{name}/views/`               | PascalCase `.vue` files                    |
| Module schema     | `src/modules/{name}/schema/index.js`      | Always `index.js`                          |
| Module locales    | `src/modules/{name}/locales/{ar,en}.json` | ISO language codes                         |
| Module components | `src/modules/{name}/components/`          | PascalCase `.vue` files                    |
| Common components | `src/components/common/`                  | PascalCase `.vue` files                    |
| Composables       | `src/composables/`                        | `use{Name}.{js,ts}`                        |
| Utils             | `src/utils/`                              | camelCase `.{js,ts}` files                 |
| Global stores     | `src/stores/`                             | camelCase `.js` files                      |
| Theme files       | `src/themes/{name}/`                      | PascalCase `.vue` or camelCase `.js/.json` |

### Backend File Placement

| File Type         | Must Be In                                                      | Naming                              |
| ----------------- | --------------------------------------------------------------- | ----------------------------------- |
| Core controller   | `app/Http/Controllers/API/{Group}/`                             | `{Name}Controller.php` (PascalCase) |
| Module controller | `Modules/{Name}/app/Http/Controllers/Api/Admin/`                | `{Name}Controller.php`              |
| Core request      | `app/Http/Requests/{Group}/`                                    | `{Name}Request.php`                 |
| Module request    | `Modules/{Name}/app/Http/Requests/`                             | `{Name}Request.php`                 |
| Core resource     | `app/Http/Resources/{Group}/`                                   | `{Name}Resource.php`                |
| Module resource   | `Modules/{Name}/app/Http/Resources/Admin/`                      | `{Name}Resource.php`                |
| Core model        | `app/Models/`                                                   | `{Name}.php` (singular PascalCase)  |
| Module model      | `Modules/{Name}/app/Models/`                                    | `{Name}.php`                        |
| Filter            | `app/Filters/{Group}/`                                          | `{Name}Filter.php`                  |
| Rule              | `app/Rules/`                                                    | `{Name}.php` (PascalCase)           |
| Trait             | `app/Trait/Global/`                                             | `{Name}.php`                        |
| Service           | `app/Services/{Group}/`                                         | `{Name}Service.php`                 |
| Policy            | `app/Policies/`                                                 | `{Name}Policy.php`                  |
| Enum              | `app/Enum/{Group}/`                                             | `{Name}Enum.php`                    |
| Migration         | `database/migrations/` or `Modules/{Name}/database/migrations/` | `{date}_create_{table}_table.php`   |
| Seeder            | `database/seeders/` or `Modules/{Name}/database/seeders/`       | `{Name}Seeder.php`                  |
| Routes            | `routes/api.php` or `Modules/{Name}/routes/api.php`             | Always `api.php`                    |

### Naming Conventions

| Context           | Convention                    | Example                    |
| ----------------- | ----------------------------- | -------------------------- |
| FE module folder  | kebab-case                    | `src/modules/user-groups/` |
| FE component file | PascalCase                    | `UserCard.vue`             |
| FE composable     | camelCase with `use` prefix   | `useUserGroups.js`         |
| FE store file     | camelCase                     | `userGroups.js`            |
| FE locale keys    | snake_case                    | `user_name`, `created_at`  |
| BE class name     | PascalCase                    | `UserGroupController`      |
| BE table name     | snake_case plural             | `user_groups`              |
| BE route prefix   | kebab-case plural             | `user-groups`              |
| BE permission     | kebab-case `{action}-{model}` | `create-user-group`        |
| BE enum           | PascalCase + `Enum` suffix    | `StatusTypeEnum`           |

### Module Completeness Check

**Frontend module must have ALL of:**

```
src/modules/{name}/
├── config.js              ✓ uses createModuleConfig()
├── router/index.js        ✓ exports route array with permissions
├── stores/{name}.js       ✓ uses useCrudFactory()
├── views/index.vue        ✓ uses useLookupPage()
├── schema/index.js        ✓ uses FieldUtils creators
└── locales/
    ├── ar.json            ✓ Arabic translations
    └── en.json            ✓ English translations
```

**Backend module must have ALL of:**

```
Modules/{Name}/
├── app/Http/Controllers/Api/Admin/{Name}Controller.php  ✓ extends BaseController
├── app/Http/Requests/{Name}Request.php                  ✓ extends BaseFormRequest
├── app/Http/Resources/Admin/{Name}Resource.php          ✓ extends JsonResource
├── app/Models/{Name}.php                                ✓ extends BaseModel
├── routes/api.php                                       ✓ auth:sanctum middleware
├── database/migrations/                                 ✓ soft deletes + timestamps
├── database/seeders/{Name}Seeder.php                    ✓ optional
└── Providers/{Name}ServiceProvider.php                  ✓ registered in module.json
```

## Output

A checklist of pass/fail validations. If any fail, provide the exact fix.
