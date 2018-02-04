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
installing packages and dependencies. All of this functionality is as automatic
as possible, providing our users with a lightning-fast workflow for creating
and working on Luna packages. 

This RFC's automation treads a fine-line between enabling a diverse range of 
use-cases, and easily supporting the Luna tenet of there being one correct way
to perform a task. It covers package creation and deployment, to user install
management. Without these features, it is likely that Luna will find such duties
taken over by a fractured mess of external tools, thus robbing the community of
the best possible experience. 

# Guide-Level Explanation
Luna provides comprehensive functionality for managing projects and packages 
using the integrated tool called Luna build! Today, we're going to look at a
basic example of how you can create your own project and upload it to the 
package repository.

Creating a Luna project is as easy as typing `luna init test-project`, giving us
a new folder in our current directory called `test-project` containing all we 
need to get started! If you take a look inside, your project will look something
like this:

```
project-name/
 ├─ .luna_project/
 ├─ dist/ 
 ├─ src/
 ├─ test/
 ├─ LICENSE
 └─ .gitignore
```

So we just want to make a basic 'hello, world' application, so all you need to
do is head into the `src/` directory. Inside, you'll find a file called 
`Main.luna`. Now, Luna Build is pretty sophisticated, and based on the presence
of a main function in this file it can work out that we're trying to build an 
application. So all we need to do is open it up, and put in our main:

```
def main:
    print "Hello, world!"
```

We have some code, so let's build it! All it takes is running `luna build` on 
our command line, and we have an executable. You can run it via `luna exec`, and
you'll see our greeting printed to `stdout`!

Now, _real_ projects are a bit more complex than the stalwart hello world, so 
let's say we want to fire off some missiles! We're very fortunate that someone
has written a library just for us `acme-missiles`, providing the module 
`AcmeMissiles`. Now dependency management in Luna is _far_ more automatic than
you might be used to, so just stick with me!

First, we open up our `Main.luna` again, and add an import to the top. We can
then modify our main function to fire some missiles:

```
import AcmeMissiles

def main:
    AcmeMissiles.fire 3
```

Now, type `luna build` again. This is where the magic happens. Luna knows about
the structure of your project, and automatically detects your dependencies. As
part of this build it's going to download and install the latest version of 
AcmeMissiles, and then build your project. We can again run this with 
`luna exec`, and you'll see `Fire! Fire! Fire!` printed to `stdout`.

Now say, for example, that we've added this missile launch to our code, but we
want to be able to check that we're firing _exactly_ the right number of 
missiles. Did someone say a test suite? 

Luna Build includes robust testing support as well, and all you have to do is 
go into the `test/` directory, and add your tests to `TestMain.luna`. Let's open
this up and write a test!

```
def myTest :: Bool
def myTest:
    mainResult = main
    numMissiles = mainResult.split " "
    result = numMissiles.length
    result

def TestMain:
    testResult = myTest
    testResult
```

Let's run `luna test` and see what we get! Luna automatically converts a boolean
output from `TestMain` into a return code on stdout, so we see `0` and we're all
happy! Our code works properly.

You haven't touched your project in a while, but you've heard that there's a new
version of your dependency `acme-missiles` that makes it even more exciting! We
want it, but Luna doesn't automatically upgrade dependencies (getting you all
the reproducible build goodness). In this kind of situation, however, we _want_
the new version! All we have to do is type `luna upgrade acme-missiles`, and 
Luna Build will automatically upgrade us to the latest version! Now if we run
`luna build; luna exec` again, we see the more exciting result:

```
Fire!!! Fire!!! Fire!!!
```

