---
title: Basics of PE Format
tags: [win32, pe, dll, cracking]
---

The post will not show you every detail about the PE format. However, you ought to be given some knowledge helpful to crack an PE when you're done reading. By the way, the details of PE format can be found [here](https://docs.microsoft.com/en-us/windows/desktop/debug/pe-format "click for more"), if you want some precise insights from me, please keep reading.





Each PE file has a header, and you can dig out much crucial information from this tiny part.



## Byte Order

Before we cut into the theme, let's prepare ourselves for correctly reading the byte stream. As known to us all, the Intel CPUs all comply to the little endian byte order, whose standard has a great influence on the PC market. So bear in mind that the byte sections I'm gonna show to you are all little endian. That's for every data unit as the offset grows, the bit gets less and less significant. Hopefully the illustration above will help you get over some confusion later.



First of all, there are **DOS header**



## DOS Header

The structure representing the DOS header is `_IMAGE_DOS_HEADER`, which takes up 64 bytes of storage. There are dozens of fields in this structure but what we really care about is the last `DWORD` (4 bytes).

It's declaration is `DWORD e_lfanew;` . Its job is to point out the offset of **PE header**.

Note that most content of the PE file's header is designed for program loader of the operating system to work flawlessly.



## PE Header

It consists of **Standard PE Header** and **Optional PE Header**, both of which are members of C structure `_IMAGE_NT_HEADERS`.

The declaration shows the layout:

```C
struct _IMAGE_NT_HEADERS{
    DWORD Signature;   //"PE\0\0" if valid
    _IMAGE_FILE_HEADER FileHeader;
    _IMAGE_OPTIONAL_HEADER OptionalHeader;
};
```



### `_IMAGE_FILE_HEADER`

This structure occupies 20 bytes.

```C
struct _IMAGE_FILE_HEADER{
    WORD Machine;			//0x0: All platform; 0x14C: Intel i386 and later;
    WORD NumberOfSections;
	DWORD TimeDateStamp;    //Set on finishing linking
	DWORD PointerToSymbolTable; 
	DWORD NumberOfSymbols; 
	WORD SizeOfOptionalHeader;
	WORD Characteristics;   // Each bit represents a different property
};
```

The member mostly referred to is the `Characteristic` and meaning of its each bit can be found in [this official web page](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-_image_file_header)



### `_IMAGE_OPTIONAL_HEADER`

This structure is placed adjacent to the former one and is much larger in size ------ 224 bytes.

The declaration is the following:

```C
struct _IMAGE_OPTIONAL_HEADER{
    WORD Magic;                    //0x0107:ROM image, 0x010B:32bit PE, 0X020B:64bit PE 
    BYTE MajorLinkerVersion; 
    BYTE MinorLinkerVersion; 
    DWORD SizeOfCode;             
    DWORD SizeOfInitializedData;
    DWORD SizeOfUninitializedData; 
    DWORD AddressOfEntryPoint;     // A relative virtual address
    DWORD BaseOfCode;              // Base address of code
    DWORD BaseOfData;              // Base address of data
    DWORD ImageBase;               // The base address in memory
    DWORD SectionAlignment;  	   // Decides the alignment size
    DWORD FileAlignment;           // Mostly 1000H
    WORD MajorOperatingSystemVersion;
    WORD MinorOperatingSystemVersion;  
    WORD MajorImageVersion;           
    WORD MinorImageVersion;             
    WORD MajorSubsystemVersion;    
    WORD MinorSubsystemVersion; 
    DWORD Win32VersionValue;       // Always 0
    DWORD SizeOfImage;             // Size of memory occupied after loading into memory
    DWORD SizeOfHeaders;           // Total size of the DOS header plus PE Header.
    DWORD CheckSum;           
    WORD Subsystem;          
    DWORD DllCharacteristics;  	// Always 0
    DWORD SizeOfStackReserve;  // Default reserved stack size
    DWORD SizeOfStackCommit;   // Commited stack size
    DWORD SizeOfHeapReserve;   // Default reserved heap size
    DWORD SizeOfHeapCommit;    // Commited heap size
    DWORD LoaderFlags;         // Always 0
    DWORD NumberOfRvaAndSizes; // Self-evident
    _IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];// #define IMAGE_NUMBEROF_DIRECTORY_ENTRIES 16
};
```

All commented lines are especially instrumental. Please note that all size related stuff referred above obeys the memory alignment mechanism. 

#### FileBuffer and ImageBuffer

The execution of an `.exe` file requires ImageBuffer in memory. The ImageBuffer is a copy of stretched FileBuffer and the stretching process is operated by the **PE Loader**.

The `ImageBase` address is not RVA. After the FileBuffer is loaded, the PE Loader sets `EIP` register to `ImageBase + AddressOfEntryPoint`.



## Section Table

The section table can be found at the offset `_IMAGE_DOS_HEADER.e_lfanew + IMAGE_SIZEOF_SIGNATURE + IMAGE_SIZEOF_FILE_HEADER + _IMAGE_FILE_HEADER.SizeOfOptionalHeader` relative to the binary file or ImageBase.



## RVA and RAW

Each section as well as directory has a RVA and PointerToRawData.

VirtualAddress is relative to ImageBase just like what RVA do. They are actually just a tag marking start of one section.



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

There should always be a `IMAGE_IMPORT_DESCRIPTOR` for each library to import, and they end with `NULL` structure.

Here, the `OriginalFirstThunk` is a pointer to one of the instances of `IMAGE_IMPORT_BY_NAME` array.

The `IAT` works together with  `INT`, which also provides information about function entries of library. They both end with a `DWORD` `NULL`.



## EAT

```C
typedef struct _IMAGE_EXPORT_DIRECTORY {
    DWORD Characteristics;
    DWORD TimeDateStamp;
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