# RBAC & Role Templates

All plugin APIs (auto-generated CRUD + custom) are restricted to super-admin by default. To allow other users access, define **role templates**.

> Source references (Halo main branch):
>
> - [Role template anonymous](https://github.com/halo-dev/halo/blob/main/application/src/main/resources/extensions/role-template-anonymous.yaml)
> - [Role template authenticated](https://github.com/halo-dev/halo/blob/main/application/src/main/resources/extensions/role-template-authenticated.yaml)
> - [Plugin photos role templates (production example)](https://github.com/halo-sigs/plugin-photos/blob/main/src/main/resources/extensions/roleTemplate.yaml)

## Role Template File

Place YAML files in `src/main/resources/extensions/`.

```yaml
apiVersion: v1alpha1
kind: Role
metadata:
  name: my-plugin-role-view-persons
  labels:
    halo.run/role-template: "true"
  annotations:
    rbac.authorization.halo.run/module: "Persons Management"
    rbac.authorization.halo.run/display-name: "View Persons"
    rbac.authorization.halo.run/ui-permissions: |
      ["plugin:my-plugin:person:view"]
rules:
  - apiGroups: ["my-plugin.halo.run"]
    resources: ["my-plugin/persons"]
    verbs: ["get", "list"]
---
apiVersion: v1alpha1
kind: Role
metadata:
  name: my-plugin-role-manage-persons
  labels:
    halo.run/role-template: "true"
  annotations:
    rbac.authorization.halo.run/dependencies: |
      ["my-plugin-role-view-persons"]
    rbac.authorization.halo.run/module: "Persons Management"
    rbac.authorization.halo.run/display-name: "Manage Persons"
    rbac.authorization.halo.run/ui-permissions: |
      ["plugin:my-plugin:person:manage"]
rules:
  - apiGroups: ["my-plugin.halo.run"]
    resources: ["my-plugin/persons"]
    verbs: ["*"]
```

## Key Rules

| Element                                                  | Description                                        |
| -------------------------------------------------------- | -------------------------------------------------- |
| `metadata.name`                                          | Must use plugin name as prefix to avoid collisions |
| `labels.halo.run/role-template: "true"`                  | Required to mark as template                       |
| `annotations.rbac.authorization.halo.run/dependencies`   | Other roles this role requires                     |
| `annotations.rbac.authorization.halo.run/module`         | UI grouping name                                   |
| `annotations.rbac.authorization.halo.run/display-name`   | Human-readable name                                |
| `annotations.rbac.authorization.halo.run/ui-permissions` | Frontend permission strings                        |

## Resource Rules

For APIs matching `/apis/<group>/<version>/<resource>[/<name>/<subresource>]`:

```yaml
rules:
  - apiGroups: ["my-plugin.halo.run"]
    resources: ["my-plugin/persons"]
    resourceNames: ["zhangsan"] # optional, for single resource
    verbs: ["get", "list"]
```

## Non-Resource Rules

For APIs not matching resource patterns (e.g., `/healthz`):

```yaml
rules:
  - nonResourceURLs: ["/healthz", "/healthz/*"]
    verbs: ["get"]
```

## Verbs

| Verb               | HTTP Method     | Description                             |
| ------------------ | --------------- | --------------------------------------- |
| `create`           | POST            | Create new resource                     |
| `get`              | GET             | Get single resource (with name in path) |
| `list`             | GET             | List resources (without name in path)   |
| `watch`            | GET (WebSocket) | Watch resource changes                  |
| `update`           | PUT             | Full update                             |
| `patch`            | PATCH           | Partial update                          |
| `delete`           | DELETE          | Delete single resource                  |
| `deletecollection` | DELETE          | Delete collection                       |

## Aggregation

Merge plugin permissions into existing Halo roles:

```yaml
metadata:
  labels:
    halo.run/role-template: "true"
    halo.run/hidden: "true" # hide from UI
    rbac.authorization.halo.run/aggregate-to-anonymous: "true"
rules:
  - apiGroups: ["api.my-plugin.halo.run"]
    resources: ["public-data"]
    verbs: ["get", "list"]
```

Available aggregation targets: `anonymous`, `authenticated`, `editor`, etc.

## Special Roles

| Role            | Description                                              |
| --------------- | -------------------------------------------------------- |
| `anonymous`     | Unauthenticated visitors                                 |
| `authenticated` | All logged-in users (minimum permissions)                |
| `super-role`    | Full system access                                       |
| `guest`         | No explicit permissions (only anonymous + authenticated) |

## Web Filters

Add custom WebFlux filters for request interception.

```java
@Component
public class MyFilter implements AdditionalWebFilter {
    @Override
    public int getOrder() { return Ordered.LOWEST_PRECEDENCE; }

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        // Runs before security chain
        return chain.filter(exchange);
    }
}
```

`AfterSecurityWebFilter` runs after the security chain (for caching, analytics, etc.).

## ReactiveSecurityContextHolder

Access the current authentication in reactive code:

```java
ReactiveSecurityContextHolder.getContext()
    .map(SecurityContext::getAuthentication)
    .map(Authentication::getName)
    .defaultIfEmpty(AnonymousUserConst.PRINCIPAL);
```

## UI Permissions

Used in frontend route guards:

```ts
meta: {
  permissions: ["plugin:my-plugin:person:view"];
}
```

Check at runtime:

```ts
import { utils } from "@halo-dev/ui-shared";

utils.permission.has(["plugin:my-plugin:person:view"]); // true/false
```
