___
- **Feature Name:** User-Defined Operators
- **Start Date:** 2018-01-29
- **Change Type:** Non-Breaking 
- **RFC Dependencies:** 
- **RFC PR:** 
- **Luna Issue:** 
- **Implemented:** 

# Summary
This proposal introduces the ability to properly define infix operators, 
including their associativity and precedence relations. It also introduces the
ability to define prefix operators, useful in special domains. Finally, it 
provides a specification of the definition and usage of mixfix operators in 
Luna. In doing so, it introduces a comprehensive set of tools to round out the
expressive power of Luna, without compromising on the instant readability of
Luna's textual or visual syntaxes. 

# Motivation
The current state of operators in Luna is both underspecified and incomplete.
In order to rectify this situation, this proposal aims to fully specify 
additions to Luna to address the ability to create a variety of operators for
use in Luna programs. 

In particular, this addresses the ability to write programs containing common
functional programming idioms using infix operators (such as the use of `$` for
low precedence function application). It does this through providing the user 
with the ability to specify operator associativity and precedence, the latter 
specified using highly maintainable precedence relations.

Furthermore, it addresses a use-case for the mathematical community, where in 
languages like MATLAB, Octave and Mathematica, the use of infix operators to 
denote common matrix operations such as the inverse (`'` in MATLAB) is often 
seen. This uses a similar precedence system as that proposed for infix 
operators.

Finally, it also aims to properly specify the way that users are able to define
mixfix operators, such as the `if_then_else` operator from the Luna standard
library. Much like the above, this provides a way to specify the operator 
precedence. 

In the status quo, it is possible to write textual infix-like operators and 
postfix-like operators using the object-dot syntax already provided by Luna. 
These already extant expressive features effectively obviate the need for this
proposal to provide additional capabilities for postfix and textual infix 
operators. In essence, it is ensuring that the ecosystem is rounded enough to 
ensure that Luna is an expressive functional programming language.

# Guide-Level Explanation
Luna offers the user the ability to define prefix, infix and mixfix operators as
part of their code. Why just these three, you might ask? Well we've found that
textual infix (`a foo b`) and postfix (`a kg`) can both be emulated using Luna's
object-dot syntax notation (`a.foo b` and `a.kg` respectively).

So how does one define each of these new kinds of operators? Let's take a look
at them, one-by-one.

## Infix Operators
These are the bread and butter of operators in many programming languages. 
Things like `+`, `*` and `/` are all infix operators. They are called 'infix' 
because they are _in_-between their arguments. 

Being able to define infix operators can bring _huge_ benefits to the expressive
nature of your code. If you're defining a data pipeline, for example, where you
want to chain computations, you could create the pipe operator (`|>`). So let's
have a look at the definition of this operator using Luna's textual syntax. 

```
def a |> b:
    assoc = Left
    binds = [> *, > .]

    b a
```

This may all look a bit confusing right now, but don't worry. Let's break it 
down:

1. The first thing you'll notice is the definition of the symbolic operator 
   name: `def a |> b`. Infix operators in Luna can only contain symbol 
   characters, so no, you can't define `a foo b` using this syntax. To declare 
   an infix operator, you use `def`, like a normal function, but you place the
   arguments `a` and `b` _either side_ of the function name (`|>`).
2. Next up, we have the line `assoc = Left`. This defines the _associativity_ of
   our new operator. Associativity determines how expressions containing 
   multiple infix operators are evaluated. Say we have `a |> b |> c`, we want 
   `a |> b`, to evaluate before `b |> c`, otherwise it wouldn't make sense! As a
   result, we define the operator's associativity as `Left`. Luna also supports
   `Right` and `None` for associativity, where the latter will always require
   parentheses to disambiguate. 
3. Next up, you can see the line `binds = [> *, > .]`, and this isn't a bit of
   syntax you'll see anywhere else in Luna, though it might remind you of a list
   literal. You'd be right, by the way. This is the precedence list, and helps
   to determine the order of operations for expressions containing our operator.
   Luna doesn't use the concept of fixed precedences, as we've found them to be 
   hard to maintain. Instead, we use _precedence relations_. This cool device 
   lets you specify your operator's precedence _in relation_ to other operators,
   and the compiler will automatically build a precedence table. Here, we've 
   said that `|>` has greater precedence than `*` (multiplication) and `.` 
   (function composition), but we can define these relations using `>`, `<` and
   `==` in relation to any pre-existing operators in scope!
4. The rest is just the function body!

## Prefix Operators
Now you know how to define infix operators, let's take a look at prefix 
operators. Prefix operators are unary operators (taking one argument) that go
before their argument. There aren't _many_ of these in programming languages, 
but they are in almost _every_ programming language: `-`, the unary minus 
(e.g. `-1`). 

