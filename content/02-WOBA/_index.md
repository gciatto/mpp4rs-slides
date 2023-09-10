+++
title = "Write once, build anywhere with Kotlin"
description = "Kotlin as the super language for multi-platform programming"
outputs = ["Reveal"]
aliases = [
    "/woba/",
    "/kotlin/"
]
+++

## Write once, build anywhere with __Kotlin__

- JetBrains-made modern programming language
    * focused on "practical use" (whatever that means)

- Gaining momentum since Google adopted is as *official Android language*
    * along with Java and C++

- Clearly inspired by a mixture of Java, C#, Scala, and Groovy
    * standard library clearly inspired to Java's one

- Born in industry, for the industry
    * initially considered *a better java*
    * focused on getting productive quickly and reducing programming errors

- Acts as the "super language" supporting the "write once, build anywhere" approach
    * Kotlin compilers are available for several platforms
    * standard library implemented for those platforms

--- 

## Kotlin: supported platforms (a.k.a. targets)

Reference: https://kotlinlang.org/docs/reference/mpp-dsl-reference.html

- JVM
    + first and best supported platform
    + may target a specific JDK version: prefer `11`

- JavaScript (JS)
    + requires choosing among Browser support, Node support or both
    + the two sub-targets have different runtimes

- Native
    + supporting native may imply dealing with 3+ targets
    + cross-compilation is not yet supported

--- 

## Kotlin: supported platforms, Native sub-targets

Reference: https://kotlinlang.org/docs/reference/mpp-dsl-reference.html

