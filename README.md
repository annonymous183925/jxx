# jxx
Programming Languiage

# Value representation
- Values are represented with 8 byte integers.
- Each value has a 3 bit tag part to know the type of the value
- Each value has a 61 bit payload part to know the value of the value :)

There are 6 types of values: integers, booleans, floats, objects, small strings, and extended types.

Integers are used to represent 61 (62?) bit signed integers, booleans are used to store either the value true or false, floats are used to represent double precision floating point values, object values are used to store pointers to objects stored in the heap with more flexibilities, small strings are used as an optimization to allow for faster operations, being less memory hungry, spending less time in gc, and avoiding cache misses by storing 7 ascii character strings in the value payload directly. Extended types are for less common values, such as some singletons (None, StopIteration, NotImplemented, Ellipsis), small string iterators (to avoid allocation), linear object iterators, object data field descriptors, and more.

With this representation, a lot of operations require no branching, no allocation, no memory reads and are just a few instructions of fast bit manipulation. This means that there won't be many cache misses and the garbage collector will have to do minimal work. 

Object tag is represented with 0b000, booleans with 0b001, floats with 0b010, integers with 0b011, small strings with 0b100, and extended tag is represented with 0b110. An invalid value (like NULL PyObject pointer in CPython) would be represented with the tag of an object, and all the rest bits are 0. This means that in binary, an invalid value is just zeroes, which makes it fast to initialize and test for invalid values.

Objects use the tag 0b000 to make it zero overhead to convert values to objects and allowing to use x86 addressing modes directly. Only booleans and integers use the least significant tag bit to make testing for booleans or integers just (value & 1), because most of the operations can be done on both of them (doesn't matter if boolean or integer).

Encoding small strings in an 8 byte integer is very helpful as it allows many fast routines like reversing (using bswap instruction) and finding characters without memory accesses or branches.


