___
- **Feature Name:** New Syntax
- **Start Date:** 2018-??-??
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
dependent types, it appears that the syntax is not ready to it, introducing 
massive confusion among users with such constructs as data kind promotion.


# Motivation
Our first approach to Luna syntax was not perfect. It was developed to nicely
play with the graphical representation, be easy to use and expressive, however
after some time of using it we see space for many improvements. The following
document addresses exactly this topic, while carefully taking in consideration
Luna type system development plans. 


# Overview
This RFC is a big one. It is impossible to touch syntax topics without thinking
about almost every other entity including even the semantics of a variable
assignment. Every single thing can be designed in many different ways. Over the
past years we have learned that the only way which brings us a step closer to a
design that actually works and fits well into all requirements is a design that
bases on a small set of well defined invariants. Searching for invariants should
be always the first step when searching for a complex solution and testing new
ideas against them is a very efficient way to fast filter bad decisions.


# Design

The proposed syntax design bases on few principles, fundamental assumptions 
about how the language should look and feel like.  

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




## Explicit type provider

### Current approach problems
The current explicit type provider operator `::` is used by small group of
languages (mostly Haskell related), is not used in math and is harder to type
than just single `:` mark.


### The solution
We propose to replace the double colon `::` symbol with single colon one `:`.
This change collides with current lambda syntax, however a change to lambda
syntax, which will solve this issue, is proposed below.



</br></br>
## Lambda syntax

### Current approach problems

Consider the following simple function:

```haskell
def foo :: Int -> Int
def foo a = a + 1
```

The problem with the definition is that the type level symbol `->` does not have
any value level counterpart, which breaks the {invariant-3}. We can of course
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
## Types and constructors

### Current approach problems

There are two main problems with current solution. Firstly, accessing type
constructors is more complex than we want to be (ideas include using double type
name like `Point.Point` or auto generated smart constructors). Secondly, the
proposals to solve the former issue negatively affect the clearness of other
constructs, like pattern matching.

In order to properly solve the problem, let's carefully analyse needs and use
cases. If we assume that both atomic type names as well as group type names
start with capitalized letter, then we have to allow for some syntax to create
new group types, like the following one:

```haskell
type Foo = Int | String

a = 5 : Foo
```

The pipe (`|`) is an ordinary operator used to construct new group types. It is
very useful in various places, like in-line group type composition. Based on the
"No type/value level syntax difference" principle, the following code is correct
as well:

```haskell
foo = Int | String

a = 5 : foo
```

Thus, we can use both syntax forms to express exactly the same logic, which is
bad. 

Even worse, it seems that such in reality allows for creating functions named
with capitalized first letter. The new group alias have to accept any valid type
level expression, like `type Foo x = if x then Int else String`, so the
following code is also valid:

```haskell
type Sum a b = a + b

main = 
    print (Sum 3 4)
```

Which clearly shows a flow in the design.


### The solution

The distinction between capitalized and uncapitalized names is often used to
disambiguate the meaning of entities, like answering the question "are we
pattern matching against a constructor or a variable?". 

In the Luna type system the one primitive entity which we want to distinguish
for the needs of pattern matching is the atomic type. Atomic types are used as
variants (constructors), the primitive builders of algebraic data types. All the
other entities, including group types are just results of some expressions. In
the light of these facts, a very elegant solution emerges, namely only the atom
type names will be capitalized, while the rest of names (including names of
modules, interfaces, group types or functions) will start with lower case
letter. Consider the following code:

```haskell

type Circle =
    radius = 1

type Rectangle =
    width  = 1
    height = 1

shape a = Circle a | Rectangle a a

shape.area = case self of
    Circle    r   -> math.pi * r ^ 2
    Rectangle w h -> w * h

```

There is an important thing to note here, take a look at the type definition for
`Circle`. If we use capitalized letter in the type definition, we are defining 
a new atomic type. Such type can only list fields with optional default values,
however it cannot be parameterized with type variables nor it cannot provide 
explicit type information for it fields. To learn more, read about how atomic 
and group types work.

We can provide a syntactic sugar for the above code. If we use the type
declaration with lower case name, we are defining new type which can be used as
a module, an interface or an algebraic data type. The code above could be
re-written to:

```haskell

type shape a =
    type Circle =
        radius = 1 : a

    type Rectangle =
       width  = 1 : a
       height = 1 : a  

    area = case self of
       Circle    r   -> math.pi * r ^ 2
       Rectangle w h -> w * h

```

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

We are not convinced that code blocks should become first class citizen.
Definitely more real-life examples are needed to decide if the code will benefit
from them or they will result in lower quality, deeply nested code. In case the
results will be positive and we will need to introduce them, a possible solution
would be to introduce a `do` keyword, which will open a new code block. The
keyword will be optional after `=` and `->` operators. The above examples could
be then rewritten to:

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