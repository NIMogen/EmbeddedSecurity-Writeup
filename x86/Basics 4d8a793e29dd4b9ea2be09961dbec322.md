# Basics

Two syntaxes

Intel

dst, src

AT&T

Src, dst

immediate values prefixed with $, registers prefixed with %

Adds a suffix to the instruction to indicate "operation width"

Data Movement

Assembly consists of movement of values around in memory, and basic operations on those values. Instead of variables, you have registers and the stack to store data. 

Movement can occur between

memory to memory

register to memory

immediate to register

register to register

immediate to memory 

immediate to memory 

Instructions operate on values that come from registers in main memory

Loops

```c
do {
x++;
} while (x != 10);
/*
Is equivalent to...
mov eax, $x
beginning:
inc eax
cmp eax, 0x0A
jne beginning
mov $x, eax
*/
```

```c
while (x != 10) {
x++;
}
/*
Is equivalent to...
mov eax, $x
cmp eax, 0x0A
jg end
beginning:
inc eax
cmp eax, 0x0A
jle beginning
end:
*/
```