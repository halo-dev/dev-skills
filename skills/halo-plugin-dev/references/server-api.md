# Custom APIs

Halo plugins can define custom APIs in addition to auto-generated CRUD APIs.

> Source references (Halo main branch):
>
> - [CustomEndpoint](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/core/extension/endpoint/CustomEndpoint.java)
> - [ApiVersion](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/plugin/ApiVersion.java)
> - [SpringdocRouteBuilder](https://github.com/halo-dev/halo/blob/main/application/src/main/java/run/halo/app/infra/utils/SpringdocRouteBuilder.java)
> - [SortableRequest](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/extension/router/SortableRequest.java)
> - [PostEndpoint (SpringdocRouteBuilder example)](https://github.com/halo-dev/halo/blob/main/application/src/main/java/run/halo/app/core/endpoint/console/PostEndpoint.java)

## API Group Conventions

| Scope            | URL Prefix                                | Example                                                 |
| ---------------- | ----------------------------------------- | ------------------------------------------------------- |
| Console          | `/apis/console.api.{group}/{version}/...` | `/apis/console.api.my-plugin.halo.run/v1alpha1/persons` |
| UC (User Center) | `/apis/uc.api.{group}/{version}/...`      | `/apis/uc.api.my-plugin.halo.run/v1alpha1/persons`      |
| Public (theme)   | `/apis/api.{group}/{version}/...`         | `/apis/api.my-plugin.halo.run/v1alpha1/persons`         |

> `{group}` is the `group` value from the `@GVK` annotation.

## Method 1: CustomEndpoint (Recommended)

Implements `run.halo.app.core.extension.endpoint.CustomEndpoint`:

```java
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.ServerResponse;
import reactor.core.publisher.Mono;
import run.halo.app.core.extension.endpoint.CustomEndpoint;
import run.halo.app.extension.GroupVersion;

import static org.springframework.web.reactive.function.server.RequestPredicates.accept;
import static org.springframework.web.reactive.function.server.RouterFunctions.route;
import static org.springframework.http.MediaType.APPLICATION_JSON;

@Component
public class PersonEndpoint implements CustomEndpoint {

    @Override
    public RouterFunction<ServerResponse> endpoint() {
        return route()
            .GET("/persons/{name}", accept(APPLICATION_JSON), this::getPerson)
            .POST("/persons", this::createPerson)
            .build();
    }

    private Mono<ServerResponse> getPerson(ServerRequest request) {
        String name = request.pathVariable("name");
        return ServerResponse.ok().bodyValue("Hello, " + name);
    }

    private Mono<ServerResponse> createPerson(ServerRequest request) {
        // ...
        return ServerResponse.ok().build();
    }

    @Override
    public GroupVersion groupVersion() {
        return new GroupVersion("console.api.my-plugin.halo.run", "v1alpha1");
    }
}
```

The `endpoint()` paths are automatically prefixed with `/apis/{group}/{version}/`.

## Method 2: MVC-style @Controller

```java
import run.halo.app.plugin.ApiVersion;

@ApiVersion("my-plugin.halo.run/v1alpha1")
@RestController
@RequiredArgsConstructor
@RequestMapping("/persons")
public class PersonController {
    private final PersonService personService;

    @GetMapping("/{name}")
    public Mono<Person> getPerson(@PathVariable("name") String name) {
        return personService.getPerson(name);
    }
}
```

> ⚠️ **@ApiVersion is required** — controllers without it will not be registered.

## OpenAPI Documentation

Use `SpringdocRouteBuilder` to document Functional Endpoints:

```java
import run.halo.app.infra.utils.SpringdocRouteBuilder;

@Override
public RouterFunction<ServerResponse> endpoint() {
    final var tag = "PersonV1alpha1Console";
    return SpringdocRouteBuilder.route()
        .GET("/persons", this::listPersons,
            builder -> builder
                .operationId("ListPersons")
                .description("List all persons")
                .tag(tag)
                .response(responseBuilder()
                    .implementation(ListResult.generateGenericClass(Person.class))
                )
        )
        .build();
}
```

Tag naming convention: `{Kind}{Version}{Scope}` e.g., `PersonV1alpha1Console`.

## Query Parameters

Extend `run.halo.app.extension.router.SortableRequest` for list queries:

```java
import static run.halo.app.extension.index.query.Queries.equal;
import static run.halo.app.extension.index.query.Queries.contains;
import static run.halo.app.extension.index.query.Queries.or;

public class PersonQuery extends SortableRequest {
    public PersonQuery(ServerWebExchange exchange) {
        super(exchange);
    }

    public String getKeyword() {
        return queryParams.getFirst("keyword");
    }

    @Override
    public ListOptions toListOptions() {
        var keyword = getKeyword();
        if (StringUtils.hasText(keyword)) {
            return ListOptions.builder(super.toListOptions())
                .fieldQuery(or(
                    equal("metadata.name", keyword),
                    contains("spec.name", keyword)
                ))
                .build();
        }
        return super.toListOptions();
    }
}
```

Usage:

```java
public Mono<ListResult<Person>> list(ServerRequest request) {
    var query = new PersonQuery(request.exchange());
    return client.listBy(Person.class, query.toListOptions(), query.toPageRequest());
}
```

## Request Validation (Bean Validation)

```java
public class PersonParam {
    @NotNull
    @Size(max = 64)
    private String name;

    @Min(0)
    private int age;
}
```

Enable validator:

```java
@Configuration
public class PluginConfig {
    @Bean
    public LocalValidatorFactoryBean validator() {
        return new LocalValidatorFactoryBean();
    }
}
```

Inject and use:

```java
@Component
@RequiredArgsConstructor
public class PersonEndpoint implements CustomEndpoint {
    private final Validator validator;

    private Mono<ServerResponse> createPerson(ServerRequest request) {
        return request.bodyToMono(PersonParam.class)
            .doOnNext(this::validate)
            .flatMap(person -> /* ... */);
    }

    private void validate(PersonParam param) {
        var result = new BeanPropertyBindingResult(param, "person");
        validator.validate(param, result);
        if (result.hasErrors()) {
            throw new RequestBodyValidationException(result);
        }
    }
}
```

## Swagger Groups

API docs at `/swagger-ui.html` are grouped as:

| Group                    | Content                  |
| ------------------------ | ------------------------ |
| Aggregated API V1alpha1  | All APIs combined        |
| Extension API V1alpha1   | Auto-generated CRUD APIs |
| Console API V1alpha1     | Console custom APIs      |
| User-center API V1alpha1 | UC custom APIs           |
| Public API V1alpha1      | Public/theme custom APIs |
