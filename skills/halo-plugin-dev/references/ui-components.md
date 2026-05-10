# UI Components

Halo provides two layers of UI primitives for plugin frontends:

1. **Base component library** (`@halo-dev/components`) — install and import explicitly
2. **Business components & directives** — globally registered, use directly without import

> Full base component docs: https://halo-ui-components.pages.dev

## Base Component Library

Install:

```bash
pnpm install @halo-dev/components
```

Import and use:

```vue
<script lang="ts" setup>
import { ref } from "vue";
import { VButton, VModal, VCard } from "@halo-dev/components";
const visible = ref(false);
</script>

<template>
  <VButton type="secondary" @click="visible = true">Open</VButton>
  <VModal v-if="visible" @close="visible = false" title="Title">
    <VCard>Content</VCard>
  </VModal>
</template>
```

Actual exported components from `@halo-dev/components`:

| Component                                       | Purpose                                                                    |
| ----------------------------------------------- | -------------------------------------------------------------------------- |
| `VAlert`                                        | Alert banner                                                               |
| `VAvatar` / `VAvatarGroup`                      | User avatar(s)                                                             |
| `VButton`                                       | Button with variants: `default`, `primary`, `secondary`, `danger`, `ghost` |
| `VCard`                                         | Card container                                                             |
| `VDescription` / `VDescriptionItem`             | Key-value description list                                                 |
| `VDialog` + `Dialog` (manager)                  | Dialog with imperative API                                                 |
| `VDropdownDivider` / `VDropdownItem`            | Dropdown menu items                                                        |
| `VEmpty`                                        | Empty state placeholder                                                    |
| `VEntity` / `VEntityContainer` / `VEntityField` | Entity list item layout                                                    |
| `VLoading`                                      | Loading spinner/overlay                                                    |
| `VMenu` / `VMenuItem` / `VMenuLabel`            | Menu navigation                                                            |
| `VModal`                                        | Modal with `v-model:visible`                                               |
| `VPagination`                                   | Pagination control                                                         |
| `VPageHeader`                                   | Page header with back button/title/actions                                 |
| `VSpace`                                        | Flex spacing layout                                                        |
| `VStatusDot`                                    | Status indicator dot                                                       |
| `VSwitch`                                       | Toggle switch                                                              |
| `VTabbar` / `VTabs` / `VTabItem`                | Tab navigation                                                             |
| `VTag`                                          | Colored tag/badge                                                          |
| `Toast` (manager)                               | Toast notification imperative API                                          |
| `VTooltipComponent` / `vTooltip`                | Tooltip component/directive                                                |

## Business Components (Globally Registered)

These are available without import in any plugin Vue component.

### VCodemirror

Code editor.

```vue
<VCodemirror v-model="value" height="300px" language="yaml" />
```

| Prop         | Type     | Default  | Description                                   |
| ------------ | -------- | -------- | --------------------------------------------- |
| `modelValue` | `string` | `""`     | Binding value                                 |
| `height`     | `string` | `"auto"` | Editor height                                 |
| `language`   | `string` | `"yaml"` | Language: `yaml`, `html`, `js`, `css`, `json` |
| `extensions` | `array`  | `[]`     | Codemirror extensions                         |

### AttachmentSelectorModal

Attachment picker modal (Console only).

```vue
<script setup>
const visible = ref(false);
function onSelect(attachments) {
  console.log(attachments); // AttachmentLike[]
}
</script>

<template>
  <VButton @click="visible = true">Select</VButton>
  <AttachmentSelectorModal
    v-if="visible"
    @close="visible = false"
    :accepts="['image/*']"
    :min="1"
    :max="5"
    @select="onSelect"
  />
</template>
```

| Prop          | Type       | Default   | Description             |
| ------------- | ---------- | --------- | ----------------------- |
| `visible`     | `boolean`  | `false`   | Controlled visibility   |
| `accepts`     | `string[]` | `["*/*"]` | Accepted MIME types     |
| `min` / `max` | `number`   | —         | Min/max selection count |

### UppyUpload

File upload component.

```vue
<UppyUpload
  endpoint="/apis/api.console.halo.run/v1alpha1/attachments/upload"
  :meta="{ policyName, groupName }"
  @uploaded="onUploaded"
  @error="onError"
/>
```

| Prop          | Type                      | Default  | Description                     |
| ------------- | ------------------------- | -------- | ------------------------------- |
| `endpoint`    | `string`                  | required | Upload API endpoint             |
| `meta`        | `Record<string, unknown>` | —        | Extra metadata sent with upload |
| `autoProceed` | `boolean`                 | `false`  | Auto-upload on select           |
| `method`      | `string`                  | `"post"` | HTTP method                     |

### SearchInput

Search input that only triggers on Enter (not while typing).

```vue
<SearchInput v-model="keyword" placeholder="Search..." />
```

### AnnotationsForm

Renders the Annotations form for a given Extension group/kind.

```vue
<script setup>
const formRef = ref();

async function handleSubmit() {
  formRef.value?.handleSubmit();
  await nextTick();
  const { customAnnotations, annotations, customFormInvalid, specFormInvalid } =
    formRef.value || {};
  if (customFormInvalid || specFormInvalid) return;
  const merged = { ...annotations, ...customAnnotations };
  // ...submit merged
}
</script>

<template>
  <AnnotationsForm ref="formRef" :value="currentAnnotations" kind="Post" group="content.halo.run" />
  <VButton @click="handleSubmit">Save</VButton>
</template>
```

### FilterDropdown / FilterCleanButton

Generic filter dropdown and clear button for list pages.

```vue
<FilterDropdown
  v-model="sortValue"
  label="Sort"
  :items="[
    { label: 'Newest', value: 'creationTimestamp,desc' },
    { label: 'Oldest', value: 'creationTimestamp,asc' },
  ]"
/>

<FilterCleanButton @click="resetFilters" />
```

### PluginDetailModal

Open a plugin's detail/settings modal inline.

```vue
<PluginDetailModal v-if="visible" name="my-plugin" @close="visible = false" />
```

| Prop   | Type     | Description          |
| ------ | -------- | -------------------- |
| `name` | `string` | Plugin metadata.name |

## Directives (Globally Registered)

### v-permission

Conditionally render based on permissions.

```vue
<VButton type="danger" v-permission="['system:posts:manage']">Delete</VButton>
```

Equivalent component: `<HasPermission :permissions="["..."]">...</HasPermission>`

### v-tooltip

Add tooltip to any element.

```vue
<IconDeleteBin v-tooltip="'Delete this item'" />
```
