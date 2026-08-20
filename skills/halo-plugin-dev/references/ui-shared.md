# @halo-dev/ui-shared Utilities

Available from Halo 2.22. Requires plugin `spec.requires: ">=2.22.0"`.

## Stores (Pinia)

Install Pinia: `pnpm install pinia`

### uiPlugins (Halo 2.26+)

Use this store to inspect whether a plugin or activated-theme UI provider was
discovered and whether it registered successfully:

```ts
import { stores } from "@halo-dev/ui-shared";
import { computed } from "vue";

const uiPlugins = stores.uiPlugins();

uiPlugins.isEnabled("plugin-search");
const registered = computed(() => uiPlugins.isRegistered("plugin-search"));
const registration = uiPlugins.get("plugin-search");
```

Theme provider names use `theme:{metadata.name}`. The registration records are
read-only. Do not mutate them, depend on provider load order, or use the store to
import or invoke another provider's implementation. Replace checks for IIFE
globals such as `window.PluginName` or `window.enabledUiPlugins` with
`isEnabled()` when targeting Halo 2.26+.

### currentUser

```ts
import { stores } from "@halo-dev/ui-shared";
import { storeToRefs } from "pinia";

const userStore = stores.currentUser();
await userStore.fetchCurrentUser();

console.log(userStore.currentUser?.user.metadata.name);
console.log(userStore.isAnonymous);

// Reactive refs
const { currentUser, isAnonymous } = storeToRefs(stores.currentUser());
```

| Property      | Type                        | Description                  |
| ------------- | --------------------------- | ---------------------------- |
| `currentUser` | `DetailedUser \| undefined` | Current user info            |
| `isAnonymous` | `boolean`                   | Whether visitor is anonymous |

### globalInfo

```ts
const globalInfoStore = stores.globalInfo();
await globalInfoStore.fetchGlobalInfo();

console.log(globalInfoStore.globalInfo?.externalUrl);
console.log(globalInfoStore.globalInfo?.siteTitle);
```

| Property     | Type                      | Description                                                                                            |
| ------------ | ------------------------- | ------------------------------------------------------------------------------------------------------ |
| `globalInfo` | `GlobalInfo \| undefined` | Site config: externalUrl, siteTitle, timeZone, locale, allowComments, allowRegistration, favicon, etc. |

## Utils

### date (dayjs-based)

Dayjs is already bundled in `@halo-dev/ui-shared` and exposed via `utils.date`. **Always use this for any date handling in plugin UI code — do NOT install `dayjs`, `date-fns`, or any other date library, and do NOT write your own date formatting functions.**

```ts
import { utils } from "@halo-dev/ui-shared";

utils.date.format(new Date()); // "2025-11-05 14:30"
utils.date.format("2025-10-22", "YYYY/MM/DD"); // "2025/10/22"
utils.date.toISOString(new Date()); // ISO string
utils.date.toDatetimeLocal(new Date()); // "2025-10-22T14:30"
utils.date.timeAgo("2025-10-23"); // "1 天后"
utils.date.dayjs(); // raw dayjs instance for advanced usage
```

### permission

```ts
utils.permission.has(["core:posts:manage"]); // any match
utils.permission.has(["core:posts:manage", "core:posts:delete"], false); // all match
utils.permission.getUserPermissions(); // string[]
```

### attachment

```ts
// Generate thumbnail URL
utils.attachment.getThumbnailUrl("/uploads/image.jpg", "M"); // "?width=800"
// Sizes: "XL" (1600), "L" (1200), "M" (800), "S" (400)

// Extract URL from various attachment formats
utils.attachment.getUrl(attachmentObject);

// Convert to simplified format
utils.attachment.convertToSimple(attachmentObject);
// -> { url: "...", alt?: "...", mediaType?: "..." }
```

### id

```ts
utils.id.uuid(); // UUID v7 (time-sortable)
```

## Events

```ts
import { events } from "@halo-dev/ui-shared";

// Listen for plugin config updates
events.on("core:plugin:configMap:updated", (data) => {
  console.log(`Plugin ${data.pluginName} config updated, group: ${data.group}`);
});
```

| Event                           | Payload                                 | Description                  |
| ------------------------------- | --------------------------------------- | ---------------------------- |
| `core:plugin:configMap:updated` | `{ pluginName: string, group: string }` | Plugin configuration changed |
