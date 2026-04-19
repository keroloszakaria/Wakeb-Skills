# Skill: i18n + RTL Awareness

Ensures all generated code is fully internationalized (Arabic-first) and
RTL-compatible. Arabic is the primary language — English is secondary.

## When to Use

- During ALL code generation (automatic — this is mandatory)
- When creating locale files for a module
- When debugging layout issues in Arabic or English
- When reviewing code for RTL compliance

## i18n Rules

### Template Strings

```vue
<!-- ✅ Correct -->
<h1>{{ $t('users.title') }}</h1>
<Button :title="$t('actions.save')" />
<Table :emptyMessage="$t('no_data')" />

<!-- ❌ Wrong — bare strings -->
<h1>Users</h1>
<Button title="Save" />
<Table emptyMessage="No data available" />
```

### Script Strings

```js
// ✅ Correct
import i18n from "@/utils/i18n";
const t = i18n.global.t;
const message = t("messages.created_success");

// ❌ Wrong
const message = "Created successfully";
```

### Locale File Structure

```json
// ar.json (PRIMARY — write this first)
{
  "moduleName": {
    "title": "اسم الموديول",
    "fields": {
      "name": "الاسم",
      "email": "البريد الإلكتروني",
      "status": "الحالة",
      "created_at": "تاريخ الإنشاء"
    },
    "messages": {
      "created": "تم الإنشاء بنجاح",
      "updated": "تم التعديل بنجاح",
      "deleted": "تم الحذف بنجاح",
      "confirm_delete": "هل أنت متأكد من الحذف؟"
    },
    "status": {
      "active": "نشط",
      "inactive": "غير نشط"
    }
  }
}

// en.json (SECONDARY — mirror ar.json structure exactly)
{
  "moduleName": {
    "title": "Module Name",
    "fields": {
      "name": "Name",
      "email": "Email",
      "status": "Status",
      "created_at": "Created At"
    },
    "messages": {
      "created": "Created successfully",
      "updated": "Updated successfully",
      "deleted": "Deleted successfully",
      "confirm_delete": "Are you sure you want to delete?"
    },
    "status": {
      "active": "Active",
      "inactive": "Inactive"
    }
  }
}
```

### Key Naming Convention

```
module.title           → Page/section title
module.fields.{name}   → Form field labels
module.messages.{verb}  → Success/error messages
module.status.{value}   → Enum/status labels
module.actions.{verb}   → Button labels (if module-specific)
```

## RTL Rules

### CSS Direction

```css
/* ✅ RTL-safe — works in both Arabic and English */
margin-inline-start: 1rem;
padding-inline-end: 0.5rem;
text-align: start;
border-inline-start: 2px solid;
inset-inline-start: 0;
float: inline-start;

/* ❌ Breaks in RTL */
margin-left: 1rem;
padding-right: 0.5rem;
text-align: left;
border-left: 2px solid;
left: 0;
float: left;
```

### Tailwind Classes

```html
<!-- ✅ RTL-safe Tailwind -->
<div class="ms-4 me-2 ps-3 pe-1 text-start">
  <!-- ❌ Breaks in RTL -->
  <div class="ml-4 mr-2 pl-3 pr-1 text-left"></div>
</div>
```

| LTR Class     | RTL-Safe Class | Description    |
| ------------- | -------------- | -------------- |
| `ml-*`        | `ms-*`         | Margin start   |
| `mr-*`        | `me-*`         | Margin end     |
| `pl-*`        | `ps-*`         | Padding start  |
| `pr-*`        | `pe-*`         | Padding end    |
| `left-*`      | `start-*`      | Position start |
| `right-*`     | `end-*`        | Position end   |
| `text-left`   | `text-start`   | Text alignment |
| `text-right`  | `text-end`     | Text alignment |
| `rounded-l-*` | `rounded-s-*`  | Border radius  |
| `rounded-r-*` | `rounded-e-*`  | Border radius  |
| `border-l-*`  | `border-s-*`   | Border side    |
| `border-r-*`  | `border-e-*`   | Border side    |

### Layout Direction

```
□ Use gap for spacing between elements — it's direction-agnostic
□ Use v-row/v-col for grid — it handles RTL automatically
□ Use flexbox with gap — flex-direction handles RTL
□ Icons that indicate direction (arrows, chevrons) should flip:
  - Use CSS transform: scaleX(-1) in RTL context
  - Or use RTL-aware icon variants
```

### Bidirectional Content

```vue
<!-- Mixed Arabic + English text -->
<p dir="auto">{{ item.name }}</p>

<!-- Known-direction content -->
<code dir="ltr">{{ item.code }}</code>
<p dir="rtl">{{ item.arabic_name }}</p>
```

## Validation

Run this check on all generated code:

```
□ No bare strings in templates — every visible text uses $t()
□ Both ar.json and en.json created with matching keys
□ No ml-/mr-/pl-/pr- classes — use ms-/me-/ps-/pe-
□ No left/right in CSS — use start/end/inline-start/inline-end
□ No text-left/text-right — use text-start/text-end
□ No hardcoded text direction — use dir="auto" for user content
□ Date formatting is locale-aware (useDateTimeFormatter)
□ Number formatting is locale-aware (useNumberConverter)
```
