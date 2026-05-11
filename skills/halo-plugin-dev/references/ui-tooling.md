# UI Tooling

Recommended third-party tools for plugin frontend development that work well alongside Halo's UI framework.

---

## Icons: unplugin-icons + Iconify

[unplugin-icons](https://github.com/unplugin/unplugin-icons) provides on-demand icon imports from [Iconify](https://iconify.design/) with full IDE support (auto-completion, hover preview).

### Installation

```bash
pnpm add -D unplugin-icons @iconify/json
```

> For smaller bundles, install only the icon sets you need: `pnpm add -D @iconify-json/ri @iconify-json/mdi`

### Vite Configuration

```ts
// ui/vite.config.ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import Icons from 'unplugin-icons/vite';

export default defineConfig({
  plugins: [
    vue(),
    Icons({
      compiler: 'vue3',
      autoInstall: true,
    }),
  ],
});
```

### Rsbuild Configuration

```ts
// ui/rsbuild.config.mjs
import { defineConfig } from '@rsbuild/core';
import { pluginVue } from '@rsbuild/plugin-vue';
import Icons from 'unplugin-icons/rspack';

export default defineConfig({
  plugins: [pluginVue()],
  tools: {
    rspack: {
      plugins: [
        Icons({ compiler: 'vue3' }),
      ],
    },
  },
});
```

> Always set `compiler: 'vue3'` so icons are rendered as Vue components.

### Usage in Vue

Import icons directly from the `~icons/` virtual module:

```ts
// ui/src/index.ts
import RiImage2Line from '~icons/ri/image-2-line';
import MdiHome from '~icons/mdi/home';

// Use as a regular Vue component
// Common pattern: pass to markRaw() for menu icons
{
  menu: {
    icon: markRaw(RiImage2Line),
  },
}
```

```vue
<script setup lang="ts">
import RiImage2Line from '~icons/ri/image-2-line';
</script>

<template>
  <RiImage2Line class="w-4 h-4" />
</template>
```

> Browse icons at [icones.js.org](https://icones.js.org/) or [iconify.design](https://icon-sets.iconify.design/).

---

## Atomic CSS: UnoCSS

[UnoCSS](https://unocss.dev/) is an instant atomic CSS engine that provides Tailwind-compatible utilities with zero runtime overhead.

### Installation

```bash
pnpm add -D unocss @unocss/webpack
```

### Vite Configuration

```ts
// ui/vite.config.ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import UnoCSS from 'unocss/vite';

export default defineConfig({
  plugins: [
    vue(),
    UnoCSS(),
  ],
});
```

Add the CSS import in your entry file:

```ts
// ui/src/index.ts
import 'uno.css';
```

### Rsbuild Configuration

```ts
// ui/rsbuild.config.mjs
import { defineConfig } from '@rsbuild/core';
import { pluginVue } from '@rsbuild/plugin-vue';
import { UnoCSSRspackPlugin } from '@unocss/webpack/rspack';

export default defineConfig({
  plugins: [pluginVue()],
  tools: {
    rspack: {
      plugins: [
        UnoCSSRspackPlugin(),
      ],
    },
  },
});
```

Add the CSS import:

```ts
// ui/src/index.ts
import 'uno.css';
```

### Configuration File

Create `ui/uno.config.ts`:

```ts
import { defineConfig, presetWind3, transformerCompileClass } from 'unocss';

export default defineConfig({
  presets: [presetWind3()],
  transformers: [transformerCompileClass()],
});
```

### Usage in Vue

```vue
<template>
  <div class="flex items-center gap-2 px-4 py-2 bg-gray-100 rounded">
    <span class="text-sm font-medium text-gray-700">Hello</span>
  </div>
</template>
```

> Use UnoCSS for layout, spacing, and any custom component styling when `@halo-dev/components` does not provide the needed primitive; use `@halo-dev/components` for standard business UI.
