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

- __common *main* code__: platform-agnostic code which only depends on:
    * the Kotlin std-lib 
    * plus other third-party Kotlin multi-platform libraries

- __common *test* code__: platform-agnostic _test_ code which only depends on:
    * the common main code and all its dependencies
    * some Kotlin multi-platform testing library (e.g. [`kotlin.test`](https://kotlinlang.org/api/latest/kotlin.test/) or [Kotest](https://kotest.io/))

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
        kotlin("multiplatform") version "1.9.10"
    }
    ```

- which repositories should Gradle use when looking for dependencies
    ```kotlin
    repositories { 
        mavenCentral() 
        // custom repository here
    } 
    ```

- which platforms to target (reference [here](https://kotlinlang.org/docs/multiplatform-dsl-reference.html#targets))
    ```kotlin
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
        // other targets here
    }
    ```

---

## Kotlin multi-platform build configuration (pt. 2)

- which third-party library should each target depend upon
    ```kotlin
    kotlin {
        sourceSets {
            val commonMain by getting {
                dependencies {
                    api(kotlin("stdlib-common"))
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
                    implementation("ordinaryGroupID", "ordinaryArtifactID", "X.Y.Z")
                }
            }
            val jsTest by getting {
                dependencies {
                    implementation(kotlin("test-junit"))
                }
            }
            val jsMain by getting {
                dependencies {
                    api(kotlin("stdlib-js"))
                    api(npm("npmProjectName", "npmProjectVersion"))
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
                languageVersion = "1.8" // possible values: "1.4", "1.5", "1.6", "1.7", "1.8", "1.9"
                apiVersion = "1.8" // possible values: "1.3", "1.4", "1.5", "1.6", "1.7", "1.8", "1.9"
                enableLanguageFeature("InlineClasses") // language feature name
                optIn("kotlin.ExperimentalUnsignedTypes") // annotation FQ-name
                progressiveMode = true // false by default
            }
        }
    }
    ```

---

## Fuild configuration: full example


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
                implementation("ordinaryGroupID", "ordinaryArtifactID", "X.Y.Z")
            }
        }
        val jsTest by getting {
            dependencies {
                implementation(kotlin("test-junit"))
            }
        }
        val jsMain by getting {
            dependencies {
                api(kotlin("stdlib-js"))
                api(npm("npmProjectName", "npmProjectVersion"))
            }
        }
        val jsTest by getting {
            dependencies {
                implementation(kotlin("test-js"))
            }
        }

        all {
            languageVersion = "1.8" // possible values: "1.4", "1.5", "1.6", "1.7", "1.8", "1.9"
            apiVersion = "1.8" // possible values: "1.3", "1.4", "1.5", "1.6", "1.7", "1.8", "1.9"
            enableLanguageFeature("InlineClasses") // language feature name
            optIn("kotlin.ExperimentalUnsignedTypes") // annotation FQ-name
            progressiveMode = true // false by default
        }
    }
}


```
