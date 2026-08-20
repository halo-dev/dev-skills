# Page-layout contract (Halo 2.26+)

Use this contract when plugin-rendered frontend pages should reuse the active
theme's header, footer, styles, and responsive shell.

Official documentation:

https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/theme/page-layout.md

## Theme contract

The contract must be located at `templates/layout.html` and declare exactly the
`html(head, content)` fragment:

```html
<!doctype html>
<html
  xmlns:th="https://www.thymeleaf.org"
  th:lang="${#locale.toLanguageTag}"
  th:fragment="html (head, content)"
>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title th:text="${site.title}"></title>
    <th:block th:if="${head != null}">
      <th:block th:replace="${head}" />
    </th:block>
  </head>
  <body>
    <main><th:block th:replace="${content}" /></main>
    <halo:footer />
  </body>
</html>
```

- `head` is supplied by the plugin for title, metadata, or page resources.
- `content` is the plugin page body.
- Keep `<halo:footer />` so Halo and plugins can inject required footer content.
- Do not require callers to provide additional fragments or private theme
  variables for essential rendering.

An existing root `layout.html` can serve both theme pages and this contract when
its fragment signature matches. Move an internal-only layout to another path,
such as `templates/modules/layout.html`, so Halo does not treat it as the
integration contract.

## Status and fallback

After theme installation, update, or reload, `Theme.status.pageLayout` reports:

- `SUPPORTED`: the contract is valid.
- `MISSING`: no root layout contract exists.
- `INVALID`: the root file exists but its signature is incompatible.

Missing or invalid contracts do not fail theme activation. Plugin pages that use
the contract fall back to Halo's built-in layout.

## Vite themes

`vite-plugin-halo-theme` build-time `<include>` and `<slot>` partials are not the
same mechanism as the runtime Thymeleaf contract. Keep a root `src/layout.html`
entry that builds to `templates/layout.html`; internal source pages may continue
using `src/partials/layout.html` for build-time composition.
