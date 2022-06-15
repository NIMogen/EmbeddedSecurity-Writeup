# Practical Binary Analysis

[Taint analysis](Practical%20Binary%20Analysis%20e836f6dda0664fff9e9ee6cec9dad05a/Taint%20analysis%203b33e323644d46888ef1daabd2a2f9a1.md)

[Symbex](Practical%20Binary%20Analysis%20e836f6dda0664fff9e9ee6cec9dad05a/Symbex%207cfb386ddf384611ba667bfe9b716f21.md)

Compilation process

Pre-processing → Compilation → Assembly → Linking

Pre-processing expands macros

Compilation translates to assembly

Assembly takes the assembly language files and outputs object files (also called modules)

Object files are relocatable—that is, they don’t rely on being placed in any particular place in memory. This is the case because they are compiled independently

The linking phase combines the object files into a single executable