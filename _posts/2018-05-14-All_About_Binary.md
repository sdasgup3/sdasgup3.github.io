---
layout: post
title:  X86 FAQs & Binary Analysis Tools
tags:
- Binary
date: 14-05-2018
---
X86 FAQs & Binary Analysis Tools.

# Tools

### objdump
```
objdump -d -M=x86-64,att --no-show-raw-insn ./a.out
```
### Calling C from Assembly
```
main:
  pushq   %rax
  movq $-1, %rdi
  movq $-1, %rax
  movl    $65, %edi
  callq   putchar
  xorl    %eax, %eax
  popq    %rcx
  retq
// as test.s -o test.o
// gcc test.o
```
### Some useful gcc options
```
// Command used by compiler explorer V0.1
gcc test.c -02 -c -S -o - -masm=att | c++filt | grep -vE '\s+\.'

-fno-asynchronous-unwind-tables: disable CFI directives on gas assembler output


-march=haswell // Targetting ISA
```

# Articles

## Ida
 - [IDA Tricks - Handling dynamic imports](https://www.usualsuspect.re/article/ida-tricks-handling-dynamic-imports)
 - [Beginner's Guide](https://leanpub.com/IDAPython-Book)

## Tutorial on x86 assembly programming (Syntax/Semantics)
 - [A fundamental introduction](https://www.nayuki.io/page/a-fundamental-introduction-to-x86-assembly-programming#5-memory-addressing-reading-writing)
 - [Introduction to X86-64 Assembly for Compiler Writers](https://www3.nd.edu/~dthain/courses/cse40243/fall2015/intel-intro.html)
 - [x86 Assembly Language Reference Manual
](https://docs.oracle.com/cd/E26502_01/html/E28388/toc.html)
- [Introduction to x64 Assembly](https://software.intel.com/en-us/articles/introduction-to-x64-assembly)

## Calling Conventions
  - [x86 Disassembly/Floating Point Numbers](https://en.wikibooks.org/wiki/X86_Disassembly/Floating_Point_Numbers)

## Adressing modes

X86-64 is a complex instruction set (CISC), so the MOV instruction has many different variants that move different types of data between different cells.

MOV, like most instructions, has a single letter suffix that determines the amount of data to be moved. The following names are used to describe data values of various sizes:

| Suffix | 	Name | 	Size |
|:------:|:------:|:----:|
|B	|BYTE	|1 | byte |(8 bits)|
|W	|WORD	|2 |bytes |(16 bits)|
|L	|LONG	|4 |bytes |(32 bits)|
| Q	| QUADWORD	|8 |bytes |  (64 bits)|

It is possible to leave off the suffix, and the assembler will attempt to choose the right size based on the arguments. However, this is not recommended, as it can have unexpected effects.

The arguments to MOV can have one of several addressing modes. Here is an example of using each kind of addressing mode to load a 64-bit value into %rax:

| Mode | 	Example |
|:-----:|:--------:|
|Global Symbol	|MOVQ x, %rax|
|Immediate	|MOVQ $56, %rax|
|Register|	MOVQ %rbx, %rax|
|Indirect|	MOVQ (%rsp), %rax|
|Base-Relative	|MOVQ -8(%rbp), %rax|
|Offset-Scaled-Base-Relative	|MOVQ -16(%rbx,%rcx,8), %rax|

``-16(%rbx,%rcx,8)`` refers to the value at the address ``-16+%rbx+%rcx*8``

For the most part, the same addressing modes may be used to store data into registers and memory locations. However, not all modes are supported. For example, it is not possible to use base-relative for both arguments of MOV: MOVQ -8(%rbx), -8(%rbx).


# FAQs
## How main works
- [How main() is executed on Linux ](http://www.tldp.org/LDP/LG/issue84/hawk.html)
- [How statically linked programs run on Linux](https://eli.thegreenplace.net/2012/08/13/how-statically-linked-programs-run-on-linux/)
- [Linux assemblers: A comparison of GAS and NASM](https://www.ibm.com/developerworks/library/l-gas-nasm/index.html)
- [Displaying all argv in x64 assembly](https://eli.thegreenplace.net/2013/07/24/displaying-all-argv-in-x64-assembly)

## Why %eax is made zero before printf
Code in C:
```C
printf("%d", 1);
Output:
```

Assembly
```asm
movl    $1, %esi
leaq    LC0(%rip), %rdi
movl    $0, %eax  ; WHY?
call    _printf
```

From the x86_64 System V ABI:
```
  Register    Usage
  %rax        temporary register; with variable arguments
            passes information about the number of vector
            registers used; 1st return register

For calls that may call functions that use varargs or
stdargs (prototype-less calls or calls to functions
containing ellipsis (. . . ) in the declaration) %al is
used as hidden argument to specify the number of vector
registers used. The contents of %al do not need to
match exactly the number of registers, but must be an
upper bound on the number of vector registers used
and is in the range 0–8 inclusive.
```

printf is a function with variable arguments, and the number
of vector registers used is zero. Note that printf must
check only %al, because the caller is allowed to leave
garbage in the higher bytes of %rax.

### Reference
  - [link](https://stackoverflow.com/questions/6212665/why-is-eax-zeroed-before-a-call-to-printf/6212835#6212835)
