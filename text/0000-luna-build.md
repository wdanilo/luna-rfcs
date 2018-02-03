___
- **Feature Name:** Luna Build
- **Start Date:** 2018-02-03
- **Change Type:** Non-Breaking
- **RFC Dependencies:**
- **RFC PR:** 
- **Luna Issue:** 
- **Implemented:** 

# Summary
The success of a language is often inextricably tied to the way that you manage
projects and packages in it. Luna wants to have one way to do things, and so
this proposal introduces a combination package manager and build tool that 
operates as part of the Luna toolchain. This tool is Luna Build.

# Motivation
An effective system for managing projects and package downloads is central to
the success of any language. However, it is often the case that a language 
without an opinionated solution, or an officially endorsed solution ends up with
a proliferation of alternatives. This leaves users of the language segmented,
with different projects using disparate build systems (e.g. Maven and Gradle in
the Java community). This fragmentation story is often repeated, but in 
situations where the community has rallied around a single tool (e.g. Cargo for
Rust), the ecosystem turns out to be far more coherent.

Far from wanting Luna's ecosystem to suffer the same fate as Java or C++, this 
proposal introduces an upgrade to the Luna toolchain that provides an integrated
package management and build system usable directly from the `luna` 
command-line. 'Luna Build', as this RFC terms it, provides functionality for
managing projects and dependencies, creating and manipulating sandboxes, and 
installing packages and dependencies. 

In doing so, this RFC supports a diverse range of use-cases, from package 
development and deployment to user-side package and install management. Without
this, it is likely that Luna will find such duties taken over by a fractured 
mess of external tools, robbing the community of the best-possible experience.

# Guide-Level Explanation
Explain the proposal as if teaching the feature(s) to a newcomer to Luna. This
should usually include:

- Introduction of any new named concepts.
- Explaining the feature using motivating examples.
- Explaining how Luna programmers should _think_ about the feature, and how it
  should impact the way they use the language. This should aim to make the 
  impact of the feature as concrete as possible.
- If applicable, provide sample error messages, deprecation warnings, migration
  guidance, etc.
- If applicable, describe the differences in teaching this to new Luna
  programmers and experienced Luna programmers.

For implementation-oriented RFCs (e.g. compiler internals or luna-studio 
internals), this section should focus on how contributors to the project should
think about the change, and give examples of its concrete impact. For 
policy-level RFCs, this section should provide an example-driven introduction to
the policy, and explain its impact in concrete terms.

# Reference-Level Explanation
This RFC proposes a multi-pronged extension to the command-line `luna` interface
to integrate the build tool and package management features into the already
extant Luna command-line application. This reference of the new features breaks
these extensions down into categories based on the intended use of their 
features. 

## Compiler Management
The `luna` command line tool will be extended with an additional sub-commend
`update`. This will automatically download and update the system-wide install
of Luna to the latest available version.

## Sandboxing
The `luna` command line tool with be extended with an additional sub-command
`sandbox`. This sub-command will support the following sub-commands:

- `init [VERSION] [PATH]`: Initializes a new sandbox in the current directory, 
  or, if provided, in the `PATH` given. This will download and install an 
  isolated version of the Luna compiler to the sandbox, using `VERSION` if 
  specified. 
- `update [VERSION]`: Updates the luna compiler installed in the sandbox to the
  latest version or, if specified, to `VERSION`. 
- `delete PATH`: Removes the sandbox specified by `PATH` and all its contents.

All other `luna` commands will operate inside the sandbox, on local copies of 
the Luna compiler and package database.

Sandboxes are intended to isolate a project or build into a truly reproducible 
state. A sandbox has an isolated and fixed version of the Luna compiler, and all
packages installed into a sandbox are visible only within that sandbox. 

## Build Management
The `luna` command-line tool will be extended with an additional sub-command 
`project`. This command will have additional sub-commands that assist in the
management of Luna projects. These are as follows:

