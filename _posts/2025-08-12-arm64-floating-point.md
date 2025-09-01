---
title: ARM64 floating point arithmetic
layout: post
---

## ARM64 floating point registers
Aarch64 has 32 floating point registers. Each is 128-bit under name Q0-Q31. If one needs less precision for example 32-bit, one can access the lowest 32-bit part of each register under a different name. For example,

| --------- | --------- |
|     Q0    |  128-bit  |
|    D0     |  the lowest 64-bit part of Q0 |
|    S0     |  the lowest 32-bit part of Q0 |
|    H0     |  the lowest 16-bit part of Q0 |
|    B0     |  the lowest 8-bit part of Q0  |

Below is a graphical illustration,

![a64 floating point registers](/assets/arm64/a64_fp_registers.png)

## Floating point registers preserved by the callee function
Certain registers need to stay the same at the exit as at the entry of a function/subroutine being called (callee).

The first eight registers, v0-v7, are used to pass argument values into a subroutine and to return result values from a function. They may also be used to hold intermediate values within a routine (but, in general, only between subroutine calls).

Registers v8-v15 must be preserved by a callee across subroutine calls; the remaining registers (v0-v7, v16-v31) do not need to be preserved (or should be preserved by the caller). Additionally, only the bottom 64 bits of each value stored in v8-v15 need to be preserved; it is the responsibility of the caller to preserve larger values. [Procedure Call Standard for Aarch64 6.1.2 SIMD and Floating-Point registers](https://github.com/ARM-software/abi-aa/blob/main/aapcs64/aapcs64.rst#simd-and-floating-point-registers)

## Project Overview
In previous posts, the result in an interger is returned by the Linux `exit()` call. But in this post the result is a pointing point value, so we change the game plan -- we'll use the C function `printf()` to print it out.

We first write `add.h` and `main.c`:

`add.h`:

```c
float add(float a, float b);
```

`main.c`:

```c
#include "add.h"
#include <stdio.h>

int main() {
    printf("%f\n", add(3.4, -2.7));
    return 0;
}
```

what remains is to write an `add.s`, assemble and link it with `main.c` to produce the executable file.

## The adding function in assembly
Recall that all constants are preceded by `#`. `#0x10` is the hexadecimal 10, equivalent to decimal 16. `#12` is just the decimal 12.

The C `float` type is typically 32-bit. In C, the interface of `add()` was declared as

```c
float add(float a, float b);
```

At the entry of the `add` routine in the assembly, the input parameters a and b are in the registers s0, s1 of 32-bit width. The result will be in s0. They don't have to be preserved by the callee.

Refer to the [ARM64 function call](/2025/08/04/arm64-func.html) post for the use of the stack.

`add.s`:

```asm
.global add

.text
add:
    sub sp, sp, #0x10       // set sp to sp - 16 bytes
    str lr, [sp, #8]        // store link register lr at sp + 8 bytes
    str fp, [sp]            // store frame pointer fp at sp
    mov fp, sp              // set frame pointer fp to sp

    fadd s0, s0, s1         // floating-point add
                            // store result in s0

    ldr fp, [sp]            // restore frame pointer fp from sp
    ldr lr, [sp, #8]        // restore link register lr from sp + 8 bytes
    add sp, sp, #0x10       // set sp to sp + 16 bytes
    ret
```

## Run the project
We assemble the `add.s` into a binary file `add.o`:

```sh
$ as -o add.o add.s
```

then compile the remaing C files, and link with `add.o` to produce the final executable file `main`:

```sh
$ gcc -o main main.c add.o
```

We run it:

```sh
$ ./main
0.700000
```

## References
Arm Compiler armasm User Guide. On [https://developer.arm.com](https://developer.arm.com), search for "armasm user guide". In the result list, find the latest version of "Arm Compiler armasm User Guide".

Aarch64 registers, [https://developer.arm.com/documentation/102374/0102/Registers-in-AArch64---general-purpose-registers](https://developer.arm.com/documentation/102374/0102/Registers-in-AArch64---general-purpose-registers)

Procedure Call Standard for Aarch64, [https://github.com/ARM-software/abi-aa/blob/main/aapcs64/aapcs64.rst](https://github.com/ARM-software/abi-aa/blob/main/aapcs64/aapcs64.rst)

ARM64 load/save register instructions, [/2025/07/14/arm64-ldr-and-str.html](/2025/07/14/arm64-ldr-and-str.html)

ARM64 function calls, [/2025/08/04/arm64-func.html](/2025/08/04/arm64-func.html).

Introduction to Aarch64 architecture, 8. The Stack, [https://hrishim.github.io/llvl_prog1_book/stack.html](https://hrishim.github.io/llvl_prog1_book/stack.html).
