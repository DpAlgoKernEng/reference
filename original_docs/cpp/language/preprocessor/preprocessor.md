Preprocessor

The preprocessor is executed at translation phase 4, before the compilation. The result of preprocessing is a single file which is then passed to the actual compiler.

Directives
The preprocessing directives control the behavior of the preprocessor. Each directive occupies one line and has the following format:

the # character.
a sequence of:
a standard-defined directive name (listed below) followed by the corresponding arguments, or
one or more preprocessing tokens where the beginning token is not a standard-defined directive name, in this case the directive is conditionally-supported with implementation-defined semantics (e.g. a common non-standard extension is the directive #warning which emits a user-defined message during compilation)(until C++23), or
nothing, in this case the directive has no effect.
a line break.
The module and import directives are also preprocessing directives.

(since C++20)
Preprocessing directives must not come from macro expansion.

#define EMPTY
EMPTY   #   include <file.h> // not a preprocessing directive
Capabilities
The preprocessor has the source file translation capabilities:

conditionally compile parts of source file (controlled by directive #if, #ifdef, #ifndef, #else, #elif, #elifdef, #elifndef(since C++23), and #endif).
replace text macros while possibly concatenating or quoting identifiers (controlled by directives #define and #undef, and operators # and ##).
include other files (controlled by directive #include and checked with __has_include(since C++17)).
cause an error or warning(since C++23) (controlled by directive #error or #warning respectively(since C++23)).
The following aspects of the preprocessor can be controlled:

implementation-defined behavior (controlled by directive #pragma and operator _Pragma(since C++11)). In addition, some compilers support (to varying degrees) the operator __pragma as a non-standard extension.
file name and line information available to the preprocessor (controlled by directive #line).
Defect reports
The following behavior-changing defect reports were applied retroactively to previously published C++ standards.

DR	Applied to	Behavior as published	Correct behavior
CWG 2001	C++98	the behavior of using non-standard-defined directives was not clear	made conditionally-supported
See also
C++ documentation for Predefined Macro Symbols
C++ documentation for Macro Symbol Index