The main benefit for us of being able to define these is for scientific 
computing libraries. MATLAB, for example, uses `'` to denote the transpose of a
matrix, so let's take a look at how this operator can be written below:

```
def ' a:
    binds = [> *, > **]

    # Implement matrix transpose here
```

Having read the section on infix operators, this probably looks pretty familiar.
Let's go through it just in case:

1. The first line, like for infix operators, is the definition of the operator
   name. Here, we define a name for the operator (here it's `'`, and no, you 
   can't make it textual), and also its _single_ argument `a`.
2. `binds = [> *, > **]`, is just as for infix operators. Here we're saying that
   matrix transpose has higher precedence than multiplication and exponentiation
   which, when you think about it, is just right!
3. Again, the rest is just the function body (and no, we're not including it
   here).

## Mixfix Operators
Now, you may well have been familiar with infix and prefix operators, but mixfix
operators could well be new to you! Mixfix operators are cool in that they let
you define your own expressions for the language, where the fixities of the 
parts of the expression all differ. They're not super common, but Luna itself 
uses them to implement some common constructs. You'll use one of them a lot 
`if a then b else c`. So let's take a look at how it's defined.

```
def if_then_else pred trueExpr falseExpr:
    binds = []

    case pred of:
        True:  trueExpr
        False: falseExpr
```

Now this probably looks quite funny. How do we get from `if_then_else` to 
`if a then b else c`. That's all in the magic of the compiler. Let's take a
look:

1. Much like in the above two examples, the first line defines the function 
   name. You might notice, however, that `if_then_else` has segments separated
   by underscores. Now we don't like underscores in Luna, preferring good old
   `camelCase`, so you may intuit that something special is going on! When you
   define a mixfix function, you have to have the same number of 'sections' 
   (separated by underscores), as you do arguments. Each of these sections 
   becomes associated with an argument as part of the expression! This means 
   that the compiler automatically translates an expression of the form 
   `if a then b else c` into `if_then_else a b c`. 
2. Again, just as for infix and prefix operators, we have the binding list. 
   Here we leave it empty to signify that our operator has lowest precedence. 
3. After that we define the implementation of our if expression using Luna's
   `case` construct for pattern matching (and yes, this _is_ the real 
   implementation).

# Reference-Level Explanation
Luna has a philosophy of defining function behaviour inside the function body.
As a result, the specification of operator functions ensures that all details of
the operator's specification are contained within the associated function. 

The first point to address is that all user-defined operators are specified as
Luna functions, allowing them to be easily understood and read. This means that
_all symbolic operators_ are subject to normal name resolution rules, and hence
operators cannot share names (e.g. you cannot define both infix `**` and prefix
`**` operators). However, if you define identically named operators in different
scopes, normal name resolution rules apply (e.g. `A.**` can be used to 
disambiguate operator usages). Please see note [1] in [Notes](#notes) for an
exception to this rule.

Each operator type proposed by this RFC has its name determined from its
declaration syntax. These are as follows:

- **Infix:** `def <arg> <name> <arg>:` -- You must define a binary function 
  where `<name>` consists only of non-textual characters (defined by the 
  expression `isAlpha || isDigit` according to Haskell's `Data.Char`). For 
  example, `def a ** b` defines an infix operator with name `**`.
- **Prefix:** `def <name> <arg>:` -- You must define a unary function where
  `<name>` consists only of non-textual characters defined as above. For example
  `def ~ a` defines an prefix with name `~`. 
- **Mixfix:** `def <name-1>_...<name-N> <arg-1> ... <arg-N>` -- You must define 
  an N-ary function, where each `<nameI>` consists of characters allowed in a 
  normal luna function name. The function name is separated on `_` to determine
  the multiple portions of the function call signature. For example, 
  `def if_then_else a b c` is a definition of the if-then-else operator where 
  `a` is the condition, `b` is the 'true' branch and `c` is the 'false' branch.
  This function is called as follows: `if a then b else c`. 

Each kind of operator under the purview of this RFC must specify the following
attributes in its body:

- **Infix:**
    + `assoc`: An attribute taking value `Left | Right | None` that specifies 
      the associativity of the infix operator.
    + `binds`: A list literal that defines the binding precedence for this
      operator in relation to other operators.
- **Prefix:**
    + `binds`: A list literal that defines the binding precedence for this
      operator in relation to other operators.
- **Mixfix:**
    + `binds`: A list literal that defines the binding precedence for this
      operator in relation to other operators.

These attributes are to be considered globally reserved words in Luna, as 
contextual keywords lead to difficulty in mentally parsing the language.

The single-definition rule is used to disambiguate between prefix function calls
and operator sections. In absence of this rule `~1` could be interpreted as
either a binary operator with its second argument applied, or a prefix operator.
This is because Luna does not require parentheses for operator sections. 

The binding list specified in the attributes above is used to construct a 
precedence relation between operators automatically at compile time. This is far
more maintainable than fixed fixities (as in haskell `infixl 5`, for example).
Each entry in the binding list is a relation between the operator being defined,
and another, already defined operator [2]. It supports the following relational 
operators: `<`, `>` and `==`. For example, the relation `> *` specifies that
the precedence of the new operator is greater than that of `*`. It is considered
a compile-time error to specify two different fixity relations with respect to
an already-defined operator.

Implementation can be briefly summarised as follows, though this is lacking in
any significant technical detail:

1. Functions defining operators should associate a name with the operator symbol
   table in the parser.
2. Uses of operators should be desugared to applications of their function 
   bodies at compile time. 

## Textual Syntax Explanation
This feature can be used from Luna's textual syntax much akin to the definitions
given in the above section. Within the body of each operator, the precedence and
associativity (if present) are defined as values within the function. This can
be seen as follows for an infix function `|>`:

```
def a |> b:
    assoc = Left
    binds = [> *, > .]

    b a
```

## Graph Syntax Explanation
As the textual syntax just uses bindings of values to names for the operator
attributes, the graph syntax will initially create nodes for each of the 
attributes that float free in the graph of the function. This is an adequate
solution for the first iteration of this feature, but could benefit from some
improvement later on in the development process. 

# Drawbacks
While this feature brings additional expressiveness to Luna as a language, it 
does bring some additional complexity to the language syntax. As Luna aims to be
as learnable as possible, it is perhaps the case that this additional syntax
does not bring enough ergonomic benefit to users of Luna to warrant the 
additional teaching burden. 

Furthermore, it requires the reservation of two additional keywords for the
language. While this is not necessarily a problem at an early stage in the 
language development for breakage reasons, both words are conceivably useful as
binder names in user code.

Along the same lines, the syntax used in the list of precedence relations is
entirely new. As a result, it requires extending the parser, thereby increasing
maintenance burden on the compiler developers. 

Finally, the syntax for defining mixfix operators borrows a convention from the
widely-used 'snake_case' coding convention (seen in some C++ and Python 
codebases). While Luna aims to encourage the use of 'camelCase' for function and
variable names, it may prove confusing to a first-time user to be unable to 
define a function using snake case. 

# Rationale and Alternatives
While many designs of the possible designs for adding additional operators to 
Luna would have worked, this is the only design that required the least changes
to existing language features, especially regarding operator sections. 

The design presented above brings the additional expressiveness of operators 
from other functional programming languages to Luna, without requiring in-depth
alterations to syntax, breaking changes, or alterations to the core philosophy 
for the language. 

Concerns about the immediate readability of syntax are paramount for Luna, and 
hence ensuring that users could 'visually parse' the final design was of great
importance. If the proposal allowed visually similar operators with different
fixities (as was considered in prior designs), this visual parsing would require
additional information to perform.

Furthermore, an initial design that allowed prefix and infix operators with the
same name had a similar difficulty. Under such a circumstance, it wouldn't have
been possible to easily _visually_ disambiguate between an operator section and
a unary operator application. If we consider the example of `~a`, from a parsing
standpoint this can be disambiguated by requiring spaces and parentheses `(~ a)`
for a section, but this required a breaking change and compromise to the current
expressive nature of Luna as a language. Furthermore, this change would have 
clashed with the implicit lambda syntax (`.method` => `x:x.method`), which still
would've not required parentheses for disambiguation.

While not including this feature is always an alternative, it seems that the 
slight increase to parser maintenance burden is a small price to pay for the
increase in language expressiveness such a proposal brings. The inclusion of
capable infix operators allows many common functional programming patterns to
be expressed in Luna, while the creation of prefix operators allows for people
coming from the scientific computing and data-visualisation worlds to easily 
adapt to Luna as a language.

# Unresolved Questions
This section should address any unresolved questions you have with the RFC at 
the current time. Some examples include:

- The syntax for defining the operator attributes needs bikeshedding, 
  particularly in the instance of the graph syntax. 
- Concrete implementation details will also need to be nailed down, at least in
  more detail than the provided list.

# Notes

1. Unary negation and minus are exceptions to this name resolution rule, and 
   will undergo resolution via special rules in the parser. 
2. Function application is always of the highest precedence, and the user cannot
   specify a precedence higher than function application. Existing operators 
   must already be in scope.
