# DevTools (run.halo.plugin.devtools)

A Gradle plugin for streamlined plugin development. Requires Docker.

> - [halo-gradle-plugin repo](https://github.com/halo-sigs/halo-gradle-plugin)
> - [halo-gradle-plugin releases](https://github.com/halo-sigs/halo-gradle-plugin/releases)

## Quick Commands

| Command                           | Purpose                                             |
| --------------------------------- | --------------------------------------------------- |
| `./gradlew haloServer`            | Start Halo in Docker with plugin loaded in dev mode |
| `./gradlew reload`                | Reload plugin changes without restarting Halo       |
| `./gradlew watch`                 | Auto-reload on file changes                         |
| `./gradlew build`                 | Build plugin JAR (includes frontend build)          |
| `./gradlew generateApiClient`     | Generate TypeScript API client from OpenAPI         |
| `./gradlew generateRoleTemplates` | Generate role template YAML from OpenAPI            |

## build.gradle Setup

```groovy
plugins {
    id 'java'
    id "io.freefair.lombok" version "8.13"
    id "run.halo.plugin.devtools" version "0.6.0"
}

group = 'com.example.myplugin'

repositories {
    mavenCentral()
}

dependencies {
    implementation platform('run.halo.tools.platform:plugin:2.22.0')
    compileOnly 'run.halo.app:api'

    testImplementation 'run.halo.app:api'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

test {
    useJUnitPlatform()
}

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

tasks.withType(JavaCompile).configureEach {
    options.encoding = "UTF-8"
    options.release = 21
}

// UI build integration (if UI is in a separate subproject)
tasks.register('processUiResources', Copy) {
    from project(':ui').layout.buildDirectory.dir('dist')
    into layout.buildDirectory.dir('resources/main/console')
    dependsOn project(':ui').tasks.named('assemble')
    shouldRunAfter tasks.named('processResources')
}

tasks.named('classes') {
    dependsOn tasks.named('processUiResources')
}

halo {
    version = '2.22'
    superAdminUsername = 'admin'
    superAdminPassword = 'admin'
    port = 8090
    // debug = true
    // debugPort = 5005
    // suspend = true
    externalUrl = 'http://localhost:8090'
}
```

## halo {} Block Options

| Option               | Description                                | Default                                   |
| -------------------- | ------------------------------------------ | ----------------------------------------- |
| `version`            | Halo Docker image version                  | `'2.9.1'`                                 |
| `superAdminUsername` | Auto-created admin username                | `'admin'`                                 |
| `superAdminPassword` | Auto-created admin password                | `'admin'`                                 |
| `port`               | Halo server port                           | `8090`                                    |
| `externalUrl`        | External access URL                        | `'http://localhost:8090'`                 |
| `debug`              | Enable JDWP debug                          | `false`                                   |
| `debugPort`          | JDWP port                                  | auto-assigned                             |
| `suspend`            | Suspend on startup until debugger connects | `false`                                   |
| `docker.url`         | Docker daemon URL                          | `unix:///var/run/docker.sock` (Mac/Linux) |
| `docker.apiVersion`  | Docker API version                         | `'1.42'`                                  |

## Watch Configuration

```groovy
haloPlugin {
    watchDomains {
        consoleSource {
            files files('ui/src/')
            // exclude '**/node_modules/**'
        }
    }
}
```

## Generate API Client

Requires OpenAPI grouping configuration in `build.gradle`:

```groovy
haloPlugin {
    openApi {
        groupingRules {
            extensionApis {
                displayName = 'Extension API for MyPlugin'
                pathsToMatch = ['/apis/my-plugin.halo.run/v1alpha1/**']
            }
        }
        groupedApiMappings = [
            '/v3/api-docs/extensionApis': 'extensionApis.json'
        ]
        generator {
            outputDir = file("${projectDir}/ui/src/api/generated")
            additionalProperties = [
                useES6: true,
                useSingleRequestParameter: true,
                withSeparateModelsAndApi: true,
                apiPackage: "api",
                modelPackage: "models"
            ]
            typeMappings = [
                set: "Array"
            ]
        }
    }
}
```

Usage in TypeScript:

```ts
import { axiosInstance } from "@halo-dev/api-client";
import { MyResourceV1alpha1Api } from "./api/generated";

const api = new MyResourceV1alpha1Api(undefined, "", axiosInstance);
const { data } = await api.listMyResources({});
```

## Generate Role Templates

After configuring OpenApi grouping, run:

```bash
./gradlew generateRoleTemplates
```

Generates `roleTemplates.yaml` in the `workplace/` directory. Review and customize before adding to `src/main/resources/extensions/`.

## Custom Halo Config

Place `workplace/config/application.yaml` to override Halo defaults:

```yaml
logging:
  level:
    run.halo.app: DEBUG
```
