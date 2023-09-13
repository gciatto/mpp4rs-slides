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
            useCommonJs() // use CommonJS for JS depenencies management
            // or useEsModules() 
            binaries.executable() // enables tasks for Node packages generation
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

## Gradle tasks for multi-platform projects (pt. 1)

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

## Gradle tasks for multi-platform projects (pt. 2)

- `compileProductionExecutableKotlinJs` compiles the JS main code into a Node project
    * requires the `js` target to be enabled
    * requires the `binaries.executable()` configuration to be enabled

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

## About the example

- We'll follow a domain-driven approach:
    1. focus on designing / implementing the domain entities
    2. then, we'll add functionalities for manipulating them
    3. finally, we'll write unit and integration tests

- Domain entities (can be realised as common code):
    - `Table`: in-memory representation of a CSV file
    - `Row`: generic row in a CSV file
    - `Header`: special row containing the names of the columns
    - `Record`: special row containing the values of a line in a CSV file

- Main functionalities (require platform specific code):
    1. creating a `Table` programmatically
    2. reading columns, rows, and values from a `Table`
    3. parsing a CSV file into a `Table`
    4. saving a `Table` into a CSV file

---

## Domain entities

{{< plantuml >}}
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
}
{{< /plantuml >}}

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
skinparam monochrome false
top to bottom direction

package "io.github.gciatto.csv.**impl**" {
    interface Row

    abstract class AbstractRow {
        - values: List<String>
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
        + constructor(header: Header, values: Iterable<String>)
    }

    Row <|-- Record
    Record <|-- DefaultRecord
    AbstractRow <|-- DefaultRecord

    class DefaultTable {
        + constructor(header: Header, records: Iterable<Record>)
    }

    Table <|-- DefaultTable
}
@enduml
{{< /gravizo >}}

---

## The `AbstractRow` class

```kotlin
// Base implementation for the Row interface.
internal abstract class AbstractRow(protected open val values: List<String>) : Row {
    override val size: Int
        get() = values.size

    override fun get(index: Int): String = values[index]

    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (other == null || this::class != other::class) return false
        return values == (other as AbstractRow).values
    }

    override fun hashCode(): Int = values.hashCode()

    // Returns a string representation of this row as "<type>(field1, field2, ...)".
    protected fun toString(type: String?): String {
        var prefix = ""
        var suffix = ""
        if (type != null) {
            prefix = "$type("
            suffix = ")"
        }
        return values.joinToString(", ", prefix, suffix) { "\"$it\"" }
    }

    // Returns a string representation of this row as "Row(field1, field2, ...)".
    override fun toString(): String = toString("Row")

    // Makes it possible to iterate over the fields of this row, via for-each loops.
    override fun iterator(): Iterator<String> = values.iterator()
}
```

---

## The `DefaultHeader` class

```kotlin
// Default implementation for the Header interface.
internal class DefaultHeader(columns: Iterable<String>) : Header, AbstractRow(columns.toList()) {

    // Cache of column indexes, for faster lookup.
    private val indexesByName = columns.mapIndexed { index, name -> name to index }.toMap()

    override val columns: List<String>
        get() = values

    override fun contains(column: String): Boolean = column in indexesByName.keys

    override fun indexOf(column: String): Int = indexesByName[column] ?: -1

    override fun iterator(): Iterator<String> = values.iterator()

    override fun toString(): String = toString("Header")
}
```

---

## The `DefaultRecord` class

```kotlin
internal class DefaultRecord(override val header: Header, values: Iterable<String>) : Record, AbstractRow(values.toList()) {

    init {
        require(header.size == super.values.size) {
            "Inconsistent amount of values (${super.values.size}) w.r.t. to header size (${header.size})"
        }
    }

    override fun contains(value: String): Boolean = values.contains(value)

    override val values: List<String>
        get() = super.values

    override fun get(column: String): String =
        header.indexOf(column).takeIf { it in 0..< size }?.let { values[it] }
            ?: throw NoSuchElementException("No such column: $column")

    override fun toString(): String = toString("Record")
}
```

---

## The `DefaultTable` class

```kotlin
internal class DefaultTable(override val header: Header, records: Iterable<Record>) : Table {

    // Lazy, defensive copy of the records.
    override val records: List<Record> by lazy { records.toList() }

    override fun get(row: Int): Row = if (row == 0) header else records[row - 1]

    override val size: Int
        get() = records.size + 1

    override fun iterator(): Iterator<Row> = (sequenceOf(header) + records.asSequence()).iterator()
    
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (other == null || other !is DefaultTable) return false
        if (header != other.header) return false
        if (records != other.records) return false
        return true
    }

    override fun hashCode(): Int {
        var result = header.hashCode()
        result = 31 * result + records.hashCode()
        return result
    }

    override fun toString(): String = this.joinToString(", ", "Table(", ")")
}
```

---

## Need for factory methods

- To enforce separation among API and implementation code, it's better:
    * to keep interfaces public, and classes internal
    * to provide factory methods for creating instances of the interfaces

