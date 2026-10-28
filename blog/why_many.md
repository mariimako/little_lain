---
title: Why the Multi-Language Repositories
layout: default   
date: 2025-09-21
parent: Blog
---
# Why the Multi-Language Repositories
Like actually why. This is due to dynamic linking.

Instead of embedding the full bytecode of commonly used libraries, we keep the library in a .so (or .dll) file and reference it in bytecode. Then we replace the reference to the function implementation in the process' address space. This is more storage efficient.

# GCC Pipeline
pre-processor -> compiler -> assembler -> linker

GCC works with Objective C, C#, C++, Fortran, Go

Using things like the extern keyword, we can compile the linked code:

```
gcc main.c -S -o main
```
This will make it stop after generating assembly. We can use this to view performance critical code and verify proper optimizations.

We can also simply pass assembly files and tell gcc to link them:

```
gcc main.s -o main
```
We can write code in assembly, pass it in at different stages, and the linker will take care of creating one executable.

```
extern int is_prime(n);
```

Write assembly code for `is_prime`, then pass both files to gcc and link.
Done in ffmpeg, openssl, the Linux kernel. Fall back to assembly.

We can replace the compiler and pre-processor with others.

Other than assembly, we can mix high level languages.

We can compile and assemble both languages separately into .o files and then link them.

We can even do this with Rust, because it still has linkers. Rust often needs to do this due to existing old implementations in C.

This has many advantages, such as:
1. Performance optimizations by writing some code in C/C++/Rust.
2. Leveraging existing libraries
3. Leverage system level access
4. Faster development for non critical parts
5. Team expertise
6. Cross platform support
7. Migration of existing code

However, how do we make caller callee conventions consistent? Perhaps call by reference or call by value affects this?
# About ABIs
Application Binary Interface

How binary code interacts with other binary code at the hardware level.

1. Processor instruction set, details like register file structure, memory access types
2. Size, layout, and alignment of basic data types
3. Calling conventions, how the call stack is organized
4. Exception propagation
5. How an application should make system calls to the operating system, if the ABI specifies direct system calls rather than procedure calls to system call stubs, syscall numbers
6. Binary format of object files

We need to make language X conform to language Y's ABI or vice versa.
Compiler flags and extern are what hide this process.

They tell the compiler the generated assembly follows the ABI.
