Memory model

Defines the semantics of computer memory storage for the purpose of the C++ abstract machine.

The memory available to a C++ program is one or more contiguous sequences of bytes. Each byte in memory has a unique address.

Byte
A byte is the smallest addressable unit of memory. It is defined as a contiguous sequence of bits, large enough to hold

the value of any UTF-8 code unit (256 distinct values) and of
any member of the basic execution character set.
(until C++23)
the ordinary literal encoding of any element of the basic literal character set.
(since C++23)
Similar to C, C++ supports bytes of sizes 8 bits and greater.

The types char, unsigned char, and signed char use one byte for both storage and value representation. The number of bits in a byte is accessible as CHAR_BIT or std::numeric_limits<unsigned char>::digits.

Memory location
A memory location is the storage occupied by the object representation of either an object of scalar type that is not a bit-field, or the largest contiguous sequence of bit-fields of non-zero length.

Note: Various features of the language, such as references and virtual functions, might involve additional memory locations that are not accessible to programs but are managed by the implementation.

struct S
{
    char a;     // memory location #1
    int b : 5;  // memory location #2
    int c : 11, // memory location #2 (continued)
          : 0,
        d : 8;  // memory location #3
    struct
    {
        int ee : 8; // memory location #4
    } e;
} obj; // The object “obj” consists of 4 separate memory locations
Defect reports
The following behavior-changing defect reports were applied retroactively to previously published C++ standards.

DR	Applied to	Behavior as published	Correct behavior
CWG 1953	C++98	objects occupying the same storage were
considered as different memory locations	memory location
now refers to storage
