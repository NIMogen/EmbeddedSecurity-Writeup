# Conditionals

x86 uses comparisons and jumps for conditional branches.

```c
if (x == 1) {
x = 1;
}
x++;

/*Asm
mov eax, $x
cmp eax, 0
jne end
mov eax, 1
end:
inc eax
mov $x, eax
*/
```

```c
if (x == 10) {
	x = 0;
} else {
	x++
}
/*Asm
mov eax, $x
cmp eax, 0x0A
jne else
mov eax, 0
jne end
else:
inc eax
end:
mov $x, eax
*/
```