- Convention in Kotlin is to create factory methods as package-level `fun`ctions
    * e.g. contained into the `io/github/gciatto/csv/Csv.kt` file

- Factory methods may be named after the concept they create: `<concept>Of(args...)`
    * e.g. `headerOf(columns...)`, `recordOf(header, values...)`, etc.

- Class diagram:
    
    {{< gravizo width="30%">}}
    @startuml
    skinparam monochrome false
    class Csv{
        + headerOf(columns): Header
        + anonymousHeader(size: Int): Header
        ..
        + recordOf(header, values): Record
        ..
        + tableOf(header, records): Table
        + tableOf(rows): Table
    }
    @enduml
    {{< /gravizo >}}

---

## The `Csv.kt` file

```kotlin
// Headers creation from columns names
fun headerOf(columns: Iterable<String>): Header = DefaultHeader(columns)
fun headerOf(vararg columns: String): Header = headerOf(columns.asIterable())

// Creates anonymous headers, with columns named after their index
fun anonymousHeader(size: Int): Header = headerOf((0 ..< size).map { it.toString() })

// Records creation from header and values
fun recordOf(header: Header, columns: Iterable<String>): Record = DefaultRecord(header, columns)
fun recordOf(header: Header, vararg columns: String): Record = recordOf(header, columns.asIterable())

// Tables creation from header and records
fun tableOf(header: Header, records: Iterable<Record>): Table = DefaultTable(header, records)
fun tableOf(header: Header, vararg records: Record): Table = tableOf(header, records.asIterable())

// Tables creation from rows (anonymous header if none is provided)
fun tableOf(rows: Iterable<Row>): Table {
    val records = mutableListOf<Record>()
    var header: Header? = null
    for (row in rows) {
        when (row) {
            is Header -> header = row
            is Record -> records.add(row)
        }
    }
    require(header != null || records.isNotEmpty())
    return tableOf(header ?: anonymousHeader(records[0].size), records)
}
```

- Notice that each factory method is __overloaded__
    + to support both `Iterable` and `vararg` arguments
    + this is convenient for Kotlin programmers that will use our library

---

## Time for unit testing

- Dummy instances object:
    ```kotlin
    object Tables {
        val irisShortHeader = headerOf("sepal_length", "sepal_width", "petal_length", "petal_width", "class")

        val irisLongHeader = headerOf("sepal length in cm", "sepal width, in cm", "petal length", "petal width", "class")

        fun iris(header: Header): Table = tableOf(
            header,
            recordOf(header, "5.1", "3.5", "1.4", "0.2", "Iris-setosa"),
            recordOf(header, "4.9", "3.0", "1.4", "0.2", "Iris-setosa"),
            recordOf(header, "4.7", "3.2", "1.3", "0.2", "Iris-setosa")
        )
    }
    ```

