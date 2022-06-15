# Practical Reverse Engineering

First 3 chapters consist in an introduction to assembly (x86 and ARM), and then an introduction to the Windows kernel.

**Obfuscation**

- Most obfuscation centers around choosing the most verbose method of performing an operation, sprinkled with junk operations
- Pattern-based obfuscation, constant unfolding, junk code insertion, stack-based obfuscation, and the use of unconventional instructions are all obfuscation methods
- The previous methods can be reversed through peephole optimization, constant folding, dead statement elimination, and stack optimization

Constant folding-unfolding: Constant folding replaces computations whose results are known at compile time with the results themselves.

Dead Code Insertion-Elimination: Removing or adding code that has no real effect on the result of the program.

Substituting arithmetic with bit operations

 

- Pattern based obfuscation
    
    “Transformations that map one or more adjacent instructions into a more complicated sequence of instructions that have the same semantic effect”
    
    for example
    
    ```
    push reg32
    
    turns into
    
    push imm32
    mov dword ptr [esp], reg32
    
    or
    
    push reg32
    mov reg32, esp
    xchg [exp], reg32
    pop esp
    ```
    
    Transformations like this can be applied indefinitely, turning a small number of instructions into what could be a million
    
    You can try to deobfuscate code that has been obfuscated (using pattern based obfuscation) by mapping the mangles sequences (called target sequences) to the original. This roughly corresponds to peephole optimization, which is something that is done by compilers. It basically consists of changing a small set of instructions to an equivalent set with improved performance. 
    

Spaghetti code: Dynamic analysis can help with this, as you can explicitly follow the control flow. Outlining functions, turning some subpart of a function into its own functions.

```
push target_address
ret

is semantically equivalent to the following

jmp target_address 
```

Calls are often assumed to return, but this is not absolutely necessary

```
Consider

basic_block_a:
add [esp], 9
ret

basic_block_b:
call basic_block_a
<junk code here>
true_return_address:
nop

basic_block_a only updates the return address stored at the
top of the stack before it is used by ret
```

OS-based control indirection: SEH, VEH, UEH on Windows. setjmp, longjmp in Unix.

- Obfuscated code triggers on exception.
- OS calls its exception handlers.
- Handler dispatches instructions flow and gives the program a clean slate.

```
push address_seh_handler
push fs:[0]
mov fs:[0], esp
xor eax, eax
mov [eax], 1234h ; trigger exception by writing to 0x0
<junk code>
address_seh_handler:
<condition execution>
pop fs:[0]
add esp, 4
```

Control-flow graph...

Consists of using a dispatcher switch statement to control flow, instead of more conventional control flow, so that its harder to see relationships between procedures. Dummy states can be inserted as well...

**De-obfuscation and Analysis**

abstract interpretation: abstracting the concrete behaviors of the program into a “decidable approximation”

Semantics: “...represent all if its [the programs] possible concrete behavior, including its interaction with any possible computer system environment”

Trace Semantics: “all finite and infinite sequences of states and transitions...”. Trace semantics are apparently more concrete.

Abstract domain: an “abstraction” of a concrete semantics.

Slicing a program can simplify the process. This amounts to looking at sections of interest, and ignoring the rest temporarily. Divide and conquer.

Symbolic Execution: “a means analyzing a program to determine which inputs cause each part of  program to execute” - angr is a tool for symbolic execution

“Abstract interpretation can be used for modelling any program transformation. By considering the syntax of a program as an abstraction of its concrete semantics, we can formalize any syntactic program transformation as an abstract interpretation of the corresponding semantic transformation”

Metasm, mixsm

Some trivial obfuscation techniques can be deobfuscated with a search and replace algorithm.