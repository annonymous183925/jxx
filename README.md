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

This value representation doesn't give room to a lot of cache misses and memory dereferences. It is very cpu friendly because it uses short and simple arthimetic / bitwise instructions do perform operations. The more types of values there are, the less objects there are and then the less work the garbage collector has to do. This also makes it very efficient to write machine code instructions directly, since this value representation will work the best with those. For example, assuming you know `n` is an integer value, you could convert the following function:

```
def fib(n) {
  if(2 >= n)
      return 1;
  return fib(n - 1) + fib(n - 2);
}
```
To the following assembly:
``` assembly
fib:                  ; assume rdi is 'n'
  cmp rdi, 19         ; compare n with 19
  jbe .return_one     ; jump to .return_one if n is below or equal 19
  
  push rdi            ; save n on the stack
  push rdx            ; allocate rdx register
  sub rdi, 8          ; subtract 1 from n
  call fib            ; recursive call fib
  mov rdx, rax        ; store return result on rdx
  and rdx, -8         ; remove type tag (to later perform addition)
  sub rdi, 8          ; decrease 1 from n 
  call fib            ; recursive call fib
  add rax, rdx        ; perform addition
  pop rdx             ; deallocate rdx register
  pop rdi             ; deallocate n 
  ret
.return_one:
  mov eax, 11
  ret
.overflow:
  ret
```

Calling functions (like the following)
```
def call(callable) {
  return callable();  
}
```
can get turned into something like
```
call:                                                       ; assume callable is rdi
  mov rax, qword ptr [rdi]                                  ; put class object on 'rax'
  xor edi, edi                                              ; put first argument to nullptr
  xor esi, esi                                              ; put last argument to nullptr
  jmp qword ptr [rax + class_function_pointer_offset]       ; tail call to the class function pointer
```
However this is not simple because of exception handling. In this case, the exception handling job would be moved into the caller.
