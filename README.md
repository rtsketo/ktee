<div align="center">
  <h1><img src="assets/ktee-logo.svg" alt="KTee — see the value, keep the flow" width="760"></h1>
  <p><strong>Peek at values. Keep your pipeline.</strong></p>
  <p>A tiny Kotlin <code>tee</code> for inspecting values as they flow through a chain.</p>
  <p>
    <a href="https://github.com/rtsketo/ktee/releases/tag/1.0.5"><img alt="Release" src="https://img.shields.io/github/v/release/rtsketo/ktee?label=release&amp;color=7c3aed"></a>
    <a href="https://kotlinlang.org/"><img alt="Kotlin 1.7.20" src="https://img.shields.io/badge/Kotlin-1.7.20-22d3ee"></a>
    <a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-a3e635"></a>
  </p>
  <p><a href="#quick-start">Quick start</a> · <a href="#installation">Install</a> · <a href="#global-prefix">Global prefix</a> · <a href="#api-at-a-glance">API</a></p>
</div>

## Why KTee?

A pipeline is easier to follow when you can inspect intermediate values without breaking the chain. KTee prints or logs a value **and returns that same value**, so the rest of the expression stays intact.

`tee()` writes to standard output in the active artifact, `yetee`. Pair it with the `notee` artifact for release builds: the same calls compile, but tee output and tee lambdas are skipped.

## Quick start

```kotlin
import ktee.tee

val total = (1..10)
    .filter { it % 2 == 0 }.tee("even: ")
    .map { it * 2 }.tee { "doubled: $it" }
    .reduce(Int::plus).tee("total: ")
```

```text
even: [2, 4, 6, 8, 10]
doubled: [4, 8, 12, 16, 20]
total: 60
```

`total` is still `60`. Add or remove tee calls without changing the value returned by the pipeline.

## Installation

Add [JitPack](https://jitpack.io/#rtsketo/ktee) to the repositories used for dependencies. For Android projects with debug and release variants, use **yetee for debug** and **notee for release**:

```groovy
repositories {
    maven { url 'https://jitpack.io' }
}

dependencies {
    debugImplementation 'com.github.rtsketo.ktee:yetee:1.0.5'
    releaseImplementation 'com.github.rtsketo.ktee:notee:1.0.5'
}
```

With Gradle Kotlin DSL, the dependency declarations are:

```kotlin
dependencies {
    debugImplementation("com.github.rtsketo.ktee:yetee:1.0.5")
    releaseImplementation("com.github.rtsketo.ktee:notee:1.0.5")
}
```

If your project manages repositories in `settings.gradle(.kts)`, put the JitPack repository there instead. In a plain JVM project, depend on the artifact appropriate for that build; `debugImplementation` and `releaseImplementation` are Android variant configurations.

## Global prefix

Set `KTee.prefix` to prepend the same string to **every KTee stdout and SLF4J message**. It starts as `""`, so existing `tee()` and `tee { }` calls keep their original output until you set it.

```kotlin
import ktee.KTee
import ktee.tee

KTee.prefix = "[app] "
"request-42".tee("Processed: ")    // [app] Processed: request-42
"request-42".tee()                  // [app] request-42
"request-42".tee { "Processed: $it" } // [app] Processed: request-42
```

The global prefix comes **before** a per-call marker or lambda result. The no-op artifact exposes `KTee.prefix` too, but does not print anything. This setting is mutable and shared across calls; choose and set it in your app's initialization code.

## Log with SLF4J

Pass your SLF4J `Logger` to log at the desired level. Each function also returns its receiver, just like `tee()`.

```kotlin
import ktee.teeToDebug
import ktee.teeToInfo
import ktee.teeToTrace
import org.slf4j.LoggerFactory

val logger = LoggerFactory.getLogger("KTeeDemo")
"request-42".teeToDebug(logger)
"request-42".teeToTrace(logger, "Tracing {}")
"request-42".teeToInfo(logger) { "Processed: $it" }
```

The message overload defaults to `"{}"`, and the global prefix applies to both message and lambda overloads. Your SLF4J binding controls the final log format; `tee()` prints directly to stdout instead.

## API at a glance

| Call | Active artifact (`yetee`) | No-op artifact (`notee`) |
| --- | --- | --- |
| `value.tee()` | Print the value | Return the value |
| `value.tee("label: ")` | Print the label and value | Return the value |
| `value.tee { "value: $it" }` | Print the lambda result | Return the value without evaluating the lambda |
| `value.teeToInfo(logger)` | Log at INFO | Return the value |
| `value.teeToDebug(logger)` | Log at DEBUG | Return the value |
| `value.teeToTrace(logger)` | Log at TRACE | Return the value |
| `debug { ... }` | Run the block | Skip the block |
| `release { ... }` | Skip the block | Run the block |

The `teeTo*` functions also accept a message template or a lambda. All tee calls preserve the original value and its type.

## Development

Run the tests and check that the release substitute compiles:

```sh
./gradlew test compileNoopKotlin
```

## Origins and license

This project builds on [Medly's KTee](https://github.com/medly/ktee). The fork adds a no-op artifact and a shared prefix for tee output. Licensed under [MIT](LICENSE).
