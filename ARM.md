# ARM

Less instructions, more registers than x86

**Syntax**

*opcode{S}{condition} dest, Operand1, Operand2*

ADD r0, r1, r1

MOVLE r0, #5 /* Move val 5 to r0 only if condition LE (less than or equal) is satisfied */

**Data Types**

3 supported data types with a signed and unsigned variation on each

Byte

These values have an extension of -b or -sb

Half-Word

Extension of -h or -sh

Word

No extension

You will also see number of other suffixes and probably combinations. For example, an s means signed...

**Registers**

*17 Registers

R0 - R15 and CPSR, which holds status flags

A few special purpose (by convention) registers

R15 = PC = Program Counter = Instruction Pointers

R13 = SP = Stack Pointer

R11 = FP = Frame Pointer (similar to EBP)

R14 = LR = Link Register

Holds return address for function

**Thumb**

The ARM instruction set has changed over time and an enhanced "thumb" instruction set was added at some point. The thumb enhancement has changed across versions as well, so there isn't much point in learning all the different variations. References are available when necessary.

Important differences are:

conditional execution through the it instruction

"barrel shifting"

32-bit instructions

**LDR and STR**

The ldr and str instructions are used for loading register value and storing values in registers. ARM cannot operate on values in any other way (compare to x86, which can operate on values directly in memory)

ldr Ra, [Rb] /* loads value at addr Rb into register Ra */

str Ra, [Rb] /* stores value at Ra in addr Rb*/

**Conditionals**

*IT{x{y{z}}} cond*

Like x86, they occur after a cmp instruction. The conditionals themselves are not a separate instruction, however. They appear as a qualifier for another instruction. See syntax section above.

Every instruction technically has a 4 bit condition code field. However, for most instructions, always (AL) condition is specified, so that it executes no matter the flag set in CPSR.

Thumb adds conditionals that lay out similarly to switch statements. There is are conditional blocks with a few conditions determining which statements will execute.

IT (**I**f **T**hen)

ITT

ITE (If then else)

ITTE

ITTEE

**Multiples**

You can operate on multiple registers.

STM R1, {R6-R10} stores registers r6 through r10 at a memory location that r1 points to. They would be stored as R1 ← R6, R1+4 ← R7, R1+8 ← R8, and so on...

STM R1!, {R6-R10} would be the same as above, but R1 would be updated with the address right after where R10 was stored. 

**Stack Organization and Functions**

Stack frame are used by functions as their localized memory spaces. 

Like x86, functions have a prologue, body, and epilogue

```c
push {rll, lr} /* Save fp and lr onto stack */
add r11, sp, #0 /* Setting up bottom of stack frame */
sub sp, sp, #16 /* Allocate some buffer on the stack */
```

**Process Memory and Memory Corruption**

Think of the address space as a memory array, where each address is an index, and each one of those indices holds a single byte. Larger multi-byte values are stored in a sequential series of indices.

The common memory corruption vulnerabilities are heap overflows, buffer overflows, format string bugs, and dangling pointers.

gets, strcpy, memcpy
