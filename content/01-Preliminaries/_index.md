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

* Code archives (`.jar`) are Zip files containing compiled classes

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
  * e.g. indentation-aware syntax, blank line conventions, etc.
  * ...

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

* Code archives (`.whl`) are Zip files containing Python sources

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

* Code archives (`.tar.?z`) are compressed tarball files containing JS sources

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

{{< figure src="abstraction-gap.svg" width="70%" >}}

---

## How platform affects the **implementation** phase

- Developers build solutions by leveraging the API of the platform

- ... as well as the API of any third-party library available for that platform

{{< figure src="third-party.svg" width="80%" >}}

---

## How platform affects the **testing** phase

- Test suites are a "project in the project"
  * so remarks are similar w.r.t. the implementation phase

- One may test the system against as many versions as possible of the underlying platforms

- One may test the system against as many OS as possible
  * virtual platforms may behave differently depending on the OS

---

## How platform affects the **release** phase

> __Release__ $\approx$ publishing some packaged software system onto a repository, hence enabling its import and exploitation

- Packaging systems are platform-specific...

- Repositories are platform-specific...

- ... release is therefore platform specific

{{% /section %}}

---

## Platform choice for a new SW project

- Choice is commonly driven by design / technical decisions
  + which platform would ease developers' work the most?

- However, choosing the platform is a business decision as well
  + platforms have user communities
  + SW project benefit from wide(r) user communities

- Business decision: which user communities to target?
  + what are the most relevant platforms for that communities?

- Coherency is key for success in platform selection
  1. coherently choose the target community w.r.t SW goal
  2. coherently choose the platform w.r.t. target community
  
---

## General benefits of coherence

+ The abstraction gap is likely lower

+ More third party libraries are likey available

+ The potential audience is wider
  * implies the SW project is more valuable

+ Easier to find support / help in case of issues

+ More likely that third-party issues are timely fixed

---

{{% section %}}

## What about __research-oriented__ software? (pt. 1)

> Researchers may act as software developers

1. to elaborate data

2. to create in-silico experiments

3. to study software system

4. to create software tools improving their research

5. to create software tools for the community
    * this is how many FOSS tools have been created

---

## What about __research-oriented__ software? (pt. 2)

> Research institutions are _not_ software houses

- Personnel can only dedicate a fraction of their time to development

- Most software artifacts are disposable (1, 2, 4)

- Development efforts are discontinuous

- Development teams are small 

- Software is commonly a means, not an objective

- Commitment to software development is:
  * commonly on the individual 
  * sometimes on the project
  * rarely on the institution

---

## What about __research-oriented__ software? (pt. 3)

> Research-oriented software development should maximise audience and impact, while minimising development and maintenance effort

- Science requires reproducibility
  * wide(r) user base is a facilitator for reproducility

- The wider the community, the wider the impact of community-driven research software
  * more potential citations

- Minimising effort can be done by improving efficiency
  * by means of software engineering

---

## General benefits of coherence, __for researchers__

> Choosing the right community / platform is strategical for research-oriented software

+ The abstraction gap is likely lower
  * less development effort

+ More third party libraries are likely available
  * less development effort

+ The potential audience is wider
  * more value, more impact, more reproducibility

+ Easier to find support / help in case of issues
  * potentially, less development effort

+ More likely that third-party issues are timely fixed
  * potentially, less development effort

{{% /section %}}

---

{{% section %}}

## About technological silos

> __Silos__ (in IT) are software components / systems / ecosystems having poor external interoperability 
> (i.e. software from silos A hardly interoperates with software from silos B)

- Platforms are (pretty wide) software silos
  * making software from any two platforms interoperate is non-trivial
  * making software from the same platform interoperate is easier

- Examples:
  * intra-platform: Kotlin program calling Java library
  * inter-platform: Python program calling native library

--- 

## Open research communities vs. technological silos