- Win by [MinGW](https://sourceforge.net/projects/mingw), Linux
    + must pay attention to architecture
    + MinGW is not exactly "vanilla" Windows environment

- iOS, macOS
    + requires Apple facilities (e.g. XCode)
    + may imply further costs

- Android
    + requires Android SDK

---

## __Gradle__ as the tool for build automation

- Building a multi-platform Kotlin project is _only_ supported via [Gradle](https://gradle.com/)
    * no real alternatives here

- Gradle is a build automation + dependency management system

- Gradle simultaneously enables & constrains the multi-platform workflow
    * pre-defined project structure
    * pre-defined development workflow
    * enforced code partitioning
    * ad-hoc tasks for platform specific/agnostic compilation, testing, deploy, etc.

---

## Kotlin multi-platform project structure

### Common

Kotlin enforces strong segregation of the platform-agnostic and platform-specific parts

- __common *main* code__: platform-agnostic code which only depends on:
    * the Kotlin std-lib 
    * plus other third-party Kotlin multi-platform libraries

- __common *test* code__: platform-agnostic _test_ code which only depends on:
    * the common main code and all its dependencies
    * some Kotlin multi-platform testing library (e.g. [`kotlin.test`](https://kotlinlang.org/api/latest/kotlin.test/) or [Kotest](https://kotest.io/))

---

## Kotlin multi-platform project structure 

### Platform-specific

- for each target platform `T` (e.g. `jvm`, `js`, etc.)
    * __`T`-specific *main* code__: main Kotlin code targeting the `T` platform, depending on:
        + the common main code and all its dependencies
        + `T`-specific standard library
        + `T`-specific third-party libraries

    * __`T`-specific *test* code__: test Kotlin code targeting the `T` platform, depending on:
        + `T`-specific main code
        + `T`-specific test libraries

---

## Kotlin multi-platform project structure (example)

```bash
<root>/
│
├── build.gradle.kts        # multi-platform build description 
├── settings.gradle.kts     # root project settings
│
└── src/
    ├── commonMain/
    │   └── kotlin/         # platform-agnostic Kotlin code here	font-weight: normal;

    ├── commonTest/
    │   └── kotlin/         # platform-agnostic test code written in Kotlin here
    │
    ├── jvmMain/
    │   ├── java/           # JVM-specific Java code here
    │   ├── kotlin/         # JVM-specific Kotlin code here
    │   └── resources/      # JVM-specific resources here
    ├── jvmTest
    │   ├── java/           # JVM-specific test code written in Java here
    │   ├── kotlin/         # JVM-specific test code written in Kotlin here
    │   └── resources/      # JVM-specific test resources here
    │
    ├── jsMain/
    │   └── kotlin/         # JS-specific Kotlin code here
    ├── jsTest/
    │   └── kotlin/         # JS-specific Kotlin code here
    │
    ├── <TARGET>Main/       
    │   └── kotlin/         # <TARGET>-specific Kotlin code here
    └── <TARGET>Test/
        └── kotlin/         # <TARGET>-specific test code written in Kotlin here
```

---

## Kotlin multi-platform build configuration (pt. 1)

Defines several aspects of the project:

- which version of the Kotlin compiler to adopt
    ```kotlin
    plugins {
        kotlin("multiplatform") version "1.9.10" // defines plugin and compiler version
    }
    ```

- which repositories should Gradle use when looking for dependencies
    ```kotlin
    repositories { 
        mavenCentral() // use MCR for downloading dependencies (recommneded)

        // other custom repositories here (discouraged)
    } 
    ```

- which platforms to target (reference [here](https://kotlinlang.org/docs/multiplatform-dsl-reference.html#targets))
    ```kotlin
    kotlin {
        // declares JVM as target
        jvm {
            withJava() // jvm-specific targets may include java sources
        }
        // declares JavaScript as target
        js {
            // the target will consist of a Node project (with NodeJS's stdlib)
            nodejs {
                runTask { /* configure project running in Node here */ }
                testRuns { /* configure Node's testing frameworks */ }
            }
            // alternatively, or additionally to nodejs:
            browser { /* ... */ } 
        }

        // other targets here
    }
    ```

---

## Kotlin multi-platform build configuration (pt. 2)

- other admissible targets: 
    * `android`
    * various native sub-targets (details [here](https://kotlinlang.org/docs/native-target-support.html))

- which third-party library should each target depend upon
    ```kotlin
    kotlin {
        sourceSets {
            val commonMain by getting {
                dependencies {
                    api(kotlin("stdlib-common"))
                    implementation("group.of", "multiplatform-library", "X.Y.Z") // or api
                }
            }
            val commonTest by getting {
                dependencies {
                    implementation(kotlin("test-common"))
                }
            }
            val jvmMain by getting {
                dependencies {
                    api(kotlin("stdlib-jdk8"))
                    implementation("group.of", "jvm-library", "X.Y.Z") // or api
                }
            }
            val jvmTest by getting {
                dependencies {
                    implementation(kotlin("test-junit"))
                }
            }
            val jsMain by getting {
                dependencies {
                    api(kotlin("stdlib-js"))
                    implementation(npm("npm-module", "X.Y.Z")) // lookup on https://www.npmjs.com
                }
            }
            val jsTest by getting {
                dependencies {
                    implementation(kotlin("test-js"))
                }
            }
        }
    }
    ```

---

## Kotlin multi-platform build configuration (pt. 3)

- configure Kotlin compiler options
    ```kotlin
    kotlin {
        sourceSets.all {
            languageSettings.apply {
                // provides source compatibility with the specified version of Kotlin.
                languageVersion = "1.8" // possible values: "1.4", "1.5", "1.6", "1.7", "1.8", "1.9"

                // allows using declarations only from the specified version of Kotlin bundled libraries.
                apiVersion = "1.8" // possible values: "1.3", "1.4", "1.5", "1.6", "1.7", "1.8", "1.9"

                // enables the specified language feature
                enableLanguageFeature("InlineClasses") // language feature name

                // allow using the specified opt-in
                optIn("kotlin.ExperimentalUnsignedTypes") // annotation FQ-name

                // enables/disable progressive mode
                progressiveMode = true // false by default
            }
        }
    }
    ```
    
- details about:
    * progressive mode [here](https://kotlinlang.org/docs/whatsnew13.html#progressive-mode)
    * opt-ins [here](https://kotlinlang.org/docs/opt-in-requirements.html#opt-in-to-using-api)

---

## Build configuration: full example


```kotlin
plugins {
    kotlin("multiplatform") version "1.9.10"
}

repositories { 
    mavenCentral() 
} 

kotlin {
    jvm {
        withJava()
    }

    js {
        nodejs {
            runTask { /* ... */ }
            testRuns { /* ... */ }
        }
        // alternatively, or additionally to nodejs:
        browser { /* ... */ } 
    }

    sourceSets {
        val commonMain by getting {
            dependencies {
                api(kotlin("stdlib-common"))
                implementation("group.of", "multiplatform-library", "X.Y.Z") // or api
            }
        }
        val commonTest by getting {
            dependencies {
                implementation(kotlin("test-common"))
            }
        }
        val jvmMain by getting {
            dependencies {
                api(kotlin("stdlib-jdk8"))
                implementation("group.of", "jvm-library", "X.Y.Z") // or api
            }
        }
        val jvmTest by getting {
            dependencies {
                implementation(kotlin("test-junit"))
            }
        }
        val jsMain by getting {
            dependencies {
                api(kotlin("stdlib-js"))
                implementation(npm("npm-module", "X.Y.Z")) // lookup on https://www.npmjs.com
            }
        }
        val jsTest by getting {
            dependencies {
                implementation(kotlin("test-js"))
            }
        }

        all {
            languageVersion = "1.8"
            apiVersion = "1.8"
            enableLanguageFeature("InlineClasses")
            optIn("kotlin.ExperimentalUnsignedTypes")
            progressiveMode = true // false by default
        }
    }
}
```

---

## Gradle tasks for multi-platform projects 

Let `T` denote the target name (e.g. `jvm`, `js`, etc.)

- `<T>MainClasses` compiles the main code for platform `T`
    + e.g. `jvmMainClasses`, `jsMainClasses`

- `<T>TestClasses` compiles the test code for platform `T`
    + e.g. `jvmTestClasses`, `jsTestClasses`

- `<T>Jar` compiles the main code for platform `T` and produces a JAR out of it
    + e.g. `jvmJar`, `jsJar`
    + depends on (hence implies) `<T>MainClasses`, but NOT `<T>TestClasses`

- `<T>Test` executes tests for platform `T`
    + e.g. `jvmTest`, `jsTest`
    + depends on (hence implies) `<T>MainClasses`, AND on `<T>TestClasses`

---

## Gradle tasks for multi-platform projects 

- `assemble` creates all JARs (hence compiling for main code for all platforms)
    + it also generates documentation and sources JAR
        * if publishing is configured

- `test` executes tests for all platforms

- `check` like `test` but it may also include other checks (e.g. style) if configured

- `build` $\approx$ `check` + `assemble`

---

## Multi-platform Kotlin sources

- Ordinary Kotlin sources

- When in `common`:
    + only the platform-agnostic std-lib can be used
        * API reference [here](https://kotlinlang.org/api/latest/jvm/stdlib/) (recall to disable all targets except `Common`)
    + one may use third-party libraries, as long as they are __multi-platform__ too
    + one may use the [`expect` keyword](https://kotlinlang.org/docs/multiplatform-connect-to-apis.html)
    + one may use platform-specific annotations 
        * e.g. `@JvmStatic`, `@JsName`, etc.

---

## Multi-platform Kotlin sources

Let `T` denote some target platform

- When in `T`-specific source sets
    + one may use the `T`-specific std-lib
    + one may use the `T`-specific Kotlin libraries
    + one may use the `T`-specific libraries
    + one may use the [`actual` keyword](https://kotlinlang.org/docs/multiplatform-connect-to-apis.html)

- Each platform `T` may allow for specific keywords
    * e.g. the [`external` modifier](https://kotlinlang.org/docs/js-interop.html#external-modifier)

---

## The `expect`/`actual` mechanism for specialising common API

- Declaring an `expect`ed function / type __declaration__ in common code...

- ... enforces the existence of a corresponding `actual` __definition__ for _all_ platforms
    * compiler won't succeed until an actual definition is provided _for each target_

{{< figure src="expect-actual.png" width="50%" >}}

---

## Workflow (top-down)

1. Draw a platform-agnostic design for your domain entities
    + keep in mind that you can only rely on a very small std-lib / runtime
    + keep in mind that some API may be missing in the common std-lib:
        - e.g. file system API 
    + try to reason in a platform-agnostic way
        - e.g. file system makes no sense for in-browser JS
    + separate interfaces from classes
    + rely on abstract classes with [template methods](https://en.wikipedia.org/wiki/Template_method_pattern) to maximise common code
    + use `expect` keyword to declare platform-agnostic factories

2. Assess the _abstraction gap_, __for each target platform__

3. For each target platform:
    + draw a platform-specific design extending the platform-agnostic one
    + ... and filling the abstraction gap for that target
    + implement interfaces via platform-specific classes
    + implement platform-specific concrete classes for common abstract classes
    + use `actual` keyword to implement platform-specific factories

---

{{% section %}}

# Running example

## Multi-platform CSV parsing lib

- CSV (comma-separated values) files, e.g.:
    ```bash
    # HEADER_1, HEADER_2, HEADER_3, ...
    # character '#' denotes the beginning of a single-line comment
    # first line conventionally denotes columns names (headers)

    field1, filed2, field3, ...
    # character ',' acts as field separator
    
    "field with spaces", "another field, with comma", "yet another field", ...
    # character '"' acts as field delimiter

    # other characters may be used to denote comments, separators, or delimiters
    ```

- JVM and JS std-lib do not provide direct support for CSV

- Requirements:
    * library for parsing / manipulating CSV files
    * parsing = loading from file
    * manipulating = creating / editing / saving CSV files, programmatically
    * support for both JVM and JS

---

## Domain entities

{{< gravizo >}}
@startuml
top to bottom direction

package "kotlin" {
    interface Iterable<T>
}

package "io.github.gciatto.csv" {
    interface Row {
        + get(index: Int): String
        + size: Int
    }

    Row -up-|> Iterable: //T// = String

    interface Header {
        + columns: List<String>
        + contains(column: String): Boolean
        + indexOf(column: String): Int
    }

    Row <|-- Header

    interface Record {
        + header: Header
        + values: List<String>
        + contains(value: String): Boolean
        + get(column: String): String
    }

    Row <|-- Record

    Record "1" *-right- "*" Header

    interface Table {
        + header: Header
        + records: List<Record>
        + get(row: Int): Row
        + size: Int
    }

    Table "1" *-u- "*" Header
    Table "*" o-u- "*" Record
    Table -u----|> Iterable: //T// = Row

    class Configuration {
        + separator: Char
        + delimiter: Char
        + comment: Char
    }
}
@enduml
{{< /gravizo >}}

---

## The `Row` interface

```kotlin
// Represents a single row in a CSV file.
interface Row : Iterable<String> {

    // Gets the value of the field at the given index.
    operator fun get(index: Int): String

    // Gets the number of fields in this row.
    val size: Int
}
```

--- 

## The `Header` interface

```kotlin
// Represents the header of a CSV file.
// A header is a special row containing the names of the columns.
interface Header : Row {

    // Gets the names of the columns.
    val columns: List<String>

    // Checks whether the given column name is present in this header.
    operator fun contains(column: String): Boolean

    // Gets the index of the given column name.
    fun indexOf(column: String): Int
}
```

---

## The `Record` interface

```kotlin
// Represents a record in a CSV file.
interface Record : Row {

    // Gets the header of this record (useful to know column names).
    val header: Header

    // Gets the values of the fields in this record.
    val values: List<String>

    // Checks whether the given value is present in this record.
    operator fun contains(value: String): Boolean

    // Gets the value of the field in the given column.
    operator fun get(column: String): String
}
```

---

## The `Table` interface

```kotlin
// Represents a table (i.e. an in-memory representation of a CSV file).
interface Table : Iterable<Row> {

    // Gets the header of this table (useful to know column names).
    val header: Header

    // Gets the records in this table.
    val records: List<Record>

    // Gets the row at the given index.
    operator fun get(row: Int): Row

    // Gets the number of rows in this table.
    val size: Int
}
```

---

## Common implementations of domain entities

{{< gravizo >}}
@startuml
top to bottom direction

package "io.github.gciatto.csv.**impl**" {
    interface Row

    class AbstractRow {
        # constructor(values: List<String>)
        # toString(type: String?): String
    }

    Row <|-- AbstractRow

    interface Header

    class DefaultHeader {
        - indexesByName: Map<String, Int>
        + constructor(columns: Iterable<String>)
    }

    Row <|-- Header
    Header <|-- DefaultHeader
    AbstractRow <|-- DefaultHeader

    interface Record

    class DefaultRecord {
        # constructor(header: Header, values: Iterable<String>)
    }

    Row <|-- Record
    Record <|-- DefaultRecord
    AbstractRow <|-- DefaultRecord

    class DefaultTable {
        # constructor(header: Header, records: Iterable<Record>)
    }

    Table <|-- DefaultTable
}
@enduml
{{< /gravizo >}}

{{% /section %}}