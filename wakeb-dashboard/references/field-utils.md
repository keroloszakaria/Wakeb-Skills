# Field Utilities & Validation Reference

## Contents

- Available field creators
- createTextField full signature
- Dynamic field generation
- Form data helpers
- Validation rules (complete list)

---

## Field Creators (`@/utils/FieldUtils`)

```js
import {
  createTextField, // Text input
  createTextAreaField, // Multi-line text
  createSelectField, // Dropdown select
  createComboBoxField, // Combobox (searchable select)
  createNumberField, // Numeric input
  createDateTimeField, // Date/time picker
  createCheckBoxField, // Checkbox / Switch
  createRadioButtonField, // Radio buttons
  createImageInput, // Image/File upload
  createPasswordField, // Password with toggle
  createPhoneField, // Phone with country code
  createEditorField, // Rich text (Quill)
  createColorField, // Color picker
  createOtpInput, // OTP input
  createCaptchaField, // Captcha
  createMapField, // Google Maps field
  createButton, // Action button in form
} from "@/utils/FieldUtils";
```

---

## createTextField — Full Signature

All field creators follow a similar pattern. `createTextField` shows the complete options:

```js
createTextField({
  key: "fieldName", // API field name (required)
  label: "name", // i18n label key

  // Translation
  translatable: false,
  translateTo: null, // 'ar' | 'en'
  translateFrom: null,
  pairKey: null, // e.g., 'name.en' for translation pair

  // Validation
  required: true,
  minLength: 0,
  maxLength: null,
  isEmail: false,
  isArabicOnly: false,
  isEnglishOnly: false,
  isNumberOnly: false,
  rules: [], // Extra string-based rules (see below)

  // Display
  placeholder: null,
  icon: null,
  description: null,
  disabled: false,
  hide: false,

  // Layout
  cols: { md: 6, lg: 6 },
  mode: "vertical", // 'vertical' | 'horizontal'
  inputStyle: "flat", // 'flat' | 'outlined' | 'filled' | 'minimal' | 'viewMode'
  classList: "",

  // Data
  value: null,
  defaultValue: null,
  clearable: false,

  // Advanced
  setToFilter: false, // Add to filter schema
  allowVariables: false, // Enable template variables
  updateValueHandler: null, // Custom change handler
});
```

---

## Dynamic Field Generation

```js
import { generateDynamicField } from "@/utils/dynamicFieldFactory";

const field = generateDynamicField({
  type: "text", // text|textarea|number|password|select|combobox|radio|
  // checkbox|switchbox|phone|datetime|image|file|otp|captcha|editor
  key: "name",
  label: "Name",
  ...options,
});
```

---

## Form Data Helpers (`@/utils/formDataHandler`)

```js
import {
  transformSchemaToObject, // schema → flat object for API payload
  updateSchemaValues, // Populate schema fields from API response data
  resetSchemaValues, // Reset all fields to defaults
  handleErrors, // Apply 422 validation errors to schema fields
  findSchemaItem, // Find field by key (supports nested keys)
  handleFormData, // Convert to FormData (for file uploads)
} from "@/utils/formDataHandler";
```

**Field payload types** (for `transformSchemaToObject`):

- `'object'` — single nested object
- `'array'` — items pushed to array
- `'spread'` — spread properties to parent
- `'duplicateObject'` — array of objects
- `'mergedDuplicateObject'` — merged duplicate arrays

---

## Validation Rules

### Rule generator function

```js
import { generateValidationRules } from "@/utils/validationRules";

const rules = generateValidationRules({
  required: true,
  minLength: 3,
  maxLength: 255,
  isEmail: true,
  isArabicOnly: false,
  isEnglishOnly: false,
  isNumberOnly: false,
  isIntegerOnly: false,
  extraRules: ["url"],
});
```

### String-based rules (pass in field `rules: [...]` array)

**Simple type rules:**
`required`, `nullable`, `string`, `integer`, `numeric`, `decimal`, `boolean`,
`array`, `date`, `email`, `url`, `uuid`, `ip`, `ipv4`, `ipv6`, `json`,
`image`, `file`

**Parameterized rules:**

| Rule                             | Description                       |
| -------------------------------- | --------------------------------- |
| `min:5`                          | Minimum length/value              |
| `max:100`                        | Maximum length/value              |
| `between:1,10`                   | Value range                       |
| `size:5`                         | Exact size                        |
| `gt:5`, `gte:5`                  | Greater than / greater or equal   |
| `lt:5`, `lte:5`                  | Less than / less or equal         |
| `min_digits:3`                   | Minimum digit count               |
| `max_digits:5`                   | Maximum digit count               |
| `digits:6`                       | Exact digit count                 |
| `digits_between:3,5`             | Digit count range                 |
| `mimes:jpg,png,pdf`              | Allowed file extensions           |
| `mimetypes:image/jpeg,image/png` | Allowed MIME types                |
| `in:val1,val2`                   | Allowed values                    |
| `not_in:val1,val2`               | Disallowed values                 |
| `starts_with:prefix`             | Must start with                   |
| `ends_with:suffix`               | Must end with                     |
| `regex:^[a-z]+$`                 | Regex match                       |
| `not_regex:pattern`              | Regex mismatch                    |
| `after:2024-01-01`               | Date after                        |
| `before:2025-12-31`              | Date before                       |
| `after_or_equal:now`             | Date on or after                  |
| `before_or_equal:now`            | Date on or before                 |
| `date_equals:2024-01-01`         | Exact date match                  |
| `same:field`                     | Must match other field            |
| `different:field`                | Must differ from other field      |
| `confirmed`                      | Must match `{field}_confirmation` |
