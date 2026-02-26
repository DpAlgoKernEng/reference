Basic concepts

This section provides definitions for the specific terminology and the concepts used when describing the C programming language.

A C program is a sequence of text files (typically header and source files) that contain declarations. They undergo translation to become an executable program, which is executed when the OS calls its main function (unless it is itself the OS or another freestanding program, in which case the entry point is implementation-defined).

Certain words in a C program have special meaning, they are keywords. Others can be used as identifiers, which may be used to identify objects, functions, struct, union, or enumeration tags, their members, typedef names, labels, or macros.

Each identifier (other than macro) is only valid within a part of the program called its scope and belongs to one of four kinds of name spaces. Some identifiers have linkage which makes them refer to the same entities when they appear in different scopes or translation units.

Definitions of functions include sequences of statements and declarations, some of which include expressions, which specify the computations to be performed by the program.

Declarations and expressions create, destroy, access, and manipulate objects. Each object, function, and expression in C is associated with a type.
