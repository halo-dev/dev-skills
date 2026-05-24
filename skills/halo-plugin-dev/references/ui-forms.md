# UI Forms (FormKit)

Halo uses [FormKit](https://formkit.com/) as its form solution. FormKit is **globally registered** in both Console and UC (User Center) — you do **not** need to install or import FormKit in plugin UI code. Use `<FormKit>` components directly, or define forms via Schema in `Setting` resources.

> **Critical**: Do NOT build custom form components from scratch (e.g. raw `<input>` elements) in plugin pages. Always use FormKit inputs so your UI stays consistent with the rest of Halo.

## Two Ways to Use Forms

### 1. Setting Schema (Plugin Config)

For plugin settings that users configure in the plugin detail page, define the form in a `Setting` resource using FormKit Schema syntax (written in YAML):

```yaml
# src/main/resources/extensions/settings.yaml
apiVersion: v1alpha1
kind: Setting
metadata:
  name: my-plugin-settings # must match spec.settingName in plugin.yaml
spec:
  forms:
    - group: basic
      label: Basic Settings
      formSchema:
        - $formkit: text
          name: apiKey
          label: API Key
          value: ""
          validation: required
        - $formkit: switch
          name: enabled
          label: Enable Feature
          value: true
```

Then reference it in `plugin.yaml`:

```yaml
spec:
  settingName: my-plugin-settings
  configMapName: my-plugin-configmap
```

See [plugin-manifest.md](plugin-manifest.md#settings--configmap) for full setup.

### 2. Vue Component (Direct `<FormKit>`)

For forms inside plugin pages (e.g. a custom admin page), use `<FormKit>` components directly:

```vue
<template>
  <FormKit id="my-form" type="form" :actions="false" @submit="handleSubmit">
    <FormKit type="text" name="title" label="Title" validation="required" />
    <FormKit type="textarea" name="description" label="Description" :auto-height="true" />
    <FormKit type="switch" name="published" label="Published" :value="true" />
    <VButton type="primary" @click="$formkit.submit('my-form')"> Save </VButton>
  </FormKit>
</template>

<script setup lang="ts">
import { Toast } from "@halo-dev/components";

function handleSubmit(values: Record<string, unknown>) {
  console.log(values);
  Toast.success("Saved");
}
</script>
```

No `import { FormKit } from "@formkit/vue"` is needed — FormKit is globally registered.

## Available Inputs

### FormKit Built-ins (Official)

All standard FormKit inputs work out of the box:

| Input                     | Type                   | Description                                |
| ------------------------- | ---------------------- | ------------------------------------------ |
| `text`                    | `string`               | Single-line text                           |
| `textarea`                | `string`               | Multi-line text (with `auto-height` addon) |
| `email`                   | `string`               | Email with validation                      |
| `number`                  | `number`               | Numeric input                              |
| `password`                | `string`               | Password (Halo disables autocomplete)      |
| `date` / `datetime-local` | `string`               | Date pickers                               |
| `checkbox`                | `boolean` / `string[]` | Single or multi checkbox                   |
| `radio`                   | `string`               | Radio group                                |
| `range`                   | `number`               | Slider                                     |
| `file`                    | `FileList`             | File input                                 |
| `group`                   | `object`               | Nested object container                    |

### Halo Custom Inputs

Halo registers additional inputs for common CMS use cases. Use them exactly like built-ins:

#### `select` — Enhanced Select

Custom select with static or remote data source, multi-select, sorting, and search.

```yaml
- $formkit: select
  name: country
  label: Country
  searchable: true
  clearable: true
  options:
    - label: China
      value: cn
    - label: USA
      value: us
```

Remote data source:

```yaml
- $formkit: select
  name: post
  label: Post
  clearable: true
  action: /apis/api.console.halo.run/v1alpha1/posts
  requestOption:
    method: GET
    labelField: post.spec.title
    valueField: post.metadata.name
```

Key props: `options`, `action`, `requestOption`, `multiple`, `searchable`, `clearable`, `sortable`, `maxCount`.

#### `switch` — Toggle Switch

```yaml
- $formkit: switch
  name: enabled
  label: Enable
  value: false
  onValue: "active"
  offValue: "inactive"
```

#### `attachment` / `attachmentInput` — Attachment Picker

`attachment` (Halo 2.22+): supports preview, direct upload, and library selection.

```yaml
- $formkit: attachment
  name: logo
  label: Logo
  accepts:
    - "image/png"
    - "image/jpeg"
  width: "200px"
  aspectRatio: "1/1"
```

`attachmentInput`: simpler input that opens the attachment library modal.

```yaml
- $formkit: attachmentInput
  name: cover
  label: Cover
  accepts: ["image/*"]
  min: 1
  max: 1
```

#### `code` — Code Editor

Integrated with CodeMirror. Supports `yaml`, `html`, `javascript`, `css`, `json`.

```yaml
- $formkit: code
  name: custom_css
  label: Custom CSS
  language: css
  height: "300px"
```

#### `iconify` — Icon Selector

Based on [Iconify](https://iconify.design/).

```yaml
- $formkit: iconify
  name: social_icon
  label: Social Icon
  format: svg # svg | dataurl | url | name
```

With sizing options:

```yaml
- $formkit: iconify
  name: icon
  label: Icon
  format: svg
  sizing:
    enabled: true
    default: "24"
    presets: ["16", "24", "32", "48"]
```

#### `toggle` — Visual Toggle

For image, color, or text option toggling.

```yaml
- $formkit: toggle
  name: theme
  label: Theme
  render-type: color
  options:
    - label: Dark
      value: dark
      render: "#1a1a1a"
    - label: Light
      value: light
      render: "#ffffff"
```

#### `array` — Object Array (Recommended over `repeater`)

For defining arrays of objects with add/remove/reorder.

```yaml
- $formkit: array
  name: socials
  label: Social Accounts
  value: []
  max: 5
  min: 1
  itemLabels:
    - type: image
      label: $value.logo
    - type: text
      label: $value.name
  children:
    - $formkit: attachment
      name: logo
      label: Icon
    - $formkit: text
      name: name
      label: Name
      validation: required
    - $formkit: text
      name: url
      label: URL
      validation: required|url
```

> Use `itemLabels` to show preview content on collapsed array items. `$value` refers to the current item.

#### `list` — Primitive Array

For arrays of primitives (strings, numbers, booleans).

```yaml
- $formkit: list
  name: tags
  label: Tags
  itemType: string
  min: 1
  max: 10
  addLabel: Add Tag
  children:
    - $formkit: text
      index: "$index"
      validation: required
```

#### `verificationForm` — Remote Validation

Wraps a group of fields and validates them against a remote endpoint.

```yaml
- $formkit: verificationForm
  action: /apis/console.api.halo.run/v1alpha1/verify/verify-password
  label: Verify Account
  children:
    - $formkit: text
      name: username
      label: Username
      validation: required
    - $formkit: password
      name: password
      label: Password
      validation: required
```

> Unlike other inputs, `verificationForm` does NOT wrap values in its own key. The saved values stay flat: `{ "username": "...", "password": "..." }`.

#### CMS Entity Selectors

Halo provides dedicated selectors for core CMS entities. All return the resource's `metadata.name`.

| Input                    | Description                | Multi-select |
| ------------------------ | -------------------------- | ------------ |
| `menuSelect`             | Navigation menu selector   | Yes          |
| `menuCheckbox`           | Menu checkbox group        | Yes (array)  |
| `menuRadio`              | Menu radio selection       | No           |
| `postSelect`             | Post selector              | No           |
| `singlePageSelect`       | Single page selector       | No           |
| `categorySelect`         | Category selector          | No           |
| `categoryCheckbox`       | Category checkbox          | Yes (array)  |
| `tagSelect`              | Tag selector               | No           |
| `tagCheckbox`            | Tag checkbox               | Yes (array)  |
| `userSelect`             | User selector              | Yes          |
| `roleSelect`             | Role selector              | Yes          |
| `attachmentGroupSelect`  | Attachment group selector  | Yes          |
| `attachmentPolicySelect` | Attachment policy selector | Yes          |

Example:

```yaml
- $formkit: postSelect
  name: featuredPost
  label: Featured Post
  value: ""

- $formkit: categoryCheckbox
  name: categories
  label: Categories
  value: []
```

#### `secret` — Secret Resource Selector

For selecting a Halo Secret resource (stores sensitive data like API keys).

```yaml
- $formkit: secret
  name: apiSecret
  label: API Secret
  requiredKeys:
    - key: apiKey
      help: API Key
    - key: secretKey
      help: Secret Key
```

#### `color` — Color Picker

```yaml
- $formkit: color
  name: themeColor
  label: Theme Color
  value: "#1890ff"
```

## Programmatic Form Submission

In Vue components, trigger form submission programmatically:

```vue
<VButton type="primary" @click="$formkit.submit('my-form-id')">
  Submit
</VButton>
```

Or using `@formkit/core`:

```ts
import { submitForm } from "@formkit/core";

submitForm("my-form-id");
```

## Validation

FormKit supports built-in validation rules. Use them in Schema or Vue components:

```yaml
- $formkit: text
  name: email
  label: Email
  validation: required|email
```

```vue
<FormKit
  type="text"
  name="slug"
  label="Slug"
  :validation="[['required'], ['matches', /^[a-z0-9-]+$/]]"
/>
```

Common rules: `required`, `email`, `url`, `number`, `min`, `max`, `matches`, `confirm`.

## Conditional Rendering

Use `if` in Schema to conditionally show fields:

```yaml
- $formkit: select
  name: type
  label: Type
  options:
    - label: Internal
      value: internal
    - label: External
      value: external

- $formkit: text
  name: url
  label: URL
  if: "$value.type === 'external'"
  validation: required|url
```

> In `if` expressions, `$value` refers to the current form values object.

## Schema vs Vue Component: When to Use Which

| Scenario                             | Approach                                       |
| ------------------------------------ | ---------------------------------------------- |
| Plugin settings (config page)        | `Setting` resource with Schema                 |
| Custom admin page with dynamic logic | Vue `<FormKit>` components                     |
| Simple CRUD form in a modal          | Vue `<FormKit>` components                     |
| Reusable form across plugins         | Vue `<FormKit>` components in a shared package |

## Important Notes

- **Do NOT install FormKit in your plugin** — it's already globally registered. Installing it again can cause conflicts.
- **Do NOT use FormKit Pro inputs** — they are not included in Halo.
- Schema is JSON format natively, but Halo uses YAML for `Setting` resources. Write Schema in YAML syntax.
- When using `array` or `list`, always provide `value: []` as default to avoid undefined issues.
- For `attachment` with `multiple: true`, the value is a `string[]` of attachment URLs/names.
