---
title: Reverse Engineering Of Flash Disk Virus
tags:
---

## FS and GS registers

Their usages in fact varie from system to system.

https://stackoverflow.com/questions/10810203/what-is-the-fs-gs-register-intended-for



## Two parts

1. _WGN.nil 
2. The TrustedInstaller
   1. Mutex



### Self Creation





## sete Instruction

https://stackoverflow.com/questions/53011701/what-does-the-instruction-sete-do-in-assembly





## `test` Instruction

https://en.wikipedia.org/wiki/TEST_(x86_instruction)

This instruction performs a bitwise AND operation to both operands, and sets ZF if result is 0. And the CF is directly set 0.





## `jae` Instruction

Jump if above or zero.



## `jb` Instruction

Jump when CF = 1



## Part of the code analysis

http://www.drdobbs.com/configuring-vc-multithreaded-memory-mana/184416249

It's from a really old blog post.

Whenever a VC++ application starts up, the C runtime library is initialized. One of the first actions is to determine which heap manager to use. The VC++ v6 runtime library is capable of using either its own internal set of heap-management routines, using heap-management routines compatible with the previous version of the compiler, or just calling the operating-system heap-management routines (**HeapAlloc()** and friends) directly. Internally, **__heap_select()** does the following three steps:

1. Check the operating-system version. If running on NT and the major version is five or higher (Windows 2000 or later), then **HeapAlloc()** is used.

2. Look for the environment variable **__MSVCRT_HEAP_SELECT**. If present, it influences which heap is to be used. If the value is **__GLOBAL_HEAP_SELECTED**, the behavior for all programs is changed. When a full path to an executable is found, the code checks if it’s the actual application by calling **GetModuleFileName()**. Which heap type is selected depends on a value appended after a separating colon: 1 to use **HeapAlloc()**, 2 to use the VC++ v5 heap, and 3 to use the VC++ v6 heap.

3. Evaluate the linker build flags in the executable. If the application was built with VC++ v6 or later, the version 6 heap is used; all others use version 5.



## Win32 Heap

https://blog.csdn.net/playboy1/article/details/6940213

一个保留区域, 在堆上分配了内存之后, 系统才将区域提交到物理内存当中.1





## GetStdHandle

https://docs.microsoft.com/en-us/windows/console/getstdhandle



## stosd Instruction

http://www.hgy413.com/hgydocs/IA32/instruct32_hh/vc304.htm