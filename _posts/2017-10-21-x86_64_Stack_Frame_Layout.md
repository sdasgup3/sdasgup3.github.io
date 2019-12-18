---
layout: post
title:  x86-64 Stack Frame Layout
tags:
- Syntax, X86/X86-64,
date: 12-10-2017
---


__The following is an exceprt from the article mentioned in Reference section.__

# Process Stack
```
High Addres --> -----------
                |         |
                |         |
                |         |
                |         |
                |         |
                -----------
  Oxffffff08    | foo     |  <-- XSP
                -----------
  Oxffffff00    |         |
Low Addres -->  |         |

```

To push new data onto the stack we use the push instruction
``push %rax``
Is actually equivalent to this:
```
sub $8, %rsp
mov %rax, (%rsp)
```

```
High Addres --> -----------
                |         |
                |         |
                |         |
                |         |
                |         |
                -----------
  Oxffffff08    | foo     |
                -----------
  Oxffffff00    | %rax val| <-- XSP
                -----------
Low Addres -->  |         |

```

Similarly, the pop instruction takes a value off the top of stack and places it in its operand, increasing the stack pointer afterwards. In other words, this:
``pop rax``
Is equivalent to this:
```
mov (%rsp), %rax
add  $8, %rsp
```
```
High Addres --> -----------
                |         |
                |         |
                |         |
                |         |
                |         |
                -----------
  Oxffffff08    | foo     |  <-- XSP
                -----------
  Oxffffff00    | %rax val| <-- Still there
                -----------
Low Addres -->  |         |

```

# Stack frames & Calling convention
```
int foobar(int a, int b, int c)
{
    int xx = a + 2;
    int yy = b + 3;
    int zz = c + 4;
    int sum = xx + yy + zz;

    return xx * yy * zz + sum;
}

int main()
{
    return foobar(77, 88, 99);
}
```
Right before the return statement, the stack frame for foobar looks like this:

![]({{site.url}}/assets/images/stackframe1.png)
![](images/stackframe1.png)


The green data were pushed onto the stack by the calling function, and the blue ones by foobar itself.


An x86-64 instruction may be at most 15 bytes in length. It consists of the following components in the given order, where the prefixes are at the least-significant (lowest) address in memory.

# Argument Passing

 According to the System V AMD 64 ABI, the first 6 integer or pointer arguments to a function are passed in registers. The order being:

```
 rdi:rsi:rdx:rcx:r8:r9
```
 The 7th argument and onwards are passed on the stack.

```
long myfunc(long a, long b, long c, long d,
            long e, long f, long g, long h)

{

}
rdi: a
rsi: b
rdx: c
rdx: d
r8: e
r9: f
g & h are passed onto stack
```

![]({{site.url}}/assets/images/x64_frame_nonleaf.png)
![](images/x64_frame_nonleaf.png)



## Reference
1. [Stack frame layout on x86-64](https://eli.thegreenplace.net/2011/09/06/stack-frame-layout-on-x86-64#id8)
