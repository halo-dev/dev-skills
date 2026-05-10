# TemplateHeadProcessor

Inject scripts, styles, or meta tags into the theme's `<head>` section.

> Source: [TemplateHeadProcessor](https://github.com/halo-dev/halo/blob/main/api/src/main/java/run/halo/app/theme/dialect/TemplateHeadProcessor.java)

## Usage

```java
@Component
public class MyHeadProcessor implements TemplateHeadProcessor {

    private final ReactiveSettingFetcher settingFetcher;

    public MyHeadProcessor(ReactiveSettingFetcher settingFetcher) {
        this.settingFetcher = settingFetcher;
    }

    @Override
    public Mono<Void> process(ITemplateContext context, IModel model,
                              IElementModelStructureHandler structureHandler) {
        return settingFetcher.fetch("basic", MyConfig.class)
            .flatMap(config -> {
                if (!config.isEnabled()) {
                    return Mono.empty();
                }
                // Add script tag to <head>
                var script = createScriptTag(config.getTrackingId());
                model.add(script);
                return Mono.empty();
            })
            .then();
    }

    private IModel createScriptTag(String trackingId) {
        // Build Thymeleaf model with script element
        var modelFactory = new ModelFactory();
        var script = modelFactory.createOpenElementTag("script");
        // ... configure attributes
        var model = modelFactory.createModel();
        model.add(script);
        return model;
    }
}
```

## Ordering

Use `@Order` to control execution order. Higher values execute first, allowing later processors to override earlier ones.

```java
@Component
@Order(100)  // Higher = earlier
public class HighPriorityHeadProcessor implements TemplateHeadProcessor { ... }

@Component
@Order(200)  // Lower = later, can override above
public class LowPriorityHeadProcessor implements TemplateHeadProcessor { ... }
```

## Common Use Cases

- Analytics tracking (Google Analytics, Plausible)
- Code highlighting (highlight.js, Prism, Shiki)
- Math rendering (KaTeX, MathJax)
- SEO meta tags (OpenGraph, Twitter Cards)
- Custom CSS/JS injection
- Comment widgets
