---
title: ARM64 load/store register instructions
layout: post
---

This article accompanies Lesson 3 "LDR, STR" of LaurieWired in the [ARM assembly tutorial](https://www.youtube.com/playlist?list=PLn_It163He32Ujm-l_czgEBhbJjOUgFhg). Refer to the [The first ARM64 assembly program](/2025/07/13/first-arm64-code.html) for how to call Linux (for ARM64) `exit()` to end execution.

ARM64 has x0-x30 64-bit general purpose registers that usually store integers. Each register is a memory cell to store some value. Their respective lower 32-bit parts are named w0-w30.


## Documentation for ARM32 and ARM64 assembly
On [https://developer.arm.com](https://developer.arm.com), search for "armasm user guide". In the result list, one may find 

Arm Compiler armasm User Guide. Version: 6.6.5. Recommended

Version 6.6.5 is the latest version as of writing. Click on it. The LDR and STR instructions are documented under the section "A64 Data Transfer Instructions".

## ARM64 general purpose registers
According to ARM's documentation [Aarch64 registers](https://developer.arm.com/documentation/102374/0102/Registers-in-AArch64---general-purpose-registers), Aarch64 has 31 general purpose registers. A register is a memory cell into which one can write values, usually integer. Each register can be used as a 64-bit X register (X0..X30), or its lower 32-bit part as a 32-bit W register (W0..W30).

![Aarch64 general purpose registers](/assets/arm64/a64_registers.png)

When X register is used, 64-bit calculation is performed. When W register is used, 32-bit calculation is performed. For example,

```asm
add x0, x1, x2
```

adds the 64-bit content, usually integer, in x1 and x2, and store the 64-bit result into x0.

When you use the 32-bit form of an instruction, the upper 32 bits of the source registers are ignored and the upper 32 bits of the destination register are set to zero. For example, we make a 64-bit integer in x0,

```asm
mov x0, #0x52
lsl x0, x0, #32      // shift left by 32 bits
add x0, x0, #0x21
```

we would obtain

x0 = 0x5200000021

by now. If we append

```asm
add w0, w0, #0x100
```

this line would set the upper 32-bit of x0 to zero. Effectively we obtain

x0 = w0 = 0x121.

Compare with the following code:

```asm
mov w0, #0x12        // x0 = w0 = 0x12
lsl x0, x0, #32      // x0 = 0x1200000000
add x0, x0, #0x35    // x0 = 0x1200000035
```

There is no register named W31 or X31. Depending on the instruction, register 31 is either the stack pointer or the zero register. When used as the stack pointer, you refer to it as SP. When used as the zero register, you refer to it as WZR in a 32-bit context or XZR in a 64-bit context.

## LDR (load register)
LDR reads a word (32-bit) at specified address. We will look at two versions of `ldr.s` that does the same thing.

### version 1
```asm
.global _start

.text
_start:
    // load content of var2, the decimal 6
    ldr x0, var2     // x0 stores return value for exit()
    mov x8, #0x5d    // x8 stores function number for exit()
    svc #0

.data
// a word is 32-bit
var1: .word 5     // decimal 5
var2: .word 6     // decimal 6
```

The code in this article has to be linked with `-static` flag, or an error will be produced. To run it in the shell:

```sh
$ as -o ldr.o ldr.s
$ gcc -o ldr ldr.o -nostdlib -static
$ ./ldr; echo $?
6
```

If we dissemble it in the shell:

```sh
$ objdump -d ldr

ldr:     file format elf64-littleaarch64


Disassembly of section .text:

00000000004000b0 <_start>:
  4000b0:	58080080 	ldr	x0, 4100c0 <var2>
  4000b4:	d2800ba8 	mov	x8, #0x5d                  	// #93
  4000b8:	d4000001 	svc	#0x0
```

We suppose at the address 0x4100c0 is stored the number 6. Can we prove it?

```
$ readelf -x .data ldr

Hex dump of section '.data':
  0x004100bc 05000000 06000000                   ........
```

We see at the address 0x004100bc is stored `0x05000000`. An 8-digit hexademical number would occupy 4 bytes (one byte occupying 8 binary bits):

each single hexadecimal digit can represent 16 different numbers. In binary, a single hexadecimal digit has to be represented with 4 binary bits so when flipping each bit between 0 and 1, that would produce 2 ** 4 = 16 different choices. 8 of hexadecimal digits would therefore be represented by 4 x 8 = 32 binary bits, or 4 bytes. One byte occupies 8 binary bits.

Addresses are measured in bytes. As the address 0x004100bc has stored four bytes of content, the following address would be 0x004100bc incremented by 4, 0x004100bc + 0x00000004 = 0x004100c0, and `0x06000000` is stored there.

But `0x06000000` looks many times larger than the 6 we obtained? We have to talk about how ARM64 stores numbers, or particularly the order in which it stores the bytes of a number. If we search for "aarch64 little endian", we may find:

> AArch64, which is the 64-bit architecture for ARM, typically uses little-endian format by default. This means that the least significant byte is stored at the smallest memory address.

By this rule, to record the number 0x00000006, the four bytes 0x00, 0x00, 0x00, 0x06 in the number would go into addresses

| The Address  | Recorded Byte |
| ------- | ------- |
| 0x004100c0 | 0x06 |
| 0x004100c1 | 0x00 |
| 0x004100c2 | 0x00 |
| 0x004100c3 | 0x00 |

with the least significant byte 0x06 going into the lowest address 0x004100c0.

### version 2
We look at an variety of `ldr.s`:

```asm
.global _start

.text
_start:
    ldr x1, =var2
    ldr x0, [x1]       // x0 stores return value for exit()
    mov x8, #0x5d      // x8 stores function number for exit()
    svc #0

.data
var1: .word 5
var2: .word 6
```

It would still return the number 6:

```sh
$ as -o ldr.o ldr.s
$ gcc -o ldr ldr.o -nostdlib -static
$ ./ldr; echo $?
6
```

Let's look at the dissembled `ldr`:

```sh
$ objdump -d ldr

ldr:     file format elf64-littleaarch64


Disassembly of section .text:

00000000004000b0 <_start>:
  4000b0:	58000081 	ldr	x1, 4000c0 <_start+0x10>
  4000b4:	f9400020 	ldr	x0, [x1]
  4000b8:	d2800ba8 	mov	x8, #0x5d                  	// #93
  4000bc:	d4000001 	svc	#0x0
  4000c0:	004100cc 	.word	0x004100cc
  4000c4:	00000000 	.word	0x00000000
```

The line `ldr x1, 4000c0` would load the 32-bit word at address 0x4000c0 into register `x1`. Several lines down, `4000c0: 004100cc` indicates that at 0x4000c0 is stored the word 0x004100cc, therefore 0x004100cc is read into `x1`.

The next line `ldr x0, [x1]` would interpret what's stored in x1 as an address, and read the word stored at that address. Let's check if `0x004100cc` is an address in the section `.data`.

```sh
$ readelf -x .data ldr

Hex dump of section '.data':
  0x004100c8 05000000 06000000                   ........

```

So at the address 0x004100c8 is stored some content 0x05000000. The next address would be 0x004100c8 + 0x00000004 = 0x004100cc, and 0x06000000 is stored there. For what we have explained about endianness, that would be hexadecimal number 0x6, equivalent to decimal number 6.

## STR (store register)
STR stores the content in a register to a specified address. The ARM64 code is not much different from the ARM32 one in Laurie's lesson.

```asm
.global _start

.text
_start:
    ldr x2, =var1
    ldr x3, =var2
    mov x1, #5
    str x1, [x3]
    ldr x0, [x3]
    mov x8, #0x5d
    svc #0

.data
var1: .word 6
var2: .word 7
```

To run it,

```sh
$ as -o str.o str.s
$ gcc -o str str.o -nostdlib -static
$ ./str; echo $?
5
```

## References
ARM assembly tutorial, @LaurieWired, [https://www.youtube.com/playlist?list=PLn_It163He32Ujm-l_czgEBhbJjOUgFhg](https://www.youtube.com/playlist?list=PLn_It163He32Ujm-l_czgEBhbJjOUgFhg)

First ARM64 assembly program, [/2025/07/13/first-arm64-code.html](/2025/07/13/first-arm64-code.html)

Arm Compiler armasm User Guide. On [https://developer.arm.com](https://developer.arm.com), search for "armasm user guide". In the result list, find the latest version of "Arm Compiler armasm User Guide".

Aarch64 registers, [https://developer.arm.com/documentation/102374/0102/Registers-in-AArch64---general-purpose-registers](https://developer.arm.com/documentation/102374/0102/Registers-in-AArch64---general-purpose-registers)

Registers in Aarch64 state, [https://developer.arm.com/documentation/dui0801/l/Overview-of-AArch64-state/Registers-in-AArch64-state](https://developer.arm.com/documentation/dui0801/l/Overview-of-AArch64-state/Registers-in-AArch64-state)
