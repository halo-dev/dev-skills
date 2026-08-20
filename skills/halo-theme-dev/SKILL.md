---
name: halo-theme-dev
description: >
  Create or modify Halo CMS themes, including Thymeleaf templates, theme.yaml and
  settings.yaml, Finder APIs, static assets, Vite integration, theme UI providers,
  page-layout integration, annotations, i18n, and error pages. Use for Halo site
  theme implementation, migration, debugging, or packaging; do not use for
  unrelated Thymeleaf applications or plugin-only changes.
---

# Halo Theme Development

Halo is built on **Spring Boot + Spring WebFlux + Thymeleaf**. Themes use Thymeleaf templates for frontend page rendering.

> **Version first**: inspect `theme.yaml` `spec.requires` and the target Halo version before choosing an API. Use the references below for Halo-specific invariants and workflow guidance. For version-sensitive template variables and Finder signatures, follow the official documentation routes linked from the relevant reference instead of guessing or silently raising `spec.requires`.

## Thymeleaf Quick Reference

Full docs: https://raw.githubusercontent.com/thymeleaf/thymeleaf-docs/refs/heads/master/docs/tutorials/3.1/usingthymeleaf.md

Core syntax cheatsheet:

```html
<!-- Output text -->
<h1 th:text="${site.title}"></h1>

<!-- Output unescaped HTML -->
<div th:utext="${post.content.content}"></div>

<!-- Links -->
<a th:href="@{${post.status.permalink}}">Post link</a>
<link rel="stylesheet" th:href="@{/assets/dist/style.css}" />

<!-- Loop -->
<li th:each="post : ${posts.items}" th:text="${post.spec.title}"></li>

<!-- Conditionals -->
<div th:if="${posts.hasNext()}">Next page</div>
<div th:unless="${posts.hasNext()}">Last page</div>

<!-- Local variable -->
<div th:with="menu = ${menuFinder.getPrimary()}">...</div>

<!-- Fragment include -->
<div th:replace="~{fragments/header :: header}"></div>

<!-- Layout reuse: pages pass fragments into a parameterized layout -->
<html th:replace="~{layout :: html(head = null, content = ~{::content})}">
  <th:block th:fragment="content"><!-- page body --></th:block>
</html>

<!-- Inline JavaScript -->
<script th:inline="javascript">
  var url = '[(${#theme.assets("/dist/main.iife.js")})]';
</script>
```

## Development Workflow

1. Create a theme folder under `themes/` in the Halo working directory (must match `metadata.name` in `theme.yaml`)
2. Write `theme.yaml` (required) and `settings.yaml` (optional)
3. Create template files under `templates/`
4. Install and activate the theme in Console → Theme Management
5. Visit the frontend to verify

Disable Thymeleaf caching during development: set env var `SPRING_THYMELEAF_CACHE=false` (Docker), or `spring.thymeleaf.cache: false` in config (source mode).

## Starter Templates

The `assets/` directory provides two ready-to-use theme templates:

- **`assets/theme-minimal/`** — Zero-build-tool minimal theme with the standard route templates; ideal for quick prototyping or simple themes
- **`assets/theme-vite/`** — Vite project template with `vite-plugin-halo-theme` (**recommended for new themes**); includes partial layout reuse and CSS toolchain

Usage: copy the directory into `themes/` in your Halo working directory, ensure the folder name matches `metadata.name` in `theme.yaml`, then install and activate in Console.

## References Index

| File                                                                     | Content                                                                                                          | When to read                                                                    |
| ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| [references/api-changelog.md](references/api-changelog.md)               | High-impact theme API changes by Halo version, with docs routes                                                  | Before using version-sensitive APIs or raising `spec.requires`                  |
| [references/structure-and-config.md](references/structure-and-config.md) | Directory structure, theme.yaml fields, root screenshot, settings.yaml form definition                           | Creating a theme, configuring theme.yaml/settings.yaml                          |
| [references/vite-plugin.md](references/vite-plugin.md)                   | vite-plugin-halo-theme integration guide, include/slot template syntax, TailwindCSS integration                  | Setting up a Vite-based theme (recommended)                                     |
| [references/page-layout.md](references/page-layout.md)                   | Halo 2.26+ page-layout contract for plugin-rendered frontend pages                                               | Providing or changing `templates/layout.html`                                   |
| [references/ui-plugin.md](references/ui-plugin.md)                       | Halo 2.26+ theme UI provider for Console and User Center                                                         | Extending Console or UC from an activated theme                                 |
| [references/templates.md](references/templates.md)                       | Template route mapping, available variables per template                                                         | Writing template files                                                          |
| [references/global-variables.md](references/global-variables.md)         | Global variables (site, theme, theme.config) and type definitions                                                | Accessing site info or theme setting values                                     |
| [references/finder-apis.md](references/finder-apis.md)                   | All Finder APIs (postFinder, categoryFinder, tagFinder, menuFinder, singlePageFinder, etc.)                      | Querying data from any template                                                 |
| [references/static-resources.md](references/static-resources.md)         | Static asset reference methods (`@{}`, `#theme.assets()`)                                                        | Referencing CSS/JS/images in plain HTML themes                                  |
| [references/template-tags.md](references/template-tags.md)               | Custom tags (halo:comment extension point, halo:footer injection)                                                | Integrating comment plugins, injecting footer code                              |
| [references/i18n.md](references/i18n.md)                                 | Internationalization via `.properties` files, `#messages`, `#locale`, frontend i18n injection                    | Adding multi-language support to a theme                                        |
| [references/official-plugins.md](references/official-plugins.md)         | Official plugin integration: pluginFinder.available(), search widget, dark mode color scheme adaptation          | Adding search, adapting dark mode for plugin UI                                 |
| [references/annotations.md](references/annotations.md)                   | AnnotationSetting for model custom fields, `#annotations` utility for reading metadata in templates              | Adding custom fields to menu items/posts/categories and using them in templates |
| [references/packaging.md](references/packaging.md)                       | Packaging a theme as a ZIP using `@halo-dev/theme-package-cli`                                                   | Preparing a theme for release or upload                                         |
| [references/thymeleaf-tips.md](references/thymeleaf-tips.md)             | Halo-specific Thymeleaf best practices: literal substitutions, safe navigation, meta tag rules, permalink syntax | Writing any template file                                                       |
