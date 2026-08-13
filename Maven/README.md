
# Apache Maven — Deep Reference Notes

> A practical, from-first-principles reference for understanding Maven deeply enough to read, debug, and maintain Maven-based Java projects later.
>
> **Context:** These notes are written with Spring Core / Spring Framework exploration in mind, but the concepts apply to Java libraries, applications, multi-module builds, CI/CD pipelines, and most Maven projects.
>
> **Current documentation check:** Apache Maven's official download page currently lists **Maven 3.9.16** as the latest Maven 3 release and **Maven 4.0.0-rc-5** as a preview release. Maven 3.9+ requires JDK 8+ to run; Maven 4 requires JDK 17+ to run. Always check the official download page for the version supported by the project you are studying. See [Apache Maven downloads](https://maven.apache.org/download.cgi) and [Maven release history](https://maven.apache.org/docs/history.html).

---

## Table of Contents

- [1. What Maven Actually Is](#1-what-maven-actually-is)
- [2. The Problem Maven Was Created to Solve](#2-the-problem-maven-was-created-to-solve)
- [3. What Was Used Before Maven?](#3-what-was-used-before-maven)
- [4. Maven's Main Responsibilities](#4-mavens-main-responsibilities)
- [5. The Core Mental Model](#5-the-core-mental-model)
- [6. Maven Is Not the Compiler](#6-maven-is-not-the-compiler)
- [7. Maven Architecture at a High Level](#7-maven-architecture-at-a-high-level)
- [8. How `mvn` Interacts With Your Computer](#8-how-mvn-interacts-with-your-computer)
- [9. Installing Maven and Verifying the Environment](#9-installing-maven-and-verifying-the-environment)
- [10. Maven Wrapper](#10-maven-wrapper)
- [11. The `pom.xml` — Maven's Project Model](#11-the-pomxml--mavens-project-model)
- [12. Maven Coordinates: GAV and Beyond](#12-maven-coordinates-gav-and-beyond)
- [13. Packaging Types](#13-packaging-types)
- [14. Standard Maven Directory Layout](#14-standard-maven-directory-layout)
- [15. Maven Build Lifecycle](#15-maven-build-lifecycle)
- [16. The Three Built-in Lifecycles](#16-the-three-built-in-lifecycles)
- [17. Default Lifecycle Phases in Detail](#17-default-lifecycle-phases-in-detail)
- [18. Phase vs Goal vs Plugin](#18-phase-vs-goal-vs-plugin)
- [19. What Happens When You Run `mvn package`?](#19-what-happens-when-you-run-mvn-package)
- [20. Dependency Management](#20-dependency-management)
- [21. Direct vs Transitive Dependencies](#21-direct-vs-transitive-dependencies)
- [22. Dependency Scopes](#22-dependency-scopes)
- [23. Dependency Mediation and Version Conflicts](#23-dependency-mediation-and-version-conflicts)
- [24. `dependencyManagement` vs `dependencies`](#24-dependencymanagement-vs-dependencies)
- [25. BOMs](#25-boms)
- [26. Maven Repositories](#26-maven-repositories)
- [27. The Local Maven Repository: `.m2/repository`](#27-the-local-maven-repository-m2repository)
- [28. What Exactly Lives Inside `.m2`?](#28-what-exactly-lives-inside-m2)
- [29. How Maven Downloads and Caches a JAR](#29-how-maven-downloads-and-caches-a-jar)
- [30. Why Maven Downloads POM Files Too](#30-why-maven-downloads-pom-files-too)
- [31. How a Maven Coordinate Maps to a File Path](#31-how-a-maven-coordinate-maps-to-a-file-path)
- [32. SNAPSHOT Dependencies](#32-snapshot-dependencies)
- [33. Remote Repositories, Mirrors, Proxies, and Credentials](#33-remote-repositories-mirrors-proxies-and-credentials)
- [34. `settings.xml` vs `pom.xml`](#34-settingsxml-vs-pomxml)
- [35. Plugins](#35-plugins)
- [36. Lifecycle Bindings and Packaging](#36-lifecycle-bindings-and-packaging)
- [37. Profiles](#37-profiles)
- [38. Multi-Module Maven Projects and the Reactor](#38-multi-module-maven-projects-and-the-reactor)
- [39. Parent POMs and Inheritance](#39-parent-poms-and-inheritance)
- [40. `target/` and Build Output](#40-target-and-build-output)
- [41. IDE Integration](#41-ide-integration)
- [42. Maven in IntelliJ IDEA](#42-maven-in-intellij-idea)
- [43. Maven in Eclipse](#43-maven-in-eclipse)
- [44. Maven in CI/CD](#44-maven-in-cicd)
- [45. Important Maven Commands](#45-important-maven-commands)
- [46. Useful Maven Command Patterns](#46-useful-maven-command-patterns)
- [47. How to Read Maven Output](#47-how-to-read-maven-output)
- [48. Troubleshooting Dependencies](#48-troubleshooting-dependencies)
- [49. Offline Mode and Caching](#49-offline-mode-and-caching)
- [50. Maven and Reproducible Builds](#50-maven-and-reproducible-builds)
- [51. Maven and Java Toolchains](#51-maven-and-java-toolchains)
- [52. Maven Security and Supply-Chain Considerations](#52-maven-security-and-supply-chain-considerations)
- [53. Common Misunderstandings](#53-common-misunderstandings)
- [54. Maven vs Ant — The Mental Shift](#54-maven-vs-ant--the-mental-shift)
- [55. Maven vs Gradle — Very High-Level](#55-maven-vs-gradle--very-high-level)
- [56. A Concrete Spring Core Example](#56-a-concrete-spring-core-example)
- [57. A Complete Mental Simulation](#57-a-complete-mental-simulation)
- [58. Practical Learning Path](#58-practical-learning-path)
- [59. Quick Reference Cheat Sheet](#59-quick-reference-cheat-sheet)
- [60. Official Documentation Links](#60-official-documentation-links)

---

# 1. What Maven Actually Is

## One-sentence definition

**Apache Maven is a Java build and project-management tool that uses a declarative project model (`pom.xml`) to describe a project, its dependencies, its build configuration, and the lifecycle used to transform source code into artifacts.**

Maven is commonly used for:

- compiling Java code
- running tests
- packaging code into JAR/WAR/etc.
- resolving and downloading dependencies
- managing transitive dependencies
- publishing artifacts
- building multi-module projects
- running static analysis, code generation, formatting, coverage, documentation, and other tasks through plugins
- integrating builds into IDEs and CI/CD

The official Maven introduction describes the goals as making builds easier, providing a uniform build system, and providing project information.

Official reference: [What is Maven?](https://maven.apache.org/what-is-maven.html)

---

# 2. The Problem Maven Was Created to Solve

The important historical problem was not simply “Java compilation is hard.”

The deeper problem was **build consistency and dependency management across many projects**.

The Apache Maven history explains that Maven started in the Jakarta ecosystem, with the Jakarta Turbine project as an important test bed. Developers were maintaining multiple Ant builds that were similar but slightly different. JARs were also being checked into CVS. The goal was to define the project in one place, standardize the directory structure/build process, publish project information, and share JARs between projects.

Official references:

- [What is Maven?](https://maven.apache.org/what-is-maven.html)
- [History of Maven](https://maven.apache.org/background/history-of-maven.html)

The big idea was roughly:

```text
Before:

Project A -> custom build.xml -> custom paths -> manually maintained JARs
Project B -> another build.xml -> other paths -> manually maintained JARs
Project C -> another build.xml -> different conventions

After Maven:

Project -> pom.xml -> standard conventions -> declared dependencies -> lifecycle
```

The project became more understandable because a developer could inspect the project model instead of reverse-engineering a collection of custom shell scripts, Ant targets, library folders, and environment assumptions.

---

# 3. What Was Used Before Maven?

## 3.1 Manual compilation

At the most basic level, Java can be compiled directly with `javac`:

```bash
javac -cp "lib/*" -d target/classes src/main/java/com/example/App.java
```

That works, but the developer now has to manage:

- source paths
- output directories
- classpaths
- dependency JARs
- test compilation
- test execution
- packaging
- manifests
- resource copying
- clean-up
- platform differences
- versioning
- publishing

For a small project this is manageable. For a large project it becomes tedious.

## 3.2 Ant

**Apache Ant** became a popular build tool for Java. Its model is more task-oriented and procedural: the developer writes `build.xml` and defines targets/tasks.

A simplified Ant-style approach looks like:

```xml
<target name="compile">
    <javac srcdir="src"
           destdir="bin"
           classpath="lib/*" />
</target>
```

Then:

```bash
ant compile
```

Ant was powerful and remains useful, but a key difference is that the developer is responsible for describing the build process in detail. Maven moved much of that build knowledge into standard lifecycle behavior and reusable plugins.

Apache's archived “Maven for Ant Users” guide explicitly describes this difference: Ant puts more responsibility on the developer to understand and script the build, while Maven captures build-process knowledge in plugins and conventions.

Reference: [Maven for Ant Users](https://maven.apache.org/archives/maven-1.x/start/maven-for-ant-users.html)

## 3.3 Keeping JAR files inside source control

Historically, projects commonly stored binary dependencies with the source code. Maven's historical documentation specifically discusses why keeping JARs in CVS was undesirable: common libraries were duplicated across projects, source checkouts became heavier, and dependency reuse was poor.

Reference: [Maven 1.x repositories: Why not store JARs in CVS?](https://maven.apache.org/components/archives/maven-1.x/using/repositories.html)

Maven's repository model separated these concerns:

```text
Git repository / source control
    -> source code
    -> pom.xml
    -> project configuration

Maven repositories
    -> dependencies
    -> plugins
    -> project artifacts
```

That separation is one of Maven's most important ideas.

---

# 4. Maven's Main Responsibilities

Think of Maven as performing several connected jobs.

| Responsibility | What Maven does |
|---|---|
| Project model | Reads `pom.xml` and creates a model of the project |
| Dependency management | Figures out which libraries are needed |
| Dependency resolution | Finds artifacts in local/remote repositories |
| Build lifecycle | Provides standardized build stages |
| Plugin execution | Runs plugin goals that perform actual work |
| Packaging | Produces JAR, WAR, etc. |
| Installation | Places the project's artifact in the local repository |
| Deployment | Publishes artifacts to a remote repository |
| Multi-module orchestration | Orders and builds related modules |
| Profiles | Enables environment/configuration-specific behavior |
| Reporting | Allows plugins to generate reports/documentation |
| Integration | Lets IDEs and CI understand the build model |

So Maven is better thought of as an **orchestrator of a build system** than as a compiler.

---

# 5. The Core Mental Model

The easiest mental model is:

```text
                 pom.xml
                    |
                    v
             Maven reads model
                    |
          +---------+---------+
          |                   |
          v                   v
   dependencies          build config
          |                   |
          v                   v
 repositories            plugins/goals
          |                   |
          +---------+---------+
                    |
                    v
              lifecycle phase
                    |
                    v
              plugin execution
                    |
                    v
          files/processes/output
                    |
        +-----------+-----------+
        |                       |
        v                       v
    target/                local/remote repo
```

A particularly useful rule is:

> **The POM declares intent; the lifecycle defines the journey; plugins perform the work; repositories provide artifacts; the operating system supplies the machine resources.**

---

# 6. Maven Is Not the Compiler

This distinction matters a lot.

Maven itself does not contain a Java compiler that replaces `javac` conceptually.

Instead, Maven invokes plugins, and a plugin may invoke Java tooling or perform some other operation.

For example:

```text
mvn compile
   |
   v
compile phase
   |
   v
compiler plugin goal
   |
   v
Java compiler / compiler APIs
   |
   v
.class files
```

Likewise, Maven does not inherently “know how to run JUnit” as a hard-coded compiler feature. It delegates that work to testing plugins and their configured goals.

That plugin architecture is one of Maven's central design ideas.

---

# 7. Maven Architecture at a High Level

A modern Maven execution can be pictured like this:

```text
Terminal / IDE / CI
        |
        |  mvn package
        v
Maven launcher / Maven Wrapper
        |
        v
JVM
        |
        v
Maven core
        |
        +--> settings.xml
        |
        +--> pom.xml / parent POMs / profiles
        |
        +--> Maven model building
        |
        +--> repository system / dependency resolution
        |
        +--> lifecycle calculation
        |
        +--> plugin resolution
        |
        +--> plugin execution
        |
        +--> reactor for multi-module builds
        |
        v
Filesystem + network + JDK + external tools
```

The exact internal implementation is more detailed, but this model is very useful when debugging.

---

# 8. How `mvn` Interacts With Your Computer

This is the part that is often not explained clearly enough.

## 8.1 You type a command

Example:

```bash
mvn clean package
```

Your shell receives the command.

Depending on the operating system, `mvn` resolves to a Maven launcher script/executable made available through `PATH`.

## 8.2 Maven starts Java

Maven is a Java application.

The launcher establishes the Maven runtime environment and starts a JVM with Maven's libraries on the Maven runtime classpath.

Useful environment/configuration concepts include:

- `PATH` — allows the shell to find `mvn`
- `JAVA_HOME` — points to the JDK/JRE installation used by the environment where required/configured
- `MAVEN_HOME` / Maven installation directory conventions — identify the Maven installation in some environments/tools
- `MAVEN_OPTS` — JVM options for Maven processes
- project `.mvn/` configuration
- user `~/.m2/settings.xml`

Always inspect the actual Maven environment with:

```bash
mvn --version
```

The official Maven guide demonstrates that this command reports Maven version, Maven home, Java version/home, OS, architecture, and other environment information.

Reference: [Maven in 5 Minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes)

## 8.3 Maven reads configuration

Maven may read:

```text
Project:
    pom.xml
    .mvn/*

User:
    ~/.m2/settings.xml

Maven installation:
    <maven-home>/conf/settings.xml
```

The settings reference documents the two main `settings.xml` locations:

- `${maven.home}/conf/settings.xml`
- `${user.home}/.m2/settings.xml`

Reference: [Settings Reference](https://maven.apache.org/settings.html)

## 8.4 Maven builds a project model

Maven does not simply parse a few XML tags and immediately run `javac`.

It builds an effective project model using information such as:

- current `pom.xml`
- parent POMs
- inherited configuration
- properties
- dependency declarations
- dependency management
- profiles
- repositories
- plugin configuration
- Maven defaults / Super POM

This effective model determines what Maven believes the project is.

Useful debugging command:

```bash
mvn help:effective-pom
```

The effective POM is often the fastest way to answer:

> “Where did Maven get this configuration from?”

## 8.5 Maven resolves required artifacts

Maven may need to resolve:

- your dependencies
- dependency POMs
- parent POMs
- build plugins
- plugin dependencies
- plugin metadata
- reporting plugins
- extensions
- other artifacts required by the build

Those artifacts may come from:

```text
local repository -> ~/.m2/repository
        |
        | if missing / stale / required
        v
remote repository / mirror
        |
        v
local repository cache
```

## 8.6 Maven executes plugin work

When a lifecycle phase is requested, Maven determines which goals are bound to the relevant phases and runs them.

A plugin goal can:

- read source files
- create directories
- compile Java
- copy resources
- create archives
- run tests
- run analysis
- generate source code
- invoke external processes
- start/stop infrastructure in integration-test scenarios
- upload artifacts
- generate reports

Maven itself therefore interacts with the PC mostly through the JVM, filesystem APIs, networking, processes, environment variables, and the JDK/tooling made available to plugins.

---

# 9. Installing Maven and Verifying the Environment

## 9.1 Install Java first

Maven needs Java in order to run.

Check:

```bash
java -version
javac -version
```

A JDK is generally what you want for development because compilation/tooling requires JDK capabilities.

## 9.2 Install Maven

Use the official distribution or your operating system's supported package manager.

Official guide:

[Installing Apache Maven](https://maven.apache.org/install.html)

## 9.3 Verify Maven

```bash
mvn --version
```

Example shape of output:

```text
Apache Maven 3.x.x
Maven home: ...
Java version: ...
Java home: ...
OS name: ...
OS arch: ...
```

## 9.4 Important version distinction

Do not confuse these three things:

```text
Maven version
Java version
POM model version
```

Example:

```xml
<modelVersion>4.0.0</modelVersion>
```

`4.0.0` here is the POM model/schema version. It is **not** saying “this project uses Maven 4”.

---

# 10. Maven Wrapper

The Maven Wrapper lets a project specify the Maven version it expects and lets contributors invoke that version through files committed with the project.

Typical files/scripts include:

```text
mvnw
mvnw.cmd
.mvn/wrapper/maven-wrapper.properties
```

Depending on wrapper type/version, additional wrapper support files may be present.

Run:

```bash
./mvnw clean verify
```

Windows:

```powershell
mvnw.cmd clean verify
```

Why this is useful:

```text
Without wrapper:
Developer machine
   -> whichever Maven version is installed

With wrapper:
Project
   -> requested Maven distribution/version
```

The official wrapper documentation states that if the required Maven version is not already available, the wrapper can download and install it before running the build. By default, wrapper distributions are installed under a path such as:

```text
~/.m2/wrapper/dists
```

References:

- [Apache Maven Wrapper](https://maven.apache.org/tools/wrapper/)
- [Maven Wrapper internals](https://maven.apache.org/components/tools/wrapper/maven-wrapper/)

A good modern Git project often prefers the wrapper because it reduces “works on my machine” differences in Maven versions.

---

# 11. The `pom.xml` — Maven's Project Model

`pom.xml` means **Project Object Model**.

It is the central declarative description of a Maven project.

A minimal POM can look like:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>hello-app</artifactId>
    <version>1.0.0</version>
</project>
```

The Maven documentation identifies `project`, `modelVersion`, `groupId`, `artifactId`, and `version` as the core minimum structure for a basic POM.

Reference: [Introduction to the POM](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)

## 11.1 What can go into a POM?

Common sections include:

```xml
<project>
    <modelVersion>...</modelVersion>

    <parent>...</parent>

    <groupId>...</groupId>
    <artifactId>...</artifactId>
    <version>...</version>
    <packaging>...</packaging>

    <name>...</name>
    <description>...</description>
    <url>...</url>

    <properties>...</properties>

    <dependencies>...</dependencies>
    <dependencyManagement>...</dependencyManagement>

    <build>...</build>

    <profiles>...</profiles>

    <modules>...</modules>

    <repositories>...</repositories>
    <pluginRepositories>...</pluginRepositories>
</project>
```

Not every POM uses every section.

## 11.2 Declarative vs procedural

A useful difference:

```text
Ant mindset:
    “Do these actions in this order.”

Maven mindset:
    “This is what my project is, these are its dependencies,
     and this is how its lifecycle should behave.”
```

You can still configure Maven very extensively, but the default style is declarative and convention-driven.

---

# 12. Maven Coordinates: GAV and Beyond

Maven identifies artifacts through coordinates.

The familiar shorthand is **GAV**:

```text
groupId : artifactId : version
```

Example:

```text
org.example : my-library : 1.2.3
```

The Maven artifact documentation explains that real coordinates can contain more information, commonly represented conceptually as:

```text
groupId : artifactId : version : classifier : extension
```

Reference: [Maven Artifacts](https://maven.apache.org/repositories/artifacts.html)

## 12.1 `groupId`

Represents the project group/organization.

Common convention:

```text
com.mycompany
org.springframework
org.apache.maven
```

Java-style reversed-domain naming is recommended.

## 12.2 `artifactId`

Identifies the artifact within the group.

Example:

```text
spring-core
maven-core
my-service
```

## 12.3 `version`

Identifies the version.

Examples:

```text
1.0.0
2.5.1
1.0-SNAPSHOT
```

## 12.4 Classifier

A classifier distinguishes related artifacts of the same base coordinates.

Examples can include:

```text
sources
javadoc
tests
linux-x86_64
```

Not every artifact has a classifier.

## 12.5 Extension

The extension identifies the artifact file type, commonly:

```text
jar
pom
war
zip
```

The official artifact model explains that the familiar GAV shorthand omits some coordinate dimensions for convenience.

Reference: [Maven Artifacts](https://maven.apache.org/repositories/artifacts.html)

---

# 13. Packaging Types

A Maven project has a packaging type.

Common values:

```xml
<packaging>jar</packaging>
```

or:

```xml
<packaging>war</packaging>
```

The default packaging is `jar` when omitted.

Packaging matters because Maven uses it to determine lifecycle behavior and default plugin bindings.

Examples:

```text
jar -> Java library/application JAR
war -> web application archive
pom -> aggregator/parent/BOM-style project, depending on usage
```

Reference: [POM Introduction](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)

---

# 14. Standard Maven Directory Layout

Maven strongly encourages a conventional layout.

Typical project:

```text
my-project/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   └── test/
│       ├── java/
│       └── resources/
└── target/
```

Meaning:

| Path | Purpose |
|---|---|
| `pom.xml` | Project model |
| `src/main/java` | Main Java source |
| `src/main/resources` | Main resources |
| `src/test/java` | Test Java source |
| `src/test/resources` | Test resources |
| `target` | Build output |

The standard layout is important because Maven can do a lot with little configuration when you follow the conventions.

Reference: [Standard Directory Layout](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)

---

# 15. Maven Build Lifecycle

The Maven build lifecycle is one of the most important concepts to understand.

A lifecycle is a defined sequence of build phases.

For the normal/default lifecycle, think roughly:

```text
validate
   -> initialize
   -> generate-sources
   -> process-sources
   -> generate-resources
   -> process-resources
   -> compile
   -> process-test-sources
   -> test-compile
   -> test
   -> package
   -> verify
   -> install
   -> deploy
```

There are more phases than the shortened diagram above; see the official lifecycle reference.

Reference: [Build Lifecycle Introduction](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)

## The most important rule

When you ask Maven for a phase, Maven runs that phase **and all preceding phases in that lifecycle**.

Therefore:

```bash
mvn package
```

does not mean “only package the code”.

It means:

```text
run all required phases up to package
```

For a normal Java JAR build, that means Maven normally validates, compiles, tests, and then packages.

---

# 16. The Three Built-in Lifecycles

Maven has three built-in lifecycles:

| Lifecycle | Main purpose |
|---|---|
| `default` | Build, test, package, verify, install, deploy |
| `clean` | Remove previous build output |
| `site` | Generate project documentation/site |

Reference: [Build Lifecycle](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)

Important examples:

```bash
mvn clean
mvn package
mvn verify
mvn install
mvn deploy
mvn site
```

You can combine lifecycles/phases:

```bash
mvn clean package
```

This first executes the `clean` lifecycle and then the default lifecycle through `package`.

---

# 17. Default Lifecycle Phases in Detail

Below is a practical mental model rather than an exhaustive plugin-binding table.

| Phase | Mental meaning |
|---|---|
| `validate` | Is the project model valid enough to build? |
| `initialize` | Set up initial build state |
| `generate-sources` | Generate source code if configured |
| `process-sources` | Process source files |
| `generate-resources` | Generate resources |
| `process-resources` | Copy/process resources |
| `compile` | Compile main code |
| `process-classes` | Process compiled classes |
| `generate-test-sources` | Generate test sources |
| `process-test-sources` | Process test sources |
| `generate-test-resources` | Generate test resources |
| `process-test-resources` | Process test resources |
| `test-compile` | Compile test code |
| `test` | Run unit tests |
| `prepare-package` | Prepare final packaging |
| `package` | Create JAR/WAR/etc. |
| `pre-integration-test` | Prepare integration-test environment |
| `integration-test` | Run/deploy into integration-test setup |
| `post-integration-test` | Clean up integration-test environment |
| `verify` | Verify the package/build quality |
| `install` | Put artifact into local repository |
| `deploy` | Publish artifact to remote repository |

Reference: [Lifecycle Introduction](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)

## Why `verify` is a valuable default

For many projects, `verify` is a better conceptual “complete build” than `package` because the lifecycle includes integration-test and verification stages before it.

Apache's “Building a Project with Maven” guide points out that `mvn verify` is sufficient for the vast majority of Maven-built projects.

Reference: [Building a Project with Maven](https://maven.apache.org/run-maven/)

---

# 18. Phase vs Goal vs Plugin

This is one of the most important distinctions in Maven.

## Phase

A **phase** is a named point in a lifecycle.

Example:

```text
compile
package
verify
```

## Goal

A **goal** is a specific executable operation provided by a Maven plugin.

Examples have the form:

```text
plugin-prefix:goal
```

For example:

```text
compiler:compile
surefire:test
jar:jar
```

The exact fully qualified plugin coordinates are often:

```text
org.apache.maven.plugins:maven-compiler-plugin:...:compile
```

## Plugin

A **plugin** packages executable build functionality.

So:

```text
Lifecycle phase
      |
      | is associated/bound with
      v
Plugin goal
      |
      v
Actual work
```

## Example

When `compile` is reached, a compiler-plugin goal can do the actual compilation.

When `test` is reached, a test plugin goal can execute the tests.

When `package` is reached for a JAR project, the JAR plugin is involved in creating the JAR.

---

# 19. What Happens When You Run `mvn package`?

Suppose the project contains:

```text
src/main/java/com/example/App.java
src/main/resources/application.properties
src/test/java/com/example/AppTest.java
pom.xml
```

You run:

```bash
mvn package
```

A simplified flow is:

```text
1. Shell starts Maven
2. Maven starts/uses JVM
3. Maven finds project POM
4. Maven reads settings
5. Maven builds the effective project model
6. Maven activates relevant profiles
7. Maven resolves required project/build artifacts
8. Maven identifies lifecycle phases up to package
9. validate/initialize/... execute
10. resources are copied/processed
11. Java is compiled
12. test code is compiled
13. tests execute
14. package artifact is created
15. result appears under target/
```

For a JAR project, a result may look like:

```text
target/my-project-1.0.0.jar
```

The exact plugin goals and extra steps depend on packaging and project configuration.

---

# 20. Dependency Management

Dependency management is a core Maven feature.

Instead of downloading a library manually, you declare its coordinates in the POM.

Example shape:

```xml
<dependencies>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>example-library</artifactId>
        <version>1.2.3</version>
    </dependency>
</dependencies>
```

The Maven dependency mechanism then uses that information to construct a dependency graph and resolve the required artifacts.

Official reference: [Introduction to the Dependency Mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

---

# 21. Direct vs Transitive Dependencies

## Direct dependency

You declare it yourself:

```text
Your project -> A
```

## Transitive dependency

Your project declares A, and A declares B:

```text
Your project
   |
   v
   A
   |
   v
   B
```

Maven can resolve B transitively by reading dependency information from A's POM.

This is a huge improvement over manually collecting every nested JAR yourself.

Example:

```text
app
 |
 +--> library-A
        |
        +--> library-B
               |
               +--> library-C
```

Your POM can directly mention only `library-A` when that is the intended direct dependency relationship.

Reference: [Dependency Mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

---

# 22. Dependency Scopes

A dependency's `scope` tells Maven where that dependency is needed.

Common scopes:

| Scope | Main idea |
|---|---|
| `compile` | Normal compile + runtime dependency; default scope |
| `provided` | Needed to compile/test, expected to be supplied by the runtime/container |
| `runtime` | Not needed to compile application code, but needed when running/testing |
| `test` | Needed only for compiling/running tests |
| `system` | Explicit local filesystem artifact; generally discouraged |

Example:

```xml
<dependency>
    <groupId>org.example</groupId>
    <artifactId>test-library</artifactId>
    <version>1.0.0</version>
    <scope>test</scope>
</dependency>
```

The official POM reference describes the semantics of these scopes.

Reference: [POM Reference](https://maven.apache.org/pom.html)

---

# 23. Dependency Mediation and Version Conflicts

A real application usually has a graph, not a simple list.

Example:

```text
app
├── A -> C:1.0
└── B -> C:2.0
```

Now Maven must decide which version of C belongs on the classpath.

This is called **dependency mediation**.

Do not think of Maven dependency resolution as simply “take every JAR and put all of them on the classpath”.

Maven has rules for selecting a version and allows you to override/manage versions explicitly.

A very useful inspection command is:

```bash
mvn dependency:tree
```

Example shape:

```text
com.example:my-app:jar:1.0.0
+- org.example:A:jar:1.0
|  \- org.example:C:jar:1.0
\- org.example:B:jar:1.0
   \- org.example:C:jar:2.0
```

This is one of the best commands to learn when investigating Spring dependency graphs.

Reference: [Dependency Mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

---

# 24. `dependencyManagement` vs `dependencies`

This distinction is extremely important in Spring and large multi-module projects.

## `dependencies`

This says:

> “My project actually uses this dependency.”

Example:

```xml
<dependencies>
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>library-a</artifactId>
        <version>1.2.3</version>
    </dependency>
</dependencies>
```

## `dependencyManagement`

This says roughly:

> “If a project in this inheritance tree uses this dependency, use these managed defaults.”

A declaration in `dependencyManagement` alone does **not** make the dependency appear on the project's classpath.

Example:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>library-a</artifactId>
            <version>1.2.3</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Then a child can write:

```xml
<dependency>
    <groupId>org.example</groupId>
    <artifactId>library-a</artifactId>
</dependency>
```

and inherit the managed version.

Reference: [Maven dependency management](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

---

# 25. BOMs

A **Bill of Materials (BOM)** is a POM used to manage a consistent set of dependency versions.

A common pattern is:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>example-bom</artifactId>
            <version>1.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

BOMs are especially useful when many libraries must move together.

The Maven dependency mechanism documentation explicitly covers BOM POMs and importing dependencies.

Reference: [Dependency Mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

---

# 26. Maven Repositories

Maven documentation describes two types of repositories:

1. **Local repository** — a directory on the computer where Maven runs.
2. **Remote repositories** — repositories accessed over paths/protocols such as HTTPS or file-based repositories.

Reference: [Introduction to Repositories](https://maven.apache.org/guides/introduction/introduction-to-repositories.html)

A simplified model:

```text
                INTERNET / COMPANY NETWORK
                        |
                        v
               +------------------+
               | Remote Repository|
               +------------------+
                        ^
                        |
                     download
                        |
                        v
               +------------------+
               | Local Repository |
               |    ~/.m2         |
               +------------------+
                        ^
                        |
                        |
                    Maven build
```

## Common remote repositories

### Maven Central

The default central repository used by normal Maven builds is the Maven Central repository.

Official repository service:

[https://repo.maven.apache.org/maven2/](https://repo.maven.apache.org/maven2/)

Repository search can also be done via:

[https://central.sonatype.com/](https://central.sonatype.com/)

### Internal repository managers

Organizations often use repository managers such as:

- Nexus Repository
- JFrog Artifactory
- cloud-hosted artifact repositories

These can act as:

- proxy/cache for Maven Central
- private repositories for company libraries
- release repositories
- snapshot repositories

---

# 27. The Local Maven Repository: `.m2/repository`

This is the directory you specifically asked to understand.

By default:

```text
Linux/macOS:
~/.m2/repository

Windows:
C:\Users\<username>\.m2\repository
```

The official Maven settings documentation defines the default local repository as:

```text
${user.home}/.m2/repository
```

References:

- [Settings Reference](https://maven.apache.org/settings.html)
- [Local Repositories](https://maven.apache.org/repositories/local.html)

## What is `.m2`?

`.m2` is a per-user Maven directory under the user's home directory.

It can contain things such as:

```text
.m2/
├── repository/
├── settings.xml
└── wrapper/
    └── dists/
```

Exact contents vary by Maven version, plugins, wrapper usage, and local configuration.

The most important directory for normal dependency resolution is:

```text
~/.m2/repository
```

---

# 28. What Exactly Lives Inside `.m2`?

A project dependency may create a structure like:

```text
~/.m2/repository/
└── org/
    └── example/
        └── example-library/
            └── 1.2.3/
                ├── example-library-1.2.3.pom
                ├── example-library-1.2.3.jar
                ├── example-library-1.2.3.jar.sha512
                └── ...other metadata/checksums as applicable
```

For a snapshot, you may also see metadata and timestamped snapshot artifacts.

## Important point

The local repository is **not just a JAR cache**.

It can contain:

- JARs
- POMs
- source artifacts
- Javadoc artifacts
- test JARs
- WARs
- ZIPs
- plugin artifacts
- plugin POMs
- metadata
- checksum/signature-related files
- locally installed project artifacts
- Maven Wrapper distributions under `.m2/wrapper` when the wrapper is used

Apache's repository documentation specifically describes the local repository as both a cache of downloaded artifacts and a place for locally installed artifacts.

Reference: [Local Repositories](https://maven.apache.org/repositories/local.html)

---

# 29. How Maven Downloads and Caches a JAR

Suppose your POM contains:

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>example-library</artifactId>
    <version>1.2.3</version>
</dependency>
```

Maven has to obtain at least the artifact and usually its POM metadata too.

## Simplified resolution process

```text
Step 1
Read dependency declaration
        |
        v
com.example:example-library:1.2.3
        |
Step 2
Check local repository
        |
        +---- found ----> use local artifact
        |
        +---- missing --> continue
        |
Step 3
Consult repository configuration
        |
        v
local / mirrors / remote repositories
        |
Step 4
Download required metadata/artifact
        |
        v
Save into ~/.m2/repository
        |
Step 5
Use resolved artifact in build classpath
```

This local cache is why a second build is often much faster than the first.

The official repositories guide states that Maven downloads a dependency when it is not present in the local repository (or when a newer snapshot is required).

Reference: [Introduction to Repositories](https://maven.apache.org/guides/introduction/introduction-to-repositories.html)

## Does Maven download the JAR every time?

Normally, no.

Once an artifact is available locally and still valid for the requested resolution, Maven can reuse it.

That is the purpose of the local repository.

---

# 30. Why Maven Downloads POM Files Too

This is a critical detail.

Suppose:

```text
Your application
      |
      v
Library A
```

You might think Maven only needs:

```text
library-A.jar
```

But Maven often also needs:

```text
library-A.pom
```

Why?

Because the POM can contain metadata such as:

- transitive dependencies
- packaging
- dependency management
- parent information
- repository information
- properties
- other project metadata

So Maven may need to read:

```text
A.pom
   |
   +--> B dependency
   +--> C dependency
   +--> version metadata
```

and then resolve B and C.

This is how a dependency graph can be built from published project metadata.

Reference: [Dependency Mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

---

# 31. How a Maven Coordinate Maps to a File Path

For ordinary Maven repository layout:

```text
groupId      -> directory path
artifactId   -> artifact directory
version      -> version directory
```

Example:

```text
org.example:demo-lib:1.2.3
```

maps conceptually to:

```text
org/example/demo-lib/1.2.3/
```

and then:

```text
demo-lib-1.2.3.pom
demo-lib-1.2.3.jar
```

The Maven repository layout documentation explains that coordinates are translated into paths using the Maven repository layout.

Reference: [Repository Layout](https://maven.apache.org/repositories/layout.html)

This explains why a local repository often looks like:

```text
~/.m2/repository/org/springframework/spring-core/...
```

when working on Spring projects.

---

# 32. SNAPSHOT Dependencies

A `SNAPSHOT` version represents a development version.

Example:

```xml
<version>1.0-SNAPSHOT</version>
```

A release is generally immutable by version identifier:

```text
1.0.0
```

A snapshot can change over time:

```text
1.0-SNAPSHOT
```

Maven may need metadata to determine whether a newer snapshot exists remotely, and downloaded snapshot artifacts can be stored locally using timestamped forms.

The official local repository documentation explains the distinction between `baseVersion` such as `1.0-SNAPSHOT` and timestamped snapshot versions retrieved from remote repositories.

Reference: [Local Repositories](https://maven.apache.org/repositories/local.html)

Useful command:

```bash
mvn -U verify
```

`-U` asks Maven to check for updated snapshots/releases according to Maven's update behavior.

---

# 33. Remote Repositories, Mirrors, Proxies, and Credentials

Maven does not always talk directly to Maven Central.

A company may configure:

```text
Developer Maven
      |
      v
Corporate mirror
      |
      +--> Maven Central
      +--> Internal releases
      +--> Internal snapshots
```

This gives organizations control over:

- caching
- availability
- auditing
- authentication
- private libraries
- approved artifacts
- external network access

## Mirrors

A mirror can redirect repository access through another repository endpoint.

## Proxies

Corporate networks may require Maven to use an HTTP/HTTPS proxy.

## Credentials

Repository credentials are normally kept in `settings.xml`, not committed into the project's POM.

The Maven settings reference covers:

- mirrors
- servers
- proxies
- repository configuration
- profiles

Reference: [Settings Reference](https://maven.apache.org/settings.html)

---

# 34. `settings.xml` vs `pom.xml`

A useful rule:

```text
pom.xml
    -> project-specific build/project model

settings.xml
    -> user/machine/environment-specific Maven behavior
```

Examples that belong naturally in `settings.xml`:

- local repository location
- mirrors
- proxy configuration
- credentials/server configuration
- environment-specific profiles

Examples that belong naturally in `pom.xml`:

- project coordinates
- dependencies
- plugins
- project build behavior
- project modules
- project properties
- dependency management

This separation prevents machine-specific secrets and network settings from being baked into source-controlled project files.

Reference: [Settings Reference](https://maven.apache.org/settings.html)

---

# 35. Plugins

Maven plugins provide most executable build behavior.

Think:

```text
Maven core
    -> lifecycle orchestration

Plugins
    -> actual build operations
```

Examples of plugin areas:

- compiler
- test execution
- JAR packaging
- WAR packaging
- dependency analysis
- source/Javadoc generation
- release/version operations
- site generation
- code quality
- code coverage
- code generation
- enforcer rules

## Plugin coordinates

A plugin is itself a Maven artifact.

Conceptually:

```text
groupId:artifactId:version
```

For example, Maven's official plugin ecosystem commonly uses:

```text
org.apache.maven.plugins
```

The plugin configuration guide recommends explicitly defining plugin versions to improve build reproducibility.

Reference: [Guide to Configuring Plugins](https://maven.apache.org/guides/mini/guide-configuring-plugins.html)

---

# 36. Lifecycle Bindings and Packaging

The lifecycle alone does not explain exactly how a project performs every action.

Packaging influences default bindings.

For a standard JAR project, phases such as `compile`, `test`, and `package` are associated with plugin goals that perform the corresponding work.

Conceptually:

```text
package = jar

compile phase
    -> compiler plugin goal

process-resources phase
    -> resources plugin goal

test phase
    -> test plugin goal

package phase
    -> jar plugin goal
```

This is why a simple POM can accomplish a surprising amount without you writing hundreds of lines of build instructions.

The lifecycle documentation calls this behavior “built-in lifecycle bindings.”

Reference: [Build Lifecycle](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)

---

# 37. Profiles

Profiles allow Maven behavior to change according to a selected environment/configuration.

Example:

```xml
<profiles>
    <profile>
        <id>production</id>
        <properties>
            <environment>production</environment>
        </properties>
    </profile>
</profiles>
```

Activate it with:

```bash
mvn -Pproduction verify
```

Profiles may be activated through things such as:

- explicit command-line activation
- settings
- active-by-default behavior
- JDK conditions
- operating system conditions
- properties
- file existence/missing conditions

Reference: [Introduction to Build Profiles](https://maven.apache.org/guides/introduction/introduction-to-profiles.html)

## Caution

Profiles can make a build more flexible but can also make it harder to understand.

A good project should make it reasonably clear:

```text
Which profile is active?
Why is it active?
What changes when it is active?
```

---

# 38. Multi-Module Maven Projects and the Reactor

Large projects often contain modules.

Example:

```text
spring-like-project/
├── pom.xml
├── core/
│   └── pom.xml
├── context/
│   └── pom.xml
├── beans/
│   └── pom.xml
└── tests/
    └── pom.xml
```

A top-level POM can aggregate modules:

```xml
<modules>
    <module>core</module>
    <module>context</module>
    <module>beans</module>
</modules>
```

Maven uses the **reactor** to process related projects.

The reactor can:

- collect modules
- determine build order
- build selected modules
- account for relationships between modules

Official reference: [Working with Multiple Modules](https://maven.apache.org/guides/mini/guide-multiple-modules.html)

## Example dependency relationship

```text
root
├── module-a
└── module-b
       |
       +---- depends on module-a
```

Maven must build `module-a` before `module-b` when both participate in the reactor build.

---

# 39. Parent POMs and Inheritance

A parent POM provides configuration that child projects can inherit.

Example:

```xml
<parent>
    <groupId>com.example</groupId>
    <artifactId>parent</artifactId>
    <version>1.0.0</version>
</parent>
```

A parent can provide:

- properties
- dependency management
- plugin management
- plugin configuration
- organization metadata
- shared build rules

## Parent vs aggregator

These are related concepts but not identical.

### Parent

Primarily about **inheritance**.

```text
Parent configuration
        |
        v
Child inherits configuration
```

### Aggregator

Primarily about **building multiple modules together**.

```text
Root POM
   |
   +--> module-a
   +--> module-b
   +--> module-c
```

A POM can be both parent and aggregator, but the concepts should be kept separate in your mind.

---

# 40. `target/` and Build Output

The `target/` directory contains generated build output.

Typical contents might include:

```text
target/
├── classes/
├── generated-sources/
├── generated-test-sources/
├── test-classes/
├── surefire-reports/
└── my-project-1.0.0.jar
```

Exact contents depend on the plugins used.

## Why `mvn clean` works

The clean lifecycle removes generated build output, normally including `target/` contents.

```bash
mvn clean
```

Then:

```bash
mvn package
```

creates a fresh build.

Do not confuse:

```text
target/       -> project build output
~/.m2/        -> user Maven repository/cache
```

These solve completely different problems.

---

# 41. IDE Integration

A modern Java IDE generally does not want you to maintain a second, completely separate build model by hand.

Instead, the IDE can read Maven's POM and use Maven information to configure the project.

The IDE can learn things such as:

- source roots
- test roots
- dependency JARs
- dependency scopes
- generated sources
- module relationships
- compiler configuration
- profiles
- lifecycle goals
- Maven plugins

The important architecture is:

```text
             pom.xml
                |
       +--------+--------+
       |                 |
       v                 v
     Maven              IDE
       |                 |
       v                 v
actual build       project model/editor
```

The IDE may invoke Maven directly, may use Maven's project model/import mechanisms, and may also provide IDE-specific features around the same project model.

Therefore:

> **Maven is the source of truth for Maven build semantics; the IDE is a client/consumer of that model plus its own IDE functionality.**

---

# 42. Maven in IntelliJ IDEA

IntelliJ IDEA has first-class Maven support.

JetBrains documents that IntelliJ can:

- create Maven projects
- open/sync existing Maven projects
- add Maven support to existing projects
- configure/manage multi-module projects
- use a selected Maven installation/version

Reference: [IntelliJ IDEA — Maven](https://www.jetbrains.com/help/idea/maven-support.html)

## Typical IntelliJ flow

Open a project containing:

```text
pom.xml
```

IntelliJ recognizes it as Maven-based and imports the Maven project model.

It can then show:

```text
Maven tool window
   |
   +-- Lifecycle
   +-- Plugins
   +-- Dependencies
   +-- Profiles
```

You can run things such as:

```text
Lifecycle -> clean
Lifecycle -> compile
Lifecycle -> test
Lifecycle -> package
Lifecycle -> verify
```

or use the terminal:

```bash
./mvnw test
```

## IDE build vs Maven build

This distinction matters.

An IDE may be able to compile code using its own incremental compiler and project model. That does not automatically mean the same thing as executing the full Maven lifecycle.

For CI and reproducibility, always know whether you are running:

```text
IDE build action
```

or:

```text
Maven build:
mvn verify
```

A useful practice is to ensure the project can be built from a terminal as well as from the IDE.

---

# 43. Maven in Eclipse

Eclipse has Maven integration through **m2e (Maven Integration for Eclipse)**.

m2e allows Maven project information to be integrated into Eclipse's workspace model.

Reference: [m2e documentation](https://eclipse.dev/m2e/)

Typical idea:

```text
Import -> Existing Maven Project -> pom.xml
```

The IDE can then understand Maven dependencies, project structure, lifecycle integration, and plugin-related project configuration where supported.

Again, the important distinction is:

```text
Maven model
   -> build semantics

IDE integration
   -> developer experience around that model
```

---

# 44. Maven in CI/CD

Maven is very common in automated builds.

A CI system typically does something like:

```text
checkout source
      |
      v
install/select JDK
      |
      v
restore Maven cache (optional)
      |
      v
./mvnw verify
      |
      v
publish test reports/artifacts
```

GitHub's documentation for Java + Maven demonstrates caching the local Maven repository in CI and running Maven in batch mode.

Reference: [GitHub Actions — Build and test Java with Maven](https://docs.github.com/en/actions/tutorials/build-and-test-code/java-with-maven)

## Why cache `.m2` in CI?

Because the dependency repository can be large and downloading the same immutable release artifacts on every build is wasteful.

Conceptually:

```text
CI run #1
    -> download dependencies
    -> populate cache

CI run #2
    -> restore cache
    -> reuse dependencies
```

The exact cache key should account for dependency/build configuration changes.

---

# 45. Important Maven Commands

This section is intended as the day-to-day command reference.

## Verify Maven installation

```bash
mvn --version
```

## Show help

```bash
mvn help:help
```

## Clean build output

```bash
mvn clean
```

## Compile

```bash
mvn compile
```

## Compile tests

```bash
mvn test-compile
```

## Run tests

```bash
mvn test
```

## Package

```bash
mvn package
```

## Verify

```bash
mvn verify
```

## Install into local repository

```bash
mvn install
```

## Deploy to remote repository

```bash
mvn deploy
```

## Clean and package

```bash
mvn clean package
```

## Clean and verify

```bash
mvn clean verify
```

## Skip tests but still compile test code

A common convention is:

```bash
mvn package -DskipTests
```

This usually skips running tests while still allowing test compilation.

## Skip test compilation and test execution

```bash
mvn package -Dmaven.test.skip=true
```

Use this carefully: you are not even compiling tests.

## Offline

```bash
mvn -o verify
```

Useful when everything needed is already locally available.

## Force/update checks

```bash
mvn -U verify
```

Useful when troubleshooting stale snapshot metadata/artifacts or when you explicitly want Maven to check for updates.

## Debug logging

```bash
mvn -X verify
```

This is extremely verbose but useful when investigating resolution/configuration problems.

## Show exceptions

```bash
mvn -e verify
```

## Quiet mode

```bash
mvn -q verify
```

## Show dependency tree

```bash
mvn dependency:tree
```

## Show effective POM

```bash
mvn help:effective-pom
```

## Show active profiles

```bash
mvn help:active-profiles
```

## Run a plugin goal directly

```bash
mvn <plugin-prefix>:<goal>
```

Example shape:

```bash
mvn dependency:tree
```

---

# 46. Useful Maven Command Patterns

## Build one module

In a module directory:

```bash
mvn verify
```

## Build selected modules from a multi-module root

Maven supports project selection options such as `-pl` (projects) along with dependency relationships such as `-am` (“also make”, build required reactor dependencies).

Typical pattern:

```bash
mvn -pl module-a -am verify
```

This is especially useful in large multi-module projects.

## Resume after failure

For supported reactor builds, Maven can resume from a failed module using a command such as:

```bash
mvn -rf :module-name verify
```

Check `mvn --help` for the exact options available in the installed Maven version.

## Change the project POM/file

```bash
mvn -f path/to/pom.xml verify
```

Useful in scripts or unusual repository layouts.

## Batch mode for CI

```bash
mvn -B verify
```

This reduces interactive behavior and is well suited to CI.

---

# 47. How to Read Maven Output

A typical Maven log has a pattern.

Example shape:

```text
[INFO] Scanning for projects...
[INFO]
[INFO] ---------------------< com.example:demo >---------------------
[INFO] Building demo 1.0.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-resources-plugin:...:resources (...) @ demo ---
[INFO]
[INFO] --- maven-compiler-plugin:...:compile (...) @ demo ---
[INFO]
[INFO] --- maven-surefire-plugin:...:test (...) @ demo ---
[INFO]
[INFO] --- maven-jar-plugin:...:jar (...) @ demo ---
[INFO]
[INFO] BUILD SUCCESS
```

The key line pattern is:

```text
--- plugin:version:goal (execution-id) @ artifact-id ---
```

This tells you:

```text
which plugin
which version
which goal
which execution
which project/module
```

When debugging Maven, learn to read these lines.

---

# 48. Troubleshooting Dependencies

## Problem: “Could not find artifact”

Think through this order:

```text
1. Is the coordinate correct?
2. Is the repository configured?
3. Is a mirror overriding the repository?
4. Is the artifact actually published?
5. Is authentication needed?
6. Is a proxy needed?
7. Is the local metadata stale/broken?
8. Is the network/DNS/TLS connection failing?
9. Are you in offline mode?
```

Useful commands:

```bash
mvn -X verify
mvn dependency:tree
mvn help:effective-settings
mvn help:effective-pom
```

## Problem: The dependency appears in the POM but code cannot compile

Check:

```text
scope
version
exclusions
profiles
dependencyManagement
optional/transitive behavior
IDE Maven sync state
```

## Problem: “It works in IntelliJ but fails in terminal”

Check:

```bash
mvn --version
java -version
```

Then check whether IntelliJ is using:

- a different JDK
- a different Maven installation
- the Maven Wrapper
- different profiles
- different environment variables
- IDE-only generated sources/classpath settings

## Problem: “It works on my machine”

Check:

```text
Maven version
JDK version
OS
local ~/.m2 state
settings.xml
mirrors/proxies
environment variables
profile activation
uncommitted/generated files
```

The Maven Wrapper is particularly valuable for reducing Maven-version drift.

---

# 49. Offline Mode and Caching

Maven's local repository makes offline work possible when all required artifacts are already available.

Use:

```bash
mvn -o verify
```

But offline mode does **not** mean:

> “Maven can magically build any project without internet.”

It means:

> “Do not attempt remote resolution during this execution.”

If a required artifact is missing locally, the build can fail.

That is why the first build of a project may download many artifacts, while later builds may be much faster.

---

# 50. Maven and Reproducible Builds

Maven can support reproducible builds, but you should not assume that merely using Maven makes every build perfectly reproducible.

Good practices include:

- pin dependency versions
- pin plugin versions
- avoid unnecessary dynamic versions
- use the Maven Wrapper where appropriate
- control Java/JDK versions
- use dependency management/BOMs consistently
- avoid machine-specific filesystem dependencies
- keep secrets/environment details out of source-controlled POMs
- control repositories and mirrors
- inspect dependency trees for accidental upgrades

The official plugin configuration guide specifically recommends defining plugin versions because predictable plugin versions improve build reproducibility.

Reference: [Guide to Configuring Plugins](https://maven.apache.org/guides/mini/guide-configuring-plugins.html)

---

# 51. Maven and Java Toolchains

A subtle but important topic is the difference between:

```text
JDK used to run Maven
```

and:

```text
JDK used to build/test the project
```

Maven can use Java toolchains so the project can request an appropriate JDK even when Maven itself is running under another JDK that is compatible with the Maven runtime.

This is useful in projects that need to build against older/newer Java targets or support multiple JDKs.

Official starting point:

[Apache Maven Toolchains documentation](https://maven.apache.org/guides/mini/guide-using-toolchains.html)

This is particularly important when learning Spring Framework code because the required Java version can vary by Spring Framework release line.

---

# 52. Maven Security and Supply-Chain Considerations

Maven downloads executable code from repositories, including:

- project dependencies
- build plugins
- plugin dependencies
- wrapper distributions in some setups

So Maven builds are part of your software supply chain.

## Important habits

### Know your repositories

Ask:

```text
Where are my dependencies coming from?
```

### Avoid unsafe credential handling

Do not commit passwords or tokens into `pom.xml`.

Prefer secure Maven settings/credential handling supported by your environment.

### Pin important versions

Unexpected plugin upgrades can change build behavior.

### Verify trusted artifacts where required

Maven repositories can contain metadata, checksums, and signatures depending on the artifact/repository setup.

### Be careful with arbitrary plugins

A Maven plugin is executable build code. Running a plugin can execute code on your machine with the permissions of your user/process.

Treat unfamiliar Maven builds with the same caution as unfamiliar software.

---

# 53. Common Misunderstandings

## Misunderstanding 1: “Maven is only a dependency downloader.”

No.

Dependency management is one of its major features, but Maven also orchestrates builds, lifecycles, plugins, packaging, installation, deployment, project metadata, multi-module builds, and more.

## Misunderstanding 2: “`mvn package` only packages.”

No.

It executes the required phases before `package`.

## Misunderstanding 3: “The local `.m2` folder is just downloaded JARs.”

No.

It can contain POMs, metadata, plugins, artifacts, locally installed artifacts, and other Maven repository data.

## Misunderstanding 4: “The IDE is Maven.”

No.

The IDE integrates with Maven. Maven remains a separate build system/tool.

## Misunderstanding 5: “Every dependency in `dependencyManagement` is automatically on the classpath.”

No.

`dependencyManagement` manages versions/configuration for dependencies that are actually declared/used.

## Misunderstanding 6: “Maven compiles Java itself without plugins.”

The Maven lifecycle orchestrates plugin goals; plugins perform the build operations.

## Misunderstanding 7: “Deleting `~/.m2` is the same as `mvn clean`.”

No.

```text
mvn clean
    -> project build output

Deleting ~/.m2/repository
    -> local Maven artifact cache/repository
```

These are very different.

## Misunderstanding 8: “If a dependency is in the local repository, Maven always uses exactly that copy.”

Not necessarily.

Resolution rules, repository metadata, snapshot behavior, checksums, update policies, and configuration affect what Maven considers usable.

---

# 54. Maven vs Ant — The Mental Shift

This is useful if you are studying the historical evolution.

## Ant

You might think:

```text
build.xml
    |
    +--> target compile
    +--> target test
    +--> target jar
```

The project author explicitly scripts the tasks.

## Maven

You might think:

```text
pom.xml
    |
    +--> project identity
    +--> dependencies
    +--> build configuration
    +--> lifecycle
```

The Maven lifecycle plus plugins supply the standard process.

## The conceptual shift

```text
Ant:
    “Tell the build system what to do.”

Maven:
    “Describe the project and use the standard build lifecycle.”
```

This is simplified, but it is a very useful first approximation.

Official historical comparison: [Maven for Ant Users](https://maven.apache.org/archives/maven-1.x/start/maven-for-ant-users.html)

---

# 55. Maven vs Gradle — Very High-Level

These tools solve overlapping problems but have different styles.

| Maven | Gradle |
|---|---|
| XML-based POM model | Script-based DSLs (Groovy/Kotlin) |
| Strong convention/lifecycle model | Highly programmable build model |
| Very established Java ecosystem | Very strong JVM/Android ecosystem |
| Explicit plugin configuration | Plugin/configuration via DSL |
| Predictable standard structure | Flexible/customizable build logic |

The goal here is not to decide which is “better”.

For Spring Core source exploration, knowing Maven deeply is valuable because Maven is deeply embedded in a large amount of Java ecosystem infrastructure.

---

# 56. A Concrete Spring Core Example

Imagine a project declares a Spring library dependency conceptually like:

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>...</version>
</dependency>
```

Do not think:

```text
POM
 -> magically gets spring-core.jar
```

Think through the complete chain:

```text
pom.xml
   |
   v
Maven reads org.springframework:spring-core:version
   |
   v
Maven builds dependency graph
   |
   +--> reads spring-core POM metadata
   |
   +--> discovers any transitive dependencies
   |
   v
Maven checks ~/.m2/repository
   |
   +--> artifact already available
   |       -> reuse it
   |
   +--> artifact missing/outdated
           -> consult repositories/mirrors
           -> download metadata/artifact
           -> store in local repository
   |
   v
Resolved artifacts become part of the appropriate Maven classpaths
   |
   v
compiler/test/package plugins use those classpaths
   |
   v
Build output appears in target/
```

This is the mental model to use when reading Spring's `pom.xml` files.

---

# 57. A Complete Mental Simulation

Let's simulate a clean machine.

Suppose you clone a Maven project:

```bash
git clone <repository>
cd project
```

The machine has:

```text
JDK installed
Maven installed (or project has mvnw)
```

But:

```text
~/.m2/repository
```

is mostly empty.

Now run:

```bash
./mvnw clean verify
```

## Step 1 — Wrapper starts

The wrapper determines which Maven distribution/version the project requests.

If necessary, the wrapper downloads the Maven distribution into its wrapper cache.

## Step 2 — Maven starts

Maven runs as a Java process.

## Step 3 — Maven finds the POM

It reads the project model.

## Step 4 — Maven reads settings

Maven determines repository/mirror/proxy/authentication/local-repository behavior.

## Step 5 — Maven calculates the effective model

Parents, properties, dependency management, profiles, plugin configuration, and defaults are combined.

## Step 6 — Maven resolves artifacts

For each artifact needed by the build:

```text
Check local repository
       |
       +--> found -> use/cache
       |
       +--> not found -> remote repository lookup/download
```

## Step 7 — Maven executes lifecycle

For `verify`, phases before and including `verify` execute.

## Step 8 — Plugins do the work

Compiler plugin:

```text
.java -> .class
```

Resources plugin:

```text
src/main/resources -> target/classes
```

Test plugin:

```text
test classes -> test process -> reports
```

Packaging plugin:

```text
compiled classes/resources -> JAR/WAR/etc.
```

Verification/integration plugins may perform additional checks.

## Step 9 — Build output

```text
target/
```

contains generated output/reports/artifacts.

## Step 10 — Re-run the build

Now much of the dependency graph may already exist in:

```text
~/.m2/repository
```

so Maven can avoid downloading those artifacts again.

That is the “download once, reuse locally” behavior you were asking about.

---

# 58. Practical Learning Path

For remembering Maven later, learn in this order.

## Level 1 — Core concepts

Understand these words:

```text
POM
coordinate
artifact
repository
local repository
remote repository
dependency
plugin
lifecycle
phase
goal
```

## Level 2 — Build one project

Be comfortable with:

```bash
mvn clean
mvn compile
mvn test
mvn package
mvn verify
```

## Level 3 — Dependency resolution

Be comfortable with:

```bash
mvn dependency:tree
```

and knowing what the local `.m2/repository` contains.

## Level 4 — POM understanding

Understand:

```text
parent
properties
dependencies
dependencyManagement
build
plugins
pluginManagement
profiles
modules
```

## Level 5 — Large project behavior

Understand:

```text
reactor
parent inheritance
BOMs
profiles
plugin execution
repository mirrors
settings.xml
Maven Wrapper
```

## Level 6 — Maven internals

Then investigate:

- Maven Resolver
- lifecycle mapping
- effective model building
- plugin class realms/classloaders
- reactor graph/build order
- toolchains
- build extensions
- repository metadata
- snapshot resolution

This order prevents you from getting lost in internals before you have the core model.

---

# 59. Quick Reference Cheat Sheet

## What is Maven?

```text
Java build/project-management tool
+ dependency management
+ lifecycle
+ plugins
+ repositories
+ packaging
+ publishing
+ multi-module orchestration
```

## Main project file

```text
pom.xml
```

## Default local repository

```text
~/.m2/repository
```

Windows:

```text
C:\Users\<username>\.m2\repository
```

## Main standard layout

```text
src/main/java
src/main/resources
src/test/java
src/test/resources
target
```

## Coordinates

```text
groupId:artifactId:version
```

## Main lifecycles

```text
default
clean
site
```

## Important phases

```text
validate
compile
test
package
verify
install
deploy
```

## Local build vs repository

```text
mvn clean
    -> cleans project target output

mvn install
    -> puts project artifact into ~/.m2/repository
```

## Remote publication

```text
mvn deploy
    -> publishes artifacts to a configured remote repository
```

## Dependency inspection

```bash
mvn dependency:tree
```

## Effective POM

```bash
mvn help:effective-pom
```

## Effective settings

```bash
mvn help:effective-settings
```

## Debug

```bash
mvn -X verify
```

## Offline

```bash
mvn -o verify
```

## Update checks

```bash
mvn -U verify
```

## Maven version

```bash
mvn --version
```

## Wrapper

```bash
./mvnw verify
```

Windows:

```powershell
mvnw.cmd verify
```

---

# 60. Official Documentation Links

Keep these bookmarked. They are more useful than memorizing every Maven feature.

## Maven fundamentals

- [Apache Maven Home](https://maven.apache.org/)
- [What is Maven?](https://maven.apache.org/what-is-maven.html)
- [Maven Documentation](https://maven.apache.org/guides/)
- [Maven in 5 Minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes)
- [Getting Started](https://maven.apache.org/guides/getting-started/)

## POM and project structure

- [Introduction to the POM](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)
- [POM Reference](https://maven.apache.org/pom.html)
- [Standard Directory Layout](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)
- [Naming Conventions](https://maven.apache.org/guides/mini/guide-naming-conventions.html)

## Lifecycle and plugins

- [Build Lifecycle](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)
- [Configuring Plugins](https://maven.apache.org/guides/mini/guide-configuring-plugins.html)
- [Maven Plugin Documentation](https://maven.apache.org/plugins/)

## Dependencies and repositories

- [Dependency Mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)
- [Repositories](https://maven.apache.org/guides/introduction/introduction-to-repositories.html)
- [Local Repositories](https://maven.apache.org/repositories/local.html)
- [Repository Layout](https://maven.apache.org/repositories/layout.html)
- [Artifacts](https://maven.apache.org/repositories/artifacts.html)

## Configuration

- [Settings Reference](https://maven.apache.org/settings.html)
- [Build Profiles](https://maven.apache.org/guides/introduction/introduction-to-profiles.html)
- [Toolchains](https://maven.apache.org/guides/mini/guide-using-toolchains.html)
- [Maven Wrapper](https://maven.apache.org/tools/wrapper/)

## Large projects

- [Multiple Modules](https://maven.apache.org/guides/mini/guide-multiple-modules.html)

## Environment / ecosystem

- [Maven Installation](https://maven.apache.org/install.html)
- [Maven Downloads](https://maven.apache.org/download.cgi)
- [Maven Release History](https://maven.apache.org/docs/history.html)
- [Maven FAQ](https://maven.apache.org/general.html)
- [Spring Framework](https://github.com/spring-projects/spring-framework)
- [Spring Framework source repository](https://github.com/spring-projects/spring-framework)

## IDE integration

- [IntelliJ IDEA Maven Support](https://www.jetbrains.com/help/idea/maven-support.html)
- [JetBrains Guide: Importing a Maven Project](https://www.jetbrains.com/guide/java/tutorials/working-with-maven/importing-a-project/)
- [Eclipse m2e](https://eclipse.dev/m2e/)

## CI/CD

- [GitHub Actions — Java with Maven](https://docs.github.com/en/actions/tutorials/build-and-test-code/java-with-maven)

---

# Final Mental Model

When you forget Maven months from now, come back to this diagram:

```text
                          YOU / IDE / CI
                               |
                               | mvn verify
                               v
                     +----------------------+
                     | Maven / Maven Wrapper|
                     +----------------------+
                               |
                    reads project + settings
                               |
                +--------------+--------------+
                |                             |
                v                             v
             pom.xml                    settings.xml
                |                             |
                +--------------+--------------+
                               |
                               v
                    EFFECTIVE PROJECT MODEL
                               |
              +----------------+----------------+
              |                                 |
              v                                 v
        Dependency graph                    Lifecycle
              |                                 |
              v                                 v
      local ~/.m2/repository              phases
              |                                 |
              | missing?                        v
              +----> remote repos           goals
                       |                       |
                       +---- download --------+
                                              |
                                              v
                                         plugin execution
                                              |
                         +--------------------+------------------+
                         |                    |                 |
                         v                    v                 v
                      compile               test             package
                         |                    |                 |
                         +--------------------+-----------------+
                                              |
                                              v
                                           target/
                                              |
                         +--------------------+------------------+
                         |                                       |
                         v                                       v
                 mvn install                              mvn deploy
                         |                                       |
                         v                                       v
                ~/.m2/repository                         remote repository
```

The deepest useful summary is:

> **Maven turns a declarative project description into a repeatable build by combining a project model, dependency graph, lifecycle, plugins, and repositories.**

And for the `.m2` part specifically:

> **`~/.m2/repository` is Maven's local artifact repository. Maven checks it before going to configured remote repositories, stores downloaded artifacts there for reuse, and also uses it as the local installation location for artifacts produced by `mvn install`.**

That is the key mechanism behind the familiar experience of:

```text
First build:
    download many artifacts

Later builds:
    reuse local artifacts
    download only what is new/needed
```

---

## Notes for Spring Core exploration

When you start reading Spring Framework's Maven metadata, repeatedly ask these questions:

```text
1. What is this module's groupId/artifactId/version?
2. Who is its parent?
3. What is inherited from the parent?
4. Which dependencies are direct?
5. Which versions come from dependencyManagement/BOMs?
6. Which dependencies are transitive?
7. Which profiles are active?
8. Which plugins are doing the actual build work?
9. Which lifecycle phase is currently executing?
10. Where would Maven look for the required artifact in ~/.m2/repository?
11. If it is missing, which remote repository/mirror will be used?
12. What will `mvn dependency:tree` reveal?
13. What will `mvn help:effective-pom` reveal?
14. What ends up in `target/`?
15. What gets installed by `mvn install` and where?
```

Those questions turn a large Maven/Spring build from a wall of XML into a system you can reason about.

---

*This file is intended as a living GitHub reference. Maven behavior can evolve, especially around Maven 4, plugins, and repository implementation details, so use the linked Apache Maven documentation as the final authority when a detail matters to a specific version.*
