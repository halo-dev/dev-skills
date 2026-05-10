# UI Entry & Routes

The frontend entry file exports a plugin definition using `definePlugin` from `@halo-dev/ui-shared`.

## Entry File (`ui/src/index.ts`)

```ts
import { definePlugin } from "@halo-dev/ui-shared";
import HomeView from "./views/HomeView.vue";
import { IconComputer } from "@halo-dev/components";
import { markRaw } from "vue";

export default definePlugin({
  components: {
    // Global component registration
    // "MyComponent": MyComponent
  },
  routes: [
    // Console routes
    {
      parentName: "Root",
      route: {
        path: "/my-plugin",
        name: "MyPluginHome",
        component: HomeView,
        meta: {
          title: "My Plugin",
          permissions: [],
          menu: {
            name: "My Plugin",
            group: "tool", // dashboard | content | interface | system | tool
            icon: markRaw(IconComputer),
            priority: 40,
          },
        },
      },
    },
  ],
  ucRoutes: [
    // UC (User Center) routes
    {
      parentName: "Root",
      route: {
        path: "/uc-my-plugin",
        name: "MyPluginUCHome",
        component: HomeView,
        meta: {
          permissions: [],
          menu: {
            name: "My Plugin",
            priority: 40,
          },
        },
      },
    },
  ],
  extensionPoints: {
    // UI extension point implementations
  },
});
```

## Route Definition

### With parentName (RouteRecordAppend)

```ts
{
  parentName: "Root",
  route: { /* RouteRecordRaw */ }
}
```

### Without parentName (RouteRecordRaw)

```ts
{
  path: "/standalone",
  name: "StandalonePage",
  component: MyView
}
```

## Console Parent Routes

| parentName        | Section               |
| ----------------- | --------------------- |
| `Root`            | Top level             |
| `AttachmentsRoot` | Attachment management |
| `CommentsRoot`    | Comments              |
| `SinglePagesRoot` | Single pages          |
| `PostsRoot`       | Posts                 |
| `MenusRoot`       | Menus                 |
| `ThemeRoot`       | Themes                |
| `OverviewRoot`    | Overview              |
| `BackupRoot`      | Backups               |
| `PluginsRoot`     | Plugins               |
| `SettingsRoot`    | Settings              |
| `UsersRoot`       | Users                 |
| `ToolsRoot`       | Tools                 |

## UC Parent Routes

| parentName          | Section       |
| ------------------- | ------------- |
| `PostsRoot`         | Posts         |
| `NotificationsRoot` | Notifications |

## RouteMeta

```ts
interface RouteMeta {
  title?: string; // Browser tab title
  searchable?: boolean; // Include in Console global search
  permissions?: string[]; // Required UI permissions
  menu?: {
    name: string; // Menu display name
    group?: CoreMenuGroupId; // Built-in group or custom group name
    icon?: Component; // Vue icon component (use markRaw)
    priority: number; // Sort order (lower = higher)
    mobile?: boolean; // Show on mobile
  };
}
```

## Using markRaw for Icons

Always wrap icon components with `markRaw`:

```ts
import { IconComputer } from "@halo-dev/components";
import { markRaw } from "vue";

icon: markRaw(IconComputer);
```

## Permissions in Routes

```ts
meta: {
  permissions: ["plugin:my-plugin:person:manage"];
}
```

The route is hidden if the user lacks any of the listed permissions.
