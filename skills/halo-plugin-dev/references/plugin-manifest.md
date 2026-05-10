# Plugin Manifest (plugin.yaml)

Located at `src/main/resources/plugin.yaml`. Required.

> - [create-halo-plugin repo](https://github.com/halo-dev/create-halo-plugin)

## Minimal Example

```yaml
apiVersion: plugin.halo.run/v1alpha1
kind: Plugin
metadata:
  name: hello-world
spec:
  enabled: true
  requires: ">=2.22.0"
  author:
    name: Halo
    website: https://www.halo.run
  logo: logo.svg
  displayName: "Hello World"
  description: "A minimal Halo plugin"
  license:
    - name: "GPL-3.0"
      url: "https://github.com/example/plugin/blob/main/LICENSE"
```

## Field Reference

| Field                                 | Description                                                                                               | Required    |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------- | ----------- |
| `apiVersion` / `kind`                 | Fixed: `plugin.halo.run/v1alpha1` / `Plugin`                                                              | Yes         |
| `metadata.name`                       | Unique plugin ID. Max 253 chars, lowercase letters/numbers/hyphens only, must start/end with alphanumeric | Yes         |
| `spec.enabled`                        | Auto-enable on install. For production, prefer `false` for security                                       | Yes         |
| `spec.requires`                       | Supported Halo version range. SemVer ranges: `>=2.22.0`, `^2.22.0`, `2.22.*`, etc.                        | Yes         |
| `spec.version`                        | Plugin version (e.g., `1.0.0`)                                                                            | Recommended |
| `spec.author.name` / `author.website` | Author info                                                                                               | Recommended |
| `spec.logo`                           | Logo file (relative to `src/main/resources/`) or URL                                                      | Recommended |
| `spec.displayName`                    | Human-readable name                                                                                       | Yes         |
| `spec.description`                    | Short description                                                                                         | Yes         |
| `spec.homepage`                       | Plugin homepage/docs URL                                                                                  | Recommended |
| `spec.repo`                           | Source repository URL                                                                                     | Recommended |
| `spec.issues`                         | Issue tracker URL                                                                                         | Recommended |
| `spec.license`                        | License name + URL array                                                                                  | Recommended |
| `spec.settingName`                    | Setting resource name for plugin config form. Suffix with `-settings`                                     | Optional    |
| `spec.configMapName`                  | ConfigMap name for persisting config. Suffix with `-configmap`. Required if `settingName` is set          | Conditional |
| `spec.pluginDependencies`             | Map of `pluginName: versionRange`. Optional deps suffix name with `?` (Halo 2.20.11+)                     | Optional    |

## Plugin Dependencies

```yaml
spec:
  pluginDependencies:
    # Required dependency
    some-plugin: ">=1.0.0 & <2.0.0"
    # Optional dependency (Halo 2.20.11+)
    optional-plugin?: "1.*"
```

## Settings & ConfigMap

To provide a user-configurable form in Console:

1. Set `spec.settingName: my-plugin-settings` and `spec.configMapName: my-plugin-configmap` in `plugin.yaml`
2. Create `src/main/resources/extensions/settings.yaml`:

```yaml
apiVersion: v1alpha1
kind: Setting
metadata:
  name: my-plugin-settings # must match spec.settingName
spec:
  forms:
    - group: basic
      label: Basic Settings
      formSchema:
        - $formkit: text
          name: apiKey
          label: API Key
        - $formkit: switch
          name: enabled
          label: Enable Feature
```

> If `settingName` is set but the corresponding `Setting` resource does not exist, the plugin will fail to start.
>
> After setting this up, read the config at runtime using [`ReactiveSettingFetcher` or `SettingFetcher`](server-shared-beans.md#reactivesettingfetcher--settingfetcher).

## App Store Annotations

For distribution via Halo App Store:

```yaml
metadata:
  annotations:
    store.halo.run/app-id: "app-XXXXX"
```
