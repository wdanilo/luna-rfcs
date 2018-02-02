___
- **Feature Name:** Embedded Syntax Architecture
- **Start Date:** 2018-02-02
- **Change Type:** Non-Breaking
- **RFC Dependencies:** Metaprogramming
- **RFC PR:** 
- **Luna Issue:** 
- **Implemented:** 

# Summary
Luna is positioned to become a scientific computing powerhouse, but it can't do
that without easy interoperability with other systems and languages. This 
proposal specifies a universal system for integrating other languages, 
libraries, and runtimes into Luna programs in a way that is flexible, and open
to extension. 

# Motivation
This, is the Embedded Syntax Architecture, or ESA, and it allows the integration
of code from other languages almost like a notebook. It provides systems for
marshalling values into and out of Luna code, and thus greatly extends the 
number of systems where Luna is useful. 

Such a system may be reminiscent of the Jupyter Notebook, which uses kernels to
work with myriad different languages. The ESA, however, goes beyond this, 
providing a 'universal FFI', a system that allows for marshalling data in and
out of Luna, and hence between all these different languages. 

The system only asks that you have the toolchain for the languages you use 
installed and available on your system, and in a place that Luna's ESA plugin
for that language can find it. All the rest is up to Luna and its runtime. 

The main reason that the ESA is useful is that it enables Luna to become a true
scientific computing powerhouse and, beyond that, a true powerhouse for systems
in general. Even if Luna doesn't support a library you want, it can support a
language that does, making it the most flexible program possible for whatever
task you might want to use it for. 

As it is today, Luna exists in a space of limited opportunity. It is a 
fantastically innovative language, but is hampered by lack of libraries. While
the Haskell FFI allows for some integration with existing libraries, the ESA
goes beyond that to allow integration of any language for which someone writes
an ESA plugin. 

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
This is the technical portion of the RFC and should be written as a 
specification of the new feature(s). It should provide a syntax-agnostic 
description of the planned feature, and include sufficient detail to address the
following points:

- Impact on existing feature(s) or functions.
- Interactions with other features.
- A clear description of how it should be implemented, though this need not 
  contain precise references to internals where relevant.
- An explanation of all corner cases for the feature, with examples.

This section should be written in comprehensive and concise language, and may
include where relevant: 

- The grammar and semantics of any new constructs. 
- The types and semantics of any new library interfaces.

## Textual Syntax Explanation
This should explain how the new feature interacts with the textual syntax 
representation of Luna. It should touch on the same categories as above.

## Graph Syntax Explanation
This should explain how the new feature interacts with the graphical syntax for
Luna. It should also touch on the same categories as above.

# Drawbacks
A description of why we _should not_ do this. Write this section as if you are
picking apart your proposal.

# Rationale and Alternatives
A few paragraphs addressing the rationale for why this design is the best 
possible design for this feature in its design space. It should address how the
proposed design resolves the motivating factors. 

A few paragraphs addressing alternative designs for the feature, and the reasons
for not choosing them.

A paragraph or two addressing the impact of not including this feature. 

# Unresolved Questions
This section should address any unresolved questions you have with the RFC at 
the current time. Some examples include:

- What parts of the design will require further elaboration before the RFC is 
  complete?
- Which portions of the design will need resolution during the implementation
  process before the feature is made stable.
- Are there any issues related to this RFC but outside its scope that could be 
  addressed in future independently of this RFC? If so, what are they?
