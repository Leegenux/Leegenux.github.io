---
title: Basics of PE Format(1)
tags: [win32, pe, dll, cracking]
---

The post will not show you every detail about the PE format. However, you ought to be given some knowledge helpful to crack an PE when you're done reading. By the way, the details of PE format can be found [here](https://docs.microsoft.com/en-us/windows/desktop/debug/pe-format "click for more"), if you want some precise insights from me, please keep reading.



## Byte Order

Before we cut into the theme, let's prepare ourselves for correctly reading the byte stream. As known to us all, the Intel CPUs all comply to the little endian byte order, whose standard has a great influence on the PC market. So bear in mind that the byte sections I'm gonna show to you are all little endian ordering. That's for every data unit, as the offset grows, the byte gets less and less significant. Hopefully the illustration above will help you get over some confusion later.













Each PE file has a header, and you can dig out much crucial information from this tiny part. The definitions of the headers serve as references when needed, so there is no need to memorize them all.

First of all, there are **DOS header**



## DOS Header

The structure representing the DOS header is `_IMAGE_DOS_HEADER`, which takes up 64 bytes of space. There are dozens of fields in this structure but what we really care about is the last `DWORD` (4 bytes).

It's declaration is `DWORD e_lfanew;` . Its job is to point out the offset of **NT header**.

Note that most content of the PE file's header is designed for program loader of the operating system to work flawlessly.

Here is its definition:
```C
struct _IMAGE_DOS_HEADER {      // DOS .EXE header
    WORD   e_magic;                     // Magic number
    WORD   e_cblp;                      // Bytes on last page of file
    WORD   e_cp;                        // Pages in file
    WORD   e_crlc;                      // Relocations
    WORD   e_cparhdr;                   // Size of header in paragraphs
    WORD   e_minalloc;                  // Minimum extra paragraphs needed
    WORD   e_maxalloc;                  // Maximum extra paragraphs needed
    WORD   e_ss;                        // Initial (relative) SS value
    WORD   e_sp;                        // Initial SP value
    WORD   e_csum;                      // Checksum
    WORD   e_ip;                        // Initial IP value
    WORD   e_cs;                        // Initial (relative) CS value
    WORD   e_lfarlc;                    // File address of relocation table
    WORD   e_ovno;                      // Overlay number
    WORD   e_res[4];                    // Reserved words
    WORD   e_oemid;                     // OEM identifier (for e_oeminfo)
    WORD   e_oeminfo;                   // OEM information; e_oemid specific
    WORD   e_res2[10];                  // Reserved words
    LONG   e_lfanew;                    // File address of new exe header
  };
```



## NT Header

