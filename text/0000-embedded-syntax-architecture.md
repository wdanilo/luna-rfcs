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

# Reference-Level Explanation
This RFC introduces the concept of a 'Syntax Plugin' to the Luna ecosystem. Each
syntax plugin provides the following capabilities to the Luna ecosystem when in
use:

- Users are able to natively write code from the language in question into their
  Luna projects.
- See this code in-line as if it is a part of the project, and pass data between
  Luna and the embedded code as if it was Luna code. 

A syntax plugin is a piece of software that plugs into the Luna toolchain, and
provides the following functionality:

- Marshalling of data between the embedded language and Luna via the use of the
  to-be-specified ESA API. 
- Establishing the conventions for calling into and returning from the embedded 
  language.
- Marshal the embedded language toolchain, whether that be embedded locally or
  remotely (e.g. running heavyweight DL computations in C++ on a remote 
  cluster). 
- Handle static compilation for linking of any build artefacts if possible. 

These plugins would require access to the appropriate toolchain, and hence place 
no burden on Luna itself to support the language. The only things that Luna 
itself must support is the API and tools for marshalling data and calling into
and out of the foreign code. This API should include tools to allow for users
to marshal data in cases where the ESA plugin falls short, but ideally we want
to significantly minimise this friction. 

These ESA plugins would also be able to be used in the static binary. Each ESA
language would have to take a different approach for this, as something like
Python would need the runtime available and operate via the C-FFI, while 
something like C++ would produce a static object for linking into the binary.

The exact shape this design should take is very much up in the air for the 
moment, but I feel it would be an invaluable addition to the Luna ecosystem. 

## Textual Syntax Explanation
From the perspective of the textual syntax, the ESA plugin consists only of a 
block scope, potentially introduced using the metaprogramming system (as this
can be easily implemented using quasi-quotation). This block would then contain
the foreign code, and indications as to how to marshal in and out of it. 

The following is a very sketchy example:

```
embedForeign CPP :: Vec n Float -> Vec m Float -> OwnedPtr (Matrix n m Float):
	
	std::unique_ptr<Matrix<n, m, Float>> processInputs (
		) {
		// Do the stuff. 

		frob(a, b);
	}

	// Another function not part of the public API of the block, but still has
	// been implemented locally.
	float frob (float, float) {
		// Do the thing.
	}
```

## Graph Syntax Explanation
In terms of the graphical syntax, such a block would be seen on the graph as a
single node. The node could either be displayed in its compact view, or, if 
chosen by the user, could be expanded into a larger view to allow for direct 
editing of the embedded code on the graph. 

# Drawbacks

# Rationale and Alternatives

# Unresolved Questions

- Exactly how would ESA plugins tie into the runtime?
- What would the static binary compilation model look like for Luna projects
  that employ ESA plugins? 
