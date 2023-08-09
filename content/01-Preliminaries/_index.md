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

> **API** $\equiv$ application programming interface(s)  $\stackrel{\Delta}{=}$ a _formal_ specification of the set of **functionalities** provided by a software (sub-)system for _external_ usage, there including their **input**, **outputs**, and _enviromental_ **preconditions** and **effects**
- client-server methaphor is implicit

---

Examples:
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

TODO: define runtime

{{% /section %}}

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