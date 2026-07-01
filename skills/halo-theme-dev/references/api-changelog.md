# Theme API Changelog

Read the official changelog before using version-sensitive theme APIs or raising
`spec.requires`.

Official docs:

- Theme API changelog: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/theme/api-changelog.md
- Form schema: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/form-schema.md

High-impact changes:

| Halo version | Change                                                                                                                                                                 | Skill reference                                    |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| 2.25.0       | `select` options support `icon` and `description`; remote selects support `requestOption.iconField` and `descriptionField`                                             | [structure-and-config.md](structure-and-config.md) |
| 2.25.0       | Theme root may include `screenshot.png`, `screenshot.jpeg`, `screenshot.jpg`, or `screenshot.webp`; Halo exposes the first readable image as `Theme.status.screenshot` | [structure-and-config.md](structure-and-config.md) |
| 2.25.0       | `#halo.matchVersion(constraint)` supports conditional rendering for newer Halo-only fragments                                                                          | [global-variables.md](global-variables.md)         |
| 2.25.0       | `postFinder.cursorByCategory(postName)` returns previous/next posts in the current post's primary category                                                             | [finder-apis.md](finder-apis.md)                   |
| 2.24.1       | `postFinder.random(maxSize)` returns random published posts                                                                                                            | [finder-apis.md](finder-apis.md)                   |
| 2.23.0       | `iconify` supports optional `sizing` config                                                                                                                            | [structure-and-config.md](structure-and-config.md) |
| 2.22.8       | `toggle` FormKit input added                                                                                                                                           | [structure-and-config.md](structure-and-config.md) |
| 2.22.2       | `switch` FormKit input added                                                                                                                                           | [structure-and-config.md](structure-and-config.md) |
| 2.22.0       | `array` FormKit input added and preferred over `repeater`; `attachment` was expanded and the older picker is `attachmentInput`                                         | [structure-and-config.md](structure-and-config.md) |
| 2.22.0       | `postFinder.cursor(postName)` return shape changed: no `current`; `previous`/`next` are `ListedPostVo`                                                                 | [finder-apis.md](finder-apis.md)                   |

Prefer `#halo.matchVersion()` for small optional fragments that require a newer
Halo version. Raise `spec.requires` when the whole theme depends on the newer
capability.
