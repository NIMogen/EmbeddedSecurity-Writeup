# Calling Conventions

cdecl

Clears the stack after the procedure call

```
push arg3
push arg2
push arg1
call function
add esp, 12 ; returns ESP
```

stdcall

Same as cdecl, though a RET instruction is used to clear the stack. Do find the number of arguments, just divide the operand by 4.

```
push arg3
push arg2
push arg1
call function
function:
... do something ...
ret 12
```

fastcall

Some arguments are passed via register, and the others passed via the stack. Can be more performant on old computers due to using less stack space, though there is little benefit on “newer” computers. First to dx, then cx, then onto the stack (note the reverse order).

```
push arg3
mov edx, arg2
mov ecx, arg1
call function
function:
.. do something ..
ret 4
```

regparm

Basically fastcall, though options can be set to decide how many registers are being used before parameters are pushed onto the stack.

watcom

Same as the previous two, except it will use 4 registers: ax, cx, bx, dx, in that order, before pushing onto the stack.

Apparently they sometimes have a underscore appended to them to indicate this calling convention.

thiscall

Passing the *this* pointer to a function method in c++.

Depending on the compiler, *this* will be passed to either cx, or will always be the first argument in the call, meaning all of function methods will have the extra *this* ptr parameter as the first arg.

return types of float and double

In all conventions minus Win64, floats and doubles are returned via the FP register S(0), while in win64 they are returned in the low bits of the XMM0 register.