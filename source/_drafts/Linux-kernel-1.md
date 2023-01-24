---
title: Linux Kernel Basics
tags: [linux, kernel, c]
---

First I will describe the programs you need to configure a kernel, build it and successfully boot.

## Tools to Build the Kernel

- Compiler
- Linker
- make

In most cases, the compiler used for building an kernel is **gcc**. The linker required is named as **binutils** in most Linux distributions. The **make** tool helps walk the kernel source tree to determine either to compile, link or clean up.

### Tools to Use the Kernel

