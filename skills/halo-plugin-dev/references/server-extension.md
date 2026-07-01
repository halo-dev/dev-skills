# Custom Extension (Data Model)

Halo uses a Kubernetes CRD-like system called **Extension** for custom data storage.

> Source references (Halo main branch):
>
> - [AbstractExtension](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/AbstractExtension.java)
> - [GVK](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/GVK.java)
> - [SchemeManager](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/SchemeManager.java)
> - [IndexSpecs](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/index/IndexSpecs.java)
> - [ReactiveExtensionClient](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/ReactiveExtensionClient.java)
> - [GroupVersion](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/GroupVersion.java)
> - [GroupVersionKind](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/GroupVersionKind.java)
> - [Queries](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/index/query/Queries.java)
> - [FieldSelector](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/router/selector/FieldSelector.java)
> - [ExtensionUtil](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/ExtensionUtil.java)
> - [MetadataUtil](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/MetadataUtil.java)
> - [ExtensionOperator](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/ExtensionOperator.java)

## Docs Routing

This file captures the common model pattern. Verify exact APIs, query helpers,
and reconciler contracts in the official docs before depending on a recent
method or version-specific behavior.

| Need                                      | Official docs                                                                                                                        |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Extension model, indexes, query params    | https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/api-reference/server/extension.md        |
| ExtensionClient / ReactiveExtensionClient | https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/api-reference/server/extension-client.md |
| Object management basics                  | https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/basics/server/object-management.md       |
| Reconciler controllers                    | https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/api-reference/server/reconciler.md       |
| Plugin API changelog for version gates    | https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/api-changelog.md                         |

When working from a local docs checkout, use the same paths under
`docs/developer-guide/...`; versioned docs live under
`versioned_docs/version-2.25/...`.

## Quick Index

