___
- **Feature Name:** Dependent Types
- **Start Date:** 2018-02-04
- **Change Type:** Non-Breaking 
- **RFC Dependencies:** Interfaces
- **RFC PR:** 
- **Luna Issue:** 
- **Implemented:** 

# Summary
Luna's type system is already a category-based dependently-typed system, but so
far that property hasn't been exposed to the user. This proposal aims to start
discussion of how best we can give users control over the type-system's power. 

# Motivation

# Guide-Level Explanation

# Reference-Level Explanation
Luna already has a dependently-typed type system. We want to expose this to the
users, allowing:

- Proofs of properties of types.
- No distinction between types and terms
- Computation of Types
- Pi and forall quantification over types.

I feel like type expressions should be computable as part of a function's body,
to avoid the issues that Idris has with significant expressions at the type 
level. 

Idris' design should be looked at for inspiration for things like named 
implicits and the like. 

## Textual Syntax Explanation

## Graph Syntax Explanation

# Drawbacks

# Rationale and Alternatives

# Unresolved Questions
This section should address any unresolved questions you have with the RFC at 
the current time. Some examples include:

- We want to expose it to the user, but pi and forall quantification shouldn't
  be required in types unless needed by the signature.
- What kind of syntax do we want?
