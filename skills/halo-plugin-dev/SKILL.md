---
name: halo-plugin-dev
description: >
  Use when creating or modifying a Halo CMS plugin: writing Java backend code,
  configuring plugin.yaml, defining custom extensions (GVK), creating custom APIs
  (CustomEndpoint, MVC controllers), building Vue-based UI with @halo-dev/ui-shared,
  setting up Gradle builds with DevTools, defining RBAC role templates, declaring
  pluginDependencies, exposing or consuming plugin extension points, publishing
  shared events, registering custom FormKit inputs, generating API clients from
  OpenAPI, registering theme-side Finder APIs, or handling plugin lifecycle
  (start/stop/delete).
---

# Halo Plugin Development

Halo is built on **Spring Boot + Spring WebFlux + Vue 3**. A plugin consists of:

- **Backend (Java)**: runs inside Halo's JVM, uses Spring DI, reactive WebFlux, custom extensions (CRD-like), and custom APIs
- **Frontend (Vue/TypeScript)**: built into `main.js` + `style.css`, injected into Console and UC (User Center)
- **Manifest (`plugin.yaml`)**: plugin metadata, dependencies, settings, and config map names

> **Important**: Halo's plugin APIs, VO field names, extension annotations, and UI APIs evolve across versions. **Do not rely on training data for specific field names, method signatures, or type structures.** When writing code that accesses extension fields or calls shared beans, always fetch the relevant online doc from the References section below first.

## Quick Start

Create a new plugin project using the official scaffolding tool:

```bash
pnpm create halo-plugin
```

Follow the prompts (plugin name, domain, author, UI build tool: Rsbuild or Vite).

Then run with DevTools (requires Docker):

```bash
./gradlew haloServer
```

Visit `http://localhost:8090/console` — username/password defaults to `admin`/`admin`.

After code changes:

```bash
./gradlew reload
```

Or use `watch` for auto-reload:

```bash
./gradlew watch
```

## Development Workflow

1. **Scaffold**: `pnpm create halo-plugin`
2. **Backend**: write Java code under `src/main/java/`
3. **Frontend**: write Vue/TS code under `ui/src/` (or `console/src/`)
4. **Manifest**: configure `src/main/resources/plugin.yaml`
5. **Extensions**: declare YAML resources under `src/main/resources/extensions/`
6. **Run**: `./gradlew haloServer` (with Docker)
7. **Test**: visit Console at `http://localhost:8090/console`
8. **Build**: `./gradlew build` produces a JAR for distribution

## References Index

| File                                                                       | Content                                                                                           | When to read                                                                                                                              |
| -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| [references/api-changelog.md](references/api-changelog.md)                 | High-impact plugin API changes by Halo version, with docs routes                                  | Before using version-sensitive APIs, upgrading Halo dependencies, or raising `spec.requires`                                              |
| [references/plugin-structure.md](references/plugin-structure.md)           | Directory structure, backend/frontend layout, build.gradle basics                                 | Creating a new plugin from scratch or understanding the directory layout                                                                  |
| [references/plugin-manifest.md](references/plugin-manifest.md)             | plugin.yaml fields, version requirements, dependencies, settings/configMap                        | Writing or editing plugin.yaml                                                                                                            |
| [references/plugin-interaction.md](references/plugin-interaction.md)       | pluginDependencies, API modules, shared events, defining and consuming plugin extension points    | Depending on another plugin, exposing an API module, sharing events, or making a plugin extensible                                        |
| [references/devtools.md](references/devtools.md)                           | haloServer, reload, watch, generateApiClient, generateRoleTasks, debug config                     | Running `./gradlew haloServer`, hot reload, or debugging a plugin                                                                         |
| [references/server-extension.md](references/server-extension.md)           | Custom Extension (GVK), AbstractExtension, CRUD APIs, indexes, field/label selectors              | Defining a custom data model, storage, or query indexes                                                                                   |
| [references/server-api.md](references/server-api.md)                       | CustomEndpoint, @Controller with @ApiVersion, query params, validation, OpenAPI docs              | Writing a new backend API endpoint or controller                                                                                          |
| [references/server-lifecycle.md](references/server-lifecycle.md)           | BasePlugin lifecycle (start/stop/delete), Scheme registration/cleanup                             | Handling plugin start, stop, delete, or scheme registration                                                                               |
| [references/server-shared-beans.md](references/server-shared-beans.md)     | ReactiveExtensionClient, SchemeManager, UserService, AttachmentService, ExtensionGetter, etc.     | Injecting or calling Halo core services from plugin Java code                                                                             |
| [references/server-security.md](references/server-security.md)             | Role templates, RBAC rules, verbs, aggregation, UI permissions                                    | Adding RBAC roles, API permissions, or UI permission checks                                                                               |
| [references/ui-entry.md](references/ui-entry.md)                           | definePlugin, routes/ucRoutes, menu config, parentName, RouteMeta                                 | Adding a new page to the Console or User Center                                                                                           |
| [references/ui-build.md](references/ui-build.md)                           | @halo-dev/ui-plugin-bundler-kit (Vite/Rsbuild), output dirs, migration                            | Configuring the frontend build (Vite/Rsbuild) or troubleshooting bundling                                                                 |
| [references/ui-shared.md](references/ui-shared.md)                         | stores (currentUser, globalInfo), utils (date, permission, attachment, id), events                | Formatting dates, checking permissions, handling attachments, generating IDs — do NOT install dayjs/date-fns or write your own date utils |
| [references/ui-extension-points.md](references/ui-extension-points.md)     | ExtensionPoint keys, editor, attachment selector, dashboard widgets, list operations/fields       | Extending existing Console UI (editor, lists, attachment picker, dashboard)                                                               |
| [references/ui-components.md](references/ui-components.md)                 | Base components (@halo-dev/components), business components, directives (v-permission, v-tooltip) | Looking for a UI component, directive, or modal API to use in Vue code                                                                    |
| [references/ui-forms.md](references/ui-forms.md)                           | FormKit schema and component usage, custom inputs (select, attachment, array, etc.), validation   | Building a form with FormKit schema or custom inputs                                                                                      |
| [references/ui-api-request.md](references/ui-api-request.md)               | @halo-dev/api-client (coreApiClient, axiosInstance), generateApiClient Gradle task                | Making HTTP requests from plugin UI to Halo APIs                                                                                          |
| [references/ui-tooling.md](references/ui-tooling.md)                       | unplugin-icons + Iconify, UnoCSS (Vite/Rsbuild config)                                            | Adding icons or writing atomic CSS (UnoCSS) in plugin UI                                                                                  |
| [references/server-reconciler.md](references/server-reconciler.md)         | Reconciler + ControllerBuilder, finalizers, retry scheduling, vs Watcher                          | Building a controller that watches and reconciles resource state                                                                          |
| [references/server-search.md](references/server-search.md)                 | HaloDocument, HaloDocumentsProvider, SearchEngine, search events                                  | Integrating with Halo search (indexing, searching, search events)                                                                         |
| [references/theme-head-processor.md](references/theme-head-processor.md)   | TemplateHeadProcessor for injecting scripts/styles/meta into theme head                           | Injecting scripts, styles, or meta tags into the theme `<head>`                                                                           |
| [references/theme-content-handler.md](references/theme-content-handler.md) | ReactivePostContentHandler / ReactiveSinglePageContentHandler for modifying rendered HTML         | Modifying post or page HTML after rendering                                                                                               |
| [references/theme-integration.md](references/theme-integration.md)         | Finder API for themes, template variables, reverse proxy, static resources, CommentSubject        | Adding theme-side template variables or Finder APIs                                                                                       |
