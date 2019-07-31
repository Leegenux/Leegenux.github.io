---
title: understanding-pe-format-next
date: 2019-07-20 18:21:39
tags:
---




## IAT

```C
typedef struct _IMAGE_IMPORT_DESCRIPTOR { // Represents the library to be imported
    union {
        DWORD  Characteristics;
        DWORD  OriginalFirstThunk;   // Import Name Table
    }
    DWORD  TimeDateStamp;
    DWORD  ForwarderChain;
    DWORD  Name;					// Library name, each library contains multiple functions
    DWORD  FirstThunk;				// Import Address Table
} IMAGE_IMPORT_DESCRIPTOR;

typedef struct _IMAGE_IMPORT_BY_NAME {
    WORD Hint;						// Ordinal
    BYTE Name[1];					// Function name
} IMAGE_IMPORT_BY_NAME, *PIMAGE_IMPORT_BY_NAME;
```

There should always be a `IMAGE_IMPORT_DESCRIPTOR` for each library to import, and they all end with a `NULL` structure.

Here, the `OriginalFirstThunk` is a pointer to one of the instances of `IMAGE_IMPORT_BY_NAME` array.

The `IAT` works together with  `INT`, which also provides information about function entries of library. They both end with a `DWORD` `NULL`.



## EAT

```C
typedef struct _IMAGE_EXPORT_DIRECTORY {
    DWORD Characteristics;
    DWORD TimeDateStamp;commented
    WORD  MajorVersion;
    WORD  MinorVersion;
    DWORD Name;					 // Address of library file name
    DWORD Base;					 // Ordinal Base
    DWORD NumberOfFunctions;
    DWORD NumberOfNames;
    DWORD AddressOfFunctions;    //
    DWORD AddressOfNames;		 //
    DWORD AddressOfNameOrdinals; //
}
```

Typical process of getting exported function address starts with obtaining the *name_index* from `AddressOfNames`, after that, get the *ordinal* with the *name_index* as key out of `AddressOfNameOrdinals`. Finally, the address of funtion is simply `AddressOfFunctions[ordinal]`.

https://blog.csdn.net/Apollon_krj/article/details/77427729

https://blog.kowalczyk.info/articles/pefileformat.html

https://www.toutiao.com/i6469404407048962573/



## ESI and EDI

寄存器*ESI*、*EDI*、*SI*和*DI*称为变址寄存器*(Index Register)*，它们主要用于存放存储单元在段内的偏移量，用它们可实现多种存储器操作数的寻址方式，为以不同的地址形式访问存储单元提供方便。变址寄存器不可分割成*8*位寄存器。作为通用寄存器，也可存储算术逻辑运算的操作数和运算结果。 它们可作一般的存储器指针使用。在字符串操作指令的执行过程中，对它们有特定的要求，而且还具有特殊的功能。

Source and Destination
