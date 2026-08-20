# UI Build (`@halo-dev/ui-plugin-bundler-kit`)

Use the bundler kit rather than recreating Halo's UI-provider output protocol.
First inspect the installed bundler-kit version, `plugin.yaml` `spec.requires`, and
the project's existing Vite or Rsbuild configuration.

Official documentation:

- https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/basics/ui/build.md
- https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/api-changelog.md

## Choose the existing build tool

Both Vite and Rsbuild support IIFE and ESM providers. Preserve the project's
current tool unless the user asks to migrate it.

For a new Halo 2.26+ setup, use the build-tool-specific entry and install its Vue
plugin peer dependency.

### Vite

```bash
pnpm install @halo-dev/ui-plugin-bundler-kit@2.26.0 vite @vitejs/plugin-vue -D
```

```ts
import { viteConfig } from "@halo-dev/ui-plugin-bundler-kit/vite";

export default viteConfig();
```

### Rsbuild

```bash
pnpm install @halo-dev/ui-plugin-bundler-kit@2.26.0 @rsbuild/core @rsbuild/plugin-vue -D
```

```ts
import { rsbuildConfig } from "@halo-dev/ui-plugin-bundler-kit/rsbuild";

export default rsbuildConfig();
```

The package-root `viteConfig` and `rsbuildConfig` exports are compatibility
aliases in 2.26 and are planned for removal in 2.27. Preserve them only when the
project is pinned to an older bundler kit that lacks the specific entry.

The presets already create the Vue plugin. Configure Vue compilation through
the top-level `vue` option; do not add another Vue plugin instance.

## Output format and Halo target

`format` accepts `"auto"`, `"iife"`, or `"esm"`. The default `"auto"` mode only
infers a target from a stable version such as `2.26.0` or a simple minimum such
as `>=2.26.0`:

- Target 2.26.0 or newer: ESM.
- Older target: IIFE.
- A wildcard or compound range that cannot establish a minimum: warn and fall
  back to IIFE.

Use an explicit `format: "iife"` when an existing project cannot yet satisfy the
ESM contract. Use `targetHaloVersion` only when an explicit ESM build cannot be
derived from `spec.requires`; it does not change the plugin's installation
compatibility. Keep `spec.requires` consistent with the runtime needed by the
emitted provider.

## ESM artifact contract

An ESM build includes generated provider metadata and may contain hashed entry,
style, chunk, and asset files:

```text
ui/build/dist/
├── ui-plugin.json
├── main.<hash>.js
├── style.<hash>.css
├── chunks/
└── assets/
```

Treat the complete directory as one artifact:

- Do not create, copy, or edit `ui-plugin.json` manually.
- Do not copy only the startup entry and stylesheet.
- Do not assume fixed entry or style filenames; use the generated metadata.
- Custom output format, asset paths, externals, or filenames can invalidate the
  manifest, Import Map resolution, dependency identity, and cache behavior.

## Shared runtime dependencies

Halo 2.26 ESM providers can import supported package roots such as Vue, Vue
Router, Pinia, Axios, FormKit, and public Halo UI packages from Halo's shared
runtime. Do not deep-import a shared package or expose an unlisted dependency as
an external merely to reduce bundle size. The exact list is version-sensitive;
read the official UI-build documentation for the target version.

An ESM entry must still default-export its `PluginModule`. Avoid top-level route
registration and other irreversible side effects. Halo isolates provider load
failures, but it cannot roll back timers or listeners created during module
evaluation. A full Console or UC refresh is the supported replacement boundary
after installing, upgrading, enabling, disabling, or reloading a provider.

## Gradle packaging

Production output normally stays in `ui/build/dist`. Copy the complete directory
to the build output, not into source-controlled resources:

```groovy
tasks.register('processUiResources', Copy) {
    from project(':ui').layout.buildDirectory.dir('dist')
    into layout.buildDirectory.dir('resources/main/ui')
    dependsOn project(':ui').tasks.named('assemble')
    shouldRunAfter tasks.named('processResources')
}

tasks.named('classes') {
    dependsOn tasks.named('processUiResources')
}
```

For development builds targeting Halo 2.25 or newer, the preset uses
`build/resources/main/ui`; older targets may continue using
`build/resources/main/console`. `console` is a compatibility path, not the
recommended path for new projects.

## Verification

After changing the build configuration:

1. Run the repository-pinned package manager and the UI build.
2. Inspect the complete emitted directory and `ui-plugin.json` when present.
3. Build the plugin JAR and confirm the same complete directory is packaged
   under `ui/`.
4. Load Console and UC in the target Halo version. Exercise at least one async
   route or asset when the provider emits chunks.
