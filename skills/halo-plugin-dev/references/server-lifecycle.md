# Plugin Lifecycle

The plugin entry class extends `run.halo.app.plugin.BasePlugin` and must be annotated with `@Component`.

> Source references (Halo main branch):
>
> - [BasePlugin](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/plugin/BasePlugin.java)
> - [PluginContext](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/plugin/PluginContext.java)

## Entry Class Template

```java
package com.example.myplugin;

import org.springframework.stereotype.Component;
import run.halo.app.plugin.BasePlugin;
import run.halo.app.plugin.PluginContext;

@Component
public class MyPlugin extends BasePlugin {

    public MyPlugin(PluginContext pluginContext) {
        super(pluginContext);
    }

    @Override
    public void start() {
        // Called after classes are loaded and before the plugin is marked active
        // Register schemes, initialize caches, start watchers, etc.
    }

    @Override
    public void stop() {
        // Called when the plugin is stopped (disabled)
        // Unregister schemes, dispose watchers, clean up resources
    }

    @Override
    public void delete() {
        // Called when the plugin is uninstalled
        // Final cleanup, delete external data if needed
    }
}
```

## Lifecycle Behavior

| Method     | When Called                                    | Typical Actions                                                     |
| ---------- | ---------------------------------------------- | ------------------------------------------------------------------- |
| `start()`  | After plugin classes loaded, before activation | `schemeManager.register()`, init caches, register watchers          |
| `stop()`   | When plugin is disabled                        | `schemeManager.unregister()`, dispose watchers, clear rate limiters |
| `delete()` | When plugin is uninstalled                     | Delete external resources, cleanup                                  |

## Important Rules

1. **Only ONE class** may extend `BasePlugin` and be annotated with `@Component`. Multiple candidates cause startup failure.
2. **Must have `@Component`** (or other Spring stereotype). Without it, lifecycle methods are never invoked.
3. **Use constructor injection** for dependencies (preferred over `@Autowired` fields).

## Scheme Registration / Cleanup

```java
@Component
public class MyPlugin extends BasePlugin {
    private final SchemeManager schemeManager;

    public MyPlugin(PluginContext ctx, SchemeManager schemeManager) {
        super(ctx);
        this.schemeManager = schemeManager;
    }

    @Override
    public void start() {
        schemeManager.register(MyExtension.class, indexSpecs -> {
            indexSpecs.add(IndexSpecs.<MyExtension, String>single("spec.slug", String.class)
                .indexFunc(ext -> ext.getSpec().getSlug()));
        });
    }

    @Override
    public void stop() {
        schemeManager.unregister(Scheme.buildFromType(MyExtension.class));
    }
}
```

## Watchers

Watch extension changes for cache invalidation or reactive workflows:

```java
@Component
public class MyPlugin extends BasePlugin {
    private final ReactiveExtensionClient client;
    private Watcher watcher;

    // ... constructor

    @Override
    public void start() {
        watcher = new Watcher() {
            private volatile boolean disposed = false;

            @Override
            public void onAdd(Extension extension) {
                if (extension instanceof MyExtension) {
                    // handle add
                }
            }

            @Override
            public void onUpdate(Extension oldObj, Extension newObj) {
                if (newObj instanceof MyExtension) {
                    // handle update
                }
            }

            @Override
            public void onDelete(Extension extension) {
                if (extension instanceof MyExtension) {
                    // handle delete
                }
            }

            @Override
            public void dispose() { disposed = true; }

            @Override
            public boolean isDisposed() { return disposed; }
        };
        client.watch(watcher);
    }

    @Override
    public void stop() {
        if (watcher != null) watcher.dispose();
    }
}
```

## RateLimiter Cleanup

If creating rate limiters via `RateLimiterRegistry`, always clean up in `stop()`:

```java
private final Set<String> limiterNames = ConcurrentHashMap.newKeySet();

@Override
public void stop() {
    limiterNames.forEach(rateLimiterRegistry::remove);
}
```
