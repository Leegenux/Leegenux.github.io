---
title: Win32 Reverse Engineering Basics
date: 2019-03-28 14:36:33
tags: [reverse, crack, win32, pe]
---

This post consists of my learning notes, insights, my experience of cracking a executable as well as answers to the assignments.
We are using OllyDbg to crack.

## Primary Usages of OllyDbg

### Keyboard Shortcuts

With these 4 shortcuts, we are able to do some reverse engineering on easy crackmes like one that I'm gonna show you.

- **Restart**			  `Ctrl` + `F2`
- **Step Into** 		 `F7`
- **Step Over** 		  `F8`
- **Execute till Return** 	 `Ctrl `+ `F9`

### Ways to Modify Strings in Binaries

- Direct modification : That's directly making changes in the **Disassembler Window**.
- Create a new string somewhere else and make it work instead of the original one.

> To know more details about the user interface. Refer to [here](https://www.oreilly.com/library/view/practical-malware-analysis/9781593272906/ch10s02.html)

Two points above reveals two fundamental approaches of patching a binary. The patch mentioned are both intuitively achievable in terms of OllyDbg's software operations. 

What need to be emphasized is that we shouldn't forget to save our changes to the binary.



## IA32 Registers

### Register Types

- General purpose registers
- Segment registers
- Program status and control registers
- Instruction pointer registers



## Stack Establishment and Destruction

The routine of conventional Win32 programs regarding stack's establishment and destruction can be elaborated in a simple way:

1. push EBP to store the old stack base value
2. mov EBP, ESP to create a brand new stack for subprogram

And the destruction is quite the opposite.



## Important Information About Cracking VB Binaries

- VB program can be compiled into P code(**Pseudo Code**) or N code(**Native code**)
- VB program operates in an event-driven manner, in which Windows mainly works.
- VB program's fundamental information resides in the executables in the form of structures.

## X86 Calling Conventions

- **cdecl** 
- **stdcall**
- **fastcall**

The `cdecl` features callers' responsibility to clean up the callee's stack which makes calling with mutable amount of parameters possible. And `stdcall` works differently. The callee clears the stack so that the program have better compatibility against other languages. So Win32 development chooses the latter convention. There are also `fastcall` which makes use of registers for faster calling (However things don't always act as expected).



## PE File Format

Portable Executables are known as PE file. There are essential information across different aspects including memory location, compatibility code, CPU family etc stored in the header part of the PE file. You can check them out by clicking [here](https://docs.microsoft.com/en-us/windows/desktop/debug/pe-format). Also I will upload a post containing the important points later.



## Hands-on Cracking

The test executable can be downloaded [here]()

### Find the Main Function

Before debugging, you should first take a glance at what this file is about. It turns out that a password is required or you are blocked. Then load the file with debugger. Do stepping over until you see the prompt: `What's your password?`. You are already in the `main` function. Trace back and at some point you see this: ![](https://ws3.sinaimg.cn/large/005BYqpgly1g1ilhjin0aj30iy0bvgls.jpg)

The `CALL` statement jumps to the "ask for password" section. Around it there are `MSVCR120.__winitenv` and `MSVCR120.exit` API calls. So we can assure that this is the `main` function entry.

### Crack the Password Section

After the password `scanf` related code. There are lines

```assembly
TEST EAX, EAX
JE SHORT Test1.011C419
```

implementing the logic that when two string matches, jump to `Test1.011C419`. Here `EAX` is set to 0 when function `strcmp` finds two strings match.

`TEST` sets the zero flag, `ZF`, when the result of the `AND` operation is 0. So only if `EAX` is 0 will `JE` statement do the jump.

To bypass the password, try changing them into this:

```assembly
CMP EAX, EAX
JE SHORT Test1.011C419
```

This modification makes the `JE` statement directly jumps to the `string matches` logic branch. Save and rerun, you now get to the next stage.

### Reveal the Encryption

The second input in the format of `%d` is stored on the `SS:[EBP-48]` and a least significant byte in the `EAX` as what this screenshot tells us:

![](https://ws3.sinaimg.cn/large/005BYqpgly1g1im1uk8lrj30im0brwer.jpg)

Execution of `CALL Test1.011C11A4` leads us to the initiation of an array:

![](https://ws3.sinaimg.cn/large/005BYqpgly1g1im4pztg2j30im0bz74p.jpg)

After that there is a loop where the encryption goes:

![](https://ws3.sinaimg.cn/large/005BYqpgly1g1im87ylenj30iv03v0sp.jpg)

Procedures above can be translated into fake code:

```
unsigned char offset = last_byte_of(input_code); 	// Get the input

init(array);     // Init the code

for (char i = 0; i<length(array); i++) {
    array[i] = array[i] xor array[i+offset];
}
```

### Store the Decryped Code

The screenshot shows all necessary information:

![](https://ws3.sinaimg.cn/large/005BYqpgly1g1imhjl4j4j30ik0870ss.jpg)

The decrypted code are stored in the `D:\kp`. (Though I changed it because I don't have no drive with label D).


## Assignments and Answers

### What's the Calling Convention of this executable ?

As you can see, there are no steps of clearing stack right behind the `CALL`. According to the description in the **X86 Calling Conventions** section several paragraphs away above . It uses `stdcall`. Registers play no part in passing parameters to callee.

### What API does the program calls when storing the decrypted data ?

Obviously according to the image: `KERNEL32.CreateFileA`, `KERNEL32.WriteFile` and `KERNEL32.CloseHandle` are used.

### Where is the `main` entry and how is the code decrypted ?

Both described concisely in the post.

### Write a PE header scanner ?

There are references all around the web. My solution will be like [this one](https://github.com/Arvanaghi/PE-Parser).
