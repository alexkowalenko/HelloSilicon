# HelloSilicon
An attempt with assembly on the Machine We Must Not Speak About

## Latest News

[Chapter 6](https://github.com/below/HelloSilicon#chapter-6) completed.

## Introduction

In this repository, I will code along with the book [Programming with 64-Bit ARM Assembly Language](https://www.apress.com/gp/book/9781484258804), adjusting all sample code for a new platform that might be very, very popular soon. The original sourcecode can be found [here](https://github.com/Apress/programming-with-64-bit-ARM-assembly-language).

### Prerequisites

While I pretty much assume that people who made it here meet most if not all required prerequisites, it doesn't hurt to list them. 

* You need [Xcode 12](https://developer.apple.com/xcode/), and to make things easier, the command line tools should to be installed. I believe they are when you say "Yes" to "Install additional components", but I might be wrong. This ensures that our tools are found in default locations (namely `/usr/bin`). If you are not sure, check _Preferences → Locations_ in Xcode.

* All application samples also require [macOS Big Sur](https://developer.apple.com/macos/), [iOS 14](https://developer.apple.com/ios/) or their respective watchOS or tvOS equivalents. Especially for the later three systems it is not a necessity per-se, but it makes things a lot simpler.

* Finally, while all samples can be adjusted to work on Apple's current production ARM64 devices, for best results you should have access to a [MWMNSA](https://developer.apple.com/programs/universal/).


With the exception of the existing iOS samples, the book is based on the Linux operating system. Apple's operating systems (macOS, iOS, watchOS and tvOS) are actually just flavors of the [Darwin](https://en.wikipedia.org/wiki/Darwin_(operating_system)) operating system, so they share a set of common core compoents. 
While Linux and Darwin share a common ancestor and appear to be very similar, some changes are needed to make the samples run on Apple hardware, and they are listed below.

### Tools

The book uses Linux GNU tools, such as the GNU `as` assembler. While there is an `as` command on macOS, it will invoce the integrated Clang assembler by default. While there is the `-Q` option to use the GNU based assembler, this was only ever an option for x86_64 — and this will be deprecated as well.
```
% as -Q -arch arm64
/usr/bin/as: can't specifiy -Q with -arch arm64
```
Thus, the GNU assembler syntax is not an option for Darwin, and the code will have to be adjusted for the Clang assembler syntax

### Operating System

Linux and Darwin, while having similar roots in AT&T Unix, are different. For the listings in the book, this mostly concerns system calls (i.e. when we want the Kernel to do someting for us), and the way Darwin accesses memory. 

The changes will be explained in details below.

## Chapter 1

If you are reading this, I assume you know that the macOS Terminal can be found in _Applications → Utilities → Terminal.app_. But if you didn't I feel honored to tell you and I wish you lots of fun on this journey! Don't be afraid to ask questions.

### Listing 1-1

To make "Hello World" run on the MWMNSA, first the changes from page 78 have to be applied to account for the differences between the Darwin and the Linux kernel.
The next trick is to insert `.align 2` (or `.p2align 2`), because Darwin likes things to be aligned on even boundaries. Thanks to @m-schmidt and @zhuowei!

To make the linker work, a little more is needed, most of it should look familiar to Mac/iOS developers. These changes need to be applied to the `makefile` and to the `build` file. The complete call to the linker looks like this:
```
ld -o HelloWorld HelloWorld.o \
	-lSystem \
	-syslibroot `xcrun -sdk macosx --show-sdk-path` \
	-e _start \
	-arch arm64
```

We know the `-o` switch, let's examine the others:


* `-lSystem` tells the linker to link our executable with `libSystem.dylib`. We do that to add the `LC_MAIN` load command to the executable. Generally, Darwin does not support [statically linked executables](https://developer.apple.com/library/archive/qa/qa1118/_index.html). It is [possible](https://stackoverflow.com/questions/32453849/minimal-mach-o-64-binary/32659692#32659692), if not especially elegant to create executables without using `libSystem.dylib`. I will go deeper into that topic when time permits. For people who read _Mac OS X Internals_ I will just add that this replaced `LC_UNIXTHREAD` as of MacOS X 10.7. 
* `-sysroot`: In order to find `libSystem.dylib`, it is mandatory to tell our linker where to find it. It seems this was not necessary on macOS 10.15 because _"New in macOS Big Sur 11 beta, the system ships with a built-in dynamic linker cache of all system-provided libraries. As part of this change, copies of dynamic libraries are no longer present on the filesystem."_. We use `xcrun -sdk macosx --show-sdk-path` to dynamically use the currently active version of Xcode.
* `-e _start`: Darwin expects an entrypoint `_main`. In order to keep the sample both as close as possible to the book, and to allow it's use with in the C-Sample from _Chapter 3_, I opted to keep `_start` and tell the linker that this is the entry point we want to use
* `-arch arm64` for good measure, let's throw in the option to cross-compile this from an Intel Mac

## Listing 2-1

Other than adding `.align 2` to silence the warning, the Clang assembler does not understand the `MOV X1, X2, LSL #1` alias, but requires `LSL X1, X2, #1`. Apple has told me (FB7855327) that they are not planning to change this. If anyone has a hint where I can find the Clang assembly syntax reference guide, I'd be happy if you'd let me know.

Also, of course, exit call and the makefile had to be adjusted like in _Listing 1-1_.

## Listing 2-3, Listing 2-4

The exit call must be adjusted.

Please note that I have not yet update the `codesnippets.s` file. Here Clang apparently also does not like the syntax.

## Chapter 3

### Listing 3-1

As an excersise, I have added code to find the default Xcode toolchain on macOS. In the book they are using this to later switch from a Linux to an Android toolchain. This process is much different for macOS and iOS: It does not usually involve a different toolchain, but instead a different Software Development Kit (SDK). You can see that in [Listing 1-1] where `-sysroot` is set. 

That said, while it is possible to build an iOS executable with the command line, it is not a trivial process. So for building apps, I will stick to Xcode.

### Listing 3-7

As [Chapter 10] focusses on building an app that will run on iOS I have chosen to simply create a Command Line Tool, which is now using the same `HelloWorld.s` file.
Thankfully @saagarjha [suggested](https://github.com/below/HelloSilicon/issues/5) how it would be possible to build the sample with Xcode _without_ `libc`, and I might come back to try that later.

## Chapter 4

Besides the changes that are common, we face a new issue, which is described in the book in Chapter 5: Darwin does not like `LSR X1, =symbol`, we will get the error `ld: Absolute addressing not allowed in arm64 code`. If we use `ASR X1, symbol`, our data has to be in the read-only `.text` section. In this sample however, we want writable data. 

On Darwin 
> All large or possibly nonlocal data is accessed indirectly through a global offset table (GOT) entry. The GOT entry is accessed directly using RIP-relative addressing. [Apple Documentation](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/MachOTopics/1-Articles/x86_64_code.html#//apple_ref/doc/uid/TP40005044-SW1)

And by default, on Darwin all data contained in the `.data` section, where data is writeable, is "possibly nonlocal".

Thankfully, @claui gave me a pointer into the right direction, and the full answer can be found [here](https://reverseengineering.stackexchange.com/a/15324): 
> The `ADRP` instruction loads the address of the 4KB page anywhere in the +/-4GB (33 bits) range of the current instruction (which takes 21 high bits of the offset). This is denoted by the `@PAGE` operator. then, we can either use `LDR` or `STR` to read or write any address inside that page or `ADD` to to calculate the final address using the remaining 12 bits of the offset (denoted by `@PAGEOFF`).

So this: 
```
	LDR	X1, =outstr // address of output string
```

becomes this:
```
	ADRP	X1, outstr@PAGE	// address of output string
	ADD	X1, X1, outstr@PAGEOFF
```

## Chapter 5

The important differences in memory addressing for Darwin were already addresed above.

### Listing 5-1
The `quad`, `octa` and `fill` keywords apparently must be in lowercase for the llvm assembler. (See bottom of this file)

### Listing 5-10

Changes like in Chapter 4

## Chapter 6

Of course, the usual alignment and addressing changes were needed.

And as we learned in Chapter 5, all assembler directives (like `.equ` must be in lowercase for the Clang assember. Also, clang uses `.endmacro` instead of GNU's `.ENDM`.

Please note that I have not tested the `codesnippts` code yet, at this point all I can say is that it builds.

## Chapter 7

As of right now, I will skip this chapter. Linux, by design, is made for tinkering, and Darwin is not. `unistd.h` is not part of the userland MacOS SDK, and the whole system call mechanism is considered private and subject to change. As @sagaarjha said: _"Go used to create static binaries on macOS but they would constantly break whenever an update came out"_.

When I find the time I may look deeper into this. Or maybe someone else wants to?

## Chapter 8

This chapter is specifically for the Raspberry Pi4, so there is nothing to do here.

## Chapter 9

For transparency reasons, I replaced `gcc` with `clang`. On macOS it doesn't matter because:
```
% gcc --version
Apple clang version 12.0.0 (clang-1200.0.22.41)
```

### Listing 9-1
Apart from the usual changes, it appears that on Linux, printf will accept arguments passed in the registers. On Darwin, this is not the case, and we must pass the arguments on the stack. As I am still learning this stuff, this may not be the most elegant way to do it, but it works:
```
mov	    X9, SP	// Move Stackpointer into X9
str	    X1, [X9]	// Push X1 onto the stack
str	    X2, [X9, #8]	// Push X2 onto the stack
str	    X3, [X9, #16]	// Push X3 onto the stack
```

It took me quite a while to figure this out, and there is minimal `test.s` and corresponding `build` script to see the printf call in isolation.

### Listing 9-5
`mytoupper` was prefixed with `_` as this is necessary for C on Darwin to find it.

### Listing 9-7
Instead of a shared `.so` library, a dynamic Mach-O libary was created. Further information can be fore here: [Creating Dynamic Libraries](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/CreatingDynamicLibraries.html)

### Listing 9-8
The size of one variable had to be changed from int to long to make the assembler happy.

More importantly, I had to change the `loop` label to a numeric label, and branch to it with the `f` — forward — option. If anyone has an idea how a non-numeric label can be used here, that would be apprecated.

### Listing 9-9

While the `uppertst5.py` file only needed a minimal change, calling the code was more challenging that I had thought: On the MWMNSA, python is a Mach-O universal binary with 2 architectures: x86_64 and arm64e. Notably absent is the arm64 architecture we were building for up to this point. This makes our dylib unusable with python.

arm64e is the [Armv-8 architecture](https://community.arm.com/developer/ip-products/processors/b/processors-ip-blog/posts/armv8-a-architecture-2016-additions), which Apple is using since the A12 chip. If you want to address devies prior to the A12, you must stick to arm64. It is public knowledge that the MWMNSA runs on an A12Z Bionic, thus Apple decided to take advangage of the new features.

So, what to do? We could compile everything as arm64e, but that would make the library useless on any iPhone but the very latests, and we would like to support those, too.

Above, you read something about _universal binary_. For a very long time, the Mach-O executable format had support for several processor architectures. This includes, but is not limited to, Motorola 68k (on NeXT computers), PowerPC, Intel x86 and x86 64-Bit, as well as 32-Bit and 64-Bit arm variants. In this case, I am building a universal dynamic library which includes arm64 and arm64e code. More information can be found [here](https://developer.apple.com/documentation/xcode/building_a_universal_macos_binary). 

## Chapter 10
No changes in the core code were required, but I created a SwiftUI app that will work on macOS, iOS, and later on watchOS and tvOS, too.

## Additional references

* [What is required for a Mach-O executable to load?](https://stackoverflow.com/a/42399119/1600891)
* [Mac OS X Internals, A Systems Approach](https://www.pearson.ch/Informatik/Macintosh/EAN/9780134426549/Mac-OS-X-Internals) Amit Singh, 2007 
* [Darwin Source Code](https://opensource.apple.com/source/xnu/)
* [ARM Archicture Reference Manual](https://static.docs.arm.com/ddi0487/ca/DDI0487C_a_armv8_arm.pdf)

## One More Thing…
_"The C language is case-sensitive. Compilers are case-sensitive. The Unix command line, ufs, and nfs file systems are case-sensitive. I'm case-sensitive too, especially about product names. The IDE is called Xcode. Big X, little c. Not XCode or xCode or X-Code. Remember that now."_ — Chris Espinosa
