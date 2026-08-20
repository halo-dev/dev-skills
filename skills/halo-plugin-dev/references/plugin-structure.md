# Plugin Project Structure

## Typical Directory Layout

```
my-halo-plugin/
├── ui/                              # Frontend source (Vue + TypeScript)
│   ├── src/
│   │   ├── assets/
│   │   ├── views/                   # Page components
│   │   ├── components/              # Reusable components
│   │   └── index.ts                 # Plugin entry (definePlugin)
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts | rsbuild.config.ts
│   └── ...
├── src/main/
│   ├── java/
│   │   └── com/example/myplugin/
│   │       └── MyPlugin.java        # Entry class extending BasePlugin
│   └── resources/
│       ├── plugin.yaml              # Plugin manifest (required)
│       ├── extensions/              # YAML extension declarations
│       ├── templates/               # Thymeleaf templates (optional, for theme integration)
│       └── static/                  # Static assets served at /plugins/{name}/assets (optional)
├── build.gradle                     # Gradle build config
├── gradle.properties
├── settings.gradle
├── gradlew
└── README.md
```

## Backend (`src/main/java/`)

- **Entry class**: One class extending `run.halo.app.plugin.BasePlugin`, annotated with `@Component`. It is the only lifecycle entry point.
- **Spring features supported**: Core IoC, WebFlux reactive stack, Testing. Standard annotations: `@Component`, `@Service`, `@Repository`, `@Configuration`, `@Controller`, etc.
- **Resources**: `src/main/resources/plugin.yaml` is mandatory. `src/main/resources/extensions/` holds YAML declarations for custom extensions, role templates, settings, etc.

## Frontend (`ui/`)

- **Entry file**: `ui/src/index.ts` normally exports a default object created by `definePlugin()`.
- **Production output**: generated under `ui/build/dist`, then copied as a complete directory to `build/resources/main/ui` for JAR packaging.
- **Format**: Halo 2.26+ supports ESM providers with `ui-plugin.json`, hashed entries, styles, and async chunks; older targets use the compatible IIFE format.
- **Build tools**: Vite or Rsbuild via `@halo-dev/ui-plugin-bundler-kit`.

> From Halo 2.11+, UC (User Center) shares the same plugin mechanism. Halo 2.x still reads the legacy `resources/console` directory, but new projects should use `resources/ui`. Do not assume an ESM provider consists only of `main.js` and `style.css`; package the complete generated directory.
