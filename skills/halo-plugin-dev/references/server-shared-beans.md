# Shared Beans (Dependency Injection)

Halo exposes several core beans that any plugin can inject via constructor injection.

> Source references (Halo main branch):
>
> - [ReactiveExtensionClient](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/ReactiveExtensionClient.java)
> - [ExtensionClient](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/ExtensionClient.java)
> - [SchemeManager](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/SchemeManager.java)
> - [ExtensionGetter](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/plugin/extensionpoint/ExtensionGetter.java)
> - [UserService](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/core/user/service/UserService.java)
> - [RoleService](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/core/user/service/RoleService.java)
> - [AttachmentService](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/core/extension/service/AttachmentService.java)
> - [PostContentService](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/content/PostContentService.java)
> - [ExternalLinkProcessor](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/infra/ExternalLinkProcessor.java)
> - [ExternalUrlSupplier](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/infra/ExternalUrlSupplier.java)
> - [NotificationReasonEmitter](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/notification/NotificationReasonEmitter.java)
> - [NotificationCenter](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/notification/NotificationCenter.java)
> - [CryptoService](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/security/authentication/CryptoService.java)
> - [RateLimiterRegistry](https://github.com/resilience4j/resilience4j/blob/master/resilience4j-ratelimiter/src/main/java/io/github/resilience4j/ratelimiter/RateLimiterRegistry.java)
> - [ReactiveSettingFetcher](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/plugin/ReactiveSettingFetcher.java)
> - [SettingFetcher](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/plugin/SettingFetcher.java)
> - [PluginConfigUpdatedEvent](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/plugin/PluginConfigUpdatedEvent.java)

## ReactiveExtensionClient

Reactive CRUD for custom extensions.

```java
private final ReactiveExtensionClient client;

// List with options
client.listBy(Person.class, query.toListOptions(), query.toPageRequest());

// Get by name
client.fetch(Person.class, "person-name");

// Create
client.create(person);

// Update
client.update(person);

// Delete
client.delete(person);
```

## ExtensionClient

Blocking version of `ReactiveExtensionClient`. Use only in non-NIO threads (e.g., background tasks).

## SchemeManager

Register/unregister custom extension types.

```java
schemeManager.register(Person.class);
schemeManager.register(Person.class, indexSpecs -> { /* ... */ });
schemeManager.unregister(Scheme.buildFromType(Person.class));
```

## ExtensionGetter

Retrieve implementations of an extension point.

```java
private final ExtensionGetter extensionGetter;

// Get all implementations
extensionGetter.getExtensions(AttachmentHandler.class);
```

## UserService

User operations: get info, update password, create users.

## ReactiveUserDetailsService

```java
Mono<UserDetails> findByUsername(String username);
```

## RoleService

Role operations: query roles, bindings, dependencies.

## AttachmentService

Upload, delete, get access URLs for attachments.

## PostContentService

Get post content with version management.

```java
postContentService.getHeadContent(postName);      // latest draft
postContentService.getReleaseContent(postName);   // published version
postContentService.getSpecifiedContent(snapshotName);
postContentService.listSnapshots(postName);
```

## ExternalLinkProcessor

Convert relative URLs to absolute using configured `externalUrl`.

```java
externalLinkProcessor.processLink("/post/1");
// -> "https://example.com/post/1"
```

## ExternalUrlSupplier

Get the configured external URL.

## NotificationReasonEmitter / NotificationCenter

Send notifications and manage subscriptions.

## ServerSecurityContextRepository

Access authentication context. Required if your filter runs before Spring Security's `ReactorContextWebFilter`.

## CryptoService

Decrypt login passwords or reuse the public key.

```java
cryptoService.readPublicKey();
cryptoService.decrypt(encryptedPassword);
```

## RateLimiterRegistry

Create rate limiters (remember to clean up in `stop()`).

```java
var rateLimiter = rateLimiterRegistry.rateLimiter(key,
    new RateLimiterConfig.Builder()
        .limitForPeriod(1)
        .limitRefreshPeriod(Duration.ofSeconds(60))
        .build());
```

## SystemInfoGetter (Halo 2.20.11+)

```java
Mono<SystemInfo> info = systemInfoGetter.get();
// Contains: title, subtitle, logo, favicon, url, version, seo, locale, timeZone, activatedThemeName
```

## ReactiveSettingFetcher / SettingFetcher

Fetch plugin configuration defined in `plugin.yaml` (`settingName` / `configMapName`).
The fetcher caches values internally and auto-refreshes when configuration changes.

**Reactive (WebFlux):**

```java
private final ReactiveSettingFetcher settingFetcher;

// Fetch a typed config object by group name
settingFetcher.fetch("seo", SeoSetting.class)
    .doOnNext(seo -> { ... });

// Get raw JsonNode by group
settingFetcher.getSettingValue("seo");

// Get all groups as a map
settingFetcher.getSettingValues();
```

**Blocking (background tasks / non-reactive code):**

```java
private final SettingFetcher settingFetcher;

Optional<SeoSetting> seo = settingFetcher.fetch("seo", SeoSetting.class);
JsonNode raw = settingFetcher.getSettingValue("seo");
Map<String, JsonNode> all = settingFetcher.getSettingValues();
```

**Listen for config changes:**

```java
@EventListener
public void onConfigUpdated(PluginConfigUpdatedEvent event) {
    if (event.getNewConfig().containsKey("seo")) {
        // re-apply configuration
    }
}
```

> **Prerequisite**: In `plugin.yaml`, declare `settingName: my-settings` and `configMapName: my-configmap`, and create a corresponding `settings.yaml` extension resource defining the form schema and groups.

## BackupRootGetter / PluginsRootGetter

`Supplier<Path>` for backup directory and plugin directory.

## LoginHandlerEnhancer

Hook into login success/failure for custom logic.
