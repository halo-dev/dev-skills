# Theme UI provider (Halo 2.26+)

An activated, version-compatible theme can provide the same `PluginModule`
contract used by plugins to extend Console and UC. Use this only when the theme
itself needs administrative UI, routes, components, or extension points.

Official documentation:

- https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/theme/ui-plugin.md
- https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/basics/ui/build.md

## Directory and entry

Halo reads the complete provider artifact only from `ui-plugin/dist`:

```text
theme-root/
├── templates/
├── theme.yaml
└── ui-plugin/
    ├── src/index.ts
    ├── package.json
    ├── vite.config.ts
    └── dist/
        ├── ui-plugin.json
        ├── main.<hash>.js
        ├── style.<hash>.css
        ├── chunks/
        └── assets/
```

The entry default-exports `definePlugin(...)`. Available routes, `ucRoutes`, and
extension points follow the plugin UI contract.

## Build configuration

Vite:

```ts
import { viteConfig } from "@halo-dev/ui-plugin-bundler-kit/vite";

export default viteConfig({
  provider: "theme",
  vite: {},
});
```

Rsbuild uses `@halo-dev/ui-plugin-bundler-kit/rsbuild` with
`provider: "theme"`. Install the matching Vue build-plugin peer dependency.

The provider normally reads the parent `theme.yaml`, outputs to `dist`, and uses
`/themes/{metadata.name}/ui-plugin/assets/` as its asset base. Preserve the
complete output directory when packaging the theme.

`format: "auto"` follows the same target inference as plugin UI builds. A simple
`spec.requires: ">=2.26.0"` selects ESM; use explicit IIFE only when compatibility
requires it. Do not override output paths, externals, or filenames without also
preserving the generated manifest and runtime contracts.

Halo registers the provider as `theme:{metadata.name}`. Its status is available
through `stores.uiPlugins()`. Do not depend on provider load order. A full
Console or UC refresh is required after theme installation, update, reload, or
switching to replace the module graph.
