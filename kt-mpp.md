# Multi-platform programming in Kotlin

## Motivation, Context, and Goal

### Platforms and multi-platforms applications

1. SW dev is commonly performed on top of some platform

0. Technically, platforms are most common OS
    - e.g. Linux, Mac OS, and Windows

0. In the past, applications were said "multi-platform" if they were buildable -- and therefore executable -- on several OS
    - this definition is __outdated__

0. Nowadays, mainstream SW dev is performed on top of __virtual__ platforms, which already support multiple OS
    - e.g. JVM, .NET, Python, JS, ART (Android RunTime), etc

0. This is made possible by virtual machines:
    1. define a low-level API/byte-code which is OS-agnostic
    1. implement several VM, one for each OS, supporting that API/byte-code

0. One can now execute a Java/Python/JS/.NET program on any (relevant) OS
    + however, no matter how beautiful my Java program is, if I want it in my browser I must either rewrite it in JS or wrap it in a WS
    + however, no matter how cool my .NET ML toolkit is, if I want it to go mainstream I need to make it usable for Python developers

0. Nowadays, by "platforms" we mean not only OS but __runtimes__ too
    + __multi-platform__ applications support several runtimes and OS

### Motivation & Context

1. Maximising the _audience_ of your SW is often desirable, if not explicitly needed
    + in some domains, a wide audience is a necessary for success
        - eg. Middlewares, Databases, Protocols, Markup Languages, Build Management Systems, VM, etc

0. To do so, it is important for modern SW to support as many platforms as possible
    + what would you think about some SW which does not support YOUR_FAVOURITE_OS in 2020?
    + what would you think about some SW which does not support YOUR_FAVOURITE_LANGUAGE in 2020?

0. However, multi-platform support is __costly__ from a SE perspective
    + as everything should work in spite of differences among platforms
    + e.g. not all platforms provide the same fucntionalities
        + and if they do, they rarely have the exact same API

0. The basic workflow for multi-platform applications dev is _very easy_ & __extrimely costly__:
    1. design you SW in a platform-agnostic way
    2. provide one implementation for each target platform
    3. test them all
    4. deploy them all

0. In the _best_-case scenario costs scale up linearly with the amount of target platforms
    + as all phases of SE must be repeated for each platform
    + yet, in the _worst_-case scenario, different platforms may have different __abstraction gaps__ w.r.t the problem at hand
        - depending on which and how many libs are available for each platform 
    
