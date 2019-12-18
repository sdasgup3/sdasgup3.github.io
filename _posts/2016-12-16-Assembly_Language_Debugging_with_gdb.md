---
layout: post
title: Assembly Language Debugging with gdb
tags:
- Assembly
- Decompilation
date: 2016-12-16
---
Assembly Language Debugging with gdb.

## Useful commands
- Signal Handlers
```
info signal SIGUSR1
handle SIGUSR1 noprint nostop
```

- Run gdb in Text User Interface (TUI) mode
```shell
gdb -tui <program>
```

- Change the layout to Assembly
```
layout asm
```
- Or split the layout to C and Assembly
```
layout split
```
- Apply the following customization
```
set disassembly-flavor att/intel
set print asm-demangle
set disassemble-next-line on  //ask gdb to show us the next instruction every time
```
- Puts break point on main and invoke run
```
start
```

- Examine memory: x/FMT ADDRESS.
  - ADDRESS is an expression for the memory address to examine.
  - FMT == [NUM][FORMAT][SIZE]
    - [Format] ::= [ o(octal), x(hex), d(decimal), u(unsigned decimal), t(binary), f(float), a(address), i(instruction), c(char), s(string), z(hex, zero padded on the left)].
    - [Size] ::= [b(byte), h(halfword), w(word), g(giant, 8 bytes)]
    - The specified number of objects of the specified size are printed according to the format.  If a negative number is specified, memory is examined backward from the address.
    - Example
      ```
      x/10hb $rsp
      x/Ni $pc
      ```
- Use `nexti,stepi` instead of `next, step` which traverse the source lines
- Printing registers
```
info registers
info all-registers
info registers regname …
info registers eflags
```
- Printing & setting xmm registers
```
(gdb) print $xmm1
$1 = {
  v4_float = {0, 3.43859137e-038, 1.54142831e-044, 1.821688e-044},
  v2_double = {9.92129282474342e-303, 2.7585945287983262e-313},
  v16_int8 = "\000\000\000\000\3706;\001\v\000\000\000\r\000\000",
  v8_int16 = {0, 0, 14072, 315, 11, 0, 13, 0},
  v4_int32 = {0, 20657912, 11, 13},
  v2_int64 = {88725056443645952, 55834574859},
  uint128 = 0x0000000d0000000b013b36f800000000
}
```
To set values of such registers, you need to tell GDB which view of the register you wish to change, as if you were assigning value to a struct member:
```
 (gdb) set $xmm1.uint128 = 0x000000000000000000000000FFFFFFFF
```

## References
- [Debugging with GDB](https://sourceware.org/gdb/onlinedocs/gdb/index.html#SEC_Contents)
