# Data Structures

**Arrays**

```c
x = array[25];
/*x86

mov ebx, $array
mov eax, [ebx + 25]
mov $x, eax
*/
```

Arrays are often allocated on the stack, so if you see large amounts of contiguous storage being allocated on the stack (sub [esp - 1000]) the  there is a chance that it might be an array

Arrays in memory or global arrays are accessed through an offset of a memory address

```c
:_MyFunction4
 push ebp
 mov ebp, esp
 mov esi, 0x77651004
 mov ebx, 0x00000000
 mov [esi + ebx], 0x00
```

Generally other data structures can be accessed in the same way, so you have to remember that arrays have to be of the same type, and they are generally accessed in a loop (though this is not always true)

If the array has elements larger than one byte, the index needs the be multiplied by the size of the element

```c
mov [ebx + eax * 4], 0x11223344 // Access array of DWORDS
...
mul eax, $20 // Access an array of structs (each 20 bytes)
lea edi, [ebx + eax] // ptr = &arr[i]
```

**Structs**

All items in a struct do not have to be the same type as they have to be in an array.

```c
This data is "packed"; the length allows you to determine the
difference in types
 [ebx + 0] ;value1
 [ebx + 4] ;value2
 [ebx + 6] ;value3
```

```c
One of the following values in the contiguous memory space is a ptr, and 
the others are not, telling you that this is a struct
:_MyFunction
 push ebp
 mov ebp, esp
 lea ecx, SS:[ebp + 8]
 mov [ecx + 0], 0x0000000A
 mov [ecx + 4], ecx
 mov [ecx + 8], 0x0000000A
 mov esp, ebp
 pop ebp
```

One pointer offset from another pointer signals a complex data structure

```c
:MyFunction1
 push ebp
 mov ebp, esp
 mov esi, [ebp + 8]
 lea ecx, SS:[esi + 8]
 ...
```

Elements in a struct need to be byte aligned

Elements that are not aligned will be padded

```c

// Two integer fields and a char field
push rbp
mov rbp, rsp
DWORD PTR [rbp - 0x14], edi
QWORD PTR [rbp - 0x20], rsi
DWORD PTR [rbp - 0x10], 0x64
BYTE PTR [rbp - 0xc], 0x58
DWORD PTR [rbp - 0x8], 0x12c
mov eax, 0x0
pop rbp 
ret
```

Linked list

```c
loop_top:
cmp [ebp + 0], 10
je loop_end
mov ebp, [ebp + 4]
jmp loop_top
loop_end:
```

Binary trees will look similar, though they will use two pointers, one for the right and left branch