0. Examples of technologies adopting this approach:
    - VM such as the JVM, .NET's CLR, CPython, NodeJS, etc.
    - Browsers such as Firefox and Chrome (and their respective JS interpreters)
    - [ReactiveX](http://reactivex.io/), which supports JVM, JS, .NET, Swift, etc
    - [Protocol Buffers](https://developers.google.com/protocol-buffers), which supports JVM, .NET, Go, Python, C++, etc
    - all such technologies are backed by __well-funded__ organisations

0. How can one keep multi-platform development feasible?
    + We may use the C/C++ appraoch: 
        1. let's choose a language for which __compilers__ or __code-generators__ exists targetting __multiple platforms__
        2. keep the implementation as _platform-agnostic_ as possible
        3. write platform-specific parts when only when __unavoidable__

0. If properly configured, Kotlin lets developers follow this approach
    + as it supports compilation for a number of platforms:
        1. JVM => Win Desktop + Linux Desktop + MacOS Desktop + Android + ...
        0. JS => Browser + Node
        0. __\[Incubating\]__ Objective C and Swift => iOS ans MacOS
        0. __\[Incubating\]__ Native => Win + Linux + MacOS + iOS 
        0. __\[Incubating\]__ Web Assembly (WASM)

### Multi-platform applications and libraries

- Multi-platform development processes vary a lot depending on what's their objective

- Multi-platform development processes may target the development of
    1. multi-platform __applications__ --- i.e. SW which can _run_ on several platforms
    2. multi-platform __libraries__ --- i.e. SW which can _be used to build SW_ on several platforms

- Developing applications multi-platform libraries is usually harder
    + one must not only ensure the software runs
    + but also its __public API__ is clear and usable on all platforms

- Making a multi-platform library actually useful on all platforms requires to pay attention to several details
    + e.g. naming conventions
    + e.g. scoping subtleties
    + e.g. syntactic sugar
    + e.g. primitive types


### Goal

> In this presentation we will briefly discuss how multi-platform projects should be managed in Kotlin + Gradle

We will encompass the peculiarities & subtleties of multi-platform dev along the following phases of SE:
1. design
0. development
0. testing
0. deployment

## Preliminaries

### Kotlin MPP Project Setup

Basic info about MPP project setup in Gradle & IntelliJ 

### Available Targets

https://kotlinlang.org/docs/reference/mpp-dsl-reference.html

Briefly describe which targets are available, and their peculiarities:

- JVM
    + best-supported target
    + may taget a specific JDK version: prefer 1.8
- Native
    + supporting native may imply dealing with 3+ targets
    + cross-compilation is not yet supported
- Win by MinGW, Linux
    + must pay attention to architecture
    + MinGW is not exactly a "Windows vanilla" environment
- iOS, macOS
    + requires Apple facilities
    + may imply further costs
- Android
    + requires Android SDK
- JS
    + requires choosing among Browser support, Node support or both
    + the two sub-targets have different runtimes

### General Structure of a Kotlin MPP Project

#### Generally speaking a Kotlin MPP partions the source code as follows:
- __common *main* code__: platform-agnostic code which only depends on the Kt StdLib or other platform-agnostic libraries
- __common *test* code__: platform-agnostic _test_ code which only depends on the Kotlin Testing API or other platform-agnostic libraries
- for each target `T` (e.g. jvm, js, etc)
    - __`T` *main* code__: `T`-specific code depending on __common *main*__ and on `T`-specific libraries
    - __`T` *test* code__: `T`-specific _test_ code depending on __common *test*__, on __`T` *main*__, and on `T`-specific _test_ libraries

#### General Project Structure:
```
rootDirectory
├── build.gradle.kts
└── src
    ├── commonMain
    │   └── kotlin         // common code is Kt only
    │       └── Hello.kt  
    ├── commonTest
    │   └── kotlin
    ├── jvmMain
    │   ├── java          // jvm target supports java sources too
    │   ├── kotlin
    │   └── resources     // jvm target supports resources too
    ├── jvmTest
    │   ├── java
    │   ├── kotlin
    │   └── resources
    ├── jsMain
    │   └── kotlin        // js code is Kt only
    ├── jsTest
    │   └── kotlin
    ├── <target>Main
    │   └── kotlin
    └── <target>Test
        └── kotlin
```

#### Minimal Build File:
```kotlin
plugins {
    kotlin("multiplatform") version "1.4.10"
}

repositories { mavenCentral() } // mandatory

kotlin {
    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation(kotlin("stdlib-common"))
            }
        }
        val commonTest by getting {
            dependencies {
                implementation(kotlin("test-common"))
                implementation(kotlin("test-annotations-common"))
            }
        }

        jvm { // this is where the `jvm` target is declared
            withJava() // project may include bare Java sources
            val jvmMain by getting {
                dependencies {
                    api(kotlin("stdlib-jdk8"))
                    implementation("ordinaryGroupID", "ordinaryArtifactID", "X.Y.Z")
                }
            }
            val jvmTest by getting {
                dependencies {
                    implementation(kotlin("test-junit"))
                }
            }
        }

        js { // this is where the `js` target is declared
            nodejs() // the js target will generate a NodeJS project
            browser() // the js target will generate a Webpack bundle
            val jsMain by getting {
                dependencies {
                    api(kotlin("stdlib-js"))
                    api(npm("npmProjectName", "npmProjectVersion")) // NPM dependencies can be included in js target
                }
            }
            val jsTest by getting {
                dependencies {
                    implementation(kotlin("test-js"))
                }
            }
        }

        otherTarget("targetName") { // https://kotlinlang.org/docs/reference/mpp-dsl-reference.html#targets
            // target specific dependencies
        }
    }
}
```

### Rationale

Kt MPP enforces strong segregation of the platform-agnostic and platform-specific parts


### Operations 

Several target-specific tasks are available for developers:

- Task `classes` depends on
    - Task `<target>MainClasses` which compiles `target`'s __main__ code
    - Task `<target>TestClasses` which compiles `target`'s __test__ code

- Task `test` depends on
    - Task `<target>Test` which executes `target`-specific tests
    - Task `<target>TestClasses` which executes `target`-specific tests

- Task `jar` depends on
    - Task `<target>Jar` which generates `target`-specific `.jar` archives

> Even non-JVM targets are packaged as `.jar`s to support the import of your project as MPP dependency

> Target-specific tasks may be added by Gradle plugins for, e.g., signing and deploying archives 

## MPP Design

### General remarks

- Try to maximise the platform-agnostic part
    + try to rely on platform-specific code only when strictly needed 

- Try to hide the platform specific-part as much as possible
    + prefer public interfaces and internal classes in platform-agnostic code
    + use static factories instead of constructors

### Workflow (top-down)

1. Draw a platform-agnostic design for your domain entities
    + keep in mind that you can only rely on a very small StdLib / runtime
    + keep in mind that some API may be missing in the general case:
        - e.g. file system API 
    + try to reason in a platform-agnostic way
        - e.g. file system makes no sense for in-browser JS

2. Assess the _abstraction gap_, __for each targe platform__

3. For each target platform:
    + draw a platform-specific design extending the platform-agnostic one
    + and filling the abstraction gap for that target

## Workflow (bottom-up)

1. Your goal is to support feature `F` in a multi-platform fashion
    + you know a library `L1` exists for target `T1`, supporting `F`

2. For each target `Ti` where `i = 2, ..., N`:
    + look for a library `Li` supporting `F` for `Ti`

3. Try to "compute" if there exists an _intersection_ `I` among the APIs of all `Li` 
    + if all `Li` support `F` you may expect similar API, despite differences in naming

4. Design a platform-agnostic interface for `I`

5. For each platform `Ti` where `i = 2, ..., N`:
    + implement the aforementioned interface using `Li`

### Example (top-down): the `Resource` interface

### Example (bottom-up): the `FooSerializer` interface

## MPP Implementation

### Overview

- Of course if one puts some code in a target-specific source set, that code is platform-specific

- Yet, there are mechanisms to enforce an implementation for a particular concept is provided for each target

- In any case, when writing both common and target-specific code, it is important to take care of that code will be compiled
    + especially if one wants the compiled code to be usable as a library on each target platform

- Finally, a number of issues arise from JS targets, as JavaScript is a dynamic language with no type-checking
    + these must be understood and handled/avoided, even in common code

### The Expect/Actual Mechanism

### Making your API Usable on Targets

#### On the JVM

##### Kt-JVM mappings
- primitive types
- interfaces (and their methods)
- classes
- methods whose arguments have default values
- varargs
- companion objects
- packages

##### The `@JvmStatic` Annotation

##### The `@JvmOverloads` Annotation

##### The `@JvmName` Annotation

#### On JavaScript

##### Kt-JS mappings
- primitive types
- interfaces (and their methods)
- classes
- methods whose arguments have default values
- varargs
- companion objects
- packages

##### The `@JsName` Annotation

- methods overloading and JS object model
- methods' names mengling

##### JS Numbers

- JS does not distings among integers and floating point numbers

### Using JS dependencies

#### The `dynamic` type

#### External declarations

#### The `js("...")` function

#### The `@JsModule` and `@JsNonModule` annotations

## MPP Test

### Overview

- Testing if pretty straightforward: nothing new to say about how to write tests

- In MPP projects it is important to test all targets
    + never assume that a target will pass tests because other targets are passing tests

- However, in MPP projects it is strategical to re-use tests as much as possible
    - recall that test is code as well, and code comes either in a platform-agnostic or -specific flavour

- If the MPP design is good, most domain entities should be reified into commond code
    - thus most tests can be written as common test code
    - low level mechanisms (e.g. expect/actual) can be exploited to fine tune tests on targets, if needed

- Kotlin MPP libraries include a minimal testing API (`kotlin("test-common")`)
    - which comes with many target-specific implementations:
        * e.g. `kotlin("test-junit")` (backed by JUnit) for JVM targets
        * e.g. `kotlin("test-js")` (backed by Mocha) for JS targets
    - API very similar to JUnit's one
    - quite limited (no parametric tests, no access to local resources)
    - yet it does the job

- Of course one can still write platform-specific tests via custom test frameworks
    - but they will only work for one target

- If one writes tests as common code, Gradle will compile them for all targets
    - then you can run target-spefic tests via the task `:<target>Test
    - this is quite well integrated with IntelliJ's GUI as well

### Example (platform-agnostic)

### Example (platform-specific)

### About target-language testing

+ Despite target support is guaranteed by Kotlin, we suggest to add some target-language tests in your code

+ While (possibly target-tuned) common tests assess the correct functioning of your code, target-language tests assess that your library can be used as you expect in the target language
    + e.g. if one forgets a `@JvmStatic` annotation, the Kotlin code is usable, while the Java one is not
    + e.g. if one forgets a `@JsName` annotation, the Kotlin code is usable, while the JS one is not

+ So, for instance, we suggest to write Java tests assessing your library's API can be used as you expect in Java
    + similarly, you may want to do the same for other targets

+ Currently, the Gradle plugin does not allow developers to include JS code in JS-targetting projects
    + thus, target-language tests for JS are better performed via separate projects

### Debugging

- Debugging is supported in IntelliJ, both for JVM and for JS targets (not sure about Native targets)
    + you can use breakpoints & co. as usual

- Keep in mind while debugging that you __always__ debugging a particular target
    + keep in mind the peculiarities of that target while debugging

- While debugging JS, keep in mind that the correspondence among Kotlin line and JS lines of code is not perfect
    + you may be willing to debug the generate JS code directly---which is quite human-readable
    + in that case, just launch the generated code in Node

## MPP Deployment

### Overview

- MPP projects packaging

- JVM-specific packaging

- Android-specific packaging

- JVM-specific fat-jars

- NPM-specific packaging

- Browser-specific packaging

### Deployment on Maven Repositories

### Deployment on NPM

### Browser-compatible bundles

## Notable Examples

### Kt Math

### 2P-Kt

### Clikt

## Possible Activities

- Experiment with Native compilation

- Experiment with Android compilation

- Experiment with iOS / Mac OS / Swift interoperability

