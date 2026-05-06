# fluxtion-playground-libs

Curated, CheerpJ-verified Maven artefacts served to the Fluxtion playground via
[jsDelivr](https://cdn.jsdelivr.net/) so the browser client can fetch them with
CORS enabled.

## How it's consumed

The playground reads `catalog.json` over jsDelivr and lazy-fetches the JAR
bytes the first time a user adds a dependency. Bytes are cached in IndexedDB,
so subsequent visits are instant.

```
https://cdn.jsdelivr.net/gh/telaminai/fluxtion-playground-libs@main/catalog.json
https://cdn.jsdelivr.net/gh/telaminai/fluxtion-playground-libs@main/libs/<group>/<artifact>/<version>/<file>.jar
```

## Layout

```
catalog.json                   # what the playground reads
libs/<group-path>/<artifact>/<version>/<artifact>-<version>.jar
```

Maven coordinate paths mirror Maven Central exactly so the same path can be
re-used in the downloaded Maven project's `pom.xml` (modulo repo URL).

## Adding a new library

1. Verify the JAR loads and runs under CheerpJ (Java 8 bytecode, no JNI / unsafe
   APIs, no Java 9+ module references). The simplest check is a one-method
   smoke test in the playground.
2. Download the JAR (and any flattened transitive deps it needs at runtime)
   from Maven Central:
   ```sh
   curl -sSfL -o libs/<group-path>/<artifact>/<version>/<file>.jar \
     "https://repo1.maven.org/maven2/<group-path>/<artifact>/<version>/<file>.jar"
   ```
3. Compute SHA-256 + size:
   ```sh
   shasum -a 256 libs/.../*.jar
   wc -c libs/.../*.jar
   ```
4. Add an entry to `catalog.json` listing the JAR plus all runtime
   transitives (flattened — no in-browser POM resolution).
5. Commit + push. jsDelivr edge-cache picks up `@main` within ~10 minutes;
   pin the playground client to a specific commit SHA for instant cache-bust.

## What's currently here

| Library | Version | License | Java 8 |
| ------- | ------- | ------- | ------ |
| Jackson databind (+ core, annotations) | 2.16.1 | Apache-2.0 | yes |
| SLF4J API + simple binding | 2.0.9 | MIT | yes |
| Guava | 33.0.0-jre | Apache-2.0 | yes |

CheerpJ smoke-test status for each of these is recorded as the playground
exercises them; if a lib gets removed from the catalog it's because it failed
to load under CheerpJ (browser-side javac classpath wiring or runtime
linkage error).

## Why this repo exists rather than a CDN bundle / backend proxy

- Zero infra. Public GitHub + jsDelivr's free Cloudflare-edge mirror.
- Versioned and auditable — git log is the change history.
- Each commit's URLs are effectively immutable, so an older playground
  release keeps fetching the JARs it was tested with.
- Adding a lib is a 4-line shell script + a JSON edit; no service deploy.
