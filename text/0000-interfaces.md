___
- **Feature Name:** Interfaces
- **Start Date:** 2018-02-04
- **Change Type:** Non-Breaking
- **RFC Dependencies:** 
- **RFC PR:** 
- **Luna Issue:** 
- **Implemented:**

# Summary
In functional programming languages, types are often key. This is no different 
in Luna. One of the key things that we can use types for is to restrict the 
behaviours that we can perform to reduce the chance of bugs. Interfaces are one
way that we can restrict this behaviour, and this proposal brings them to Luna.

# Motivation

# Guide-Level Explanation

# Reference-Level Explanation
Interfaces in Luna behave much like those in Idris or typeclasses in Haskell,
providing a structured way to control ad-hoc polymorphism. They allow you to 
define a set of methods for a type, including default implementations. They look
like this:

```
interface Show a:
	show :: a -> Text
```

Interfaces in Luna support the following features:

- Default implementations of interface methods. 
- Can be used in restrictions on function signatures (e.g 
  `print :: Show a => a -> Text`).
- Interfaces themselves can be extended with constraints

Interfaces are implemented as follows:

```
class Foo implements Show:
	show Foo = "Foo"
```

Interfaces in Luna are able to be named, allowing for disambiguation (yes I know
this syntax doesn't gel with our operator definitions yet):

```
interface Applicative a:
	(<*>) :: f (a -> b) -> f a -> f b
	pure :: a -> f a
	id :: a

class Int implements Applicative [IntPlus]:
	pure a = a 
	a <*> b = a + b 
	id = 0

class Int implements Applicative [IntMult]:
	pure a = a
	a <*> b = a * b
	id = 1
```

Doing so, we can then disambiguate at the call site as to which implementation 
of the interface should be used `foldl (<*> @IntMult) [1:10]`.

## Textual Syntax Explanation
This should explain how the new feature interacts with the textual syntax 
representation of Luna. It should touch on the same categories as above.

## Graph Syntax Explanation
This should explain how the new feature interacts with the graphical syntax for
Luna. It should also touch on the same categories as above.

# Drawbacks

# Rationale and Alternatives

# Unresolved Questions

- Syntax is far from final
- Features are far from final
- All interface constraints should be computable under dependent typing.
