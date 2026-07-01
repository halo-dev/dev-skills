# Plugin API Changelog

Read the official changelog before using version-sensitive plugin APIs,
upgrading Halo dependencies, or raising `spec.requires`.

Official docs:

- Plugin API changelog: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/api-changelog.md
- Form schema: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/form-schema.md

High-impact changes:

| Halo version | Change                                                                                                                         | Skill reference                                                      |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| 2.25.0       | `select` options support `icon` and `description`; remote selects support `requestOption.iconField` and `descriptionField`     | [ui-forms.md](ui-forms.md)                                           |
| 2.25.0       | Plugin UI can register custom FormKit inputs through `definePlugin({ formkit: { inputs } })`                                   | [ui-entry.md](ui-entry.md), [ui-forms.md](ui-forms.md)               |
| 2.25.0       | `secret` FormKit input supports `descriptionPreset`                                                                            | [ui-forms.md](ui-forms.md)                                           |
| 2.23.0       | Spring Boot 4 upgrade can break plugins using Spring APIs; upgrade the Halo platform dependency and `spec.requires` together   | [devtools.md](devtools.md), [plugin-manifest.md](plugin-manifest.md) |
| 2.23.0       | `iconify` supports optional `sizing` config                                                                                    | [ui-forms.md](ui-forms.md)                                           |
| 2.22.8       | `toggle` FormKit input added                                                                                                   | [ui-forms.md](ui-forms.md)                                           |
| 2.22.5       | SpringDoc update can break OpenAPI documentation generation; upgrade the Halo platform dependency and `spec.requires` together | [server-api.md](server-api.md), [devtools.md](devtools.md)           |
| 2.22.2       | `switch` FormKit input added                                                                                                   | [ui-forms.md](ui-forms.md)                                           |
| 2.22.0       | Custom model index/query APIs changed: use `IndexSpecs.single/multi`, `Queries`, and newer `ExtensionClient` query helpers     | [server-extension.md](server-extension.md)                           |
| 2.22.0       | `@halo-dev/console-shared` was renamed to `@halo-dev/ui-shared`                                                                | [ui-shared.md](ui-shared.md)                                         |
| 2.22.0       | Attachment selector extension results need `mediaType` on `AttachmentLike`                                                     | [ui-extension-points.md](ui-extension-points.md)                     |

When a feature requires a newer Halo runtime, update dependency versions and
`plugin.yaml` `spec.requires` together. Do not raise `spec.requires` for an
optional UI enhancement unless the plugin cannot run without it.
