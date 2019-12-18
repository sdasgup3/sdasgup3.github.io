---
layout: post
title:  LLVM Compiler Bugs related to UD
tags:
- Compilers,  Bugs, Alive
date: 06-10-2017
---
Here I am collecting some important points from various articiles mentioned in the reference section.

LLVM has three distinct kinds of undefined behavior. Together, they enable many desirable optimizations, and LLVM aggressively exploits these opportunities.

Undefined behavior in LLVM resembles undefined behavior in
C/C++: anything may happen to a program that executes it. The compiler may simply assume that undefined behavior does not oc- cur; this assumption places a corresponding obligation on the pro- gram developer (or on the compiler and language runtime, when a safe language is compiled to LLVM) to ensure that undefined op- erations are never executed. An instruction that executes undefined behavior can be replaced with an arbitrary sequence of instructions. When an instruction executes undefined behavior, all subsequent instructions can be considered undefined as well.

Following Table shows an arithmetic instructions have defined
behavior, following the LLVM IR specification. For example, the shl instruction is defined only when the shift amount is less than the bitwidth of the instruction.

Coming to the memory related instructions, the getelementptr instruction supports structured address computations: it uses a sequence of additions and multiplications to compute the address of a specific array element or structure field. For example, an array dereference in C such as val = a[b][c] can be translated to the following LLVM code:
```
%ptr = getelementptr %a, %b, %c
%val = load %ptr
```
Unstructured memory accesses are supported by the ``inttoptr`` instruction. The load and store instructions support typed memory reads and writes. Out-of-bounds and unaligned loads and stores result in true undefined behavior, but a load from valid, uninitialized memory returns an undef.

| Instruction  | Definedness Constraint  |
|---|---|
| sdiv a, b  |  b != 0 ∧ (a ?= INT MIN ∨ b != −1) |
| udiv a, b  |  b != 0 |
| srem a, b  |  b != 0 ∧ (a ?= INT MIN ∨ b != −1) |
| urem a, b  |  b != 0 |
| shl a, b   |  b <u B |
| lshr a, b  |  b <u B |
| ashr a, b  |  b <u B |

