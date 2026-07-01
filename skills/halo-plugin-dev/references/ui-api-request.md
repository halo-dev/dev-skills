# API Requests in Plugin UI

Halo provides `@halo-dev/api-client` for making API calls from plugin Vue/TypeScript code.

## Installation

```bash
pnpm install @halo-dev/api-client axios
```

> `@halo-dev/ui-plugin-bundler-kit@2.17.0+` already excludes `@halo-dev/api-client` and `axios` from the bundle — the final build will use Halo's own copies. If using these versions, set `spec.requires: ">=2.17.0"` in `plugin.yaml`.

## Built-in API Clients

`@halo-dev/api-client` exports pre-configured clients for Halo's built-in APIs. They handle base URL, auth, error handling (login expiry, permission denied), etc.

```ts
import {
  coreApiClient, // CRUD for all Extensions
  consoleApiClient, // Console APIs
  ucApiClient, // User Center APIs
  publicApiClient, // Public APIs
  axiosInstance, // Raw axios instance
} from "@halo-dev/api-client";
```

## Error Handling

Halo adds global response interceptors to the shared Axios instance used by `@halo-dev/api-client`. Request failures from `coreApiClient`, `consoleApiClient`, `ucApiClient`, `publicApiClient`, generated clients constructed with `axiosInstance`, or direct `axiosInstance` calls already show Halo-managed error toasts.

- Do not add local `try/catch`, `catch`, or `useMutation.onError` handlers that call `Toast.error` / `Toast.warning` for those Axios request failures. Doing so can show duplicate toasts.
- Keep local toasts for non-Axios errors such as client-side validation, parsing failures, missing local prerequisites, or domain-specific messages created before a request is sent.
- If a handler must run cleanup or custom logic after a failed request, guard Axios errors with `isAxiosError` and do not toast them locally.

```ts
import { Toast } from "@halo-dev/components";
import { isAxiosError } from "axios";

function toastNonAxiosError(error: unknown) {
  if (isAxiosError(error)) {
    return;
  }

  Toast.error(error instanceof Error ? error.message : "Operation failed");
}
```

### coreApiClient (Extension CRUD)

```ts
// List posts
const { data } = await coreApiClient.content.post.listPost({
  page: 1,
  size: 10,
  sort: ["spec.publishTime,desc"],
});

// Get a config map
const { data: configMap } = await coreApiClient.extension.configMap.getv1alpha1ConfigMap({
  name: "my-plugin-configmap",
});
```

### consoleApiClient / ucApiClient / publicApiClient

```ts
// Console: list attachments
const { data } = await consoleApiClient.attachment.listAttachments({
  page: 1,
  size: 20,
});

// UC: get current user notifications
const { data } = await ucApiClient.notification.listNotifications();

// Public: search
const { data } = await publicApiClient.post.searchPost({ keyword: "halo" });
```

## Calling Plugin Custom APIs

For APIs defined by your plugin (CustomEndpoint, @Controller, etc.), use the raw `axiosInstance`:

```ts
import { axiosInstance } from "@halo-dev/api-client";

// GET custom endpoint
const { data } = await axiosInstance.get("/apis/console.api.my-plugin.halo.run/v1alpha1/items");

// POST with body
await axiosInstance.post("/apis/console.api.my-plugin.halo.run/v1alpha1/items", {
  name: "new-item",
});

// Custom query params
const { data } = await axiosInstance.get("/apis/api.my-plugin.halo.run/v1alpha1/public/items", {
  params: { page: 1, size: 10 },
});
```

## Generated API Client (Recommended for Plugin APIs)

For plugin-defined APIs, use the DevTools `generateApiClient` Gradle task to generate a typed TypeScript client from your OpenAPI spec.

### 1. Configure OpenAPI grouping in `build.gradle`

```groovy
haloPlugin {
    openApi {
        groupingRules {
            extensionApis {
                displayName = 'Extension API for MyPlugin'
                pathsToMatch = ['/apis/my-plugin.halo.run/v1alpha1/**']
            }
        }
        groupedApiMappings = [
            '/v3/api-docs/extensionApis': 'extensionApis.json'
        ]
        generator {
            outputDir = file("${projectDir}/ui/src/api/generated")
            additionalProperties = [
                useES6: true,
                useSingleRequestParameter: true,
                withSeparateModelsAndApi: true,
                apiPackage: "api",
                modelPackage: "models"
            ]
            typeMappings = [
                set: "Array"
            ]
        }
    }
}
```

### 2. Generate the client

```bash
./gradlew generateApiClient
```

### 3. Use the generated client with `axiosInstance`

```ts
import { axiosInstance } from "@halo-dev/api-client";
import { MyResourceV1alpha1Api } from "./api/generated";

const api = new MyResourceV1alpha1Api(undefined, "", axiosInstance);

// List with typed parameters
const { data } = await api.listMyResources({ page: 1, size: 10 });

// Create with typed body
await api.createMyResource({ myResource: { ... } });
```

> The generated client needs `axiosInstance` as its third constructor argument so it uses Halo's pre-configured axios (with auth, base URL, error handling).

## Data Fetching with `@tanstack/vue-query`

For managing server state (caching, refetching, mutations) in plugin Vue components, use `@tanstack/vue-query`.

> **Version warning:** Halo plugins currently use **v4** of `@tanstack/vue-query` (e.g. `^4.44.0`). Do **not** install v5 — the API is incompatible.

```bash
pnpm install @tanstack/vue-query@^4.44.0
```

Halo's plugin runtime already provides `VueQueryPlugin` setup — plugin code can use `useQuery` / `useMutation` directly without additional configuration.

### Usage Example

```vue
<script setup lang="ts">
import { consoleApiClient } from "@halo-dev/api-client";
import { useQuery } from "@tanstack/vue-query";

const { data, isLoading } = useQuery({
  queryKey: ["attachments"],
  queryFn: async () => {
    const { data } = await consoleApiClient.attachment.listAttachments({
      page: 1,
      size: 20,
    });
    return data;
  },
});
</script>
```

> Use `useQuery` for read operations and `useMutation` + `queryClient.invalidateQueries()` for create/update/delete operations.

## When to Use Which

| Approach                              | Use for                      | Example                             |
| ------------------------------------- | ---------------------------- | ----------------------------------- |
| `coreApiClient` / `consoleApiClient`  | Halo built-in APIs           | List posts, fetch users             |
| `axiosInstance` directly              | Ad-hoc plugin API calls      | Simple GET/POST to custom endpoints |
| `generateApiClient` + `axiosInstance` | Plugin APIs with type safety | Full CRUD on your custom Extension  |
| `@tanstack/vue-query` + API clients   | Server state in Vue UI       | Cached lists, mutations, loading UI |