- Research communities in CS / AI may overlap with platforms communities:
  * e.g. neural networks researchers $\rightarrow$ Python
  * e.g. data science $\rightarrow$ Python | R
  * e.g. symbolic AI $\rightarrow$ JVM | Prolog
  * e.g. multi-agent systems $\rightarrow$ JVM
  * e.g. semantic-web $\rightarrow$ JVM | Python

- What about inter-community research efforts?
  * they may need interoperability among different silos
  * in lack of which, research is slowed down

> Multi-platform programming is an enabler for inter-community research

{{% /section %}}

--- 

## The need for multi-platform programming

- Ideal goal:
> let the same software tool run on multiple platforms

- More reasonable goal:
> Create multiple artifacts, one per each supported platform, sharing the same design and functioning

- Practical goal: 
> design and write the software once, then port it to several platforms

---

{{% section %}}

## Approaches for multi-platform programming

1. Write once, build anywhere
    - software is developed using some sort of "super-language"
    - code from the "super-language" is automatically compiled for all platforms

2. Write first, wrap elsewhere
    - software is developed for some principal platform
    - implementations for all other platforms are wrappers of the first one

---

## Write once, build anywhere (concept)

{{< figure src="write-once-build-everywhere.svg" width="70%" >}}

---

## Write once, build anywhere (explanation)

- Assumptions:
  * one "super-language" exists, having:
    1. a code-generator targetting multiple platforms
        + e.g. compiler, transpiler, etc.
    2. the same standard library implemented for all those platforms

- Workflow:
  1. Design, implement, and test most of the project via the super-language
      - this is the platform-agnostic (a.k.a. "common") part of the project

  2. Complete, refine, or optimise platform-specific aspects via 
      - platform-specific code
      - platform-specific third-party libraries

  3. Build platform-specific artifacts, following platform-specific rules
  
  4. Upload platform-specific artifacts on platform-specific repositories

---

## Write once, build anywhere (analysis)

Let `N` be the amount of supported platforms

- Platform-agnostic functionalities require effort which is independent from `N`

- Platform-specific functionalities require effort proportional to `N`

- Better to minimise platform-specific code
  * by maximising common code
  * this is true both for main code and for test code 

- Relevant questions: 
  * what to realise as common code? what as platform-specific code?
  * how to set the boundary of the common (resp. platform-specific) code?

---

### Takeaway

> The abstraction gap of the common code is as wide as the one of the platform having the widest abstraction gap

{{< figure src="abstraction-gap-mp.svg" width="50%" >}}

---

## Write once, build anywhere (strategy)

Whenever a new functionality needs to be developed:

1. Try to realise it with common std-lib only

2. If not possible, try to maximise the portion of platform-agnostic code
  * make it possible to plug platform-specific aspects

3. For each functionality which cannot be realised as purely platform-agnostic:
  1. design a platform-agnostic interface
  2. implement the interface `N` times, one per target platform

---

## Write first, wrap elsewhere (concept)

{{< figure src="write-first-wrap-elsewhere.svg" width="70%" >}}

---

## Write first, wrap elsewhere (explanation)

- Assumptions:
  * one "main" platform exists such that
    + code from the "main" platform can be called from other target platforms
  * the software has been designed in a platform-agnostic way

- Workflow:
  1. Fully implement, test, and deploy the software for the main platform

  2. For all other platforms:
    1. re-design and re-write platform-specific API code
    2. implement platform-specific API by calling the main platform's code
    3. re-write test for API code
    4. build platform-specific packages, wrapping the main platform's package and runtime
  
  3. Upload platform-specific artifacts on platform-specific repositories

---

## Write once, build everywhere (analysis)

Let `N` be the amount of supported platforms

- Clear separation of API code from implementation code is quintessential

- The effort required for writing API code is virtually the same on all platforms

- Global effort is sub-linearly dependent on `N`
  * implementing API and wrapper code is a manual task
  * still less effort than implementing the same design `N` times

- The `i`-th platform's wrapper code will only call the main platform's API code

- Relevant questions: 
  * how to deal with platform-specific aspects?
  * how to create platform-agnostic design?

{{% /section %}}