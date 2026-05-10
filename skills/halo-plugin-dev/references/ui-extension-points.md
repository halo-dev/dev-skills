# UI Extension Points

Plugins can extend existing Console/UC UI via `extensionPoints` in `definePlugin()`.

## ExtensionPoint Keys

### Editor

```ts
"editor:create": () => EditorProvider[] | Promise<EditorProvider[]>;
"default:editor:extension:create": () => AnyExtension[] | Promise<AnyExtension[]>;
```

### Dashboard

```ts
"console:dashboard:widgets:create": () => DashboardWidgetDefinition[] | Promise<DashboardWidgetDefinition[]>;
"console:dashboard:widgets:internal:quick-action:item:create": () => DashboardWidgetQuickActionItem[] | Promise<DashboardWidgetQuickActionItem[]>;
```

### Attachment Selector

```ts
"attachment:selector:create": () => AttachmentSelectProvider[] | Promise<AttachmentSelectProvider[]>;
```

### Comment Subject Ref

```ts
"comment:subject-ref:create": () => CommentSubjectRefProvider[];
```

### List Item Operations

Add action buttons to list items:

```ts
"post:list-item:operation:create": (post: Ref<ListedPost>) => OperationItem<ListedPost>[];
"single-page:list-item:operation:create": (singlePage: Ref<ListedSinglePage>) => OperationItem<ListedSinglePage>[];
"comment:list-item:operation:create": (comment: Ref<ListedComment>) => OperationItem<ListedComment>[];
"reply:list-item:operation:create": (reply: Ref<ListedReply>) => OperationItem<ListedReply>[];
"plugin:list-item:operation:create": (plugin: Ref<Plugin>) => OperationItem<Plugin>[];
"backup:list-item:operation:create": (backup: Ref<Backup>) => OperationItem<Backup>[];
"attachment:list-item:operation:create": (attachment: Ref<Attachment>) => OperationItem<Attachment>[];
"theme:list-item:operation:create": (theme: Ref<Theme>) => OperationItem<Theme>[];
```

### List Item Fields

Add columns to list tables:

```ts
"plugin:list-item:field:create": (plugin: Ref<Plugin>) => EntityFieldItem[];
"post:list-item:field:create": (post: Ref<ListedPost>) => EntityFieldItem[];
"single-page:list-item:field:create": (singlePage: Ref<ListedSinglePage>) => EntityFieldItem[];
```

### Tabs

```ts
"plugin:self:tabs:create": () => PluginTab[] | Promise<PluginTab[]>;
"backup:tabs:create": () => BackupTab[] | Promise<BackupTab[]>;
"plugin:installation:tabs:create": () => PluginInstallationTab[] | Promise<PluginInstallationTab[]>;
"theme:list:tabs:create": () => ThemeListTab[] | Promise<ThemeListTab[]>;
"user:detail:tabs:create": () => UserTab[] | Promise<UserTab[]>;
"uc:user:profile:tabs:create": () => UserProfileTab[] | Promise<UserProfileTab[]>;
```

## Example: Adding a Post List Operation

```ts
import { definePlugin } from "@halo-dev/ui-shared";
import { VButton } from "@halo-dev/components";
import { h } from "vue";

export default definePlugin({
  extensionPoints: {
    "post:list-item:operation:create": (post) => [
      {
        priority: 10,
        component: h(
          VButton,
          {
            size: "sm",
            onClick: () => {
              console.log("Action on post:", post.value.metadata.name);
            },
          },
          () => "My Action",
        ),
      },
    ],
  },
});
```

## Example: Registering a Dashboard Widget

```ts
import { definePlugin } from "@halo-dev/ui-shared";
import MyWidget from "./components/MyWidget.vue";

export default definePlugin({
  extensionPoints: {
    "console:dashboard:widgets:create": () => [
      {
        id: "my-plugin-widget",
        name: "My Widget",
        component: MyWidget,
        priority: 10,
        permissions: [],
      },
    ],
  },
});
```
