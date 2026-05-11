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
> - [QueryFactory](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/index/query/QueryFactory.java)
> - [FieldSelector](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/router/selector/FieldSelector.java)
> - [ExtensionUtil](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/ExtensionUtil.java)
> - [MetadataUtil](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/MetadataUtil.java)
> - [ExtensionOperator](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/ExtensionOperator.java)

## Creating an Extension

Three steps:

1. Create a class extending `run.halo.app.extension.AbstractExtension`
2. Annotate with `@GVK(group, version, kind, plural, singular)`
3. Register in plugin `start()` via `SchemeManager`

## Example

```java
import io.swagger.v3.oas.annotations.media.Schema;
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

        @Schema(description = "Age", maximum = "150", minimum = "0")
        private Integer age;
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
GET /apis/my-plugin.halo.run/v1alpha1/persons?page=1&size=10&sort=name,desc&fieldSelector=spec.age=18
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
import java.util.HashSet;
import run.halo.app.extension.index.IndexSpecs;

@Override
public void start() {
    schemeManager.register(Person.class, indexSpecs -> {
        // Single-value index (returns one value, can be null)
        indexSpecs.add(IndexSpecs.<Person, String>single("spec.name", String.class)
            .indexFunc(person -> person.getSpec().getName()));

        // Multi-value index (returns Set of values)
        indexSpecs.add(IndexSpecs.<Person, String>multi("spec.tags", String.class)
            .indexFunc(person -> {
                var tags = person.getSpec().getTags();
                return tags == null ? new HashSet<>() : new HashSet<>(tags);
            }));

        // Other supported types: Boolean, Integer, Long, Instant
        indexSpecs.add(IndexSpecs.<Person, Integer>single("spec.priority", Integer.class)
            .indexFunc(person -> person.getSpec().getPriority()));
    });
}
```

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

## Query DSL (Field Selectors)

Build typed queries for `ListOptions` field filtering:

```java
import static run.halo.app.extension.index.query.QueryFactory.*;

ListOptions.builder()
    .fieldQuery(and(
        equal("spec.status", "published"),
        contains("spec.title", keyword),
        greaterThan("spec.priority", "10")
    ))
    .build();
```

Available operators: `equal`, `notEqual`, `contains`, `startsWith`, `endsWith`, `greaterThan`, `lessThan`, `greaterThanOrEqual`, `lessThanOrEqual`, `between`, `in`, `isNull`, `isNotNull`, `and`, `or`, `not`, `all`.

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
