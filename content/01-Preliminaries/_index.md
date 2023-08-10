+++

title = "Preliminaries"
description = "Motivation and context for the course"
outputs = ["Reveal"]
aliases = [
    "/preliminaries/"
]

+++

{{% section %}}

# What are software platforms?

- Fuzzy concept
  * often mentioned in courses / literature, yet not precisely defined

- Insight:
> The **substratum** upon which **software applications** _run_

---

# What are software platforms?
## Operative systems (?)

- Most software runs on top of an __opertive system__ (OS)

- So yes, all operative systems are platforms

- Is there more?
  + e.g. [JabRef](https://www.jabref.org/): __Java__ application running on top of __JVM__
    * the JVM is an application for the OS
  + e.g. [Draw.io](https://app.diagrams.net/): __Web__ application running on top of __browsers__
    * the browser is an application for the OS

---

# What are software platforms?
## Operative systems + ???

- Programming languages?
  * not that simple
    + e.g. Scala apps can call Java code

- Runtimes
  * i.e. the set of libraries and conventions backing programming languages

{{% /section %}}

---

# What are software platforms?
## Precise definitions

> **Software platform** $\stackrel{\Delta}{=}$ anything having an **API** enabling the writing of applications,
> and the **runtime** supporting the execution of those applications

- thank you chap, what are API and runtimes then?

---

{{% section %}}

# What are API and runtimes?

> [**API**](https://en.wikipedia.org/wiki/API) $\equiv$ application programming interface(s) $\stackrel{\Delta}{=}$ a _formal_ specification of the set of **functionalities** provided by a software (sub-)system for _external_ usage, there including their **input**, **outputs**, and _enviromental_ **preconditions** and **effects**
- client-server methaphor is implicit

> [**Runtime**](https://en.wikipedia.org/wiki/Runtime_system) [system/environment] $\approx$ the set of computational resources backing the execution of a software (sub-)system
- we say that "runtimes _support_ API"

---

# What are API and runtimes?

## Examples of API

- all possible _public_ interfaces / classes / structures in an OOP module
  * and their public/protected methods / fields / properties / constructors
    + and their formal arguments, return types, throwable excepions

- all possible commands a CLI application accept as input
  * and their admissible sub-commands, options, and arguments
    + and the corresponding outputs, exit values, and side-effects

- all possible paths a Web service may accept HTTP request onto
  * and their admissible HTTP methods
    + and their admissible query / path / body / header parameters
      - and the corresponding status codes, and response bodies

---

# What are API and runtimes?

## Examples of runtimes

- any virtual machine (JVM, CRL, CPython, V8)
  * and their standard libraries
  * and their type system and internal conventions
    + eg value/reference types in JVM/CRL, global lock in CPython

- any operative system (Win, Mac, Linux)
  * and their system calls, daemons, package managers, default commands, etc
  * and their program memory, access control, file system models

- any Web service 
  * and the protocols they leverage upon
  * and their URL structuring model
  * the data schema of their input/output objects
  * the authentication / authorization mechanisms they support

{{% /section %}}

---

# Notable platforms

- The Java Virtual Machine (JVM)
  * supported languages: Java, Kotlin, Scala, Clojure, etc.

- .NET's [pronunced "dot NET"] Common Language Runtime (CRL)
  * supported languages: C#, VB.NET, F#, etc.

- Python 3
  * supported language: Python

- NodeJS (V8)
  * supported language: JavaScript, TypeScript, etc.

- Each browser may be considered as a platform per se
  * supported language: JavaScript

- ...

---

# Practical features of platforms

- standard libraries
  * i.e. pre-cooked functionalities developers / users may exploit

- predefined design decisions 
  * e.g. global lock in Python, event loop in JavaScript, etc.

- organizational, stylistic, technical conventions
  * e.g. project structure, code linting, nomenclature, etc.

- packaging conventions, import mechanisms, and software repositories
  * e.g. classpath for the JVM, NPM for JS, Pip + Pypi for Python, etc.

- user communities
  * e.g. many Data scientists use Python, many Web developers use JVM / JS

---

{{% section %}}

# Example: the JVM platform

---

## Standard libraries

- Types from the `java.*` and `javax.*` packages are usable from any JVM language 

- Many nice functionalities covering:
  * multi-threading, non-blocking IO, asynchronous programming
  * OS-independent GUI, or IO management
  * data structures and algorithms for collections and streams
  * unlimited precision arithmetic

- Many functionalities are provided by community-driven third party libraries
  * e.g. YAML/JSON parsing / generation, CSV parsing, complex numbers

---

## Predefined design decisions

- Everything is (indirectly) a subclass/instance of `Object`
  * except fixed set of primitive types, and static stuff

- Every object is potentially a lock
  * useful for concurrency

- Default methods inherited by `Object` class
  * e.g. `toString`, `equals`, `hashCode`

- All methods are virtual by default

- ...

---

## Organizational, stylistic, technical conventions

- Project should be organized according to the Maven's [standard directory layout](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)

- Official stylistic conventions for most JVM languages
  * e.g. type names in `PascalCase`, members names in `camelCase`
  * e.g. getters/setters in Java vs. Kotlin's or Scala's properties

- Many technical conventions:
 * iterable data structures should implement the `Iterable` interface
 * variadic arguments are considered arrays
 * constructors for collections subclasses accept `Iterable`s as input
 * ...

---

## Packaging conventions, import mechanisms, and software repositories

* Code is organized into packages
  - packages must correspond to directory structures

* Code archives (JAR) are Zip files containing compiled classes

* Basic import mechanism: the class path
  * i.e. the path where classes are looked for
  * commonly set at application startup

* Many third-party repositories for JVM libraries
  - [Maven central repository](https://central.sonatype.com) is the most relevant one
  - many tools for dependency management (e.g. Maven, Gradle)

---

## User communities

- Android developers

- Back-end Web developers

- Desktop applications developers

- Researchers in the fields of: semantic Web, multi-agent systems, etc.

- ...

{{% /section %}}

---

{{% section %}}

# Example: the Python platform

---

## Standard libraries

- Notably, one the richest standard libraries ever

- Many nice functionalities covering:
  * all the stuff covered by Java
  * plus many more, e.g. complex numbers, JSON and CSV parsing, etc

- Many functionalities are provided by community-driven third party libraries
  * e.g. scientific or ML libraries

---

## Predefined design decisions

- Everything is (indirectly) a subclass/instance of `object`
  * no exceptions

- [Global Interpreter Lock](https://realpython.com/python-gil/) (GIL)

- Magic methods supporting various language features
  * e.g. `__str__`, `__eq__`, `__iter__`

- No support for overloading

- Variadic and keywords arguments

- ...

---

## Organizational, stylistic, technical conventions

- Project should be organized according to [Kenneth Reitz's layout](https://docs.python-guide.org/writing/structure/)

- Official stylistic conventions for Python ([PEP8](https://peps.python.org/pep-0008/))
  * e.g. type names in `PascalCase`, members names in `snake_case`
  * e.g. getters/setters in Java vs. Kotlin's or Scala's properties

- Many technical conventions:
 * [duck typing](https://realpython.com/lessons/duck-typing/)
 * iterable data structures should implement the `__iter__` method
 * variadic arguments are considered tuples
 * keyword arguments are considered sets
 * ...

---

## Packaging conventions, import mechanisms, and software repositories

* Code is organized into packages and modules
  - packages must correspond to directory structures
  - modules must correspond to files

* Code archives (JAR) are wheel files containing Python sources

* Each Python installation has an internal folder where libraries are stored
  * `pip` simply unzips modules/packages in there
  * `import` statements look for packages/modules in there

* [Pypi](https://pypi.org/) as the official repository for Python libraries
  * `pip` as the official tool for dependency management

---

## User communities

- Data-science community

- Back-end Web developers

- Desktop applications developers

- System administrators

- ...

{{% /section %}}

---

{{% section %}}

# Example: the NodeJS platform

---

## Standard library

- Very limited standard library from JavaScript
  + enriched with many Node modules

- Many nice functionalities covering:
  * networking and IPC
  * OS, multiprocess, and cryptographic utilities

- Many functionalities are provided by community-driven third party libraries
  * you can find virtually anything on [npmjs.com](https://www.npmjs.com/)

---

## Predefined design decisions

- Object orientation based on [prototypes](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes)

- Single threaded design + [event loop](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick)

- Asynchronous programming via [continuation-passing style](https://matt.might.net/articles/by-example-continuation-passing-style/)

---

## Organizational, stylistic, technical conventions

- Project structure is somewhat arbitrary
  * the project must contain a `package.json` file 
  * declaring the entry point of the project

- Many conventions co-exist
  * e.g. [W3School's one](https://www.w3schools.com/js/js_conventions.asp)

- Many technical conventions:
  * duck typing
  * magic variables, e.g. for prototype
  * ...

---

## Packaging conventions, import mechanisms, and software repositories

* Code is organized into modules
  - modules are file containing anything
  - and declaring what to export

* Code archives (JAR) are tarball files containing JS sources

* Third party libraries can be installed via `npm`
  * locally, for the user, or globally


* [NPM](https://www.npmjs.com/) as the official repository for JS libraries
  * `npm` as the official tool for dependency management

---

## User communities

- Front-end Web developers

- Back-end Web developers

- GUI developers

- ...

{{% /section %}}

---

{{% section %}}

# Platforms from software developers' perspective

The choice of a platform impacts developers during:

- the design phase

- the implementation phase

- the testing phase

- the release phase

---

## How platform affects the **design** phase

- One may choose the platform which minimises the __abstraction gap__ w.r.t. the problem at hand

> __Abstraction gap__ $\approx$ the space among the __problem__ and the prior functionalities offered by a __platform__. Ideally, the bigger the space the more _effort_ is required to build the solution 

---

## How platform affects the **implementation** phase

- Developers build solutions by leveraging the API of the platform

- ... as well as the API of any third-party library available for that platform

---

## How platform affects the **testing** phase

- Test suites are a "project in the project"
  * so remarks are similar w.r.t. the implementation phase

- One may test the system against as many versions as possible of the underlying platforms

- One may test the system against as many OS as possible
  * virtual platforms may behave differently depending on the OS

---

## How platform affects the **release** phase

- Packaging systems are platform-specific...

- Repositories are platform-specific...

- ... release is therefore platform specific

> __Release__ $\approx$ publishing some packaged software system onto a repository, hence enabling its import and exploitation

{{% /section %}}

---

## Platforms and multi-platforms applications

1. Software is always developed/run on top of some __platform__

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