undef
-----
  - Explicit	value	in	the	IR
  - Acts	like	a	free-floaLng	hardware	register
    - Takes	all	possible	bit	pakerns	at	the	specified	width
    - Can	take	a	different	value	every	Lme	it	is	used
  - Comes	from	uniniLalized	variables
  - [Further	reading](http://sunfishcode.github.io/blog/2014/07/14/undef-introduction.html)


poison
------
- Ephemeral	effect	of	math	instrucLons	that	violate
  - nsw	–	no	signed	wrap	for	add,	sub,	mul,	shl
  - nuw	–	no	unsigned	wrap	for	add,	sub,	mul,	shl
  - exact	–	no	remainder	for	sdiv,	udiv,	lshr,	ashr
- Designed	to	support	speculative	execuLon	of
operaLons	that	might	overflow. For example we may host loop invariant ``x+1`` outside the loop as signed add might overflow.
- Poison	propagates	via	instrucLon	results
- If	poison	reaches	a	side-effecting	instrucLon,	the
result	is	true	UB.

True UD
-------
True	undefined	behavior
- Triggered	by
  - Divide	by	zero
  - Illegal	memory	accesses


Example 1
---------
```
%1 = add  %x, 1

=>
%1 = add nsw %x, 1

ERROR: Target is more poisonous than Source for i4 %1

Example:
%x i4 = 0x7 (7)
Source value: 0x8 (8, -8)
Target value: poison

```

Example 2
---------
```
%1 = add nsw %x, 1

=>
%1 = add  %x, 1

Optimization is correct
```

Example 3
---------
```
%1 = add nsw %x, 1
%2 = icmp sgt %1, %x

=>

%2 = true

Done: 1
Optimization is correct
```

shl
---
``<result> = shl <ty> <op1>, <op2>           ; yields ty:result``

Both arguments to the ‘shl‘ instruction must be the same integer or vector of integer type. ‘op2‘ is treated as an unsigned value.

The value produced is op1 * 2^op2 mod 26n, where n is the width of the result. If op2 is (statically or dynamically) equal to or larger than the number of bits in op1, this instruction returns a poison value. If the arguments are vectors, each vector element of op1 is shifted by the corresponding shift amount in op2.

If the nuw keyword is present, then the shift produces a poison value if it shifts out any non-zero bits.
or

``(a << b) >>u b = a`` where >>u is logical shift.

If the nsw keyword is present, then the shift produces a poison value it shifts out any bits that disagree with the resultant sign bit.
``(a<<b)>>b``

for n = 4; 0111 << 1 leads to poison as
```
0111 << 1 == 1110 >>1 == 1111 (!= 0111)
```


sdiv
--
Division by zero is undefined behavior. For vectors, if any element of the divisor is zero, the operation has undefined behavior. Overflow also leads to undefined behavior; this is a rare case, but can occur, for example, by doing a 32-bit division of -2147483648 by -1.

If the exact keyword is present, the result value of the sdiv is a poison value if the result would be rounded.

srem
----

Taking the remainder of a division by zero is undefined behavior. For vectors, if any element of the divisor is zero, the operation has undefined behavior. Overflow also leads to undefined behavior; this is a rare case, but can occur, for example, by taking the remainder of a 32-bit division of -2147483648 by -1. (The remainder doesn’t actually overflow, but this rule lets srem be implemented using instructions that return both the result of the division and the remainder.)

# Bug 20186

The transformation of -(X/C) to X/(-C) is invalid if C == INT_MIN.

```C
%a = sdiv %X, C
%r = sub 0, %a
=>
%r = sdiv %X, -C

ERROR: Domain of definedness of Target is smaller than Source's for i4 %r

Example:
%X i4 = 0x8 (8, -8)
C i4 = 0x1 (1)
%a i4 = 0x8 (8, -8)
Source value: 0x8 (8, -8)
Target value: undef
```

# Bug 20189

```
%B = sub 0, %A
%C = sub nsw %x, %B
=>
%C = add nsw %x, %A

ERROR: Target is more poisonous than Source for i4 %C

Example:
%A i4 = 0x8 (8, -8)
%x i4 = 0x8 (8, -8)
%B i4 = 0x8 (8, -8)
Source value: 0x0 (0) // -8 - (0 - (-8)) == -8 - (-8)
Target value: poison // -8 + -8
```

# Bug 21242
```
Pre: isPowerOf2(C1)
%r = mul nsw %x, C1

=>

%r = shl nsw %x, log2(C1)

ERROR: Target is more poisonous than Source for i4 %r

Example:
%x i4 = 0x1 (1)
C1 i4 = 0x8 (8, -8)
Source value: 0x8 (8, -8)
Target value: poison

// Source : mul nsw 1 * -8 (defined)
// Target : shl nsw 0001, 3 (poison)
```

# Bug 21243

```
Pre: !WillNotOverflowSignedMul(C1, C2)
%Op0 = sdiv %X, C1
%r = sdiv %Op0, C2

=>
%r = 0

ERROR: Mismatch in values of i4 %r

Example:
%X i4 = 0x8 (8, -8)
C1 i4 = 0x2 (2)
C2 i4 = 0x4 (4)
%Op0 i4 = 0xC (12, -4)
Source value: 0xF (15, -1)
Target value: 0x0 (0)

Source: 4/ (-8/2) = -1
Target : 0
```

# Bug 21245

```
Pre: C2 % (1<<C1) == 0
%s = shl nsw i4 %X, C1
%r = sdiv %s, C2
  =>
%r = sdiv %X, (C2 / (1 << C1))

ERROR: Mismatch in values of i4 %r

Example:
%X i4 = 0xF (15, -1)
C1 i4 = 0x3 (3)
C2 i4 = 0x8 (8, -8)
%s i4 = 0x8 (8, -8)
Source value: 0x1 (1)
Target value: 0xF (15, -1)

Source:
1111 shl 3 bit == 1000 == -8
-8/C2 = 1

Target:
-1 / 8/8 or -1/ -8/-8 == -1 (15 or -1)
```

# Bug 21255

```
%Op0 = lshr %X, C1
%r = udiv %Op0, C2
  =>
%r = udiv %X, (C2 << C1)

ERROR: Domain of definedness of Target is smaller than Source's for i4 %r

Example:
%X i4 = 0x0 (0)
C1 i4 = 0x4 (4)
C2 i4 = 0x1 (1)
%Op0 i4 = poison
Source value: 0x0 (0)
Target value: UB

Source: lshr 0, 4 (poison as shift amount >= bitwidth)
Target: %x / 1 << 4 ( == 0) i.e. UD
```

And bypassing the undef case:
```
Pre: ((C2 << C1) != 0)
%Op0 = lshr exact %X, C1
%r = udiv %Op0, C2
  =>
%r = udiv %X, (C2 << C1)


ERROR: Mismatch in values of i4 %r

Example:
%X i4 = 0x8 (8, -8)
C1 i4 = 0x2 (2)
C2 i4 = 0x9 (9, -7)
%Op0 i4 = 0x2 (2)
Source value: 0x0 (0)
Target value: 0x2 (2)

Source: (lshr exact -8,2 ) / 9 == 2/9 = 0
Target: 8 / (-7 << 2) = 8 / (1001 << 2) = 8 / 4 = 2
```

And finally we have:

```
Pre: WillNotOverflowUnsignedShl(C2, C1)
%Op0 = lshr %X, C1
%r = udiv %Op0, C2
  =>
%r = udiv %X, (C2 << C1)

Done
Optimization is correct!
```

# Bug 21256
```
%Op1 = sub 0, %X
%r = srem %Op0, %Op1
  =>
%r = srem %Op0, %X


ERROR: Domain of definedness of Target is smaller than Source's for i4 %r

Example:
%X i4 = 0xF (15, -1)
%Op0 i4 = 0x8 (8, -8)
%Op1 i4 = 0x1 (1)
Source value: 0x0 (0)
Target value: undef

Source: -8 % (0 - (-1)) = -8 % 1 = 0
Target: -8 % -1 =  UD
```

# Bug  31633
InstCombine currently folds  "select %c, undef, %foo" into %foo, because it assumes that undef can take any value that %foo may take.

```
%y2 = add nsw i32 %y, 1
%s = select i1 %c, i32 undef, i32 %y2
%r = icmp sgt i32 %s, %y

=>
%r = true

ERROR: Mismatch in values of i1 %r

Example:
%y i32 = 0x7FFFFFFF (2147483647)
%c i1 = 0x1 (1, -1)
%y2 i32 = poison
%s i32 = 0x00000000 (0)
Source value: 0x0 (0)
Target value: 0x1 (1, -1)

%y2 overflows and becomes poison, but the select should return undef only, not poison.
```

# Refereces
 - [Undefined Behavious in LLVM](https://www.cs.utah.edu/~regehr/llvm-ub.pdf)
 - [LLVM language referece manual](https://llvm.org/docs/LangRef.html)
 - [Provably correct peephole optimizations with alive](https://dl.acm.org/citation.cfm?id=2737965)
 - [Alive](https://rise4fun.com/Alive)
 - [Translation Validation of Bounded Exhaustive Test Cases](https://blog.regehr.org/archives/1510)
