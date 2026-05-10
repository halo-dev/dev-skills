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

## When to Use Which

| Approach                              | Use for                      | Example                             |
| ------------------------------------- | ---------------------------- | ----------------------------------- |
| `coreApiClient` / `consoleApiClient`  | Halo built-in APIs           | List posts, fetch users             |
| `axiosInstance` directly              | Ad-hoc plugin API calls      | Simple GET/POST to custom endpoints |
| `generateApiClient` + `axiosInstance` | Plugin APIs with type safety | Full CRUD on your custom Extension  |
