---
layout: post
title:  Compilers Leveraging Undefined Behavious
tags:
- Undefined behaviour
date: 2017-09-12
---

One of the classic examples of compilers making use of undefined behaviour is as follows:
C standars says signed integers overflow is undefined.

Knowing this information help compiler to optimize `x+1>x` to true. As compilers know that `INT_MAX+1` is undefined so it can safely make the optimization.

Had signed integer overflow been defined (with a definition of say wrap around), then we will not be able to do the optimization as x + 1 is not `>` x if x == `INT_MAX` (under the wrap around defintion)


# Leveraging undefined behavious by optimizing compilers.

Undefined behaviors facilitate optimizations by permitting a compiler to assume that programs will only execute defined operations.

## Case I
```C
#include <iostream>

int fermat() {
  const int MAX = 1000;
  int a=1,b=1,c=1;
  // Endless loop with no side effects is UB
  while (1) {
    if (((a*a*a) == ((b*b*b)+(c*c*c)))) return 1;
    a++;
    if (a>MAX) { a=1; b++; }
    if (b>MAX) { b=1; c++; }
    if (c>MAX) { c=1;}
  }
  return 0;
}

int main() {
  if (fermat())
    std::cout << "Fermat's Last Theorem has been disproved.\n";
  else
    std::cout << "Fermat's Last Theorem has not been disproved.\n";
}
```
Result:
```
Fermat's Last Theorem has been disproved.
```

Despite the fact that this program does not contain any arithmetic overflows
(multiplier factors vary in the range from 1 to 1000, the sum of their cubes
 does not exceed 2^31), the C++ standard defines an infinite loop as an
undefined action, without changing the external state. That’s why C++ compilers
are entitled to consider similar loops as finite.

The compiler can easily see that the only way out of the while(1) loop is the
return 1; statement, while the return 0; statement at the end of fermat()
cannot be reached. Therefore, it optimizes this function to

int fermat (void)
{
  return 1;
}
In other words, the only possibility to write an infinite loop that could not be
removed by the compiler is to add a modification of the external state to the
loop body.

## Case II
```C
int table[4];
bool exists_in_table(int v)
{
    for (int i = 0; i <= 4; i++) {
        if (table[i] == v) return true;
    }
    return false;
}
```

First of all, you might notice the off-by-one error in the loop control. The
result is that the function reads one past the end of the table array before
giving up. A classical compiler wouldn't particularly care. It would just
generate the code to read the out-of-bounds array element (despite the fact
    that doing so is a violation of the language rules), and it would return
true if the memory one past the end of the array happened to match.

A post-classical compiler, on the other hand, might perform the following
analysis:

- The first four times through the loop, the function might return true.
- When i is 4, the code performs undefined behavior. Since undefined behavior
lets me do anything I want, I can totally ignore that case and proceed on the
assumption that i is never 4. (If the assumption is violated, then something
    unpredictable happens, but that's okay, because undefined behavior grants
    me permission to be unpredictable.)
- The case where i is 5 never occurs, because in order to get there, I first
have to get through the case where i is 4, which I have already assumed cannot
happen.
- Therefore, all legal code paths return true.  As a result, a post-classical
compiler can optimize the function to ```C bool exists_in_table(int v) { return
  true; } ```

## Case III
```C
int foo(int x) {
    return x+1 > x; // either true or UB due to signed overflow
}
```
may be compiled as (demo)
```C
foo(int):
        movl    $1, %eax
        ret
```
Because in all legal cases true is returned.

## Case IV
```C
std::size_t f(int x)
{
    std::size_t a;
    if(x) // either x nonzero or UB
        a = 42;
    return a;
}
```
May be compiled as (demo)
```C
f(int):
        mov     eax, 42
        ret
```
Because in all legal cases 42 is returned.

```C
bool p; // uninitialized local variable
if(p) // UB access to uninitialized scalar
    std::puts("p is true");
if(!p) // UB access to uninitialized scalar
    std::puts("p is false");
```
Possible output:
```
p is true
p is false
```
The code will ub are optimized out.

## Case V
```C
int foo(int* p) {
    int x = *p;
    if(!p) return x; // Either UB above or this branch is never taken
    else return 0;
}
```
may be compiled as
```
foo(int*):
        xorl    %eax, %eax
        ret

```
If p is null, the if access is UD; If p is non null, then `return 0` will
happen. So in legal cases, 0 is returned and so is the optimization.

# Pointer Overflow
It is undefined behavior to perform pointer arithmetic where the result is outside of an object, with the exception that it is permissible to point one element past the end of an array:

```
int a[10];
int *p1 = a - 1; // UB
int *p2 = a; // ok
int *p3 = a + 9; // ok
int *p4 = a + 10; // ok, but can't be dereferenced
int *p5 = a + 11; // UB
```
## Interesting case
```C
char buffer[BUFLEN];
char *buffer_end = buffer + BUFLEN;

/* ... */
unsigned int len;

if (buffer + len >= buffer_end)
  die_a_gory_death("len is out of range\n");
```
Here, the programmer is trying to ensure that len (which might come from an untrusted source) fits within the range of buffer. There is a problem, though, in that if len is very large, the addition could cause an overflow, yielding a pointer value which is less than buffer. So a more diligent programmer might check for that case by changing the code to read:

```C
if (buffer + len >= buffer_end || buffer + len < buffer)
  loud_screaming_panic("len is out of range\n");
```

This code should catch all cases; ensuring that len is within range. There is only one little problem: recent versions of GCC will optimize out the second test (returning the if statement to the first form shown above), making overflows possible again. So any code which relies upon this kind of test may, in fact, become vulnerable to a buffer overflow attack.

This behavior is allowed by the C standard, which states that, in a correct program, pointer addition will not yield a pointer value outside of the same object. So the compiler can assume that the test for overflow is always false and may thus be eliminated from the expression. It turns out that GCC is not alone in taking advantage of this fact: some research by GCC developers turned up other compilers (including PathScale, xlC, LLVM, TI Code Composer Studio, and Microsoft Visual C++ 2005) which perform the same optimization. So it seems that the GCC developers have a legitimate reason to be upset: CERT would appear to be telling people to avoid their compiler in favor of others - which do exactly the same thing.

The right solution to the problem, of course, is to write code which complies with the C standard. In this case, rather than doing pointer comparisons, the programmer should simply write something like:

```C
if (len >= BUFLEN)
    launch_photon_torpedoes("buffer overflow attempt thwarted\n");
```

### References
- [John Regehr Blog: Compilers and Termination Revisited](https://blog.regehr.org/archives/161)
- [cppreference.com](http://en.cppreference.com/w/c/language/behavior)
- [Undefined behavior can result in time travel](https://blogs.msdn.microsoft.com/oldnewthing/20140627-00/?p=633/)
- [Pointer Overflow Checking](https://blog.regehr.org/archives/1395)
- [GCC & pointer overflows](https://lwn.net/Articles/278137/)
