___
- **Feature Name:** New Syntax
- **Start Date:** 2018-??-??
- **Change Type:** Breaking
- **RFC Dependencies:**
- **RFC PR:** 
- **Luna Issue:** 
- **Implemented:** 

# Summary
The syntax of a language is one of its most important features. It is not only a
design choice, it directly affects how easy to use, expressive and powerful the
language is. Bad design introduces confusion and often leads to unreadable code
written in many different styles across projects.
... 

# Motivation

# Overview
This RFC is a big one. It is impossible to touch syntax topics without thinking
about almost every other entity including even the semantics of a variable
assignment. Every single thing can be designed in many different ways. Over the
past years we have learned that the only way which brings us a step closer to a
design that actually works and fits well into all requirements is a design that
bases on a small set of well defined invariants. Searching for invariants should
be always the first step when searching for a complex solution and testing new
ideas agains them is a very efficient way to fast filter bad decisions.


# Design

## Invariants

1. **The textual syntax must play well with visual representation.**
   Luna is visual and textual language. Any rule which doe not fit both worlds
   well is automatically discarded.

2. **Easiness in understanding is more important than minimalism.** 
   Luna targets a broad range of developers and domain experts. Thus it should
   be fast to write, comfortable to read and should provide easy to understand
   compile time errors. This is why for example monads in Luna are a special
   entity handled by the compiler.

3. **There should be one (and preferably only one) way to achieve a goal.**
   Consiseness in design of language and its libraries allows people to really
   fast understand new functionalities and create code without surprises. 

4. **Type level syntax = value level syntax.**
   Luna is designed to be a dependent type language. Creating typelevel
   machinery is still an open research field. We can observe how different
   languages approach it and learn from their mistakes. One of the biggest
   mistake that is easy to spot is using different syntax for type and value
   level expressions, like prefixing functions with apostrophe to denote they
   were defined in the "value level scope". Such design makes the language hard
   to refactor and overlapping value / typelevel name scopes make it also hard
   to read and thus error prone.

5. **Small number of rules is better than large.**




## Variables, functions and signatures.

### Problems with the current design

Let's start with exploring problems with the current syntax design.


#### Multiple ways to define a variable and its type signature.

Consider the following, valid Luna code:

```haskell
def foo :: Int
def foo: 15
```

It's completely valid, because its a definition of function without arguments.
Function without arguments returning a pure value is obviously the same as just
the value, thus the above code is the same as the following:

```haskell
foo :: Int
foo = 15
```

If both syntaxes are valid, then the following codes are also valid:

```haskell
def foo :: Int
foo = 15

foo :: Int
def foo = 15
```

Which clearly shows problems with function definition syntax. It's important to
note that this design doesn't match the {invariant-3}.

Moreover, it's important to note that this is true only for pure computations. Any monadic computation has different semantics when used with `=` symbol or `def` keyword. For example the following definitions behave differently.

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

This code does not print "hello" any time:
```haskell
def main:
    foo = @ print "hello"
    foo
    foo
    foo
```


#### Magic `->` symbol.

Consider the following simple function:

```haskell
def foo :: Int -> Int
def foo a = a + 1
```

The problem with it is that the type level symbol `->` does not have any value
level counterpart. We can of course assume that it is a special syntax for a
"function signature" constructor, but then its unnecesarly magical.


#### Not specified type level variables scoping.

Consider the following code:

```haskell
def foo :: a -> bar a
def foo: ...
```

We were already talking about this problem multiple times in the past always
leaving it for future refinment when developing dependent types. This problem,
however, is worth considering now, because it is clearly a part of a not well
defined syntax.


#### Non first-class sequential code blocks.

Consider the following PSEUDO-code:

```haskell
result = State.run Map.empty 
    {
    samples.each sample:
        print "Sample `sample`"
        out = runSimulation sample
        State.modify (.insert sample out)
    }
```

It is currently not possible to this code in Luna. You cannot pass a sequential
code block as a function argument. In order to make the code working it should
be currently refactored to: 

```haskell
def helper:
    samples.each sample:
        print "Sample `sample`"
        out = runSimulation sample
        State.modify (.insert sample out)

result = State.run Map.empty helper
```

This design could be considered both a problem as well as feature. The need to
refactor shows the prefered way to write a code. On the other side, such
limitation could be considered very severe, especially in simpler examples like:

```haskell
def helper: print 'hi'
[1..100].times helper
``` 

It's also important to note here some design dependencies. Luna currently
supports a construction called a function without arguments, but obviously
doesnt support a lambda without arguments (lambdas without arguments should not
be called lambdas at all). Thus you can use inline lambdas, you just cannot
create code blocks without arguments. For example, the following code is
accepted by current Luna design:

```haskell
[1..100].each x:
    print x
```


### Proposed design.

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