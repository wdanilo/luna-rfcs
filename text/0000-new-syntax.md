___
- **Feature Name:** Syntax Overhaul
- **Start Date:** 2018-06-26
- **Change Type:** Breaking 
- **RFC Dependencies:** 
- **RFC PR:** 
- **Luna Issue:** 
- **Implemented:** 


Table of Contents
=================

<!-- MarkdownTOC Style autolink="true" levels="1,2,3" -->

- [Summary](#summary)
- [Motivation](#motivation)
- [Design Principles](#design-principles)
  - [Invariants](#invariants)
- [Type-System](#type-system)
  - [Brief Introduction](#brief-introduction)
  - [Types. Unified Classes, Modules and Interfaces](#types-unified-classes-modules-and-interfaces)
- [New Syntax Reference](#new-syntax-reference)
  - [Naming rules](#naming-rules)
    - [Current problems](#current-problems)
    - [Proposed solution](#proposed-solution)
  - [Type signatures](#type-signatures)
    - [Current problems](#current-problems-1)
    - [Proposed solution](#proposed-solution-1)
  - [Functions](#functions)
    - [Function declaration](#function-declaration)
    - [Named arguments](#named-arguments)
    - [Default arguments](#default-arguments)
    - [Anonymous functions](#anonymous-functions)
  - [Types as Classes](#types-as-classes)
    - [Constructors](#constructors)
    - [Methods](#methods)
    - [Constructors as types](#constructors-as-types)
    - [Type combinators](#type-combinators)
    - [Pattern matching](#pattern-matching)
    - [Polymorphism](#polymorphism)
    - [Generalized type definitions](#generalized-type-definitions)
  - [Types as Modules](#types-as-modules)
    - [Files and modules](#files-and-modules)
    - [Module Examples](#module-examples)
  - [Types as Interfaces](#types-as-interfaces)
    - [Implementing Interfaces](#implementing-interfaces)
  - [Imports](#imports)
    - [Scoping Rules and Code Modularity](#scoping-rules-and-code-modularity)
  - [Anonymous Types](#anonymous-types)
    - [Anonymous Types as Types](#anonymous-types-as-types)
    - [Anonymous Types as Values](#anonymous-types-as-values)
  - [Nested Types](#nested-types)
  - [Example - Dependent Vector](#example---dependent-vector)
  - [Example - Linked List](#example---linked-list)
  - [First-class sequential code blocks.](#first-class-sequential-code-blocks)
    - [Current problems](#current-problems-2)
    - [Proposed solution](#proposed-solution-2)
  - [TODO:](#todo)
  - [Overview of the proposed design](#overview-of-the-proposed-design)
  - [Implementation notes](#implementation-notes)
    - [Types as Generics](#types-as-generics)
    - [To be done](#to-be-done)
- [Unresolved Questions](#unresolved-questions)

<!-- /MarkdownTOC -->



Summary
=======
The importance of a language's syntax cannot be understated. Good syntax is 
concise, expressive and yet easy to use. Badly designed syntax introduces 
confusion, leads to unreadable code, and stands in the way of the language's 
evolution. In addition to being an obvious truth to any programmer, it can be
seen in real world examples: just look at Haskell, whose base syntax is over 20 
years old. These days, with Haskell entering the world of dependent types, it 
has become abundantly clear that its syntax is not able to cope, leading to 
confusion even amongst seasoned language users. 

A language's syntax directly affects how code is structured, and as a result 
affects what is considered to be idiomatic style. Any sensible programming 
language must provide users with the ability to modularise their code and split
it into well-defined parts. The ML family of languages, in particular 1ML and 
OCaml, take the concept of a module even further. Luna, with its unique system
for categorical typing, is in a position to provide the most flexible 
implementation of modules yet, unifying the concepts of modules, classes and
interfaces.



Motivation
==========
The first attempt at creating a syntax for Luna was far from perfect. The focus
of its development was that it played nicely with the language's graphical
representation, but while it was relatively easy to use, we have found it to be
unsatisfactory for the future goals of Luna as a programming language. Moreover,
the current implementation has only a rudimentary module system, a poor state of
affairs for a sophisticated language.

This RFC proposes a wholesale redesign of Luna's syntax. It covers all aspects 
of the language and aims to provide a design that is both unified and 
future-proof. It also addresses the code modularity issue with a _unified_ 
design that combines the notions of modules (in both the conventional and ML
senses), classes, and interfaces under the umbrella of the first-class `type`.

The syntax contained in this proposal supports a variety of diverse use-cases.
It supports the first-class manipulation of types, including the creation of
anonymous types. In doing so, the users of Luna have first-class modularity for
their code, with intuitive mechanisms for working within Luna's type system. 
This unification greatly simplifies Luna as a language, providing one powerful
mechanism that results in many key language features, without requiring more 
from the users than an understanding of `type` and the principles of Luna's type
system.



Design Principles
=================
It is impossible to re-design even small part of the syntax without considering
almost every other design decision. Over the past years we have learned that the
only way which will brings us a step closer to a design that fits well into all
requirements is a design that bases on a small set of well defined invariants.
Invariants derive from a careful analysis of the needs. Their definition should
always be the first step when searching for a solution to a complex problem.
They should be used as a very efficient filter to test new ideas and discovering
bad decisions.

Below we present fundamental assumptions regarding how the Luna language should
look and feel. 



Invariants
----------

1. **The textual syntax must play well with the visual representation.**  
   Both visual and textual representations are equivalently important. Any rule
   which does not fit both worlds at the same time will be rejected.

2. **Easiness in understanding is more important than minimalism.**  
   Luna is meant to be production, not a research language. It targets a broad
   range of developers and domain experts. Thus it should be fast to write,
   comfortable to read and easy to reason about. In particular, it should
   provide easy to understand compile time errors, which is why for example
   monads in Luna are a special entity handled by the compiler.

3. **There should be one (and preferably only one) way to achieve a goal.**  
   One of the greatest power of a good syntax is that it is easy to read by
   different people from different organizations. The more coding styles or
   design pattern rules users have to learn, the more codebases with different,
   often incompatible approaches will appear. In the ideal world, a language
   would provide one and only one way to write and format code and this way
   would be fast to write and easy to understand by people. Luna design should
   be aligned with this vision.

4. **Type level syntax = value level syntax.**  
   Luna type system is designed to be as expressive and as natural to use as
   rest of the code. We believe that the only true solution for next generation
   programming languages is a well designed dependent type system which will
   blend type level and value level computations into a single scope. Creating a
   future proof dependent type system is still an open research field and we can
   observe many different approaches among modern programming languages and
   learn from their mistakes. One of such biggest mistake is using different
   syntax forms or namespaces for type and value level expressions. It leads to
   having special syntactic forms to promote values between the namespaces, like
   prefixing value level data with apostrophe to bring it to type level and
   prevent name clash (see -XDataKinds in Haskell).

5. **Small number of rules is better than large.**  
   Any special case or syntactic rule has to be remembered by the user and
   consumes important cognitive power. On the other hand, the syntax can easily
   be oversimplified, which usually leads to complex, hard to understand errors.
   Usually it is preferred to choose a solution which does not introduce any new
   special cases, or removes some existing ones. 

6. **Predictable behavior.**  
   Predictable behavior is one of the most important principles which separates
   well designed languages from the bad designed ones. A language provides a
   predicable behavior when its user can write code which will not break because
   of some external conditions, like not-dependent code change. A good examples
   of breaking this rule are standard extension methods mechanism (monkey
   patching in Ruby, Python, JavaScript) or orphan overlapping instances in
   Haskell.





Type-System
===========

Brief Introduction
------------------
Luna's type system is based on the notion that each type is a name for a set of
_values_ (denoted by _constructors_). This makes the type system a [Modular
Lattice](https://en.wikipedia.org/wiki/Modular_lattice), if you're interested.
For an example, the type `nat` contains constructors `1, 2, 3, ...`, and is
hence denotable by a set of its constructors. 

As a result, typechecking doesn't work via _unification_ as one might expect if
they are familiar with other functional programming languages, but instead 
checks if a given set of values is a valid substitution for another. We can, of
course, have the empty set (∅), and also sets containing single elements 
(single constructors). 

This notion is supported by an enforced equivalence between value-level and
type-level syntax in Luna, as the compiler makes no distinction between the two.
This means that it is perfectly valid to type `Vector 1 2 3 : Vector 1 2 3`, a
set with a single member. The resultant flexibility is very intuitive, and gives
Luna a very useful type-system in practice, allowing a form of strong,
structural typing.



Types. Unified Classes, Modules and Interfaces
----------------------------------------------
We propose to unify the abstraction of classes, modules and interfaces under a
single first-class umbrella. All of the following functionalities are provided
by a new `type` keyword, resulting in a highly flexible language construct: 

- **Classes.** Types provide containers for data and associated behavior.
- **Modules.** Types provide namespacing for code and data.
- **Interfaces.** Types provide behavior description required of a type.

At a fundamental level, the definition of a new `type` in Luna is the creation
of a (usually named) category of values described by the data and behavior it
possesses. These are first-class values in Luna, and can be created and 
manipulated at runtime. 


#### Why a new keyword?
While it would've been possible to use an existing keyword for this unified
concept, we feel that existing keywords such as `module` and `class` carried too
much baggage from their uses elsewhere. The chosen `type`, however, is very
explicit as it describes exactly what it does in Luna. Furthermore, with this
proposal `module ~ class ~ interface`, and all are members of the type-universe
`Type`, making it an even more appropriate choice. 




New Syntax Reference
====================
This design proposes a major breaking change for Luna, wholesale replacing 
portions of the Language's syntax and semantics with an entirely new model. As a
result, this RFC aims to describe the changes piece-by-piece, to help the 
reader establish a more holistic idea of the design presented here. 



Naming rules
------------

### Current problems <!-- omit in toc -->
```haskell
-- OLD SYNTAX --

class Point:
    x, y, z :: Real
```

Let's consider the above code written using old Luna syntax. It defines a new
class (a new type) and an implicit constructor with the same name as the type.

Currently all types and constructors start with an upper-case letter. This
approach has one major issue, namely accessing constructors is more complex than
it should be. The original idea assumes that we can access the constructor using
qualified name, like `p = Point.Point 1 2 3 :: Point.Point 1 2 3 :: Point`.
Other ideas proposed generating smart constructors starting with lower-case
letter, however such named constructors cannot be easily distinguished from free
variables in pattern expressions.

In order to properly solve the problem, let's carefully analyse all needs and
use cases. If we allow both constructor and type names to start with upper-case
letter, then we have to allow for some syntax to create new type sets, like the
following one:

```haskell
-- OLD SYNTAX --

type Foo = Int | String
a = 5 :: Foo
```

The pipe (`|`) is an ordinary operator used to join type sets. Based on the
`{invariant:3}` the following code is correct as well, because we can refactor
every type level expression to a variable:

```haskell
-- OLD SYNTAX --

foo = Int | String
a = 5 :: foo
```

Thus, without breaking the invariant we cannot guarantee that all type names
will start with an upper-case letter.

Even worse, it seems that such syntax allows creating functions named with
capitalized first letter. The new type alias have to accept any valid type
level expression, like `type Foo x = if x then Int else String`, so the
following has to be accepted:

```haskell
-- OLD SYNTAX --

type Sum a b = a + b
def main: 
    print (Sum 3 4)
```

Which clearly shows a flow in the design.


### Proposed solution <!-- omit in toc -->
The distinction between capitalized and uncapitalized names is often used to
disambiguate the meaning of entities for the needs of pattern matching.

In Luna the entities which we want to distinguish for the needs of pattern
matching are constructors. All the other entities, including set types are just
results of some expressions. In the light of these facts a very elegant solution
emerges, namely only the constructor names will be capitalized, while everything
else (including modules, interfaces, set types or functions) will use
uncapitalized names. The proposed solution has many benefits:

- Using upper case letters for constructors has important benefits. Whenever you
  see an upper case identifier, you know it is a data structure being taken
  apart or being constructed, which makes it much easier for a human to see what
  is going on in a piece of code.

- There are no more ambiguous syntax rules regarding new type / function
  definition.

- Construction and pattern matching is as simple as writing the right name and
  does not require any magic from the compiler.

We present formal definition for constructors and variable names below. They 
will be used in later sections of this document.

```
consName = upperLetter, {nameChar}
varName  = lowerLetter, {nameChar}
```



Type signatures
---------------

### Current problems <!-- omit in toc -->
The type signature operator `::` is used by small group of languages (mostly
Haskell related), is not used in math and is harder to type than just a single
`:` mark.


### Proposed solution <!-- omit in toc -->
We propose to replace the double colon operator `::` with a single colon one
`:`. This change collides with current lambda syntax, however a change to lambda
syntax is proposed in this document as well.




Functions
---------

### Function declaration
#### Current problems <!-- omit in toc -->
The are two problems with the current function definition syntax. The signature
definition starting with the `def` keywords looks awkward and in some situations
there are multiple ways to define the same thing, which leads to confusion and
not intuitive code. Consider the following, valid Luna code:

```haskell
-- OLD SYNTAX --

def foo :: Int
def foo: 15
```

It's completely valid, because its a definition of "function without arguments".
Function without arguments returning a pure value is obviously the same as just
the value, thus the above code could be re-written as:

```haskell
-- OLD SYNTAX --

foo :: Int
foo = 15
```

If both syntaxes are valid, then the following codes are valid as well:

```haskell
-- OLD SYNTAX --

def foo :: Int
foo = 15

foo :: Int
def foo = 15
```

Which clearly shows problems arising from the value definition ambiguity. It's
important to note that this design breaks the {invariant-3}.

It is important to note however, that this situation is true only for pure
computations. Any monadic computation has different semantics when assigned to a
variable or used within a "no-argument function definition". The `=` symbol
evaluates the outer most monad and wraps the result in `Pure` monad, while the
`def` postpones the computation having the same effect like manual postpone
operator (`@`). For example, the following definitions are equivalent:

```haskell
-- OLD SYNTAX --

def foo: print "Hi!"
foo = @ print "Hi!"
```

In order to better understand the evaluation mechanism, please consider the
following examples:

```haskell
-- OLD SYNTAX --

-- This code prints "hello" 3 times:
def main:
    def foo:
        print "hello"
    foo
    foo
    foo

-- This code prints "hello" a single time:
def main:
    foo = print "hello"
    foo
    foo
    foo

-- This code does not print "hello" (the computation is postponed):
def main:
    foo = @ print "hello"
    foo
    foo
    foo
```


#### Proposed solution <!-- omit in toc -->
We propose removing the `def` keyword in favor of using the assignment operator
to define both variables as well as functions. Moreover, values defined in a new
type declaration (every non-nested function) will have postponed effects by
default.

This solution is simple, intuitive and provides only one valid syntax for every
use case. To better understand the concepts, please refer to the following code
examples.

```haskell
-- OLD SYNTAX --

def test :: Text in IO
def test: 
    def mkMsg s: s + '!' 
    print 'Whats your name?'
    name = readLine
    print (mkMsg name)
```

```haskell
-- NEW SYNTAX -- 

test : Text in IO
test =
    mkMsg s = s + '!' 
    print 'Whats your name?'
    name = readLine
    print (mkMsg name)
```


### Named arguments
### Default arguments


### Anonymous functions

#### Current problems <!-- omit in toc -->
Consider the following simple function:

```haskell
def foo :: Int -> Int
def foo a = a + 1
```

The problem with the definition is that the type level symbol `->` does not have
any value level counterpart, which breaks the `{invariant:3}`. We can of course
assume that it is a special syntax for a "function signature" constructor, but
then its unnecessarily magical. Moreover, what value could have an expression 
which type would be expressed as `a : a + 1`?


#### Proposed solution <!-- omit in toc -->
We propose unification of the value level lambda syntax `:` and the type level
arrow syntax `->`. It makes the rules much more consistent. To better understand
the concept, please refer to the following examples:

```haskell
foo : a -> b -> a + b
foo = a -> b -> a + b

-- see the new function definition proposal
bar : x -> x + 1
bar x = x + 1
```

There is however one important thing to note here. The `:` symbol association
rules were deeply magical. Only single variable on the left side was considered
the lambda argument. Such design allows for a very fancy code snippets, but on
the other hand is an exception to all other operator rules. After the
unification, the arrow symbol `->` is just an ordinary operator and all the 
standard association rules apply. Thus the following code snippets are 
equivalent:

```haskell
-- OLD SYNTAX --
out = foo x: x + 1

-- NEW SYNTAX --
out = foo (x -> x + 1)
out = foo x-> x + 1 -- no space = strong association 
```

```haskell
-- OLD SYNTAX --
cfg = open file . parse Config . catch error:
    log.debug "Cannot open `file`: `error`"
    defaultConfig

-- NEW SYNTAX --
cfg = open file . parse Config . catch error->
    log.debug "Cannot open `file`: `error`"
    defaultConfig
```




Types as Classes
----------------
The following chapter describes the replacement for the currently used concept
of _classes_. We have been always dreaming about true dependent typed language
and the way classes currently work stands on the way to achieve the dreams. The
change is, however, not as drastic as it seems. It is rather a process of
extending the current model to provide more fine grained control over the
objects and types.

Luna is an Object Oriented programming language. It provides the notion of
objects and methods so at first glance, Luna types may seem like conventional
_classes_ from traditional object-oriented languages. However, these concepts
differ significantly. Luna types have much more power, yet much simpler design,
disallowing concepts like inheritance in favour of composition and algebraic
data types.

### Constructors
While types in Luna describe categories of values, the constructors are the
values themselves. Constructors are used for defining new data structures
containing zero or more values, so called fields. Formally, constructors are
product types, a primitive building block of algebraic data types.

A constructor definition starts with the `type` keyword followed by the
constructor name and lists its fields by name with possible default values. It
is possible to create unnamed fields by using wildcard symbol instead of the
name. Constructors cannot be parametrized and their fields cannot be provided
with explicit type annotations. The formal syntax description is presented
below.

```
consDef   = "type" consName [{consField}]
fieldName = varName | wildcard
consField = fieldName ["=" value]
```

Below we present code snippets with constructors definitions. Constructors with
the same name are just alternative syntactic forms used to describe the same
entity. We will refer to these definitions in later sections of this chapter.


```haskell
-- Boolean values
type True
type False

-- Structure containing two unnamed fields
type Tuple _ _

-- Alternative Point definitions:
type Point x y z

type Point (x = 0) (y = 0) (z = 0)

type Point x=0 y=0 z=0

type Point 
    x = 0 
    y = 0
    z = 0
```


### Methods
A method is a function associated with a given constructor. The primitive method
definition syntax is very similar to function definition, however it also
includes the constructor in its head:

```haskell
True.not  = False
False.not = True
Point x y z . length = (x^2 + y^2 + z^2).sqrt
Tuple a b . swap = Tuple b a
```

Most often methods are defined in the same module as the appropriate
constructors. Please refer to sections about interfaces and extension methods to
learn more about other possibilities.


### Constructors as types
As Luna is a dependently-typed language with no distinction between value- and
type-level syntax, we are allowed to write _very_ specific type for a given
value. As described earlier, constructors are the values belonging to categories
defined by Luna types. However, they are not only members of categories, they
are also useful to describe very specific categories per se. Formally,
a constructor is capable of describing any subset of the set of all possible
values of its fields. 

For example, the `True` constructor could be used to describe the set of all
possible values of its fields. While it does not have any fields, the set
contains only two value, the `True` constructor itself and an `undefined` value.
Thus it is correct to write in Luna `True : True` and assume that the only
possible values of a variable typed as `a : True` are either `True` or
`undefined`.

On the other hand, The `Point` constructor do contain fields, thus it could be
used for example to describe all possible points, whose first coordinate is an
integral number, while the second and third coordinates are equal to zero: `a :
Point int 0 0`. 


### Type combinators 
The careful reader will notice here, that `int` is a category of all possible
integral numbers, while the numbers are considered constructors themselves. Luna
provides an operator used to join types together, the so called pipe operator.
The hypothetical `int` definition could look like `int = .. | -1 | 0 | 1 | ...`.
We can use this mechanism to easily express even complex type dependencies. For
example we can tell Luna that a particular value has the type of `int | text`.
Luna will allow us to either use pattern matching to discover at runtime which
type are we really dealing with or will allow to use only methods which have
common interface among all constructors described by the type. It will for
example allow us to print such value to the screen.


### Pattern matching
The proposed syntax changes allow us to improve pattern matching rules and make
them much more understandable, especially for new users. As we have described
earlier, there is no need to use qualified constructor names or special cases in
patterns anymore. Moreover, a new form of pattern matching is introduced, the so
called "type pattern matching". 

While constructors allow combining fields into a single structure and type
combinators allow joining types into more general ones, the pattern matching
mechanism allows going the opposite direction. In the most common use case
pattern matching will be performed during runtime, however it is worth to note
that the Luna compiler has enough information to perform pattern matching during
compilation if the appropriate values could be deduced in the compilation
process. There are two forms of pattern matching, namely constructor pattern
matching and generalized type pattern matching. The former syntax is practically
identical to the existing one, while the later uses the `type` keyword to denote
that we are performing pattern matching on type descriptor. Let's see how the
new syntax looks like in practice:

```haskell
type shape a
    type Circle
        radius :: a

    type Rectangle 
        width  :: a 
        height :: a


main = 
    c1 = Circle 5 :: shape int
    v  = if something then c1 else 0

    print case v of
        Circle r   -> 'it is circle'
        type shape -> 'it is other shape'
        _          -> 'it is something else'

    print case v of type
        shape -> 'it is shape'
        int   -> 'it is int'

```


### Polymorphism
Formally polymorphism is the provision of a single interface to entities of
different types. Luna does not provide any special construction to support
polymorphism, because even very complex polymorphic types could be described
just by using type-level functions. Consider the following example code:

```haskell
type Point x y z
point a = Point a a a

main =
    p1 = Point 1 2 3 :: point int
    print p1
```

The `point` function is the most basic form of polymorphic type definition in
Luna. It defines all such sets of points, whose all components belong to the
provided type. To better understand this relation, please consider the following valid expressions:

```haskell
p1 = Point 1 2 3 : Point 1 2 3
p1 = Point 1 2 3 : Point int int int
p1 = Point 1 2 3 : point int

Point 1 2 3 : Point 1 2 3 : Point int int int : point int
```

This is a very flexible mechanism, allowing expressing even complex ideas in a
simple and flexible manner. An example is always worth more than 1000 words, so
please consider yet another example usage:

```haskell 
taxiDistance : point real -> point real -> real 
taxiDistance p1 p2 = (p2.x - p1.x).abs + (p2.y - p1.y).abs + (p2.z - p1.z).abs

main = 
    p1 = Point 1 2 3
    print $ taxiDistance p1
```


### Generalized type definitions
While we can define constructors, methods and compose them to create more
powerful types using the described methods, such definitions require significant
amount of code and do not reflect the real dependencies between the definitions.
This is the reason why Luna provides a syntactic sugar allowing to define
everything we have learned so far in more concise form. 

It is worth emphasizing that generalized type definitions are only a simpler way
to define multiple constructors, combine them into a common type and define
common methods. They do not provide any additional value or functionality. The
generalized type definition syntax is presented below:

```
typeDef = "type" varName [":" interface] [({consDef} | {consField})] [method]
```

The body of a type can contain functions, data, or even _other types_, and _yes_
because you were wondering, types _can_ be defined inductively or using a GADT
style. We can re-write the earlier provided definitions using this form as
follow:

```haskell
type bool
    type True
    type False

    not = case self of
        True  -> False
        False -> True
```

```haskell
type point a
    x y z = 0 : a

    length = (x^2 + y^2 + z^2).sqrt
```

```haskell
type tuple a b
    _ : a
    _ : b

    swap = Tuple b a
```

While using this form we define common methods on a set of constructors, like
the method `not` and we use pattern matching to chose the right algorithm path,
this approach does not have any performance penalties, because the compiler is
provided with enough information to optimize this check away if the value was
known at compile time. 

One important thing to note here is that if you don't define any explicit 
constructors, an implicit one will be generated automatically and will be named 
the same way as the type but starting with an upper-letter instead. Now we can 
use the above definitions as follow:

```haskell
test check = 
    p1 = Point 1 2 3 : point int
    p2 = Point 4 5 6 : point real
    px = if check then p1 else p2
    print px.length
```

**Bonus question**  
What is the most concrete type of the `px` variable above if we do not have any
information about the value of `check`? The answer is of course `px : (Point 1 2
3 | Point 4 5 6)`, which is a sub type of the type `Point (1|4) (2|5) (3|6)`. 




Types as Modules
----------------
The same notion of a type can be used to provide the functionality that is
traditionally expected of a _module_ (in the common, not ML sense). In most
programming languages, their module system provides a mechanism for code-reuse
through grouping and namespacing. Indeed, Luna's types provide both of these functionalities:

- **Grouping of Code**  
  A `type` declaration acts as a container for code, with functions able to be
  declared in its scope. 
- **Namespacing**  
  Unless otherwise declared (through a direct import statement), a `type` in
  Luna also provides a namespace to constructs declared inside its scope.


### Files and modules
Files in Luna should contain at least one `type` definition, with one type named
the same as the file. This `type` is known as the 'primary' type, and it is this
type that is referred to when importing the 'module'. A file `data/map.luna` may
contain `type map`, `type helper` and various other types, but the only things
visible outside the file are the primary type and things defined or imported
into its scope. Inside the file, however, everything can be seen, with no need
to forward-declare.


### Module Examples
The concepts are best illustrated by example. Consider the following type. If it
is imported simply as `import math` (see [Importing Types](#importing-types)),
then `pi` value is only accessible within the scope of `math` (by using
`math.pi`). 

However, please note that `math.pi` is not some kind of a special construct for
a qualified data access. The `math` is a zero-argument constructor of the `math`
module (type). The expression `math.pi` is creating the module object and then
accessing its `pi` field. Of course such creation would be optimized away during
the compilation process.

File `math.luna`:
```haskell
type math
    pi: 3.14
```

File `main.luna`:
```haskell
type main
    import math
    main = print math.pi
```



Types as Interfaces
-------------------
A type in Luna can also act as a 'contract', a specification of the behavior 
expected of a type. The use of types as interfaces in Luna is, as you might 
expect, contravariant. As long as the type satisfies the category defined by
the interface, it can be used in its place. This leads to the expected semantics
where a type `Foo` implementing `Bar` can be used where a `Bar` is expected.

Interfaces in Luna can range from general to very specific. As they define a 
_category_ of values, interfaces can specify anything from function signatures
that must be present, all the way to names that must be present in the type's 
scope and default behavior. The following are all valid ways to define types
for use as interfaces in Luna.

```haskell
-- This interface requires a function called someFunction with the correct sig.
type Interface1
    someFunction : Int -> String

-- This interface requires a function and a variable both named appropriately.
type (a : Numeric) => Interface2 a
    someVar : a

    someFunction : a -> a
    someFunction = ...

-- This interface requires a function foo with the appropriate type.
type Interface3 a = { foo : a -> a }
```

For more information on the last example, please read the section on
[anonymous types](#anonymous-types).

### Implementing Interfaces
TODO: This section needs discussion. It is a very draft proposal for now.

The nature of Luna's type system means that any type that _satisfies_ an 
interface, even without explicitly implementing it, will be able to be used in
places where that interface is expected. However, in the cases of named 
interfaces (not [anonymous types](#anonymous-types)), it is a compiler warning 
to do so. (TODO: Explain why. What bad would happen otherwise?)

You can explicitly implement an interface in two ways. Examples of both can be
found at the end of the section.

1. **Implementation on the Type**  
   Interfaces can be directly implemented as part of the type's definition. In
   this case the type header is annotated with `: InterfaceName` (and filled
   type parameters as appropriate). The interface can then be used (if it has a
   default implementation), or the implementation can be provided in the type
   body. 

2. **Standalone Implementation:**  
   Interfaces can be implemented for types in a standalone implementation block.
   These take the form of `instance Interface for Type`, with any type
   parameters filled appropriately. 

Both of these methods will support extension to automatic deriving strategies in
future iterations of the Luna compiler. 

It should also be noted that it is not possible to implement orphan instances of
interfaces in Luna, as it leads to difficult to understand code. This means that
an interface must either be implemented in the same file as the interface 
definition, or in the same file as the definition of the type for which the
interface is being implemented. (TODO: To be discussed)

Consider an interface `PrettyPrinter` as follows, which has a default 
implementation for its `prettyPrint` method. 

```haskell
type (t : Textual) => PrettyPrinter t =
    prettyPrint : t
    prettyPrint = self.show
```

For types we own, we can implement this interface directly on the type. Consider
this example `Point` type.

```haskell
type Point : PrettyPrinter Text 
    x : Double
    y : Double
    z : Double

    prettyPrint : Text
    prettyPrint = ...
```

If we have a type defined in external library that we want to pretty print, we
can define a standalone instance instead. Consider a type `External`.

```haskell
instance PrettyPrint Text for External =
    prettyPrint = ...
```

<!-- #### On the Semantics of Standalone Implementations
Standalone implementations allow for limited extension methods on types. The
interface methods implemented for a type in the standalone definition can be 
used like any other method on a Luna type. 

#### Overlapping Interface Implementations
Sometimes it is beneficial to allow interfaces to overlap in one or more of 
their type parameters. This does not mean Luna allows _duplicate_ instances (
where all of the type parameters are identical). These can be implemented by
either of the methods above, but the user may often run into issues when 
attempting to make use of these interfaces.

Luna thus provides a mechanism for the programmer to manually specify which 
instance of an interface should be selected in the cases where resolution is
ambiguous. Consider the following example, using the `PrettyPrinter` interface
defined above. 

```
type Point2D : PrettyPrinter Text | PrettyPrinter ByteArray =
    x : Double
    y : Double

    prettyPrint : Point2D -> Text
    prettyPrint self = ...

    prettyPrint : Point2D -> ByteArray
    prettyPrint self = ...

loggerFn (a : PrettyPrinter b) -> Text -> a -> Text
loggerFn msg item = msg <> prettyPrint(Text) item
```

As you can see, the syntax for specifying the instance in the ambiguous case 
uses parentheses to apply the type to the `prettyPrint` function.  -->




## Imports
To go along with the new system proposed in this RFC around code modularity, 
the syntax for dealing with imports has been tweaked slightly. The following
import syntaxes are valid:

- **Direct Imports:** These import the primary module from the file. This brings
  the type and its constructors into scope. For example `import Data.Map` would
  bring `Map` and its constructors into scope.
- **Specified Imports:** These allow the specification of additional functions
  to be brought into the current scope. For example `import Data.Map: fromList`
  would bring `Map`, its constructors and `fromList` into scope.
- **Renamed Imports:** These allow for the programmer to rename the imported
  type. For example `import Data.Containers.Map as MapInterface` brings `Map`
  into scope named as `MapInterface`. Here, constructors are also imported.
- **Specialised Imports:** These allow the programmer to specialise type 
  arguments as part of the import. For example `import Data.Map String` will
  import `Map` and its constructors with their first type arguments specialised
  to `String`.

These above import styles can be combined, for example renaming a partially
specialised import (`import Data.Map String as StringMap`), or specialising
functions imported into scope (`import Data.Map String: fromList`). Much like 
curried type application seen elsewhere in this proposal, it is possible to 
partially apply the type arguments of an import, as seen above. 

<!-- #### The File Scope
Files in Luna should contain at least one `type`, with one type named the same
as the file. This `type` is known as the 'primary' type, and it is this type
that is referred to when importing the 'module'. A file `Data/Map.luna` may 
contain `type Map`, `type Helper` and various other types, but the only things
visible outside the file are the primary type and things defined in its scope.
Inside the file, however, everything can be seen, with no need to 
forward-declare. -->

### Scoping Rules and Code Modularity
Imports in Luna can be performed in _any_ scope, and are accessible from the 
scope into which they are imported. This gives rise to a particularly intuitive
way of handling re-exports. 

Consider the following file `Test.luna`. In this file, the imports of `Thing`
and `PrettyPrint` are not visible when `Test.luna` is imported. However, 
`PrettyPrint` and `printer` are made visible from within the scope of `Test`. This means that a user can write `import Test: printer` and have it work. 

```
import Experiment.Thing
import Utils.PrettyPrint

type Test a : PrettyPrint Text (Test a) =
    import Utils.PrettyPrint: printer 

    runTest : a -> Text
    runTest test = ...

    prettyPrint : Test a -> Text
    prettyPrint self = ...
```

## Anonymous Types
In addition to the syntax proposed above in [Declaring Types](#declaring-types),
this RFC also proposes a mechanism for quickly declaring anonymous types. These 
types are anonymous in that they provide a category of values without applying 
a name to their category, and can be created both as types and as values. 

While it is possible to use the primary type declaration syntax without 
providing an explicit name, this is highly impractical for most places where an
anonymous type becomes useful. This shorthand provides a way to get the same
benefit without the syntactic issues of the former. 

### Anonymous Types as Types
When used in a type context, an anonymous type acts as a specification for an
interface that must be filled. This specification can contain anything from
types to names, and features its own syntax for inline declarations. 

Consider the following examples:

- `{Int, Int, Int}`: This type declares a set of values where each value 
  contains three integers. 
- `{Int, foo : Self -> Int}`: This type declares a set of values with an integer
  and a function from `Self` to an Integer with name `foo`.
- `{Self -> Text -> Text}`: This defines an unnamed function. This may seem
  useless at first, but the input argument can be pattern-matched on as in the
  following example:

    ```
    foo : { Int, Int, Self -> Int } -> Int
    foo rec@{x, y, fn} = fn rec
    ```

`Self` is a piece of reserved syntax that allows anonymous types to refer to 
their own type without knowing its name.

### Anonymous Types as Values
Anonymous types can also be constructed as values using similar syntax. You can provide values directly, which will work in a context where names are not 
required, or you can provide named values as in the following examples:

- `{0, 0}`: This anonymous value will work anywhere a type with two numbers and 
  no other behaviour is expected.
- `{x = 0, y = 0, z = 0}`: This one provides explicit names for its values, and
  will work where names are required.
- `{x = 0, fn = someFunction}`: This will also work, defining the value for `fn`
  by use of a function visible in the scope. 
- `{x = 0, fn = (f -> pure f)}`: Lambda functions can also be used.




## Nested Types
As any kind of language construct can be nested inside a `type`, we are able to
nest types arbitrarily in Luna. This leads to a very expressive language, but
the semantics of such nested types need careful attention.

This RFC proposes that a nested type becomes a separate type held by each 
_instance_ of a type. This means that `foo.A != bar.A`, where `foo : Foo` and 
`bar : Foo`. The benefits of doing this means that you have true associated 
types, rather than just types contained by other types. 

## Example - Dependent Vector
```
type (n : Nat) => Vector n a =
    # Constructor
    vector : (n = 1 : Nat)

    # The parameter name is visible at the 'type level', allowing dependent type
    mkVec : Nat -> Vector n a
    mkVec (n = 1) = ...
```

## Example - Linked List
This example uses currently forbidden syntax for declaring operators, but let's
pretend it works.

```
type List a = 
    nil 
    cons a (List a)

    (:) = cons
    []  = nil
```





## First-class sequential code blocks.


### Current problems <!-- omit in toc -->
Consider the following PSEUDO-code:

```haskell
result = State.run Map.empty 
    {
    samples.each sample->
        print "Sample `sample`"
        out = runSimulation sample
        State.modify (.insert sample out)
    }
```

It is currently not possible to express this code in Luna. You cannot pass a
sequential code block as a an argument. In order to make the code working it
should be currently refactored to: 

```haskell
helper = samples.each sample->
    print "Sample `sample`"
    out = runSimulation sample
    State.modify (.insert sample out)

result = State.run Map.empty helper
```

This design could be considered both a problem as well as a feature. The need to
refactor shows the preferred, modular way to write the code. Arguably such
limitation could be considered too severe, especially in simpler examples like:

```haskell
helper = 
   print "I've got a new number!"
   print "I'm happy now."

100.times helper
``` 


### Proposed solution <!-- omit in toc -->
We are not convinced that code blocks should become a first class citizen.
Definitely more real-life examples are needed to judge if Luna will benefit from
their introduction or they will contribute to lower code quality instead. Deeply
nested code blocks are almost never a good idea and keeping them as separate
functions makes the code more modular and structured. If the language supports
an easy to use syntax for creating such helper functions the problem is almost
non-existent. Moreover, the code blocks need to be supported by the visual
environment and while functions are just nodes, unnamed code blocks are a very
strange entity, which the visual language would not benefit from. 

In case the results will be positive and we will need to introduce them, a
possible solution would be to introduce a `do` keyword, which will open a new
code block. The keyword will be optional after `=` and `->` operators. The above
examples could be then rewritten to:

```haskell
result = State.run Map.empty do
    samples.each sample->
        print "Sample `sample`"
        out = runSimulation sample
        State.modify (.insert sample out)
```

```haskell
100.times do
   print "I've got a new number!"
   print "I'm happy now."
```





## TODO:
- extension methods (including imports to local scope and local scope overriding)
- default arguments (definition, evaluation, explicite currying)
- strictness 
- postponing computations
- dynamic types
- unnamed multiline types

<!-- 
## Overview of the proposed design

Let's discuss the proposed design based on a comparison with the old one:

Sample code using the current syntax design:

```haskell
01 | def inc :: a -> a + 1
02 | def inc a: a + 1
03 | 
04 | def main :: Nothing in IO
05 | def main:
06 |     file = "config.yaml" 
07 |     cfg  = open file . parse Config . catch error:
08 |         log.debug "Cannot open `file`: `error`"
09 |         defaultConfig
10 |         
11 |     samples = (1 .. cfg.maxSamples) . each random . sort 
12 |             . filter (< cfg.maxValue)
13 | 
14 |     result = State.run Map.empty { -- impossible construction
15 |         samples.each sample:
16 |             print "Sample `sample`"
17 |             out = runSimulation sample
18 |             State.modify (.insert sample out)
19 |         }
20 |
21 |     print "Results:"
22 |     results.each [k,v]:
23 |         print "`k`: `v`"
```

Sample code using the proposed syntax design:

```haskell
01 | inc : a -> a + 1
02 | inc = a -> a + 1 
03 | inc a = a + 1 -- sugar
04 | 
05 | main : Nothing in IO
06 | main =
07 |     file = "config.yaml" 
08 |     cfg  = open file . parse Config . catch error->
09 |         log.debug "Cannot open `file`: `error`"
10 |         defaultConfig
11 |         
12 |     samples = (1 .. cfg.maxSamples) . each random . sort 
13 |             . filter (< cfg.maxValue)
14 | 
15 |     result = State.run Map.empty do
16 |         samples.each sample->
17 |             print "Sample `sample`"
18 |             out = runSimulation sample
19 |             State.modify (.insert sample out)
20 | 
21 |     print "Results:"
22 |     results.each [k,v]->
23 |         print "`k`: `v`"
``` -->



Implementation notes
--------------------

Types are modules, but in fact they are just created with pipe operator. We need
to access them at runtime. It seems like it is just a structure with set of 
sub-types, methods etc. To be refined and described here including the dynamic,
runtime representation.






<!-- 

### Types as Generics
To start off, types in Luna can be made _generic_ over other types. This can be
as simple as `Map k v`, where `k` is the type of the key and `v` is the type of
the value, but actually has more sophisticated use-cases. 

One of these is to be able to parametrise your types over different 
implementations of functionality, such as strings. This means that your type can
be generic over anything that implements the required interface, as shown in the
following example:

in `util/logger.luna`:

```
type logger (a : textual) =
    ...
```

in `main.luna`:

```
import data.text
import util.logger text

...
```
This means that in the scope of `main.luna`, any instance of the `logger` type
will use `text` as its underlying implementation. This works because `text` is
an instance of the `textual` interface.

Another useful extension of this is that type arguments to generic types can be
partially applied via currying. If, for example, we have a `map k v`, we can 
produce a `stringMap = map string` just by applying one of the type 
arguments. This is equivalent to explicit specification of the free type 
variable: `stringMap v = map string v`.






It is possible, then, to dynamically determine a map
implementation to use based on runtime data. For example:

```
efficientMap k v = 
    if expectedBuckets > threshold 
    then hashMap k v 
    else treeMap k v

myMap = empty : efficientMap k v
``` -->




### To be done
- Discuss the look and feel of type constraints (`type (n : Nat) => Vector n a`)
- Describe the "GADT" style


# Unresolved Questions
This section should address any unresolved questions you have with the RFC at 
the current time. Some examples include:

- We definitely need further discussion on the situation with constructors and
  pattern matching.
- We definitely need further discussion on nested types.
- Some syntax can likely be cleaned up.

