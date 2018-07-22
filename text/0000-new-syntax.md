___
- **Feature Name:** Syntax Overhaul
- **Start Date:** 2018-06-26
- **Change Type:** Breaking 
- **RFC Dependencies:** 
- **RFC PR:** 
- **Luna Issue:** 
- **Implemented:** 

# Summary
The syntax if one of the most important aspects of a language. Good syntax is
concise, expressive yet easy to use. Bad design introduces confusion, leads to
unreadable code and stands in the way of the language evolution. Think about
Haskell, arguably one of the most powerful languages ever created. It's syntax
was developed over 20 years ago and now, when Haskell enters the world of
dependent types, it appears that the syntax is not ready for it, introducing
massive confusion even among powerful users with such constructs as data kind
promotion.

The syntax directly affects how the code is structured, which is also a topic of
this proposal. Any sensible programming language must provide its users with
some way to modularise their code, splitting it up into well-defined parts. The
ML family of languages, in particular 1ML and OCaml take the concept of a module
further and Luna, with its unique category-based type system, is in a position
to provide the most flexible implementation of modules yet, unifying the
concepts of modules, classes and interfaces. 

# Motivation
Our first approach to Luna syntax was not perfect. It was developed to nicely
play with the graphical representation, be easy to use and expressive, however
after some time of its usage we have discovered many improvement possibilities.
Moreover, at present, Luna has nothing but a rudimentary module system to aid in
the complexity management and scoping of code. For a sophisticated language this
is a bad state of affairs.

This RFC aims to propose a redesign of Luna syntax, which covers almost all
aspects of the language, in order to provide much more unified, future proof
design. Moreover, we present a _unified_ design which unifies modules (in both
the conventional and ML senses), classes, and interfaces under a single
first-class umbrella. 

In doing so, the proposal supports a diversity of use-cases, allowing the 
first-class manipulation of types, including the creation of anonymous types. In
doing so, it provides users with first-class modularity for their code, and 
intuitive mechanisms for working with types in Luna's type system. This concept
thus brings a massive simplification to the Luna ecosystem, providing one 
powerful mechanism to accomplish many key language features, without requiring
the user to understand more than the mechanisms of `type`, and the principle
behind Luna's type system.

In the end, Luna's current module system is insufficient for serious development
in the language, and needs to be replaced. In doing so, this RFC proposes to 
take the time to bring a vast simplification to the language in the same swoop. 


# Overview
This RFC is a big one. It is impossible to touch syntax topics without thinking
about almost every other entity including even the semantics of a variable
assignment. Every single thing can be designed in many different ways. Over the
past years we have learned that the only way which brings us a step closer to a
design that actually works and fits well into all requirements is a design that
bases on a small set of well defined invariants. Searching for invariants should
be always the first step when searching for a complex solution and testing new
ideas against them is a very efficient way to fast filter bad decisions.


# Design principles
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

## Invariants

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


# (Brief) Type-System Primer
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

Luna provides a mechanism to join type sets, the so called pipe operator,
allowing for example defining a type of numbers between 1 and 10 or any textual
value as `type Foo = 1 | .. | 10 | Text`. Such a declaration can, of course,
have constraints and type variables, much like any of the standard type
declarations described later in this document. It cannot, however, provide
explicit lists of implemented interfaces (TODO: explain it).


# Reference-Level Explanation
This design proposes a major breaking change for Luna, wholesale replacing 
portions of the Language's syntax and semantics with an entirely new model. As a
result, this RFC aims to describe the changes piece-by-piece, to help the 
reader establish a more holistic idea of the design presented here. 


</br></br>
## Type signature

### Current approach problems
The type signature operator `::` is used by small group of languages (mostly
Haskell related), is not used in math and is harder to type than just a single
`:` mark.


### The solution
We propose to replace the double colon operator `::` with a single colon one
`:`. This change collides with current lambda syntax, however a change to lambda
syntax is proposed in this document as well.



</br></br>
## Lambda syntax

### Current approach problems

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


### The solution

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
-- OLD
out = foo x: x + 1

-- NEW
out = foo (x -> x + 1)
out = foo x-> x + 1 -- no space = strong association 
```

```haskell
-- OLD
cfg = open file . parse Config . catch error:
    log.debug "Cannot open `file`: `error`"
    defaultConfig

