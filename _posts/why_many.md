---
title: Why the Multi-Language Repositories
date: 2025-09-21
parent: _posts
---

This is due to dynamic linking.


Instead of embedding full bytecode of commonly used libraries, we keep the library in a .so (or .dll) file and reference them in bytecode. Then replace the reference to the function implementation in process' address space. This is more storage efficient.
 

# GCC Pipeline
pre-processers -> compiler -> assembler -> linker

GCC works with Objective C, C#, C++, Fortran, Go

So using things like extern keyword, we can compile the linked

```
gcc main.c -S -o main
```
This will make it stop after generating assembly. We can use this to view performance critical code and verify proper optimizations. 

 We can  also simply pass assembly files and tell gcc to link it

 ```
 gcc main.s -o main
 ```
We can write code in assembly, pass it in different stages, and linker will take care of creating one executable.

```
extern int is_prime(n);
```

Write assembly code for ```is_prime```


Pass both files to gcc and link.
Done in ffmpeg, openssl, linux kernel. Fall back to assembly.


We can replace compiler and preproccesor with others. 

Other than assembly, we can mix high level languages.

We can compile and assemble  both langauges sepertely (.o) files and then link them.

We can even do this by Rust because they still have linkers. Rust often needs to do this because existing old implementations in C.

This has any advantages such as 
1. Performance optimizations by writing some code in C/CPP/Rust.

2. Leveraging existing libraries

3. Leverage system level access
4. Faster development for non critical
5. Team Expertise
6. Cross platform support
7. Migration of existing code 



However, how to make caller callee conventions consistent? Perhaps call by reference or call by value affects this?


# About ABIs
Application Binary Interface

how binary codes interact with each other at the hardware level.

1. processor instruction set, details like register file strcuture, memoery access types
2. Size, layout, alignment of basic data types
3. Calling conventions, how call stack is organized
4. Exception propagation
5. How application should make system calls to operating system, if ABI specifies direct system calls rather than procedure calls to system call stubs, syscall numbers
6. Binary format of object files

We need to make language X conform to language Ys ABI or vice versa. 
Compiler flags, extern is what makes this process hidden. 

They tell compiler generated assembly follows ABI.
