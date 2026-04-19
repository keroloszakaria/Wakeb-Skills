# Skill: End-to-End Flow Generator

Generates a complete feature across both frontend and backend stacks in one
coordinated flow, ensuring all pieces are aligned.

## When to Use

- When the user asks to "create a module" or "build a feature"
- When a feature needs both FE and BE code
- When generating from a requirements description

## Generation Flow

```
Step 1: Analyze Requirements
  └── Extract: entity name, fields, relationships, permissions

Step 2: Generate Backend (Laravel)
  ├── Migration (table + columns)
  ├── Model (fillable, casts, relationships)
  ├── Controller (extends BaseController)
  ├── Request (validation rules)
  ├── Resource (API response shape)
  ├── Pipelines (filter + sort)
  ├── Routes (api.php with auth middleware)
  └── Seeder (sample data)

Step 3: Generate Frontend (Vue)
  ├── Config (module.ts with headers, form schema, endpoints)
  ├── Pages:
  │   ├── IndexView.vue (list page using useLookupPage)
  │   └── AddEditView.vue (form page using useCrudFactory)
  ├── Router entry (lazy-loaded routes)
  └── Locale files (ar.json + en.json)

Step 4: Contract Validation
  └── Run contract-sync skill to verify alignment

Step 5: Summary
  └── List all created files + any manual steps needed
```

## Input → Output Mapping

### From User Description

```
User: "Create an employees module with name, email, department, and status"

→ Entity: Employee
→ Table: employees
→ Fields:
  - name: string, required
  - email: string, required, unique
  - department_id: foreign key to departments
  - status: enum (active, inactive), default active
→ Relationships: belongsTo Department
→ Permissions: employees.view, employees.create, employees.update, employees.delete
```

### Field Type Mapping

| User Says     | DB Column             | BE Validation             | FE Form Type    |
| ------------- | --------------------- | ------------------------- | --------------- |
| name/title    | string('name')        | required\|string\|max:255 | text            |
| email         | string('email')       | required\|email\|unique   | text            |
| description   | text('description')   | nullable\|string          | textarea        |
| number/count  | integer('count')      | required\|integer\|min:0  | text (number)   |
| price/amount  | decimal('price',10,2) | required\|numeric\|min:0  | text (number)   |
| date          | date('date')          | required\|date            | datePicker      |
| status        | enum('status',...)    | required\|in:...          | select          |
| active/toggle | boolean('is_active')  | boolean                   | switch          |
| category/dept | foreignId('x_id')     | required\|exists:x,id     | select (lookup) |
| image/file    | string('file_path')   | nullable\|image\|max:2048 | fileUpload      |
| tags          | json('tags')          | nullable\|array           | combobox        |

## Generated File List

For a module named "Employee":

### Backend Files

```
Modules/Employee/
├── app/Http/Controllers/EmployeeController.php
├── app/Http/Requests/EmployeeRequest.php
├── app/Http/Resources/EmployeeResource.php
├── app/Models/Employee.php
├── app/Pipelines/FilterEmployee.php
├── app/Pipelines/SortEmployee.php
├── database/migrations/YYYY_MM_DD_create_employees_table.php
├── database/seeders/EmployeeSeeder.php
└── routes/api.php
```

### Frontend Files

```
src/modules/employees/
├── config/module.ts
├── pages/IndexView.vue
├── pages/AddEditView.vue
├── router/index.ts
└── locales/
    ├── ar.json
    └── en.json
```

## Rules

```
□ Generate BE first, then FE — BE defines the contract
□ Every field appears in: migration, model $fillable, resource, request, FE config
□ Run contract-sync validation after generation
□ Include both ar.json and en.json with all field labels
□ Routes are lazy-loaded in FE router
□ Permissions defined for all CRUD actions
□ List any manual steps (run migration, register module, etc.)
```