-- NEW
cfg = open file . parse Config . catch error->
    log.debug "Cannot open `file`: `error`"
    defaultConfig
```



</br></br>
## Naming of types and constructors

### Current approach problems

```haskell
-- OLD SYNTAX
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
-- OLD SYNTAX

type Foo = Int | String
a = 5 :: Foo
```

The pipe (`|`) is an ordinary operator used to join type sets. Based on the
`{invariant:3}` the following code is correct as well, because we can refactor
every type level expression to a variable:

```haskell
-- OLD SYNTAX

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
-- OLD SYNTAX

type Sum a b = a + b
def main: 
    print (Sum 3 4)
```

Which clearly shows a flow in the design.


### The solution

The distinction between capitalized and uncapitalized names is often used to
disambiguate the meaning of entities for the needs of pattern matching.

In Luna the entities which we want to distinguish for the needs of pattern
matching are constructors. All the other entities, including set types are just
results of some expressions. In the light of these facts a very elegant solution
emerges, namely only the constructor names will be capitalized, while everything
else (including modules, interfaces, set types or functions) will use
uncapitalized names.




## Declaring Types and methods
We propose to replace the `class` keyword with the `type` keyword. The detailed
rationale behind this decision is presented in the "Unifying Types, Modules and
Interfaces" section later in this document.

Type definition always starts with the `type` keyword. There are two forms of
type definition – constructor definition and set type definition. 


### Why a New Keyword?
While it would've been possible to use an existing keyword for this unified
concept, we feel that existing keywords such as `module` and `class` carried too
much baggage from their uses elsewhere. The chosen `type`, however, is very
explicit as it describes exactly what it does in Luna. Furthermore, with this
proposal `module ~ class ~ interface`, and all are members of the type-universe
`Type`, making it an even more appropriate choice. 


### Constructor definition
The constructor definition syntax expressed in EBNF is presented below:

```
consName  = upperLetter, {nameChar}
varName   = lowerLetter, {nameChar}
consDef   = "type" consName [{consField}]
fieldName = varName | wildcard
consField = fieldName ["=" value]
```

In other words, constructor definitions lists all its fields by name with
possible default values. It is also possible to create unnamed fields by using
wildcard symbol instead of the name. Constructors cannot be parametrized and
their fields cannot be provided with explicit type annotations. All standard
layout rules apply, so the definitions can be distributed over several lines.
All of the following examples show valid constructor definitions:

```haskell
type True
type False
bool = True | False
```

```haskell
type Point x y z
```

```haskell
type Point (x = 0) (y = 0) (z = 0)
point a = Point a a a
```

```haskell
type Point x=0 y=0 z=0
```

```haskell
type Point 
    x = 0 
    y = 0
    z = 0
```

```haskell
type Tuple _ _
```


### Method definition
The most basic method definition syntax is by defining them directly on
constructors:

```haskell
True.not  = False
False.not = True
Point x y z . length = (x^2 + y^2 + z^2).sqrt
Tuple a b . swap = Tuple b a
```

It is also possible to define methods on set types:

```haskell
bool.not = case self of
    True  -> False
    False -> True
```

Most often methods are defined in the same module as the appropriate
constructors. Please refer to sections about interfaces and extension methods to
learn more about other possibilities.



### Generalized type definition
Generalized type definition is a syntactic sugar allowing for an easy way to
define multiple constructors, set types and related methods. The generalized
type definition syntax expressed in EBNF is presented below:

```
typeDef = "type" varName [":" interfaces] [({consDef} | {consField})] [method]
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

type point a
    x y z = 0 : a

    length = (x^2 + y^2 + z^2).sqrt

type tuple a b
    _ : a
    _ : b

    swap = Tuple b a
```

Do not be deceived by the similarity of these definitions and classes known from
object oriented languages. While these concepts share many common ideas, they
also differ significantly. For example, Luna does not have the notion of
inheritance nor super-classes. 

One important thing to note here is that if you don't define any explicit 
constructors, an implicit one will be generated automatically and will be named 
the same way as the type but starting with an upper-letter instead. Now we can 
use the above definitions as follow:

```haskell
main = 
    check = True
    p1 = Point 1 2 3 : point int
    p2 = Point 4 5 6 : point real
    px = if check then p1 else p2
    print px.length
