# Halo Agent Skills

[简体中文](README.zh-CN.md)

A collection of agent skills for [Halo](https://github.com/halo-dev/halo) developers, designed to help AI agents assist with building themes, plugins, and other Halo ecosystem components.

## What are Agent Skills?

Agent skills are structured knowledge packages that give AI agents (like Cursor, Codex, etc.) deep, task-specific context about a domain. Depending on its workflow, a skill can include:

- A `SKILL.md` entry point with quick references and a workflow guide
- `references/` files with focused, concise documentation
- optional `assets/` with ready-to-use starter templates

## Installation

Skills are installed via the [Skills CLI](https://skills.sh/):

```bash
# Install globally (available across all projects)
npx skills add halo-dev/dev-skills@halo-theme-dev -g
npx skills add halo-dev/dev-skills@halo-plugin-dev -g

# Or install into the current project only
npx skills add halo-dev/dev-skills@halo-theme-dev
npx skills add halo-dev/dev-skills@halo-plugin-dev
```

## Skills

### [`halo-theme-dev`](skills/halo-theme-dev/)

Everything needed to build a complete Halo theme from scratch.

**Covers:**

- Theme directory structure and `theme.yaml` / `settings.yaml` configuration
- Thymeleaf layout fragments and template routing
- Template variables and Finder API usage
- Static resource management and Vite integration via [`vite-plugin-halo-theme`](https://github.com/halo-sigs/vite-plugin-halo-theme)
- Halo 2.26+ page-layout integration and optional Console/UC theme UI providers
- FormKit Schema for building theme settings UI
- Model metadata (Annotations) — defining and reading custom fields
- Custom template tags (`halo:comment`, `halo:footer`)

**Starter templates (in `assets/`):**
| Template | Description |
|----------|-------------|
| `theme-minimal/` | Zero-build-tool theme using Thymeleaf layout fragments |
| `theme-vite/` | Full Vite project with `vite-plugin-halo-theme`, partials, and TailwindCSS-ready setup |

### [`halo-plugin-dev`](skills/halo-plugin-dev/)

Everything needed to build a complete Halo plugin from scratch.

**Covers:**

- Plugin directory structure and `plugin.yaml` manifest configuration
- Java backend: custom extensions (`AbstractExtension`, `@GVK`), custom APIs (`CustomEndpoint`, `@Controller`)
- Plugin lifecycle (`BasePlugin` start/stop/delete) with `SchemeManager` registration and cleanup
- RBAC role templates with view/manage roles and anonymous aggregation
- Vue 3 frontend: `definePlugin`, route registration for Console and User Center
- IIFE/ESM UI build and packaging via `@halo-dev/ui-plugin-bundler-kit` (Vite/Rsbuild)
- Theme integration via `@Finder` annotation for Thymeleaf template variables
- DevTools workflow (`haloServer`, `reload`, `watch`) and OpenAPI client generation

## Structure

```
skills/
└── <skill-name>/
    ├── SKILL.md          # Entry point — read this first
    ├── agents/           # Optional UI metadata and invocation policy
    ├── references/       # Optional focused reference files
    └── assets/           # Optional starter templates and output assets
```

## Contributing

To add a new skill, see the [skill creator guide](https://github.com/halo-dev/halo) or follow the structure of an existing skill.

## Related

- [Halo](https://github.com/halo-dev/halo) — the open source CMS
- [Halo Developer Docs](https://docs.halo.run/developer-guide/theme/structure) — official documentation
- [theme-starter](https://github.com/halo-dev/theme-starter) — minimal theme template
- [theme-vite-starter](https://github.com/halo-dev/theme-vite-starter) — Vite-based theme template
- [plugin-starter](https://github.com/halo-dev/plugin-starter) — minimal plugin template
