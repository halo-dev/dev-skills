# Reconciler (Controller Pattern)

A Kubernetes-style controller that watches Extension resources and continuously reconciles their desired state. More commonly used than `Watcher` in production plugins.

> Source: [Reconciler](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/controller/Reconciler.java) | [ControllerBuilder](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/controller/ControllerBuilder.java) | [Controller](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/controller/Controller.java)

## Basic Pattern

```java
@Component
public class MyReconciler implements Reconciler<Reconciler.Request> {

    private final ExtensionClient client;

    public MyReconciler(ExtensionClient client) {
        this.client = client;
    }

    @Override
    public Result reconcile(Request request) {
        // Fetch the extension being reconciled
        var myExt = client.fetch(MyExtension.class, request.name());
        if (myExt.isEmpty()) {
            return Result.doNotRetry();
        }

        // Perform reconciliation logic
        var ext = myExt.get();
        // ... update annotations, labels, related resources, etc.

        // Update the extension if modified
        client.update(ext);

        return Result.doNotRetry();
    }

    @Override
    public Controller setupWith(ControllerBuilder builder) {
        return builder
            .extension(new MyExtension())
            .syncAllOnStart(false)
            .build();
    }
}
```

## ControllerBuilder Options

| Method                             | Description                                                 |
| ---------------------------------- | ----------------------------------------------------------- |
| `.extension(new MyExtension())`    | The extension type to watch (required)                      |
| `.syncAllOnStart(true)`            | Reconcile all existing instances on startup (default: true) |
| `.syncAllListOptions(listOptions)` | Filter which existing instances to sync on start            |
| `.minDelay(Duration)`              | Minimum retry delay (default: 5ms)                          |
| `.maxDelay(Duration)`              | Maximum retry delay (default: 1000s)                        |
| `.workerCount(int)`                | Number of concurrent workers (default: 1)                   |
| `.onAddMatcher(matcher)`           | Filter which add events to process                          |
| `.onUpdateMatcher(matcher)`        | Filter which update events to process                       |
| `.onDeleteMatcher(matcher)`        | Filter which delete events to process                       |

## Result Types

```java
// Success, do not retry
return Result.doNotRetry();

// Requeue after a delay (for async operations or retry)
return Result.requeue(Duration.ofSeconds(30));
```

## Lifecycle Integration

Reconcilers are auto-discovered by Spring. If you need to start/stop manually (e.g., conditional on plugin config):

```java
@Component
public class MyReconciler implements Reconciler<Request>, SmartLifecycle {
    private Controller controller;
    private boolean running = false;

    @Override
    public Controller setupWith(ControllerBuilder builder) {
        this.controller = builder.extension(new MyExtension()).build();
        return controller;
    }

    @Override
    public void start() {
        if (controller != null && !running) {
            controller.start();
            running = true;
        }
    }

    @Override
    public void stop() {
        if (controller != null && running) {
            controller.dispose(); // or controller.stop()
            running = false;
        }
    }

    @Override
    public boolean isRunning() { return running; }
}
```

## Finalizers

Use finalizers for cleanup before an extension is deleted:

```java
@Override
public Result reconcile(Request request) {
    var ext = client.fetch(MyExtension.class, request.name()).orElse(null);
    if (ext == null) return Result.doNotRetry();

    // Check if being deleted
    if (ext.getMetadata().getDeletionTimestamp() != null) {
        // Perform cleanup
        doCleanup(ext);
        // Remove finalizer to allow deletion
        ExtensionUtil.removeFinalizers(ext.getMetadata(), Set.of("my-plugin/finalizer"));
        client.update(ext);
        return Result.doNotRetry();
    }

    // Add finalizer if not present
    ExtensionUtil.addFinalizers(ext.getMetadata(), Set.of("my-plugin/finalizer"));
    client.update(ext);

    // Normal reconciliation
    return Result.doNotRetry();
}
```

## When to Use Reconciler vs Watcher

|                 | Reconciler                                           | Watcher                 |
| --------------- | ---------------------------------------------------- | ----------------------- |
| **Use case**    | Continuous state reconciliation, finalizers, retries | One-time event handling |
| **Persistence** | Queued, survives restarts                            | In-memory only          |
| **Concurrency** | Configurable worker count                            | Single threaded         |
| **Retry**       | Built-in exponential backoff                         | No retry                |
