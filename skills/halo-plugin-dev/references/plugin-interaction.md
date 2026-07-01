# Plugin Interaction

Use this reference when a plugin depends on another plugin, exposes Java types
for other plugins, shares events, or defines/consumes extension points.

Official docs:

- Dependencies: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/interaction/dependency.md
- Shared events: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/interaction/shared-events.md
- Making a plugin extensible: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/interaction/making-plugin-extensible.md
- ExtensionGetter: https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/api-reference/server/extension-getter.md

## Dependencies

Declare runtime plugin dependencies in `plugin.yaml` under
`spec.pluginDependencies`.

```yaml
spec:
  pluginDependencies:
    required-plugin: ">=1.0.0 & <2.0.0"
    optional-plugin?: "1.*"
```

- Dependency keys are plugin `metadata.name` values.
- Optional dependencies use a `?` suffix and require Halo 2.20.11+.
- Prefer explicit versions or ranges. Avoid broad `*` constraints for production
  plugins.

## API Modules

When other plugins need to compile against your public Java types, put those
types in a separate API module and publish it. Keep implementation code in the
plugin module.

Consumer plugins should normally depend on the provider API module with
`compileOnly`, not package the provider classes into their plugin jar.

```groovy
dependencies {
    compileOnly "run.halo.example:plugin-a-api:1.0.0"
}
```

## Shared Events

Use Spring events for plugin-to-plugin notifications. Mark custom event classes
with `@SharedEvent` when dependent plugins should be able to listen to them.

```java
import org.springframework.context.ApplicationEvent;
import run.halo.app.plugin.SharedEvent;

@SharedEvent
public class CustomSharedEvent extends ApplicationEvent {
    public CustomSharedEvent(Object source) {
        super(source);
    }
}
```

Listen with `@EventListener` or `ApplicationListener`. Built-in shared events
include post publish/update/delete/visibility changes, user login/logout, and
third-party login disconnection events. Check the official docs for the current
event class names before importing.

## Extension Points

To make a plugin extensible:

1. Define an interface that extends `org.pf4j.ExtensionPoint`.
2. Declare an `ExtensionPointDefinition` resource under
   `src/main/resources/extensions/`.
3. Publish the interface in an API module so extension plugins can compile
   against it.
4. Resolve enabled implementations with `ExtensionGetter`.

```yaml
apiVersion: plugin.halo.run/v1alpha1
kind: ExtensionPointDefinition
metadata:
  name: my-plugin-reactive-notifier
spec:
  className: run.halo.example.ReactiveNotifier
  displayName: "Reactive Notifier"
  description: "Extends notification delivery"
  type: MULTI_INSTANCE
```

Use a plugin-prefixed `metadata.name` to avoid collisions. Use
`SINGLE_INSTANCE` only when exactly one enabled implementation makes sense;
otherwise use `MULTI_INSTANCE`.

```java
extensionGetter.getEnabledExtensions(ReactiveNotifier.class)
    .flatMap(notifier -> notifier.notify(context))
    .then();
```