Luna Build can do a lot more than this, and if you fancy learning about all the 
other cool things that it can do, take a read of the
[Luna Book](https://luna-lang.gitbooks.io/docs/), and its section on Luna Build.

# Reference-Level Explanation
This RFC proposes to extend the `luna` command-line interface with a set of 
additional commands to allow for comprehensive, automated project and package
management. 

## Compiler Management
The `luna` command line tool will be extended with an additional sub-commend
`update`. This will automatically download and update the system-wide install
of Luna to the latest available version.

The `luna` command-line tool will be extended with two additional sub-commands
to allow for management of the Luna install. These commands are as follows:

- `update [RELEASE-BRANCH]`: Updates the system installation of Luna to the 
  most recently released version on the specified `RELEASE-BRANCH`. If no branch
  is specified it defaults to stable rather than nightly.
- `reset VERSION`: Resets the system installation of Luna to the specified 
  `VERSION`. 

## Project Management
Luna aims to automate the management of Luna projects as much as possible. As a
result, it provides just a few commands that allow you to perform every task you
need to when it comes to managing and publishing your projects. This RFC 
proposes extending the `luna` command with these following sub-commands:

- `init NAME`: Creates a new project in the current directory using the name 
  `NAME`. The project will be configured to use the latest Luna version, and 
  will have the structure described in [Project Structure](#project-structure).
- `build`: Builds the project. What is built for a given project and how it is
  determined can be found below. As part of this, any dependencies that are not
  installed will be downloaded and installed into the global package database 
  for you, before being linked into your project sandbox.
- `exec [FLAGS]`: Executes the application that has been built, passing the
  provided flags.
- `test`: This will run all the tests contained within the `test/` directory.
- `clean`: This command will remove any artefacts built from the project, 
  including documentation, cached LIR and binaries. 
- `doc`: This command will automatically traverse the source directory and build
  HTML documentation from any of the doc comments found in the sources. 
- `dep`: This command is used for managing project dependencies. While they are
  automatically discovered from the code, this command provides three
  sub-commands to assist with version management:
  + `update [DEPENDENCY]`: Updates all non-frozen dependencies (or the specified 
    `DEPENDENCY`) to the latest version.
  + `freeze DEPENDENCY[-VERSION]`: Freezes the dependency at the current version
    found in `deps.yaml`, or to the specified version.
  + `unfreeze DEPENDENCY`: Unfreezes the version bound on a dependency.
- `login [REPOSITORY]`: Logs you into the default (or specified) repository. 
  User accounts can be associated with organisations on the package repository.
- `logout [REPOSITORY]`: Logs you out of the default (or specified) repository.
- `publish [major] [minor] [REPOSITORY]`: Publishes your project to the default
  (or specified) repository. If neither `major` or `minor` is specified, it will
  be published with a minor version bump. The default project version is `0.1`.
  This command will automatically ask for publish confirmation.
- `retract VERSION [REPOSITORY]`: Retracts a specific version of your project
  from the default (or specified) repository. This can be used for removing 
  broken packages. 

### Project Structure
Luna projects created by Luna build will all have a consistent structure. This
structure will have the following tree:

```
project-name/
 ├─ .luna_project/
 │   ├─ config.yaml
 │   └─ deps.yaml
 ├─ dist/ 
 │   └─ .lir/
 ├─ src/
 │   └─ Main.luna
 ├─ test/
 │   └─ TestMain.luna
 ├─ LICENSE
 └─ .gitignore
```

The directories and files are described as follows:

- `.luna-project/`: This is a hidden configuration directory that contains the
  automatically-generated project configuration. 
  + `config.yaml`: This configuration file is automatically generated, but can
    be edited by the user to specify additional configuration information. The
    options are found in [Project Configuration](#project-configuration) below.
  + `deps.yaml`: This file is not to be edited by the user. It contains a record
    of all of the current project dependencies and their versions. It is kept
    with the project and ensures that any user of the project gets a 
    reproducible build. It contains versions of both library and tool 
    dependencies, and handles external tools for ESA plugins.
- `dist/`: This directory contains any binary artefacts produced as a result of
  building the Luna project.
  + `.lir/`: This directory is a cache of the Luna Intermediate Representation
    for the repository. LIR is versioned, and included with prebuilt packages. 
- `src/`: This is the source directory for the project. It contains a blank
  `Main.luna` that is used as the project entry point.
- `test/`: This directory is used to contain tests for the project. It contains
  a default `TestMain.luna` which is executed as the entry point for running 
  your tests. By default, this will return a success or fail exit code to 
  `stdin` or `stderr`.
- `LICENSE`: A blank license file. If you attempt to publish the project with a
  blank license, an error will be generated. 
- `.gitignore`: A default `.gitignore` set up to ignore Luna project build 
  artefacts.

It should be noted that when creating a Luna project through the GUI, one 
additional file will be created under the project root:
- `.lunaproject`: This file is not for user editing and specifies the project
  workspace configuration.

### Luna Source Files
The true form of a Luna Program is the Luna Intermediate Representation (LIR). 
In this way, Luna just views the source files as a human-readable serialisation
of the LIR. The fact that LIR is the 'true' form of Luna leads to some 
interesting functionality for luna projects, where they offer automatic 
refactoring commands. Luna Build knows about your LIR, and can automate the 
following refactorings:

- `extract [MODULE | FUNCTION | CLASS | INTERFACE]`: Extracts the 
  specified entity to an automatically named file. It automatically performs
  imports and renaming to keep the code working, and names the file 
  appropriately.
- `inline [MODULE | FUNCTION | CLASS | INTERFACE] SCOPE`: Inlines the specified
  entity into the specified `SCOPE`. 
- `rename ENTITY NEWNAME`: Renames the specified entity to `NEWNAME`.

These refactorings can be automated from both the textual and graph editors, 
meaning that Luna users need not think about their project structure. 

### Project Configuration
Most project management systems enforce significant manual configuration on the
user, duplicating effort which need not be. Luna Build takes a different 
approach, inferring as much as possible from project structure. The following 
elements are automatically discovered for Luna Projects:

- **Project Name:** This is automatically discovered from the name of the 
  directory that contains `.luna-project/`.
- **Project Version:** This is automatically tracked by the repository as the
  project and its updates are published.
- **License:** Automatically determined from the `LICENSE` file in the project.
- **Build Type:** Luna uses the presence of a `main` function in `Main.luna` to
  determine whether the project is an executable or a library. If there is no
  main function it is build as a library, and if there is a main function, it is
  built as an executable _and_ library.
- **Exposed Modules:** This is automatically discovered from the code structure.
  All modules in project `Foo` that should be internal only should be inside a
  module named `Internal` (e.g. `Foo.Internal.Bar`). The Luna compiler 
  automatically enforces that you cannot use `Internal` modules from other 
  projects unless you pass the `-d --debug` flag to the compiler. 
- **Source Directory and Test Directory:** Luna automatically finds sources in
  `src/` and tests in `test/`.

The following overrides can be placed in the `config.yaml`, and will override 
the default behaviour where relevant.

- `synopsis`: A short, one-line description of the package.
- `description`: Detailed package description supporting Luna doc-comment 
  syntax.
- `luna-version`: This field can pin the Luna version with which a project can
  be built. 
- `luna-options`: Options to pass to the Luna compiler (e.g. `-O3`)

### Dependency Management
Each Luna project is automatically created in a sandbox. When you import a 
dependency within your code, Luna Build will automatically pull down and install
the latest compatible version of that dependency into your project, listing it
in the `deps.yaml` file. 

All dependency resolution is done based on version constraints for projects in
the repository, and any information from manually frozen versions is used in 
this resolution process. 

## Package Management
In addition to the automated functionality for dependency management provided by
Luna Build, the tool also enables users to install and download packages 
independently of this. This additional featureset is intended for those who want
to work on existing packages, or even to install Luna tools from the central
repository. 

This proposal also extends the `luna` command with a set of sub-commands 
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

If executed from within a project, these commands operate on the project's local
view of the package database.

### The Global Package Database
The Luna install maintains a global package database that keeps versioned copies
of all packages and tools installed onto the system by Luna Build. When a 
project sandbox requires one of these packages, it is symlinked into the 
project, thus avoiding duplication of packages in the system.

The database knows the difference between package versions, and hence can 
contain multiple versions of the same package in use by different projects on 
the system. 

### Package Repositories
Luna will provide a centralised repository and tooling for managing and 
publishing Luna projects. This repository will use an account system to track
package ownership and authorship, and will provide prebuilt package 
distributions. 

However, Luna will also provide the tooling (the RepoManager) API to allow 
people to use the same accounts system to host their own package repository for
Luna packages (e.g. in a corporate setting). Such repositories can be source
distributions, binary distributions or a combination of the two. 

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

- Should we use TOML or YAML for the configuration files?
- How much of this functionality can we inherit from existing tools (e.g. Stack
  or Cabal). It would be worth talking to Michael Snoyman about this. 
- What algorithm and system should we use for dependency resolution?
