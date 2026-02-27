Name lookup

Name lookup is the procedure by which a name, when encountered in a program, is associated with the declaration that introduced it.

For example, to compile std::cout << std::endl;, the compiler performs:

unqualified name lookup for the name std, which finds the declaration of namespace std in the header <iostream>
qualified name lookup for the name cout, which finds a variable declaration in the namespace std
qualified name lookup for the name endl, which finds a function template declaration in the namespace std
both argument-dependent lookup for the name operator<<, which finds multiple function template declarations in the namespace std, and qualified name lookup for the name std::ostream::operator<<, which finds multiple member function declarations in class std::ostream.
For function and function template names, name lookup can associate multiple declarations with the same name, and may obtain additional declarations from argument-dependent lookup. Template argument deduction may also apply, and the set of declarations is passed to overload resolution, which selects the declaration that will be used. Member access rules, if applicable, are considered only after name lookup and overload resolution.

For all other names (variables, namespaces, classes, etc), name lookup can associate multiple declarations only if they declare the same entity, otherwise it must produce a single declaration in order for the program to compile. Lookup for a name in a scope finds all declarations of that name, with one exception, known as the "struct hack" or "type/non-type hiding": Within the same scope, some occurrences of a name may refer to a declaration of a class/struct/union/enum that is not a typedef, while all other occurrences of the same name either all refer to the same variable, non-static data member, or enumerator, or they all refer to possibly overloaded function or function template names. In this case, there is no error, but the type name is hidden from lookup (the code must use elaborated type specifier to access it).

Types of lookup
If the name appears immediately to the right of the scope resolution operator :: or possibly after :: followed by the disambiguating keyword template, see

Qualified name lookup
Otherwise, see

Unqualified name lookup
(which, for function names, includes Argument-dependent lookup)
Defect reports
The following behavior-changing defect reports were applied retroactively to previously published C++ standards.

DR	Applied to	Behavior as published	Correct behavior
CWG 2063	C++98	"struct hack" did not apply in class scope (breaks C compatibility)	applied
CWG 2218	C++98	lookup for non-function (template) names could not associate
multiple declarations, even if they declare the same entity	allowed
See also
Scope
Argument-dependent lookup (ADL)
Template argument deduction
Overload resolution