- `new NAME [PATH]`: This creates a new project named `NAME` in the current 
  directory or, if specified, in the directory specified by `PATH`. This project
  will contain the folder structure specified in 
  [Project Structure](#project-structure).
- `build`: This will build the project as specified in the `project-name.toml`.
- `test`: This will run all tests contained within the `test/` directory.
- `clean`: This command will remove any built artefacts from the project, 
  including documentation and binaries. 
- `doc`: This command will automatically traverse the `src/` directory and build
  documentation from any doc comments found in the sources. 

### Project Structure
Luna projects created by Luna build will all have a consistent structure. This
structure will have the following tree:

```
project-name/
 ├─ app/
 ├─ dist/ 
 ├─ src/
 ├─ test/
 └─ project-name.toml
```

The directories and files are described as follows:

- `app/`: This contains any application entry points and non-library-specific 
  code for a Luna application.
- `dist/`: This directory contains any binary artefacts as a result of building
  the Luna project. 
- `src/`: This directory contains the source files for the project, whether it be
  a library or application.
- `test/`: This directory will contain any tests for the project. 
- `project-name.toml`: This file contains user-specified project configuration.
  A non-exhaustive list of these configuration options can be found below in
  [TOML Configuration](#toml-configuration).

It should be noted that when creating a Luna project through the GUI, one 
additional file will be created under the project root:
- `.lunaproject`: This file is not for user editing and specifies the project
  workspace configuration.

### TOML Configuration
Project configuration is specified using the simple markup language 
[TOML](https://github.com/toml-lang/toml). This project configuration is used
to specify package information, dependencies, compiler flags and build 
information. A non-exhaustive list of the sections it supports can be found
below:

- `name`: The project name.
- `version`: The project version.
- `synopsis` and `description`: Short and long project descriptions.
- `author`/`maintainer`: Author information.
- `license`/`license-file`: Licensing information for the project.
- `build-type`: The build type for the package. This instructs Luna whether it
  should be built as a library or executable when installed by the package 
  manager. 
- `luna-version`: The version (or set of versions) of Luna that the project can
  be built with.
- `build-artefacts`: A list of build artefacts from the project. This may 
  include artefacts such as `library`, `executable` and `test-suite`. Each of 
  these may contain a selection of the following sub-fields:
  + `source-dirs`: Source directories to search for this build artefact's 
    sources.
  + `dependencies`: A list of dependencies, specifying both package/tool name 
    and version (or version bounds). 
  + `luna-options`: Any compiler options to use when building the artefact.
  + `build-tools`: The names of any external build tools required on `$PATH` to
    build the project (e.g. `ghc, clang++`). This contains sub-options to 
    specify the build command(s) and arguments for these tools.

## Package Management
In addition to being a comprehensive install management and project build tool,
this proposal also extends the `luna` command with a set of sub-commands 
intended for the management of packages (both libraries and applications) built 
in Luna. These commands allow for the install, management and removal of Luna
packages. The commands are as follows:

- `install NAME[-VERSION]`: Installs the package given by name. If `VERSION` is
  specified it installs the specific version, otherwise the latest version of
  the package is installed. When executing install, Luna Build will build the 
  artefacts for the package if not downloaded from a binary repository. If the
  package provides an executable, the executable will be placed in a directory 
  on `$PATH`. If it contains a library, the library will be made available to
  the Luna compiler. Furthermore, it will also download and install any of the
  dependencies required by the package. 
- `remove NAME[-VERSION]`: Removes the package specified by `NAME` from the 
  package database. If no version is given it will remove all versions of the 
  package with the given name. 
- `download NAME[-VERSION]`: Downloads the package specified package but does 
  not execute any build or install steps.

If executed from within a sandbox, these commands operate on the sandbox's local
package database.

### Package Database
The Luna package database maintains a hash of each package installed by version
and name. That way it is possible to have multiple versions of a given package
(required by different projects) available on the system. 

Furthermore, sandboxes do not have access to the package database, instead 
keeping a local package database and local copies of the package database. This
allows users to carefully manage their dependencies. 

### Package Repositories
While Luna will provide a centralised package repository, this is not to say the
only way to do it. Luna Build will support repositories that provide prebuilt
packages _or_ source-only distributions. This ensures that users can 

The default package repository is specified in the Luna Build configuration file
and alternative repositories can also be stored against a name and specified 
when installing a package. 

## LunaBuild API
As part of the implementation of Luna Build, a public API will be provided that
allows Luna programs to interact with the Luna Build tooling. This API will 
contain functions that perform the same behaviours as all of the above-listed
sub-commands. 

The ability to interface with the build system from Luna code itself would prove
invaluable for self-configuring packages and the like, such as those that can be
found in the python community. 

# Drawbacks
While a comprehensive package management and build-tool infrastructure is 
important to a language, the main drawback of implementing this proposal is the
additional complexity that it brings to the Luna tool. This complexity will 
manifest both in its size and also additional maintenance burden for Luna. 

# Rationale and Alternatives
The design proposed here provides a comprehensive tool for the management of 
Luna projects and packages. In doing so, it maintains the Luna principle of 
having one way to do things, rather than allowing for a proliferation of 
external build tools and package management solutions. 

In integrating this functionality into the `luna` command-line tool itself, we 
provide a one-stop-shop for anyone to get up and running with development in
Luna, much like `stack` can do these days for Haskell. This means that we 
dramatically decrease the effort to get on-board with Luna as a language, which
should aid adoption of the language. One of the most frustrating things about 
certain language ecosystems is the lack of integrated tooling, and so by 
rectifying that we remove a major pain-point to trying a new language in Luna. 

There is, however, one main alternative to this proposal in its entirety. That 
would be to rely on existing tools for the creation of build and package 
management infrastructure for Luna. This is, however, likely to lead to an 
imperfect solution and hence I strongly advocate implementation of this proposal
as a whole. 

# Unresolved Questions

- YAML is an alternative to TOML for configuration files. We should decide on
  which of them better suits the vision for Luna. I chose TOML initially as it
  is much easier to parse with a much less complex specification.
- How much of this functionality can we inherit from existing tools (e.g. Stack
  or Cabal). It would be worth talking to Michael Snoyman about this. 