```


### To be done
- Discuss the look and feel of type constraints (`type (n : Nat) => Vector n a`)
- Describe the "GADT" style





## Pattern matching

This proposal introduces also new form of pattern matching, so called "type
pattern matching". Basically, we can pattern match against group types (in
particular interfaces) by using the `type` keyword. We can also use a special
construction of `case x of type ...` to pattern match only on group types. Let's
see how we can use the above definitions together with the pattern matching
mechanism:

```haskell

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

The proposed solution has many benefits:

- Using upper case letters for constructors has important benefits. Whenever you
  see an upper case identifier, you know it is a data structure being taken
  apart or being constructed, which makes it much easier for a human to see what
  is going on in a piece of code.

- There are no more ambiguous syntax rules regarding new type / function
  definition.

- Construction and pattern matching is as simple as writing the right name and
  does not require any magic from the compiler.



</br></br>
## Function definitions 

### Current approach problems

The are two problems with the current function definition syntax. The signature
definition starting with the `def` keywords looks awkward and in some situations
there are multiple ways to define the same thing, which allows creating a very
not intuitive code.  
  
Consider the following, valid Luna code:

```haskell
def foo :: Int
def foo: 15
```

It's completely valid, because its a definition of "function without arguments".
Function without arguments returning a pure value is obviously the same as just
the value, thus the above code could be re-written as:

```haskell
foo :: Int
foo = 15
```

If both syntaxes are valid, then the following codes are valid as well:

```haskell
def foo :: Int
foo = 15

foo :: Int
def foo = 15
```

Which clearly shows problems arising from the value definition ambiguity. It's
important to note that this design breaks the {invariant-3}.

Moreover, it's important to note that this situation is true only for pure
computations. Any monadic computation has different semantics when used with `=`
symbol or "no-argument function definition". The `=` symbol evaluates the outer
most monad and wraps the result in `Pure` monad, while the `def` postpones the
computation having the same effect like manual postpone using the `@` function.
For example the following definitions behave differently.

This code prints "hello" 3 times:
```haskell
def main:
    def foo:
        print "hello"
    foo
    foo
    foo
```

This code prints "hello" a single time:
```haskell
def main:
    foo = print "hello"
    foo
    foo
    foo
```

This code does not print "hello" (the computation is postponed):
```haskell
def main:
    foo = @ print "hello"
    foo
    foo
    foo
```


### The solution

We propose removing the `def` keyword and using `=` sign to define both
variables as well as functions. Moreover, values defined in a new type
declaration (all non-nested functions) will have postponed effects by default.

This solution is simple, intuitive and provides only one valid syntax for 
function definition. To better understand the concepts, please refer to the 
following code examples.

Using the old syntax:

```haskell
def test :: Text in IO
def test: 
    def mkMsg s: s + '!' 
    print 'Whats your name?'
    name = readLine
    print (mkMsg name)
```

Using the new syntax:

```haskell
test : Text in IO
test =
    mkMsg s = s + '!' 
    print 'Whats your name?'
    name = readLine
    print (mkMsg name)
```




</br></br>
## Type level variable scoping

### Current approach problems

Consider the following code:

```haskell
def foo :: a -> bar a
def foo x: ...
```

How do we know that `a` is a free variable and does not refer to a variable
defined earlier in the code? We were already chatting about this problem
multiple times in the past, always leaving it for future refinement when
developing dependent types. This problem, however, is worth considering now,
because it is clearly a part of a not well defined syntax.


### The solution

The solution for this problem appears obvious after the merge of value and type 
level lambdas. Let's write the code using the new syntax:

```haskell
foo : a -> bar a
foo x = ...
```

If the `a -> bar a` is a lambda, then its left side is the pattern matching,
while it's right side is an expression, thus `a` is a free variable, while `bar`
is a name of function from the current scope. If we would like to use some
function from current scope to further refine the `a` variable, we can use all
syntax form valid for pattern matches, for example:

```haskell
foo : (a : testFunc 5) -> bar a
foo x = ...
```




</br></br>
## Non first-class sequential code blocks.

### Current approach problems

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


### The solution

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




</br></br>
## Modules / classes / interfaces 

TO BE MERGED WITH MODULES PROPOSAL




</br></br>
## Imports 

TO BE MERGED WITH MODULES PROPOSAL




</br></br>
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
```