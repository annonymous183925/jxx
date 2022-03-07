# jxx
Programming Languiage

# Value representation
- Values are represented with 8 byte integers.
- Each value has a 3 bit tag part to know the type of the value
- Each value has a 61 bit payload part to know the value of the value :)

There are 6 types of values: integers, booleans, floats, objects, small strings, and extended types.

Integers are used to represent 61 (62?) bit signed integers, booleans are used to store either the value true or false, floats are used to represent double precision floating point values, object values are used to store pointers to objects stored in the heap with more flexibilities, small strings are used as an optimization to allow for faster operations, being less memory hungry, spending less time in gc, and avoiding cache misses by storing 7 ascii character strings in the value payload directly. Extended types are for less common values, such as some singletons (None, StopIteration, NotImplemented, Ellipsis), small string iterators (to avoid allocation), linear object iterators, object data field descriptors, and more.

