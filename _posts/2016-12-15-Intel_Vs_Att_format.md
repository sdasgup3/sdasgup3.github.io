---
layout: post
title: Intel (NASM) vs AT&T (GAS) Syntax
tags:
- Assembly
- Decompilation
date: 15-12-2016
---
Differences in Intel (NASM) vs AT&T (GAS) Syntax.

## Register & Immediate Prefixes

- In Intel syntax there are no register prefixes or immed prefixes.
- In AT&T however registers are prefixed with a '%' and immed's are prefixed with a '$'.


```
   Intel Syntax
   mov     eax,1
   mov     ebx,0ffh
   int     80h

   AT&T Syntax
   movl    $1,%eax
   movl    $0xff,%ebx
   int     $0x80
```

## Direction of Operands.
- Intel: `instr dest src`
- Att: `instr srd dest`

```
 Intel Syntax
 instr   dest,source
 mov     eax,[ecx]

 AT&T Syntax
 instr   source,dest
 movl    (%ecx),%eax
```

## Memory Operands.

- In Intel syntax the base register is enclosed in '[' and ']'
- In AT&T syntax it is enclosed in '(' and ')'.

```
Intex Syntax
mov     eax,[ebx]
mov     eax,[ebx+3]

AT&T Syntax
movl    (%ebx),%eax
movl    3(%ebx),%eax
```

## Complex instructions
- Intel: `segreg:[base+index*scale+disp]`
- Att:  `%segreg:disp(base,index,scale) == disp + base + index*scale`, where scale defaults to 1 if absent.

```
Intel Syntax
instr   foo,segreg:[base+index*scale+disp]
mov     eax,[ebx+20h]
add     eax,[ebx+ecx*2h
lea     eax,[ebx+ecx]
sub     eax,[ebx+ecx*4h-20h]

AT&T Syntax
instr   %segreg:disp(base,index,scale),foo
movl    0x20(%ebx),%eax
addl    (%ebx,%ecx,0x2),%eax
leal    (%ebx,%ecx),%eax
subl    -0x20(%ebx,%ecx,0x4),%eax
```

## Mnemonic Suffixes

- Att: The AT&T syntax **mnemonics** have a suffix. The significance of this suffix is that of operand size. ]
- Intel: Suffix for **memory operands**

|  Style     |  8 bits     | 16 bit | 32 bits| 64 bits |
|:----:|:------:|:--:|:--:|:--:|
|  ATT(GAS)     |   'b'    | 'w'|'l' | 'q'|
|  Intel(NASM)     | byte ptr      | word ptr | dword ptr | qword ptr |

  ```
   Intel Syntax
   mov     al,bl
   mov     ax,bx
   mov     eax,ebx
   mov     eax, dword ptr [ebx]

   AT&T Syntax
   movb    %bl,%al
   movw    %bx,%ax
   movl    %ebx,%eax
   movl    (%ebx),%eax
  ```
## Data Size Declarations
| Style | 8 bits | 16 bit | 32 bits| 64 bits | Example|
|:--:|:--:|:--:|:--:|:--:|:--:|
|  ATT(GAS)     |   'db'    | 'dw'|'dd' | 'dq'|  `var1 dd 40` |
| Intel(NASM) | .byte | .int | .long | | `var1: .int 40` | 


## Comment
`;` in Intel and `#` or  C style  in Att.


## Examples
NASM
```
; Text segment begins
section .text

   global _start

; Program entry point
   _start:

; Put the code number for system call
      mov   eax, 1

; Return value
      mov   ebx, 2

; Call the OS
      int   80h
```
GAS
```
# Text segment begins
.section .text

   .globl _start

# Program entry point
   _start:

# Put the code number for system call
      movl  $1, %eax

/* Return value */
      movl  $2, %ebx

# Call the OS
      int   $0x80

```


# References
- [Linux assemblers: A comparison of GAS and NASM](https://developer.ibm.com/articles/l-gas-nasm/)