- Basic model shape: [Creating an Extension](#creating-an-extension)
- Registration: [Registering in Lifecycle](#registering-in-lifecycle)
- Generated endpoints: [Auto-Generated CRUD APIs](#auto-generated-crud-apis)
- Indexes and queries: [Indexes](#indexes)
- Query APIs: [Querying Extensions](#querying-extensions)

## Creating an Extension

Three steps:

1. Create a class extending `run.halo.app.extension.AbstractExtension`
2. Annotate with `@GVK(group, version, kind, plural, singular)`
3. Register in plugin `start()` via `SchemeManager`

## Example

```java
import io.swagger.v3.oas.annotations.media.Schema;
import java.time.Instant;
import java.util.List;
import lombok.Data;
import lombok.EqualsAndHashCode;
import run.halo.app.extension.AbstractExtension;
import run.halo.app.extension.GVK;

@Data
@EqualsAndHashCode(callSuper = true)
@GVK(
    group = "my-plugin.halo.run",
    version = "v1alpha1",
    kind = "Person",
    plural = "persons",
    singular = "person"
)
public class Person extends AbstractExtension {

    @Schema(requiredMode = Schema.RequiredMode.REQUIRED)
    private Spec spec;

    @Data
    @Schema(name = "PersonSpec")
    public static class Spec {
        @Schema(description = "Name", maxLength = 100)
        private String name;

        @Schema(description = "Slug", maxLength = 100)
        private String slug;

        @Schema(description = "Age", maximum = "150", minimum = "0")
        private Integer age;

        @Schema(description = "Tags")
        private List<String> tags;

        @Schema(description = "Priority")
        private Integer priority;

        @Schema(description = "Pinned")
        private Boolean pinned;

        @Schema(description = "Publish time")
        private Instant publishTime;
    }
}
```

## Registering in Lifecycle

```java
@Component
public class MyPlugin extends BasePlugin {
    @Autowired
    private SchemeManager schemeManager;

    @Override
    public void start() {
        schemeManager.register(Person.class);
    }

    @Override
    public void stop() {
        schemeManager.unregister(Scheme.buildFromType(Person.class));
    }
}
```

## GVK Annotation Fields

| Field      | Description                                           |
| ---------- | ----------------------------------------------------- |
| `group`    | Domain-style group, e.g., `my-plugin.halo.run`        |
| `version`  | API version, e.g., `v1alpha1`                         |
| `kind`     | Resource type name (PascalCase)                       |
| `plural`   | REST plural path segment, lowercase (e.g., `persons`) |
| `singular` | Singular name, lowercase (e.g., `person`)             |

## Auto-Generated CRUD APIs

After registration, Halo automatically exposes:

```
GET    /apis/{group}/{version}/{plural}           # List
GET    /apis/{group}/{version}/{plural}/{name}    # Get by name
POST   /apis/{group}/{version}/{plural}           # Create
PUT    /apis/{group}/{version}/{plural}/{name}    # Update
DELETE /apis/{group}/{version}/{plural}/{name}    # Delete
```

List endpoint supports:

| Param           | Description                                                           |
| --------------- | --------------------------------------------------------------------- |
| `page`          | Page number (1-based)                                                 |
| `size`          | Page size                                                             |
| `sort`          | `field,asc\|desc`. Must be an indexed field                           |
| `labelSelector` | Label filtering: `key=value`, `key!=value`, `!key`, `key`             |
| `fieldSelector` | Indexed field filtering: `field=value`, `field!=value`, `field=(a,b)` |

Example:

```
GET /apis/my-plugin.halo.run/v1alpha1/persons?page=1&size=10&sort=metadata.name,desc&fieldSelector=spec.age=18
```

## Declaring Extension Objects (YAML)

Place YAML files in `src/main/resources/extensions/`. They are created/updated on plugin startup.

```yaml
apiVersion: my-plugin.halo.run/v1alpha1
kind: Person
metadata:
  name: default-person
spec:
  name: halo
  slug: halo
  age: 18
```

> ⚠️ Resources here are overwritten on every plugin start. Do NOT place user-modifiable config here.

## Validation with @Schema

```java
@Schema(description = "Email", format = "email")
private String email;

@Schema(requiredMode = Schema.RequiredMode.REQUIRED, minLength = 1, maxLength = 50)
private String title;
```

Validation is applied on create/update automatically.

## Indexes

Indexes improve query performance for `fieldSelector` and `sort`.

```java
import java.time.Instant;
import java.util.Set;
import run.halo.app.extension.index.IndexSpecs;

@Override
public void start() {
    schemeManager.register(Person.class, indexSpecs -> {
        // Single-value index, can return null unless nullable(false) is configured
        indexSpecs.add(IndexSpecs.<Person, String>single("spec.name", String.class)
            .indexFunc(person -> person.getSpec().getName()));

        // Multi-value index, returns a set of values
        indexSpecs.add(IndexSpecs.<Person, String>multi("spec.tags", String.class)
            .indexFunc(person -> {
                var tags = person.getSpec().getTags();
                return tags == null ? Set.of() : Set.copyOf(tags);
            }));

        // Index keys are not limited to String. Use Comparable types.
        indexSpecs.add(IndexSpecs.<Person, Boolean>single("spec.pinned", Boolean.class)
            .indexFunc(person -> person.getSpec().getPinned()));
        indexSpecs.add(IndexSpecs.<Person, Integer>single("spec.priority", Integer.class)
            .indexFunc(person -> person.getSpec().getPriority()));
        indexSpecs.add(IndexSpecs.<Person, Instant>single("spec.publishTime", Instant.class)
            .indexFunc(person -> person.getSpec().getPublishTime()));

        // Optional builder flags from Halo 2.22+: unique and nullable.
        indexSpecs.add(IndexSpecs.<Person, String>single("spec.slug", String.class)
            .unique(true)
            .nullable(false)
            .indexFunc(person -> person.getSpec().getSlug()));
    });
}
```

An index spec declares an index item. Prefer building it with
`IndexSpecs.single(name, keyType)` or `IndexSpecs.multi(name, keyType)`.
Key details:

| Property    | Description                                                                                         |
| ----------- | --------------------------------------------------------------------------------------------------- |
| `name`      | Unique index name for this extension type, usually a field path                                     |
| `keyType`   | Index key type. Must implement `Comparable`, e.g. `String`, `Boolean`, `Integer`, `Long`, `Instant` |
| `indexFunc` | Function that extracts the indexed value from the extension                                         |
| `unique`    | Optional. Enforces unique index values when set to `true`                                           |
| `nullable`  | Optional. Allows null index values by default; set `false` for required keys                        |

Since Halo 2.22.0, `IndexAttributeFactory.simpleAttribute()`,
`IndexAttributeFactory.multiValueAttribute()`, and direct `new IndexSpec()`
construction are deprecated. Use `IndexSpecs.single()` and
`IndexSpecs.multi()` instead.

Built-in indexes (do not re-declare):

- `metadata.name` (unique)
- `metadata.labels`
- `metadata.creationTimestamp`
- `metadata.deletionTimestamp`

## Metadata Structure

Every extension has `metadata`:

| Field                        | Description                                                |
| ---------------------------- | ---------------------------------------------------------- |
| `metadata.name`              | Unique ID, max 253 chars, lowercase alphanumeric + hyphens |
| `metadata.creationTimestamp` | Auto-set on create, immutable                              |
| `metadata.version`           | Optimistic locking version. Mismatch on update = conflict  |
| `metadata.deletionTimestamp` | Set when marked for deletion (before actual removal)       |
| `metadata.finalizers`        | Cleanup hooks. Extension not deleted until empty           |
| `metadata.labels`            | String key-value map. Auto-indexed. Use for querying       |
| `metadata.annotations`       | String key-value map. NOT indexed. Use for extra metadata  |

## GroupVersion & GroupVersionKind

Programmatically construct API identifiers:

```java
// From strings
var gv = new GroupVersion("my-plugin.halo.run", "v1alpha1");
var gvk = GroupVersionKind.fromAPIVersionAndKind("my-plugin.halo.run/v1alpha1", "Person");

// From a @GVK-annotated class
var gvk = GroupVersionKind.fromExtension(Person.class);

// Parse from API version string
var gv = GroupVersion.parseAPIVersion("my-plugin.halo.run/v1alpha1");
```

## Querying Extensions

Prefer the newer `ReactiveExtensionClient` query methods:

```java
Flux<Person> people = client.listAll(Person.class, options, sort);
Mono<ListResult<Person>> page = client.listBy(Person.class, options, pageable);
```

Use these methods instead of the deprecated `list(Class, Predicate, Comparator, ...)`
overloads. Common query methods include:

| Method         | Description                 |
| -------------- | --------------------------- |
| `listBy`       | Page through matching data  |
| `listNamesBy`  | Page through matching names |
| `listAll`      | Return all matching data    |
| `listAllNames` | Return all matching names   |
| `listTopNames` | Return top matching names   |
| `countBy`      | Count matching data         |

`ListOptions` carries label and field conditions:

```java
import static run.halo.app.extension.index.query.Queries.equal;

ListOptions options = ListOptions.builder()
    .labelSelector()
    .eq("env", "production")
    .end()
    .fieldQuery(equal("spec.pinned", true))
    .build();
```

Call `end()` after `labelSelector()` to return to the `ListOptions` builder.
Use `andQuery` and `orQuery` when combining multiple field selector conditions
inside the builder.

Sorting and pagination are passed separately:

```java
import org.springframework.data.domain.Sort;
import run.halo.app.extension.PageRequestImpl;

var sort = Sort.by(Sort.Order.asc("metadata.name"));
var pageable = PageRequestImpl.of(1, 10, sort);

client.listBy(Person.class, options, pageable);
client.listAll(Person.class, options, sort);
```

Fields used in `fieldQuery` or `Sort` must be indexed, otherwise Halo rejects
the query as unsupported. Query values should match the index `keyType`; Halo
uses conversion where possible, but incompatible values fail at query time.

## Query DSL (Field Selectors)

Build typed queries for `ListOptions` field filtering:

```java
import static run.halo.app.extension.index.query.Queries.*;

ListOptions.builder()
    .fieldQuery(and(
        equal("spec.pinned", true),
        contains("spec.name", keyword),
        greaterThan("spec.priority", 10)
    ))
    .build();
```

`QueryFactory` is deprecated since Halo 2.22.0. Use `Queries` to build query
conditions. Negation can be built with either `Queries.not(condition)` or
`condition.not()`.

Available operators include: `empty`, `all`, `equal`, `notEqual`,
`greaterThan(field, value)`, `greaterThan(field, value, inclusive)`,
`lessThan(field, value)`, `lessThan(field, value, inclusive)`, `between`,
`in`, `isNull`, `contains`, `startsWith`, `endsWith`, `and`, `or`, `not`,
`labelExists`, `labelEqual`, `labelIn`.

Use negation for operators that no longer have direct helper methods:

```java
var isNotNull = isNull("metadata.deletionTimestamp").not();
var greaterThanOrEqual = greaterThan("spec.priority", 10, true);
var lessThanOrEqual = lessThan("spec.priority", 20, true);
var labelNotEqual = labelEqual("env", "production").not();
```

The HTTP `fieldSelector` parameter only supports the selector-style subset
(`=`, `!=`, and `in`, for example `fieldSelector=spec.slug=(halo,halo2)`).
For richer field and label conditions in Java code, use `ListOptions` with
`Queries`.

## Extension Utilities

```java
// Check deletion state
boolean deleted = ExtensionUtil.isDeleted(extension);
Predicate<ExtensionOperator> notDeleted = ExtensionOperator.isNotDeleted();

// Finalizer management
ExtensionUtil.addFinalizers(metadata, Set.of("my-plugin/finalizer"));
ExtensionUtil.removeFinalizers(metadata, Set.of("my-plugin/finalizer"));

// Default sort
Sort sort = ExtensionUtil.defaultSort(); // creationTimestamp desc, name asc

// Safe metadata access
Map<String, String> labels = MetadataUtil.nullSafeLabels(extension);
Map<String, String> annotations = MetadataUtil.nullSafeAnnotations(extension);
```

## Naming Rules

- **metadata.name**: ≤253 chars, `[a-z0-9]([-a-z0-9]*[a-z0-9])?`
- **labels keys**: Optional prefix (DNS subdomain) + name (DNS label, ≤63 chars). Reserved: no-prefix keys and `halo.run/*`
- **annotations keys**: Same rules as labels, but not indexed
