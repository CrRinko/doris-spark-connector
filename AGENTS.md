# AGENTS.md

## Project overview

Two **independent** Maven multi-module projects — no root `pom.xml`:

| Directory | Purpose | Profiles |
|---|---|---|
| `spark-doris-connector/` | Spark Doris Connector (read/write) | `spark-2.4_2.11`, `spark-2.4_2.12`, `spark-3.1` … `spark-3.5` |
| `spark-load/` | Spark-based bulk load into Doris | `spark2`/`spark3`, `scala_2.11`/`scala_2.12` |

Submodules under `spark-doris-connector/`:
- `spark-doris-connector-base/` — core client logic
- `spark-doris-connector-spark-3-base/` — Spark 3 DataSource V2 base
- `spark-doris-connector-spark-3.x/` — version-specific adapters
- `spark-doris-connector-spark-2/` — Spark 2 adapter
- `spark-doris-connector-it/` — integration tests (Testcontainers)

## Build commands

Every command runs **inside the target subdirectory**, e.g. `cd spark-doris-connector`.

### Single Spark version (preferred)

```bash
# Build spark-3.2 fat JAR (install into local repo, skip tests)
cd spark-doris-connector
mvn install -DskipTests -P spark-3.2 -pl spark-doris-connector-spark-3.2 -am

# Build spark-load
cd spark-load
mvn package -DskipTests -P spark3,scala_2.12
```

### Via build script (Unix only, interactive prompts)

```bash
cd spark-doris-connector && ./build.sh
```

### Maven Wrapper

Unix: `./mvnw` in either subdirectory.
Windows: no `.cmd` wrapper. Use `java` directly:

```powershell
$env:JAVA_HOME = '<JDK path>'
java -cp '..\.mvn\wrapper\maven-wrapper.jar' `
     "-Dmaven.multiModuleProjectDirectory=$PWD" `
     org.apache.maven.wrapper.MavenWrapperMain <goals>
```

## Spark version switching

The `general-env` profile activates when `CUSTOM_MAVEN_REPO` is unset. This **disables `activeByDefault`** — so changing the default Spark version requires editing `<properties>` directly in `spark-doris-connector/pom.xml`:

```xml
<spark.version>3.2.0</spark.version>
<spark.major.version>3.2</spark.major.version>
<scala.version>2.12.18</scala.version>
<scala.major.version>2.12</scala.major.version>
```

## Commit message convention

All commits use the format `[Type] Description`:

```
[Chore] Update copyright year in NOTICE.txt
[Improve] Enable gzip compression by default
[Fix] Fixes the issue where configuring doris.benodes is not working
[Feature](Connector) Use Array[String] instead of defaulting to String
```

Common types: `Chore`, `Improve`, `Fix`, `Feature`, `Enhancement`. Scope in parentheses is optional: `[Improve](connector)`.

## Key facts

- **Java 8 target**: `maven.compiler.source=8`, `maven.compiler.target=8`
- **Scale 2.12**: required for Spark 3.x; Scala 2.11 only for Spark 2.x
- **Shade plugin**: runs at `package` phase, relocates deps under `org.apache.doris.shaded.*`
- **Fat JAR output**: `spark-doris-connector-spark-3.2/target/spark-doris-connector-spark-3.2-*.jar`
- **`.gitignore`**: excludes `*.jar`, `custom_env.sh`, all `target/` dirs, IDE files
- **`custom_env.sh`**: optional, created from `custom_env.sh.tpl`, sourced by `env.sh` → `build.sh`
