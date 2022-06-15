# Optimization and Obfuscation

Fast

shl eax, 3

Slow

push edx

push 8

mul dword [esp]

add esp, 4

pop edx

Fast

push $next_instruction

jmp $some_function

Slow

call $some_function

Fast

pop eax

jmp eax

Slow

ret