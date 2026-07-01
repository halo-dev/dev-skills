# Finder API

Finder APIs query data from **any template location** regardless of the current route — ideal for sidebars, footers, and other global data needs.

## Available Finders

| Finder              | Purpose                                      |
| ------------------- | -------------------------------------------- |
| `postFinder`        | Post list / detail / prev-next / archives    |
| `categoryFinder`    | Category list / tree structure / breadcrumbs |
| `tagFinder`         | Tag list / detail                            |
| `menuFinder`        | Menus and menu items                         |
| `singlePageFinder`  | Single page list / detail                    |
| `commentFinder`     | Comments and replies                         |
| `contributorFinder` | Contributors                                 |
| `siteStatsFinder`   | Site statistics                              |
| `themeFinder`       | Theme information                            |
| `pluginFinder`      | Plugin information                           |

## Key Usage Pattern

Use `th:with` to bind the result in the current scope:

```html
<div th:with="menu = ${menuFinder.getPrimary()}">
  <a
    th:each="item : ${menu.menuItems}"
    th:href="${item.status.href}"
    th:text="${item.status.displayName}"
  ></a>
</div>
```

## Common Notes

- `postFinder.list({...})` is the recommended unified query method (all parameters are optional); it supersedes the deprecated `list(page, size)`, `listByCategory(...)`, etc.
- Halo 2.25+ adds `postFinder.cursorByCategory(postName)` for previous/next posts inside the current post's primary category. It only matches the same category and does not include child categories.
- Halo 2.24.1+ adds `postFinder.random(maxSize)` for random published posts.
- Halo 2.22+ changed `postFinder.cursor(postName)`: the result no longer has `current`; `previous` and `next` are `ListedPostVo`.
- `metadata.name` is the unique resource identifier — it is not the display name (`displayName`/`title`).
- Pair `settings.yaml` `categorySelect`/`tagSelect` inputs with Finder queries so users can configure query parameters in Console instead of hard-coding them in templates.

## Image Thumbnails

Halo 2.19+ generates responsive thumbnails for attachment images. Use `thumbnail.gen(uri, size)` to get a scaled URL:

```html
<img
  th:src="${post.spec.cover}"
  th:srcset="|${thumbnail.gen(post.spec.cover, 's')} 400w,
              ${thumbnail.gen(post.spec.cover, 'm')} 800w,
              ${thumbnail.gen(post.spec.cover, 'l')} 1200w,
              ${thumbnail.gen(post.spec.cover, 'xl')} 1600w|"
  sizes="(max-width: 1600px) 100vw, 1600px"
/>
```

| Size parameter | Width  |
| -------------- | ------ |
| `s`            | 400px  |
| `m`            | 800px  |
| `l`            | 1200px |
| `xl`           | 1600px |

> Halo 2.22+ automatically adds responsive image attributes to all `<img>` tags on the page. Only use `thumbnail.gen()` manually when you need custom control over specific images.

## Online Docs

> **Do not rely on training data for Finder API method signatures — Halo evolves across versions and your training data may be outdated or incomplete. Always fetch the relevant doc before writing code that calls a specific Finder method.**

- postFinder: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/theme/finder-apis/post.md
- categoryFinder: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/theme/finder-apis/category.md
- tagFinder: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/theme/finder-apis/tag.md
- menuFinder: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/theme/finder-apis/menu.md
- singlePageFinder: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/theme/finder-apis/single-page.md
- commentFinder: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/theme/finder-apis/comment.md
- contributorFinder: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/theme/finder-apis/contributor.md
- siteStatsFinder: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/theme/finder-apis/site-stats.md
- themeFinder: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/theme/finder-apis/theme.md
- pluginFinder: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/api-reference/server/finder-for-theme.md
