# Content Handlers (Post / SinglePage)

Intercept and modify rendered post/page HTML before it reaches the theme.

> Source: [ReactivePostContentHandler](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/theme/ReactivePostContentHandler.java) | [ReactiveSinglePageContentHandler](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/theme/ReactiveSinglePageContentHandler.java)

## ReactivePostContentHandler

```java
@Component
public class MyPostContentHandler implements ReactivePostContentHandler {

    @Override
    public Mono<PostContentContext> handle(PostContentContext postContent) {
        var content = postContent.getContent();

        // Modify HTML content
        var modified = content.replace("<h2>", "<h2 id=\"heading-\">");

        return Mono.just(postContent.toBuilder()
            .content(modified)
            .build());
    }
}
```

## ReactiveSinglePageContentHandler

```java
@Component
public class MyPageContentHandler implements ReactiveSinglePageContentHandler {

    @Override
    public Mono<SinglePageContentContext> handle(SinglePageContentContext pageContent) {
        // Same pattern as PostContentHandler
        return Mono.just(pageContent);
    }
}
```

## Context Fields

| Field                 | Type                  | Description                        |
| --------------------- | --------------------- | ---------------------------------- |
| `post` / `singlePage` | `Post` / `SinglePage` | The extension object               |
| `content`             | `String`              | Rendered HTML content (modifiable) |
| `raw`                 | `String`              | Raw source content                 |
| `rawType`             | `String`              | Content format (e.g., `markdown`)  |

## Common Use Cases

- Inject diagram rendering (Mermaid, text-diagram)
- Add anchor links to headings
- Wrap code blocks with copy buttons
- Process custom shortcodes
- Add lazy loading to images
- Watermark injection
