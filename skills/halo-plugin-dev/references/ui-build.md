# UI Build (@halo-dev/ui-plugin-bundler-kit)

Halo plugins use `@halo-dev/ui-plugin-bundler-kit` to build Vue/TypeScript frontend code into `main.js` + `style.css`.

> - [NPM: @halo-dev/ui-plugin-bundler-kit](https://www.npmjs.com/package/@halo-dev/ui-plugin-bundler-kit)
> - [NPM: @halo-dev/ui-shared](https://www.npmjs.com/package/@halo-dev/ui-shared)
> - [NPM: @halo-dev/api-client](https://www.npmjs.com/package/@halo-dev/api-client)

## Build Tool Options

| Feature           | Vite      | Rsbuild (recommended for complex plugins) |
| ----------------- | --------- | ----------------------------------------- |
| Code splitting    | Limited   | Excellent                                 |
| Build performance | Good      | Excellent                                 |
| Vue ecosystem     | Excellent | Good                                      |
| Dev experience    | Excellent | Excellent                                 |
| Dynamic imports   | Limited   | Excellent                                 |

## Vite Setup

```bash
pnpm install @halo-dev/ui-plugin-bundler-kit@2.22.0 vite -D
```

```ts
// vite.config.ts
import { viteConfig } from "@halo-dev/ui-plugin-bundler-kit";

export default viteConfig();
```

```json
// package.json
{
  "type": "module",
  "scripts": {
    "dev": "vite build --watch --mode=development",
    "build": "vite build"
  }
}
```

### With Customizations

```ts
import { viteConfig } from "@halo-dev/ui-plugin-bundler-kit";
import path from "path";

export default viteConfig({
  vite: {
    resolve: {
      alias: {
        "@": path.resolve(__dirname, "src"),
      },
    },
    plugins: [
      // Additional Vite plugins (Vue plugin is pre-configured)
    ],
  },
});
```

## Rsbuild Setup

```bash
pnpm install @halo-dev/ui-plugin-bundler-kit@2.22.0 @rsbuild/core -D
```

```ts
// rsbuild.config.ts
import { rsbuildConfig } from "@halo-dev/ui-plugin-bundler-kit";

export default rsbuildConfig();
```

```json
// package.json
{
  "type": "module",
  "scripts": {
    "dev": "rsbuild build --env-mode development --watch",
    "build": "rsbuild build"
  }
}
```

### With Customizations

```ts
import { rsbuildConfig } from "@halo-dev/ui-plugin-bundler-kit";

export default rsbuildConfig({
  rsbuild: {
    source: {
      alias: {
        "@": "./src",
      },
    },
    plugins: [
      // Additional Rsbuild plugins
    ],
  },
});
```

## Output Directories

| Environment | Output Path                                                                               |
| ----------- | ----------------------------------------------------------------------------------------- |
| Development | `build/resources/main/console/`                                                           |
| Production  | `ui/build/dist/` (temporary, Gradle copies to `src/main/resources/console/` during build) |

## Dynamic Imports (Rsbuild)

```ts
import { definePlugin } from "@halo-dev/ui-shared";
import { defineAsyncComponent } from "vue";
import { VLoading } from "@halo-dev/components";

export default definePlugin({
  routes: [
    {
      parentName: "Root",
      route: {
        path: "heavy-page",
        name: "HeavyPage",
        component: defineAsyncComponent({
          loader: () => import("./views/HeavyPage.vue"),
          loadingComponent: VLoading,
        }),
      },
    },
  ],
});
```

## Gradle Integration

In the root `build.gradle`:

```groovy
tasks.register('processUiResources', Copy) {
    from project(':ui').layout.buildDirectory.dir('dist')
    into layout.buildDirectory.dir('resources/main/console')
    dependsOn project(':ui').tasks.named('assemble')
    shouldRunAfter tasks.named('processResources')
}

tasks.named('classes') {
    dependsOn tasks.named('processUiResources')
}
```

In `ui/build.gradle`:

```groovy
plugins {
    id 'base'
    id "com.github.node-gradle.node" version "7.1.0"
}

tasks.register('buildFrontend', PnpmTask) {
    group = 'build'
    args = ['build']
    dependsOn tasks.named('pnpmInstall')
    inputs.dir(layout.projectDirectory.dir('src'))
    outputs.dir(layout.buildDirectory.dir('dist'))
}

tasks.named('assemble') {
    dependsOn tasks.named('buildFrontend')
}
```
