___
- **Feature Name:** Todo Comments
- **Start Date:** 2018-02-01
- **Change Type:** Non-Breaking
- **RFC Dependencies:** 
- **RFC PR:** 
- **Luna Issue:** 
- **Implemented:** 

# Summary
In many programming languages, todo comments are simple to add, and can be 
parsed by external tools. In Luna, however, the dual-syntax representation means
that they require special consideration. This proposal introduces useful todo 
comments to Luna that can be displayed in both the visual and textual syntaxes.

# Motivation
Often when developing a program, there are things that you leave out in aid of 
getting a basic prototype working. However, so they do not become forgotten, it
is conventional to mark them with a todo comment, often with a link to an 
appropriate ticked, a description of the missing feature, or some kind of 
rationale for leaving the feature out for now.

Luna, as a general-purpose programming language, should have support for this
usage, allowing such comments to be found and listed within projects. It is, 
however, slightly more complex than for a standard textual programming language,
for Luna also has to think about the graphical syntax representation.

Currently, comments in Luna are either _documentation_ or _code disabling_, 
meaning that there is no such thing as a standard 'todo' comment format that
doesn't clash with another type of comment. This RFC introduces such a comment
format, and specifies how it works in both the visual and textual syntax 
representations.

# Guide-Level Explanation
So, you're working on a new feature and you want to get out an MVP as quickly as
possible. As always, this means you leave something out, and you want to add
something to document that fact: that way you won't forget and can come back
later. 

Luna features todo comments for this express purpose. They're really simple to
use, you can use them from both syntaxes, and they work just as you'd expect 
from another programming language. Let's take a look at an example:

```
def myFunc a b:
    # TODO: Frobnicate the arguments

    ...
```

Pretty simple, yeah? The todo comment can have various styles for the word (also
supporting `Todo` and `todo`), and the colon (`:`) is optional. These comments
are associated with their enclosing scope for conversion between Luna's dual
syntaxes, and so often occur at the top of their scope. 

If you type it wrong, for example using a variant of the word 'todo' that isn't
supported, you'll get a helpful error message, and it will fail to parse:

```
Line:Column

    # ToDo: Do the thing.
      ^--^

Unexpected token 'ToDo' found after '#'. Did you perhaps mean one of:
    - # TODO: Do the...
    - # Todo: Do the...
    - # todo: Do the...
```

To create one in the _visual_ syntax, all you have to do is hit tab and start
typing the same text you would for the textual syntax. Upon hitting enter, your
new node will be created, and display the text of the comment to you.

Once you've got these comments in your code, they're super easy to search for in
the textual syntax, and readily visually apparent in the visual syntax.

# Reference-Level Explanation
A todo comment in Luna is classed as a third kind of comment entity, alongside 
the already extant code-disabling comments (`#`), and the documentation comment
(`##`, associated with the following line). Both of these existing forms have 
well-defined interactions with both syntaxes, and so the new todo comment form
must do also.

A todo comment will be parsed by Luna, and be treated as an independent entity
in the Abstract Syntax Graph, associated with the scope in which the todo 
comment occurs. If, for example, the todo comment occurs as follows, within a
function body, it will be associated with and contained within the function's 
scope:

```
def myFunc a b:
    # TODO: Frobnicate the arguments

    ...
```

This principle for associating the scope of todo comments applies to all 
constructs in Luna that introduce a scope -- files, modules, classes, and 
functions -- and should be extended to include any scoped concepts introduced to
the language in the future (e.g. interfaces).

The grammar of the new construct for parsing is as follows (using EBNF syntax), 
where `comment-char` is defined as for existing Luna comments:

```
todo-comment = "#", ("todo" | "TODO" | "Todo"), [":"], {comment-char};
```

This syntax is applicable to both visual and textual syntaxes, and can be 
written directly in both. 

If the todo comment does not match the above syntax, this is a parse error. The
error should provide helpful suggestions and a useful span to the user, as 
displayed in the guide-level explanation above. 

Implementation of this feature requires the following changes to be made to 
Luna:

1. Alterations to the language grammar to introduce the new comment format,
   specified as above.
2. Alterations to the parser to support the new syntax.
3. GUI development to support a new type of node. 

Thankfully, this feature does not have any real corner cases that require 
consideration as it proposes no alterations to the semantics of Luna: the 
addition is purely syntactic.

## Textual Syntax Explanation
Introduction of this comment type to the textual syntax can follow the 
above-specified grammar directly. As mentioned above, any introduced comment 
will be associated with its enclosing scope.

## Graph Syntax Explanation
In the graphical syntax, a todo node can be introduced by creating a new node
prefixed with the same grammar as specified above. These nodes can initially be
free-floating within their enclosing scope, though future UI design could bring
some significant improvements to this. 

Such a node should have the name 'todo', using the same variant of the word that
the user specified in the comment, and the node itself should include the 
comment text. 

However, there is one small issue with the current design for such comments. As
they are free-floating nodes, the conversion to textual syntax is able to place
the textual equivalent anywhere within the parent scope. This means that there
is no true conversion between the syntax representations. This must be resolved
before the RFC can be accepted. 

It is perhaps possible that, like documentation comments, TODO comments are 
associated with the scope of the first non-comment line below which they are 
found, but how this would be represented for the visual syntax is still an 
unresolved question. 

# Drawbacks
The main drawback to introducing this new comment type is the additional
complexity added to the parser. This maintenance burden, however, is limited in 
scope, and should not prove onerous. 

# Rationale and Alternatives
This is essentially the only reasonable design that would work with the 
principles embodied by Luna as a language. Any other design would have 
compromised severely on one or more central tenets of Luna's design. As it is, 
this proposal has one major unresolved question.

Not including such a feature in Luna ends up hampering a very common development
workflow, reducing the appeal of the language to new users. Furthermore, its 
lack of inclusion could lead to users using other comment types (namely the
documentation comment style) to record todo information for parsing by other
tools. This would impact the usability of documentation comments for actually
documenting code. 

# Unresolved Questions
This section should address any unresolved questions you have with the RFC at 
the current time. Some examples include:

- How do we ensure a _truly_ equivalent conversion between textual and visual
  syntaxes?
