---
name: halo-theme-dev
description: >
  Complete guide for Halo CMS theme development. Covers theme directory structure,
  Thymeleaf template syntax, template route mapping, template variables, Finder API,
  global variables, theme settings, static asset management, Vite integration, and
  custom tags. Use when: creating or modifying Halo themes, writing Thymeleaf templates,
  calling Finder APIs, configuring theme.yaml / settings.yaml, integrating Vite,
  adding settings forms, handling static asset references, or implementing comment/footer
  extension points.
---

# Halo Theme Development

Halo is built on **Spring Boot + Spring WebFlux + Thymeleaf**. Themes use Thymeleaf templates for frontend page rendering.

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

<!-- Layout reuse: layout.html declares a parameterized fragment; pages pass head/content via th:replace -->
<!-- templates/layout.html -->
<html th:fragment="html (head, content)">
  <head>
    <th:block th:if="${head != null}" th:replace="${head}" />
  </head>
  <body>
    <th:block th:replace="${content}" />
  </body>
</html>
<!-- templates/index.html -->
<html th:replace="~{layout :: html(head = null, content = ~{::content})}">
  <th:block th:fragment="content"><!-- page body --></th:block>
</html>

<!-- Inline JavaScript -->
<script th:inline="javascript">
  var url = '[(${#theme.assets("/dist/main.iife.js")})]';
</script>
```

## Thymeleaf Best Practices

**1. Prefer literal substitutions over string concatenation**

```html
<!-- ✅ readable, no quoting issues -->
<title th:text="|${post.spec.title} - ${site.title}|"></title>

<!-- ❌ verbose and error-prone -->
<title th:text="${post.spec.title} + ' - ' + ${site.title}"></title>
```

**2. Use safe navigation `?.` to avoid NullPointerException**

```html
<!-- ✅ returns null instead of throwing if target is null -->
<a th:target="${item.spec.target?.value}"></a>

<!-- ❌ throws if target is null -->
<a th:target="${item.spec.target.value}"></a>
```

**3. Use Elvis operator `?:` for default values**

```html
<!-- ✅ falls back to site.title when custom_footer is empty/null -->
<p th:text="${theme.config.basic.custom_footer ?: site.title}"></p>
```

**4. Use `th:block` to group without adding extra DOM elements**

```html
<!-- ✅ th:block renders no element itself -->
<th:block th:each="archive : ${archives.items}">
  <h2 th:text="${archive.year}"></h2>
  <ul>...</ul>
</th:block>
```

**5. Use `th:classappend` / `th:attrappend` for conditional classes**

```html
<!-- ✅ appends "active" without overwriting existing classes -->
<a th:classappend="${item.active} ? 'active'">...</a>

<!-- ❌ replaces all classes -->
<a th:class="${item.active} ? 'nav-link active' : 'nav-link'">...</a>
```

**6. Use `#lists.isEmpty()` and `#strings.isEmpty()` for null-safe checks**

```html
<div th:if="${not #lists.isEmpty(post.tags)}">
  <a th:each="tag : ${post.tags}" th:text="${tag.spec.displayName}"></a>
</div>

<meta th:if="${not #strings.isEmpty(site.seo.description)}"
      name="description" th:content="${site.seo.description}" />
```

**7. Do not manually add meta tags — Halo injects them automatically**

Only the `<title>` tag needs to be written by the theme. Halo automatically injects the following into `<head>` at runtime — do not duplicate them:

- `<meta name="description">` and `<meta name="keywords">`
- Open Graph tags (`og:title`, `og:description`, `og:image`, etc.)
- Twitter Card tags
- Canonical URL

```html
<!-- ✅ only manage <title> in your layout/templates -->
<head>
  <title th:text="${site.title}">Site Title</title>
  <!-- Halo handles all other meta/SEO tags automatically -->
</head>

<!-- ❌ redundant and may conflict with Halo's injected tags -->
<head>
  <title th:text="${site.title}">Site Title</title>
  <meta name="description" th:content="${site.seo.description}" />
  <meta property="og:title" th:content="${site.title}" />
</head>
```

**8. Use `@{${url}}` for dynamic permalink URLs**

```html
<!-- ✅ correct: wraps the runtime URL in Thymeleaf's URL context -->
<a th:href="@{${post.status.permalink}}">...</a>

<!-- ❌ wrong: bypasses URL processing -->
<a th:href="${post.status.permalink}">...</a>
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

- **`assets/theme-minimal/`** — Zero-build-tool minimal theme with all 8 template files; ideal for quick prototyping or simple themes
- **`assets/theme-vite/`** — Vite project template with `vite-plugin-halo-theme` (**recommended for new themes**); includes partial layout reuse and CSS toolchain

Usage: copy the directory into `themes/` in your Halo working directory, ensure the folder name matches `metadata.name` in `theme.yaml`, then install and activate in Console.

## References Index

| File | Content | When to read |
| ---- | ------- | ------------ |
| [references/structure-and-config.md](references/structure-and-config.md) | Directory structure, theme.yaml fields, settings.yaml form definition | Creating a theme, configuring theme.yaml/settings.yaml |
| [references/vite-plugin.md](references/vite-plugin.md) | vite-plugin-halo-theme integration guide, include/slot template syntax, TailwindCSS integration | Setting up a Vite-based theme (recommended) |
| [references/templates.md](references/templates.md) | Template route mapping, available variables per template | Writing template files |
| [references/global-variables.md](references/global-variables.md) | Global variables (site, theme, theme.config) and type definitions | Accessing site info or theme setting values |
| [references/finder-apis.md](references/finder-apis.md) | All Finder APIs (postFinder, categoryFinder, tagFinder, menuFinder, singlePageFinder, etc.) | Querying data from any template |
| [references/static-resources.md](references/static-resources.md) | Static asset reference methods (`@{}`, `#theme.assets()`) | Referencing CSS/JS/images in plain HTML themes |
| [references/template-tags.md](references/template-tags.md) | Custom tags (halo:comment extension point, halo:footer injection) | Integrating comment plugins, injecting footer code |
| [references/annotations.md](references/annotations.md) | AnnotationSetting for model custom fields, `#annotations` utility for reading metadata in templates | Adding custom fields to menu items/posts/categories and using them in templates |
| [references/packaging.md](references/packaging.md) | Packaging a theme as a ZIP using `@halo-dev/theme-package-cli` | Preparing a theme for release or upload |