- Some basic tests in file `TestCSV.kt`, e.g. (bad way of writing test methods, don't do this at home)
    ```kotlin
    @Test
    fun recordBasics() {
        val record = Tables.iris(Tables.irisShortHeader).records[0]
        assertEquals("5.1", record[0])
        assertEquals("5.1", record["sepal_length"])
        assertEquals("0.2", record[3])
        assertEquals("0.2", record["petal_width"])
        assertEquals("Iris-setosa", record[4])
        assertEquals("Iris-setosa", record["class"])
        assertFailsWith<IndexOutOfBoundsException> { record[5] }
        assertFailsWith<NoSuchElementException> { record["missing"] }
    }
    ```
    
- Run tests via Gradle task `test` (also try to run tests for specific platforms, e.g. `jvmTest` or `jsTest`)

--- 

## Summary of what we did so far

1. We designed a set of domain entities
    * `Row`, `Header`, `Record`, `Table`
    * we provided a set of factory methods for creating instances of those entities

2. We implemented the domain entities as common code

3. We wrote some unit tests for the domain entities
    * again as common code

4. Let's focus now on more complex functionalities
    * e.g. parsing a CSV file into a `Table`
    * e.g. saving a `Table` into a CSV file
    * all such features require I/O functionalities

> I/O is platform-specific, hence we need platform-specific code

---

## How much platform-specific code is needed?

- I/O functionalities are supported by fairly different API in JVM and JS
    * e.g. JVM's [`java.io` package](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/package-summary.html) vs. JS' [`fs` module](https://nodejs.org/docs/latest-v20.x/api/fs.html)
    * sadly, Kotlin std-lib does not provide a common API for I/O

- Do we need to rewrite the same business logic twice? (once for JVM, once for JS)
    * yes, but we can minimise the amount of code to be rewritten

- Let's try to decompose the problem as follows:
    * parsing CSV into `Table`: file $\rightarrow$ string(s) $\rightarrow$ `Table`
    * saving `Table` as CSV file: `Table` $\rightarrow$ string(s) $\rightarrow$ file

- Remarks:
    1. only the "file $\leftrightarrow$ string(s)" part is platform-specific
    2. conversely, the "string(s) $\leftrightarrow$ `Table`" part is platform-agnostic
        + let's realise this one first, as common code

---

## Three more entities to be modelled

- `Configuration`: set of characters to be used to parse / represent CSV files
    * e.g. separator, delimiter, comment, etc.

- `Formatter`: converts `Rows` into strings, according to some `Configuration`

- `Parser`: converts some source into a `Table`, according to some `Configuration`
    * source $\approx$ anything that can be interpreted as a string to be parsed (e.g. file, string, etc.)

{{< gravizo width="40%">}}
@startuml
skinparam monochrome false

package io.github.gciatto.csv {

    class Configuration {
        + separator: Char
        + delimiter: Char
        + comment: Char
        --
        + isComment(string: String): Boolean
        + isRecord(string: String): Boolean
        + isHeader(string: String): Boolean
        ..
        + getComment(string: String): String?
        + getFields(string: String): List<String>
        + getColumnNames(string: String): List<String>
    }

    interface Formatter {
        + source: Iterable<Row>
        + configuration: Configuration
        + format(): Iterable<String>
    }

    interface Parser {
        + source: Any
        + configuration: Configuration
        + parse(): Iterable<Row>
    }

    Formatter "1" o-- "1" Configuration
    Parser "1" o-- "1" Configuration

}
@enduml
{{< /gravizo >}}

---

## The `Configuration` class

```kotlin
data class Configuration(val separator: Char, val delimiter: Char, val comment: Char) {

    // Checks whether the given string is a comment (i.e. starts with the comment character).
    fun isComment(string: String): Boolean = // ...

    // Gets the content of the comment line (i.e. removes the initial comment character).
    fun getComment(string: String): String? = // ...

    // Checks whether the given string is a record (i.e. it contains the delimiter character)
    fun isRecord(string: String): Boolean = // ...

    // Retrieves the fields in the given record (i.e. splits the string at the separator character)
    fun getFields(string: String): List<String> = // ...

    // Checks whether the given string is a header (i.e. simultaneously a record and a comment)
    fun isHeader(string: String): Boolean = // ...

    // Retrieves the column names in the given header
    fun getColumnNames(string: String): List<String> = // ...
}
```
* implementation of methods is long and boring, but straightforward
    - it relies on regular expressions, which are supported by Kotlin's common std-lib
    - details [here](https://github.com/gciatto/mpp4rs-code)

---

## The `Formatter` interface

```kotlin
interface Formatter {
    
    // The source of this formatter (i.e. the rows to be formatted).
    val source: Iterable<Row>

    // The configuration of this formatter (i.e. the characters to be used).
    val configuration: Configuration

    // Formats the source of this formatter into a sequence of strings (one per each row in the source)
    fun format(): Iterable<String>
}
```

---

## The `Parser` interface

```kotlin
interface Parser {
    // The source to be parsed by this parser (must be interpretable as string)
    val source: Any

    // The configuration of this parser (i.e. the characters to be used).
    val configuration: Configuration

    // Parses the source of this parser into a sequence of rows (one per each row in the source)
    fun parse(): Iterable<Row>
}
```

--- 

## Common implementation of novel entities

{{< gravizo >}}
@startuml
skinparam monochrome false

package io.github.gciatto.csv.impl {

    class Configuration
    interface Formatter
    interface Parser

    DefaultFormatter "1" o-- "1" Configuration
    AbstractParser "1" o-- "1" Configuration

    class DefaultFormatter {
        + constructor(source: Iterable<Row>, configuration: Configuration)
        - formatRow(row: Row): String
        - formatAsHeader(row: Row): String
        - formatAsRecord(row: Row): String
    }

    Formatter <|-- DefaultFormatter

    abstract class AbstractParser {
        # constructor(source: Any, configuration: Configuration)
        # beforeParsing()
        # afterParsing()
        # {abstract} sourceAsLines(): Sequence<String>
    }

    Parser <|-- AbstractParser

    class StringParser {
        + source: String
        + constructor(source: String, configuration: Configuration)
        + sourceAsLines(): Sequence<String>
    }

    AbstractParser <|-- StringParser
}
@enduml
{{< /gravizo >}}

---

## The `DefaultFormatter` class

```kotlin
class DefaultFormatter(override val source: Iterable<Row>, override val configuration: Configuration) : Formatter {

    // Lazily converts each row from the source into a string, according to the configuration.
    override fun format(): Iterable<String> = source.asSequence().map(this::formatRow).asIterable()

    // Converts the given row into a string, according to the configuration.
    private fun formatRow(row: Row): String = when (row) {
        is Header -> formatAsHeader(row)
        else -> formatAsRecord(row)
    }

    // Formats the given row as a header (putting the comment character at the beginning).
    private fun formatAsHeader(row: Row): String = "${configuration.comment} ${formatAsRecord(row)}"

    // Formats the given row as a record (using the separator and delimiter characters accordingly).
    private fun formatAsRecord(row: Row): String =
        row.joinToString("${configuration.separator} ") {
            val delimiter = configuration.delimiter
            "$delimiter$it$delimiter"
        }
}
```

---

## The `AbstractParser` class

```kotlin
class AbstractParser(override val source: Any, override val configuration: Configuration) : Parser {

    // Empty methods to be overridden by sub-classes to initialize/finalise parsing.
    protected open fun beforeParsing() { /* does nothing by default */ }
    protected open fun afterParsing() { /* does nothing by default */ }

    // Template method that parses the source into a sequence of strings (one per line).
    protected abstract fun sourceAsLines(): Sequence<String>

    // Parses the source into a sequence of rows (skipping comments, looking for at most 1 header).
    override fun parse(): Iterable<Row> = sequence {
        beforeParsing()
        var header: Header? = null
        var i = 0
        for (line in sourceAsLines()) {
            if (line.isBlank()) {
                continue
            } else if (configuration.isHeader(line)) {
                if (header == null) {
                    header = headerOf(configuration.getColumnNames(line))
                    yield(header)
                }
            } else if (configuration.isComment(line)) {
                continue
            } else if (configuration.isRecord(line)) {
                val fields = configuration.getFields(line)
                if (header == null) {
                    header = anonymousHeader(fields.size)
                    yield(header)
                }
                try {
                    yield(recordOf(header, fields))
                } catch (e: IllegalArgumentException) {
                    throw IllegalStateException("Invalid CSV at line $i: $line", e)
                }
            } else {
                error("Invalid CSV line at $i: $line")
            }
            i++
        }
        afterParsing()
    }.asIterable()
}
```

---

## The `StringParser` class

```kotlin
class StringParser(override val source: String, configuration: Configuration) 
    : AbstractParser(source, configuration) {

    // Splits the source string into lines.
    override fun sourceAsLines(): Sequence<String> = source.lineSequence()
}
```

--- 

## Providing functionalities via extension methods

- Additions to the `Csv.kt` file:

    ```kotlin
    const val DEFAULT_SEPARATOR = ','
    const val DEFAULT_DELIMITER = '"'
    const val DEFAULT_COMMENT = '#'

    // Converts the current container of rows into a CSV string, using the given characters.
    fun Iterable<Row>.formatAsCSV(
        separator: Char = DEFAULT_SEPARATOR,
        delimiter: Char = DEFAULT_DELIMITER,
        comment: Char = DEFAULT_COMMENT
    ): String = DefaultFormatter(this, Configuration(separator, delimiter, comment)).format().joinToString("\n")

    // Parses the current CSV string into a table, using the given characters.
    fun String.parseAsCSV(
        separator: Char = DEFAULT_SEPARATOR,
        delimiter: Char = DEFAULT_DELIMITER,
        comment: Char = DEFAULT_COMMENT
    ): Table = StringParser(this, Configuration(separator, delimiter, comment)).parse().let(::tableOf)
    ```

---

## Unit tests and usage examples

- Dummy instances for testing:
    ```kotlin
    object CsvStrings {
        val iris: String = """
        |# sepal_length, sepal_width, petal_length, petal_width, class
        |5.1,3.5,1.4,0.2,Iris-setosa
        |4.9,3.0,1.4,0.2,Iris-setosa
        |4.7,3.2,1.3,0.2,Iris-setosa
        """.trimMargin()

        val irisWellFormatted: String = """
        |# "sepal_length", "sepal_width", "petal_length", "petal_width", "class"
        |"5.1", "3.5", "1.4", "0.2", "Iris-setosa"
        |"4.9", "3.0", "1.4", "0.2", "Iris-setosa"
        |"4.7", "3.2", "1.3", "0.2", "Iris-setosa"
        """.trimMargin()

        // other dummy constants here
    }
    ```

- Tests involving parsing be like:
    ```kotlin
    @Test
    fun parsingFromCleanString() {
        val parsed: Table = CsvStrings.iris.parseAsCSV()
        assertEquals(
            expected = Tables.iris(Tables.irisShortHeader),
            actual = parsed
        )
    }
    ```

- Tests involving formatting be like:
    ```kotlin
    @Test
    fun formattingToString() {
        val iris: Table = Tables.iris(Tables.irisShortHeader)
        assertEquals(
            expected = CsvStrings.irisWellFormatted,
            actual = iris.formatAsCSV()
        )
    }
    ```

--- 

## Time to go platform-specific

- Further additions to the `Csv.kt` file:
    ```kotlin
    // Reads and parses a CSV file from the given path, using the given characters.
    expect fun parseCsvFile(
        path: String,
        separator: Char = DEFAULT_SEPARATOR,
        delimiter: Char = DEFAULT_DELIMITER,
        comment: Char = DEFAULT_COMMENT
    ): Table
    ```

    * notice the `expect` keyword, and the the lack of function body
    * we're just declaring the signature of a platform-specific function
    * there is no type for representing paths is Kotlin's common std-lib
        + hence we're using `String` as a platform-agnostic representation of paths
            - this is suboptimal choice

--- 

## Time to go platform-specific on the JVM (pt. 1)

- I/O (over textual files) is mainly supported by means of the following classes:
    * [`java.io.File`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/File.html)
    * [`java.io.BufferedReader`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/BufferedReader.html)
    * [`java.io.BufferedWriter`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/BufferedWriter.html)

- Buffered readers support reading a file __line-by-line__

- On Kotlin/JVM, Java's std-lib is available as Kotlin's std-lib
    * hence, we may use Java's std-lib directly
    * the abstraction gap is close to 0

- We may simply create a new sub-type of `AbstractParser`
    * using `File`s as sources
    * creating a `BufferedReader` behind the scenes to read files line-by-line

--- 

## Time to go platform-specific on the JVM (pt. 2)

In the `jvmMain` source set

- let's define the following JVM-specific class:
    ```kotlin
    class FileParser(
        override val source: File, // source is now forced to be a File
        configuration: Configuration
    ) : AbstractParser(source, configuration) {

        // Lately initialised reader, corresponding to source
        private lateinit var reader: BufferedReader

        // Opens the source file, hence initialising the reader, before each parsing
        override fun beforeParsing() { reader = source.bufferedReader() }

        // Closes the reader, after each parsing
        override fun afterParsing() { reader.close() }

        // Lazily reads the source file line-by-line
        override fun sourceAsLines(): Sequence<String> = reader.lines().asSequence()
    }
    ```

- let's create the `Csv.jvm.kt` file containing:
    ```kotlin
    actual fun parseCsvFile(
        path: String,
        separator: Char,
        delimiter: Char,
        comment: Char
    ): Table = FileParser(File(path), Configuration(separator, delimiter, comment)).parse().let(::tableOf)
    ```

    * this is how the `parseCsvFile` is implemented __on the JVM__
    * notice the `actual` keyword, and the presence of a function body
    * notice the usage of `FileParser` in the function body
        + class `FileParser` is internal class for filling the abstraction gap on the JVM

--- 

## Time to go platform-specific on the JS (pt. 1)

- I/O (over textual files) is mainly supported by means of the following things:
    * the [`fs` module](https://nodejs.org/docs/latest-v20.x/api/fs.html)
    * the [`fs.readFileSync` function](https://nodejs.org/api/fs.html#fsreadfilesyncpath-options)
    * the [`fs.writeFileSync` function](https://nodejs.org/api/fs.html#fswritefilesyncfile-data-options)

- These function supports reading / writing a file __in one shot__
    * i.e. they return / generate the _whole_ content of the file as a string
    * quite inefficient if the file is big

- On Kotlin/JS, Node's std-lib is __not__ directly available
    * one must instruct the compiler about
        1. __where__ to look up for std-lib function (module name)
        2. __how__ to map JS functions to Kotlin functions (`external` declarations)
    * the abstraction gap is non-negligible

- To-do list for Kotlin/JS:
    1. create `external` declarations for the Node's std-lib functions to be used
    2. implement JS-specific code via the `StringParser` (after reading the whole file)

--- 

## Time to go platform-specific on the JS (pt. 2)

In the `jsMain` source set

1. let's create the `NodeFs.kt` file (containing `external` declarations for the `fs` module):
    ```kotlin
    @file:JsModule("fs")
    @file:JsNonModule

    package io.github.gciatto.csv

    // Kotlin mapping for: https://nodejs.org/api/fs.html#fsreadfilesyncpath-options
    external fun readFileSync(path: String, options: dynamic = definedExternally): String

    // Kotlin mapping for: https://nodejs.org/api/fs.html#fswritefilesyncfile-data-options
    external fun writeFileSync(file: String, data: String, options: dynamic = definedExternally)
    ```

    * the `@JsModule` annotation instructs the compiler about where to look up for the `fs` module
    * `external` declarations are Kotlin signatures of JS function
    * `dynamic` is a special type that can be used to represent any JS object
        + it overrides Kotlin's type system, hence it should be used with care
        + it prevents developers from declaring too much `external` stuff
        + good to use when the JS API is not known in advance or uses union types
    * `definedExternally` is stating that a parameter is optional (default value is defined in JS)

--- 

## Time to go platform-specific on the JS (pt. 3)

2. let's create the `Csv.js.kt` file containing:
    ```kotlin
    actual fun parseCsvFile(
        path: String,
        separator: Char,
        delimiter: Char,
        comment: Char
    ): Table = readFileSync(path, js("{encoding: 'utf8'}")).parseAsCSV(separator, delimiter, comment)
    ```

    * this is how the `parseCsvFile` is implemented __on JS__
    * notice the `actual` keyword, and the presence of a function body
    * notice the usage of `readFileSync` to read a file as a string in one shot
    * notice the exploitation of common code for parsing the string into a `Table` (namely, `parseAsCSV`)
    * notice the `js("...")` magic function
        + it allows to write JS code directly in Kotlin
        + the provided string should contain bare JS code, the compiler will output "as-is"
        + it's a way to fill the abstraction gap on JS
        + we use it to create a JS object on the fly, to provide optional parameters to `readFileSync`

---

## Platform-agnostic testing (strategy)

- We may test our CSV library in common-code
    + so as to avoid repeating testing code

- The test suite may:   
    1. create temporary files with known paths, containing known CSV data
    2. read those paths, and parse them as CSV
    3. assert that the parsed tables are as expected

- The hard part is __step 1__: two platform-specific steps
    1. discovering the temp directory of the current system
    2. writing a file

--- 

## Multi-platform testing (preliminaries)

- file `Utils.kt` in `commonTest`:
    ```kotlin
    // Creates a temporary file with the given name, extension, and content, and returns its path.
    expect fun createTempFile(name: String, extension: String, content: String): String
    ```

    notice the `expect`ed function

- file `Utils.jvm.kt` in `jvmTest`:
    ```kotlin
    import java.io.File

    actual fun createTempFile(name: String, extension: String, content: String): String
        val file = File.createTempFile(name, extension)
        file.writeText(content)
        return file.absolutePath
    }
    ```
    + cf. documentation of method [`File.createTempFile`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/File.html#createTempFile(java.lang.String,java.lang.String)) 

- file `Utils.js.kt` in `jsTest`:
    ```kotlin
    private val Math: dynamic by lazy { js("Math") }

    @JsModule("os") @JsNonModule
    external fun tmpdir(): String

    actual fun createTempFile(name: String, extension: String, content: String): String {
        val tag = Math.random().toString().replace(".", "")
        val path = "$tmpDirectory/$name-$tag.$extension"
        writeFileSync(path, content)
        return path
    }
    ```
    + cf. documentation of [`os.tmpdir`](https://nodejs.org/docs/latest-v20.x/api/os.html#ostmpdir) and [`Math`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math) 

---

## Multi-platform testing (actual tests)

- file `CsvFiles.kt` in `commonMain`:
    ```kotlin
    object CsvFiles {
        // Path of the temporary file containing the string CsvStrings.iris
        // (file lazily created upon first usage).
        val iris: String by lazy { createTempFile("iris.csv", CsvStrings.iris) }

        // Path of the temporary file containing the string CsvStrings.irisWellFormatted
        // (file lazily created upon first usage).
        val irisWellFormatted: String by lazy {
            createTempFile("irisWellFormatted.csv", CsvStrings.irisWellFormatted)
        }

        // other paths here, corresponding to other constants in CsvStrings
    }
    ```

- tests for parsing be like:
    ```kotlin
    @Test
    fun testParseIris() {
        val parsedFromString = CsvStrings.iris.parseAsCSV()
        val readFromFile = parseCsvFile(CsvFiles.iris)
        assertEquals(parsedFromString, readFromFile)
    }
    ```

{{% /section %}}

---

{{% section %}}

## Output of a multi-platform build

- Kotlin multi-platform projects can be assembled as JARs
    * enabling importing the project as dependency in other multi-platform projects

- The `jvmMain` source set is compiled into a JVM-compliant JARs
    * enabling importing the project as dependency in JVM projects
    * enabling the creation of runnable JARs
    * via the various `*Jar` or `assemble` tasks

- The `jsMain` source set is compiled into either
    * a Kotlin library (`.klib`), enabling imporing the project tas dependency in Kotlin/JS projects
        + via the various `*Jar` or `assemble` tasks
    * a NodeJS project
        + via the `compileProductionExecutableKotlinJs` task

--- 

## Output of JVM compilation

- Task for JVM-only compilation of re-usable packages: `jvmJar`

- Effect: 
    1. compile the project as JVM bytecode 
    2. pack the bytecode into a JAR file
    3. move the JAR file into `$PROJ_DIR/build/libs/$PROJ_NAME-jvm-$PROJ_VERSION`

- The JAR does not contain dependencies

- Ad-hoc Gradle plugins/code is needed for creating [fat Jar](https://www.baeldung.com/gradle-fat-jar)

--- 

## Output of JS compilation

- Task for JS-only compilation of re-usable packages: `compileProductionExecutableKotlinJs`

- Effect: 
    1. Generate Node project into `$ROOT_PROJ_DIR/build/js/packages/$ROOT_PROJ_NAME-$PROJ_NAME`:
        - JS code
        - `package.json` file
    2. [Dead code elimination](https://kotlinlang.org/docs/javascript-dce.html) (DCE) removing unused code
        - i.e. code not (indirectly) called by main

- Support for packing as NPM package via the [`npm-publish`](https://npm-publish.petuska.dev/3.4/) Gradle plugin

{{% /section %}}

---

{{% section %}}

## Calling Kotlin from other platforms

- Kotlin code can be called from the target platforms' main languages
    + e.g. Java, JavaScript, etc.

- Understading the mapping among Kotlin and other languages is key
    + it impacts the usability of Kotlin libraries for ordinary platform users

> How are Kotlin's syntactical categories mapped to other platforms/languages?

+ the answer varies on a per-platform basis

---

## Kotlin--Java Mapping (pt. 1)

- Kotlin class $\equiv$ Java class

{{% multicol %}}
{{% col %}}
```kotlin
class MyClass {}
interface MyType {}
```
{{% /col %}}
{{% col %}}
```java
public class MyClass {}
public interface MyType {}
```
{{% /col %}}
{{% /multicol %}}

- No syntactical difference among primitive and reference types
    + `Int` $\leftrightarrow$ `int` / `Integer`, `Short` $\leftrightarrow$ `short` / `Short`, etc.

{{% multicol %}}
{{% col %}}
```kotlin
val x: Int = 0
val y: Int? = null
```
{{% /col %}}
{{% col %}}
```java
int x = 0;
Integer y = null;
```
{{% /col %}}
{{% /multicol %}}

- Some Kotlin types are replaced by Java types at compile time
    + e.g. `Any` $\rightarrow$ `Object`, `kotlin.collections.List` $\rightarrow$ `java.util.List`, etc.

{{% multicol %}}
{{% col %}}
```kotlin
val x: Any = "ciao"
val y: kotlin.collections.List<Int> = listOf(1, 2, 3)
val z: kotlin.collections.MutableList<String> = mutableListOf("a", "b", "c")
```
{{% /col %}}
{{% col %}}
```java
Object x = "ciao";
java.util.List<Integer> y = java.util.Arrays.asList(1, 2, 3);
java.util.List<String> z = java.util.List.of("a", "b", "c");
```
{{% /col %}}
{{% /multicol %}}

- Other Kotlin types are mapped to homonymous Java types

---

## Kotlin--Java Mapping (pt. 2)

- Kotlin properties are mapped to getter / setter methods

{{% multicol %}}
{{% col %}}
```kotlin
interface MyType {
    val x: Int
    var y: String
}
```
{{% /col %}}
{{% col %}}
```java
public interface MyType {
    int getX();
    String getY(); void setY(String y);
}
```
{{% /col %}}
{{% /multicol %}}

- ... unless the `JvmField` annotation is adopted

{{% multicol %}}
{{% col %}}
```kotlin
import kotlin.jvm.JvmField
class MyType {
    @JvmField
    val x: Int = 0
    @JvmField
    var y: String = ""
}
```
{{% /col %}}
{{% col %}}
```java
public class MyType {
    public final int x = 0;
    public String y = "";
}
```
{{% /col %}}
{{% /multicol %}}

- Kotlin's package functions in file `X.kt` are mapped to static methods of class `XKt`

{{% multicol %}}
{{% col %}}
```kotlin
// file MyFile.kt
fun f() {}
```
{{% /col %}}
{{% col %}}
```java
public static class MyFileKt {
    public static void f() {}
}
```
{{% /col %}}
{{% /multicol %}}

- ... unless the `JvmName` annotation is exploited

{{% multicol %}}
{{% col %}}
```kotlin
@file:JvmName("MyModule")
import kotlin.jvm.JvmName
fun f() {}
```
{{% /col %}}
{{% col %}}
```java
public class MyModule {
    public static void f() {}
}
```
{{% /col %}}
{{% /multicol %}}

---

## Kotlin--Java Mapping (pt. 3)

- Kotlin's `object X` is mapped to Java class `X` with 
    * private constructor
    * public static final field named `INSTANCE` to access

{{% multicol %}}
{{% col %}}
```kotlin
object MySingleton {}
```
{{% /col %}}
{{% col %}}
```java
public static class MySingleton {
    private MySingleton() {}
    public static final MySingleton INSTANCE = new MySingleton();
}
```
{{% /col %}}
{{% /multicol %}}

- Class `X`'s companion object is mapped to public static final field named `Companion` on class `X`

{{% multicol %}}
{{% col %}}
```kotlin
class MyType {
    private constructor()
    companion object {
        fun of(): MyType = MyType()
    }
}

// usage:
val x: MyType = MyType.of()
```
{{% /col %}}
{{% col %}}
```java
public class MyType {
    private MyType() {}
    public static final class Companion {
        public MyType of() { return new MyType(); }
    }
    public static final Companion Companion = new Companion();
}

// usage:
MyType x = MyType.Companion.of();
```
{{% /col %}}
{{% /multicol %}}

---

## Kotlin--Java Mapping (pt. 4)

- Class `X`'s companion object's member `M` tagged with `@JvmStatic` is mapped to static member `M` on class `X`

{{% multicol %}}
{{% col %}}
```kotlin
import kotlin.jvm.JvmStatic
class MyType {
    private constructor()
    companion object {
        @JvmStatic
        fun of(): MyType = MyType()
    }
}

// usage:
val x: MyType = MyType.of()
```
{{% /col %}}
{{% col %}}
```java
public class MyType {
    private MyType() {}
    public static MyType of() { return new MyType(); }
}

// usage:
MyType x = MyType.of();
```
{{% /col %}}
{{% /multicol %}}

- Kotlin's variadic functions are mapped to Java's variadic methods

{{% multicol %}}
{{% col %}}
```kotlin
fun f(vararg xs: Int) {
    val ys: Array<Int> = xs
}
```
{{% /col %}}
{{% col %}}
```java
void f(int... xs) {
    Integer[] ys = xs;
}
```
{{% /col %}}
{{% /multicol %}}

- Kotlin's extension methods are mapped to ordinary Java methods with one more argument

{{% multicol %}}
{{% col %}}
```kotlin
class MyType { }
fun MyType.myMethod() {}

// usage:
val x = MyType()
x.myMethod() // postfix
```
{{% /col %}}
{{% col %}}
```java
public class MyType {}
public void myMethod(MyType self) {}

// usage:
MyType x = new MyType();
myMethod(x); // prefix
```
{{% /col %}}
{{% /multicol %}}

---

## Kotlin--Java Mapping (pt. 5)

- practical consequence: usability of fluent chains, in Java, is suboptimal
    + e.g. [sequence operations](https://kotlinlang.org/docs/sequences.html) are implemented as extension methods

{{% multicol %}}
{{% col %}}
```kotlin
import kotlin.sequences.*

val x = (0 ..< 10).asSequence() // 0, 1, 2, ..., 9
    .filter { it % 2 == 0 } // 0, 2, 4, ..., 8
    .map { it + 1 } // 1, 3, 5, ..., 9
    .sum() // 25
```
{{% /col %}}
{{% col %}}
```java
import static kotlin.sequences.SequencesKt.*;

int x = sumOfInt(
            map(
                filter(
                    asSequence(
                            new IntRange(0, 9).iterator()
                    ),
                    it -> it % 2 == 0
                ),
                it -> it + 1
            )
        );
```
{{% /col %}}
{{% /multicol %}}

- optional arguments exist in Kotlin but they do not exist in Java

{{% multicol %}}
{{% col %}}
```kotlin
fun f(a: Int = 1, b: Int = 2, c: Int = 3) = // ...

// usage:
f(b = 5)
```
{{% /col %}}
{{% col %}}
```java
void f(int a, int b, int c) { /* ... */ }

// usage:
f(1, 5, 3);
```
{{% /col %}}
{{% /multicol %}}

---

## Kotlin--Java Mapping (pt. 6)

- practical consequence: 
    + arguments are handy in Kotlin, but...
    + ... they are painful to use in Java
    + mitigation strategy: use `@JvmOverloads` to generate overloaded methods for Java
        * sadly, the does not take into account all possible ordered combinations of arguments

{{% multicol %}}
{{% col %}}
```kotlin
import kotlin.jvm.JvmOverloads

@JvmOverloads
fun f(a: Int = 1, b: Int = 2, c: Int = 3) = // ...
```
{{% /col %}}
{{% col %}}
```java
void f(int a, int b, int c) { /* ... */ }
void f(int a, int b) { f(a, b, 3); }
void f(int a) { f(a, 2, 3); }
void f() { f(1, 2, 3); }

// missing overloads:
// void f(int a, int c) { f(a, 2, c); }
// void f(int b, int c) { f(1, b, c); }
// void f(int c) { f(1, 2, c); }
// void f(int b) { f(1, b, 3); }
```
{{% /col %}}
{{% /multicol %}}

---

## Kotlin--Java Mapping Example

- Compile our CSV lib and import it as a dependency in a novel Java library

- Alternatively, add Java sources to the `jvmTest` source set, and use the library

- Listen to the teacher presenting key points

--- 

## Kotlin--JavaScript Mapping (pt. 1)

> __Disclaimer__: the generated JS code is not really meant to be read by humans

- DCE will eliminate unused code
    + "unused" $\equiv$ "not explicitly labelled as exported"
    + code is exported by means of the `@JsExport` annotation
        * to be used on API types and functions
        * requires the `kotlin.js.ExperimentalJsExport` __opt-in__
    + not all syntactical constructs or types are currently exportable
        * you should ignore the warning explicitly

- Kotlin supports overloading, JS does not
    + class / interface members names are __mengled__ to avoid clashes
    + the `@JsName` annotation can be used to control the name of a member 
        * to be used on API types and functions

- Kotlin class $\equiv$ [JS prototype](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes)

- Primitive type mappings are non-trivial (cf. [documentation](https://kotlinlang.org/docs/js-to-kotlin-interop.html#kotlin-types-in-javascript))
    + Kotlin numeric types, except for `kotlin.Long`, are mapped to JavaScript `Number`
        * hence, there is no way to distinguish numbers by type
    
    + `kotlin.Char` is mapped to JS `Number` representing character code.

    + Kotlin preserves overflow semantics for `kotlin.Int`, `kotlin.Byte`, `kotlin.Short`, `kotlin.Char` and `kotlin.Long`

    + `kotlin.Long` is not mapped to any JS object, as there is no 64-bit integer number type in JS. It is emulated by a Kotlin class

    + `kotlin.String` is mapped to JS `String`

    + `kotlin.Any` is mapped to JS `Object` (`new Object()`, `{}`, and so on)

    + `kotlin.Array` is mapped to JS `Array`

    + Kotlin collections (`List`, `Set`, `Map`, and so on) are not mapped to any specific JS type

    + `kotlin.Throwable` is mapped to JS `Error`

- Kotlin's `dynamic` overrides the type system, and it is translated "1-to-1"

- Kotlin's common std-lib is implemented in JS
    * cf. NPM package [`kotlin`](https://www.npmjs.com/package/kotlin)

- Companion objects are treated similarly to Kotlin/JVM

- Extension methods are treated similarly to Kotlin/JVM

- Variadic functions are compiled to JS functions accepting an array
    + which are not really variadic in JS

{{% /section %}}