It consists of **Standard NT Header** (AKA FileHeader below) and **Optional NT Header**, both of which are members of C structure `_IMAGE_NT_HEADERS`.

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
    WORD Machine;			// 0x0: All platform; 0x14C: Intel i386 and later;
    WORD NumberOfSections;
	DWORD TimeDateStamp;    // Set on finishing linking
	DWORD PointerToSymbolTable; 
	DWORD NumberOfSymbols; 
	WORD SizeOfOptionalHeader;
	WORD Characteristics;   // Each bit represents a different property
};
```

The member mostly referred to is the `Characteristic` and meaning of its each bit can be found in [official page.](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-_image_file_header)



### `_IMAGE_OPTIONAL_HEADER`

This structure is placed adjacent to the former one (`_IMAGE_FILE_HEADER`) and is much larger --- 224 bytes which peaks among the list of PE headers in size. (Since the array `DataDirectory` might actually have more or less than 16 elements, the size of Optional Header varies.)

The declaration is the following:

```C
struct _IMAGE_OPTIONAL_HEADER{
    WORD Magic;                    //0x0107:ROM image, 0x010B:32bit PE, 0X020B:64bit PE 
    BYTE MajorLinkerVersion; 
    BYTE MinorLinkerVersion; 
    DWORD SizeOfCode;             
    DWORD SizeOfInitializedData;
    DWORD SizeOfUninitializedData; 
    DWORD AddressOfEntryPoint;     // A relative virtual address of Program Entry Point
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
    DWORD NumberOfRvaAndSizes; // Number of members in the next array
    _IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];// #define IMAGE_NUMBEROF_DIRECTORY_ENTRIES 16
};
```

All members with comments are instrumental. Note that all size related stuff referred above obeys the memory alignment mechanism. 

For each 32bit program there are virtual memory space from `0x0` to `0xFFFFFFFF`. And the `ImageBase` specifies the default offset the PE should be loaded to.



### FileBuffer and ImageBuffer

FileBuffer is the way how PE lives in the hard drive. However, execution of a PE requires an ImageBuffer in memory storing all the machine code and other data. The ImageBuffer is a copy of stretched version FileBuffer and the stretching process is done by the **PE Loader**  of the OS.

The `ImageBase` address is not [RVA](#rva-and-raw). After the FileBuffer is loaded, the PE Loader sets `EIP` register to `ImageBase` + `AddressOfEntryPoint`, which is to make the program start from its entry point.

Note that the value of Alignment specifies the minimal storage unit of the subject. For instance, `SectionAlignment` specifies the unit for sections in ImageBuffer, the `FileAlignment` in FileBuffer. These two values might not be the same sometimes.



## Sections Table

For better organization, higher efficiency and other benifits, PE format has exploited the Section mechanism. The resource such as code, data are grouped according to their attributes and placed together respectively.

One of those benifits the Section mechanism brings is security. We avoid puting code and data together so we get away from accidently overwriting our code or run the data in some extent.

At the beginning of each Section, there is a Section Header containing the locations, access privileges, attributes and so on.

The section table can be found at the offset **`_IMAGE_DOS_HEADER.e_lfanew` + `IMAGE_SIZEOF_SIGNATURE + IMAGE_SIZEOF_FILE_HEADER` + `_IMAGE_FILE_HEADER.SizeOfOptionalHeader` **relative to the beginning of the binary file (For FileBuffer) or `ImageBase` (For ImageBuffer). That's to say, the Section Table is put right next to the Optional NT header.

> Both `IMAGE_SIZEOF_SIGNATURE` and `IMAGE_SIZEOF_FILE_HEADER` are predefined macros.
>
> From the formula above we also manage to know the alignment of all the headers I have mentioned above. Try drawing a graph for them on the paper yourself.

Note that there is no such structure corresponding to Section Table in code, this term just helps me to put thing easier.



### Section Header

Inside of the Section Table are Section Headers placed consequently.

The definition of Section Header looks like this:

```C
#define IMAGE_SIZEOF_SHORT_NAME 8
struct _IMAGE_SECTION_HEADER {
    UCHAR   Name[IMAGE_SIZEOF_SHORT_NAME];
    union {
            ULONG   PhysicalAddress;
            ULONG   VirtualSize;
    } Misc;
    ULONG   VirtualAddress;
    ULONG   SizeOfRawData;
    ULONG   PointerToRawData;
    ULONG   PointerToRelocations;
    ULONG   PointerToLinenumbers;
    USHORT  NumberOfRelocations;
    USHORT  NumberOfLinenumbers;
    ULONG   Characteristics;
}
```

The five fields : `VirtualAddress`, `VirtualSize`, `PointerToRawData`, `SizeOfRawData` and `Characteristic` are especially important for us hackers. You should read the next section [RVA and RAW](#rva-and-raw), where four of the fields are explained.



### RVA and RAW

This two terms are crucial for you to understand how PE FileBuffer is mapped to the ImageBuffer.

RVA is the bias between specified address and ImageBase. RAW is the bias between file offset and the beginning of the file. The `VirtualAddress` field of the `_IMAGE_SECTION_HEADER` is also an RVA.

So, look at the definition of `_IMAGE_SECTION_HEADER`. The `VirtualAddress` field is the RVA of the Section in ImageBuffer. The `PointerToRawData` is RAW of the Section in FileBuffer. `PointerToRawData` and `VirtualSize` are then both self-evident.

Though the section-wise layout always changes after transformation between the FileBuffer and ImageBuffer. The inner layout of each section is not likely to change. So we come to a formula: **RAW - `PointerToRawData` = RVA - `VirtualAddress`**.

For example, say we have an `ImageBase` of value `0x01000000`, a .text section in ImageBuffer at `0x01001000` (which is to say the `VirtualAddress` = `0x00001000` - `0x0100000` = `0x00001000`). The corresponding .text section's `PointerToRawData` is `0x00000400` Now there is a structure at `0x00001124` in this .text section in FileBuffer. How do you find out its location in the ImageBuffer ? 

Easy! Apply that equation to this problem : RVA = RAW(`0x00001124`) - `PointerToRawData`(`0x00000400`) + `VirtualAddress`(`0x00001000`) = `0x00001d24`. And according to the RVA, we get its VA = RVA(`0x00001d24`) + `ImageBase`(`0x01000000`) = `0x01001d24`. Now we've derived all location information with the help of that simple equation.



## Conclusion

We have played a little bit around part of the PE header. We have learnt about DOS header(`_IMAGE_DOS_HEADER`), NT header(`_IMAGE_NT_HEADERS`) and Sections(`_IMAGE_SECTION_HEADER`). Hopefully two formulas introduced in [Section Table](#section-table) and [RVA and RAW](#rva-and-raw) can help you draw a schematic diagram for part of the FileBuffer and its corresponding ImageBuffer.

In next post, I will talk about the **IAT**, **EAT** and **DLL**.
