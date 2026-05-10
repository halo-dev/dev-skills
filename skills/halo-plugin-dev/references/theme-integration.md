# Theme Integration

Plugins can provide data and pages to the theme frontend via Finder APIs, Thymeleaf templates, and reverse proxies.

> - [Finder for theme (official docs)](https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/api-reference/server/finder-for-theme.md)
> - [Template for theme (official docs)](https://raw.githubusercontent.com/halo-dev/docs/refs/heads/main/docs/developer-guide/plugin/api-reference/server/template-for-theme.md)
> - [Plugin photos Finder example](https://github.com/halo-sigs/plugin-photos/blob/main/src/main/java/run/halo/photos/finders/PhotoFinder.java)

## Finder API

A **Finder** is a Java class annotated with `@Component` that exposes methods callable from Thymeleaf templates.

### Creating a Finder

```java
package com.example.myplugin.finders;

import org.springframework.stereotype.Component;
import reactor.core.publisher.Mono;
import run.halo.app.extension.ReactiveExtensionClient;
import run.halo.app.theme.finders.Finder;

@Component
@Finder("myPlugin")  // Template variable name: ${myPlugin}
public class MyPluginFinder {

    private final ReactiveExtensionClient client;

    public MyPluginFinder(ReactiveExtensionClient client) {
        this.client = client;
    }

    public Mono<ListResult<Person>> listPersons(int page, int size) {
        return client.listBy(Person.class,
            ListOptions.builder().build(),
            PageRequestImpl.of(page, size));
    }

    public Mono<Person> getPerson(String name) {
        return client.fetch(Person.class, name);
    }
}
```

### Using in Templates

```html
<!-- List persons -->
<ul>
  <li th:each="person : ${myPlugin.listPersons(1, 10).items}" th:text="${person.spec.name}"></li>
</ul>

<!-- Get single person -->
<div th:with="person = ${myPlugin.getPerson('john')}">
  <h1 th:text="${person.spec.name}"></h1>
</div>
```

## Thymeleaf Templates

Place templates in `src/main/resources/templates/`:

```
src/main/resources/
  templates/
    my-page.html          # Accessible as a route
    modules/
      my-widget.html      # Partial templates
```

Use in a template:

```html
<div th:replace="~{modules/my-widget :: my-widget}"></div>
```

## Reverse Proxy

Serve plugin static resources or proxy external APIs through Halo:

Create `src/main/resources/extensions/reverseProxy.yaml`:

```yaml
apiVersion: plugin.halo.run/v1alpha1
kind: ReverseProxy
metadata:
  name: my-plugin-reverse-proxy
rules:
  - path: /assets
    file:
      directory: static/dist # relative to src/main/resources/
  - path: /api/proxy
    url:
      url: http://localhost:8080
```

Access at: `/plugins/{plugin-name}/assets/...`

## Static Resources

Place static files in `src/main/resources/static/`:

```
src/main/resources/
  static/
    dist/
      main.css
      main.js
```

Access at: `/plugins/{plugin-name}/assets/dist/main.css`

## Template Variables

Plugins can contribute global template variables via a `TemplateModel` bean:

```java
@Component
public class MyTemplateModel implements TemplateModel {

    @Override
    public String getVariableName() {
        return "myPluginData";
    }

    @Override
    public Mono<Object> getValue() {
        return Mono.just(Map.of("version", "1.0.0"));
    }
}
```

Then in any template:

```html
<div th:text="${myPluginData.version}"></div>
```

## CommentSubject

Enable Halo's comment system on your custom Extension:

```java
@Component
public class MyCommentSubject implements CommentSubject<MyExtension> {

    private final ReactiveExtensionClient client;

    public MyCommentSubject(ReactiveExtensionClient client) {
        this.client = client;
    }

    @Override
    public Mono<MyExtension> get(String name) {
        return client.fetch(MyExtension.class, name)
            .switchIfEmpty(Mono.error(() -> new NotFoundException("Not found")));
    }

    @Override
    public Mono<SubjectDisplay> getSubjectDisplay(String name) {
        return get(name).map(ext -> new SubjectDisplay(
            ext.getSpec().getTitle(),
            "/my-extensions/" + ext.getSpec().getSlug(),
            "My Extension"
        ));
    }

    @Override
    public boolean supports(Ref ref) {
        return GroupVersionKind.fromExtension(MyExtension.class).equals(ref.getGroupVersionKind());
    }
}
```

Also add a role template aggregating `comment` permissions to `anonymous`:

```yaml
metadata:
  labels:
    halo.run/role-template: "true"
    halo.run/hidden: "true"
    rbac.authorization.halo.run/aggregate-to-anonymous: "true"
rules:
  - apiGroups: ["my-plugin.halo.run"]
    resources: ["my-extensions/comments"]
    verbs: ["create", "list"]
```

## URL Conventions for Public APIs

When building APIs consumed by the theme:

```
/apis/api.{group}/{version}/{resource}
```

Example: `/apis/api.my-plugin.halo.run/v1alpha1/persons`

These should have role templates aggregated to `anonymous` for public access.
