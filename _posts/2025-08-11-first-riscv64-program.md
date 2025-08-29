---
title: First RISC-V64 program
layout: post
---

RISC-V64, as ARM64, is another simple [reduced instruction set computer](https://en.wikipedia.org/wiki/Reduced_instruction_set_computer) architecture. RISC-V is not owned by a corporate but a [consortium](https://riscv.org/). RISC-V is not one single instruction set, but a family of base sets and extensions, e.g, "I" for the base integer instruction set, "V" for the extension for vector operations. Any base set or extension may undergo revisions, but it tends to be overall stable. Manufacturers can combine a base set with any extensions for a chip, for example "E" only for microcontrollers, but "IFD" for integer, single-width floating point and double-width floating point arithmetic.

The main reference is the **instruction set manual** at [https://github.com/riscv/riscv-isa-manual](https://github.com/riscv/riscv-isa-manual), whose README links to the HTML snapshots of [user-level (unprivileged) instruction sets](https://riscv.github.io/riscv-isa-manual/snapshot/unprivileged/) and [privileged instruction sets](https://riscv.github.io/riscv-isa-manual/snapshot/privileged/).

We will install the RISC-V development toolchain on a x86_64 host and write the first RISC-V64 assembly program on this non-RISC-V-native host. A toolchain is the collection of assembler, compiler, and linker that translate assembly, C programs to executable binary ones.

# Bare metal development toolchain
Bare metal, means executable files produced by the toolchain will be able to run with no support of the underlying operating system (for example to translate memory addresses) or any simulator of RISC-V64.

Clone the repo at [https://github.com/riscv-collab/riscv-gnu-toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain),

```sh
git clone https://github.com/riscv-collab/riscv-gnu-toolchain
```

and follow the instructions and make the default target as under the "Installation (Newlib)" section.

```sh
mkdir /opt
configure --prefix=/opt/rv64gcv --with-arch=rv64gcv --with-abi=lp64d
make -j 2
```

After the make, 18G disk space is taken on a Ubuntu x86_64 system. After making of the toolchain, in the cloud you can switch the virtual machine to less powerful instance type, with at least 2G memory.

If you follow the instruction as under the "Installation (Linux)" section, it will take longer and more disk space to make the Linux/GNU toolchain. Executable files produced by this toolchain will be trickier to run, depending on the support of the underlying operating system and a RISC-V simulator.

## `arch` (architecture) and `abi` (application binary interface) for RISC-V64
The `--with-arch=rv64gcv` parameter configures the architecture: the "G" represents the "I" base set and the "M", "A", "F", "D", "Zicsr" and "Zifencei" extensions, enough for a general purpose computer. [The RISC-V Instruction Set Manual Volume I: Unprivileged Architecture](https://riscv.github.io/riscv-isa-manual/snapshot/unprivileged/) 36.3 Instruction-Set Extension Names

| acrynom | instruction set |
| --------- | ------------------- |
|  I      | integer    |
|  M      | integer multiplication/division |
|  A      | atomic instructions |
| Zicsr   | extension for control and status register instructions |
|  F      | single-precision floating point, depends on "Zicsr" |
|  D      | double-precision floating point, depends on "F" |
| Zifencei | extension for instruction-fetch fence |
|  C      | compressed 16-bit instructions (instead of 32-bit) |
|  V      | vector      |

`abi`, or application binary interface, specifies how parameters are passed in a function call, for example, whether it takes one 64-bit register or two 32-bit registers to send one 64-bit number. For more detail, refer to [The -march, -mabi, and -mtune arguments to RISC-V Compilers](https://www.sifive.com/blog/all-aboard-part-1-compiler-args), part of a blog series.

This whole SiFive blog series is worth a read:

* As each RISC-V instruction is encoded with 32-bit, it takes two instructions to load a 32-bit address;
* it also influences the choice of "memory model": the range of memory a program has access to.

## RISC-V64 registers
A register is a location that stores a number on a computer architecture. For either the 32-bit base integer instruction set RV32I or the 64-bit counterpart RV64I, XLEN characters the width in bits of an integer value stored in a register:

for RV32I, XLEN = 32; for RV64I, XLEN = 64.

Apart from the different XLEN, RV32I and RV64I define the same 32 general purpose registers: x0-x31. The value in x0 is always 0, even an instruction tries to write another value. There is an extra unprivileged program counter register pc that points at the current instruction in the memory.

By convention, x1 is used to store the return address that the current function shall return to, x2 as the stack pointer, etc. For this convention, the ABI (application binary interface) gives mnemonic names: ra for x1, sp for x2, etc, which are listed below.

|  Register  |  ABI mnemonic  | Use by convention | Preserved by the callee |
| ---------- |  ------------- | ----------------- | ----------------------- |
|  x0        |   zero         | hardwired to 0, ignore writes |  n/a  |
|  x1        |   ra           | return address    |   no    |
|  x2        |   sp           | stack pointer     |   yes   |
|  x3        |   gp           | global pointer    |   n/a   |
|  x4        |   tp           | thread pointer    |   n/a   |
|  x5        |   t0           | temporary register 0 |   no   |
|  x6        |   t1           | temporary register 1 |   no   |
|  x7        |   t2           | temporary register 2 |   no   |
|  x8        |   s0 or fp     | saved register 0 or frame pointer  | yes |
|  x9        |   s1           | saved register 1  |   yes   |
|  x10       |   a0           | return value or function argument 0 | no |
|  x11       |   a1           | return value or function argument 1 | no |
|  x12       |   a2           | function argument 2  |   no   |
|  x13       |   a3           | function argument 3  |   no   |
|  x14       |   a4           | function argument 4  |   no   |
|  x15       |   a5           | function argument 5  |   no   |
|  x16       |   a6           | function argument 6  |   no   |
|  x17       |   a7           | function argument 7  |   no   |
|  x18       |   s2           | saved register 2     |   yes  |
|  x19       |   s3           | saved register 3     |   yes  |
|  x20       |   s4           | saved register 4     |   yes  |
|  x21       |   s5           | saved register 5     |   yes  |
|  x22       |   s6           | saved register 6     |   yes  |
|  x23       |   s7           | saved register 7     |   yes  |
|  x24       |   s8           | saved register 8     |   yes  |
|  x25       |   s9           | saved register 9     |   yes  |
|  x26       |   s10          | saved register 10    |   yes  |
|  x27       |   s11          | saved register 11    |   yes  |
|  x28       |   t3           | temporary register 3 |   no   |
|  x29       |   t4           | temporary register 4 |   no   |
|  x30       |   t5           | temporary register 5 |   no   |
|  x31       |   t6           | temporary register 6 |   no   |
|  pc        |  (none)        | program counter      |   n/a  |

The current function (callee) has to make sure that at the exit of the callee function s0-s11 and sp preserve the same values as at the entry of it.

RV32E and RV64E are for microcontroller use. The only difference between RV32I and RV32E, or between RV64I and RV64E is that RV32E or RV64E has only 16 registers. By the convention above, all the indispensable registers are defined within x0-x15: ra, sp, etc. x16-x31 only increased the number of function arguments, saved registers and temporary registers.

One can write assembly progam either with the general names x0-x31 or the mnemonic names. The disassembler `objdump` normally displays the mnemonic names, but can output the general names with option `-M numeric`.

## The first RISC-V64 assembly program
We copy the `hello.s` in [RISC-V Assembly Hello World (part 1)](https://www.youtube.com/watch?v=0IeOaiKszLk) by LaurieWired, which simply calls the Linux `exit()`.

```asm
.global _start

.text
_start:
   li a0, 2      # return value for Linux exit()
   li a7, 93     # function number for Linux exit()
   ecall
```

We run the toolchain to produce the executable `hello`:

```sh
$ /opt/rv64gcv/bin/riscv64-unknown-elf-as -o hello.o hello.s
$ /opt/rv64gcv/bin/riscv64-unknown-elf-ld -o hello hello.o
```

As we are using the bare metal toolchain, we'll find `hello` can run by itself,

```sh
$ ./hello; echo $?
2
```

Alternatively,

```sh
$ /opt/rv64gcv/bin/riscv64-unknown-elf-run hello; echo $?
2
```

## References
RISC-V Instruction Set Manual, [https://github.com/riscv/riscv-isa-manual](https://github.com/riscv/riscv-isa-manual)

The RISC-V Instruction Set Manual Volume I: Unprivileged Architecture, [https://riscv.github.io/riscv-isa-manual/snapshot/unprivileged/](https://riscv.github.io/riscv-isa-manual/snapshot/unprivileged/)

RISC-V GNU toolchain, [https://github.com/riscv-collab/riscv-gnu-toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain)

Spike RISC-V ISA Simulator, [https://github.com/riscv-software-src/riscv-isa-sim](https://github.com/riscv-software-src/riscv-isa-sim)

RISC-V Proxy Kernel and Boot Loader, [https://github.com/riscv-software-src/riscv-pk](https://github.com/riscv-software-src/riscv-pk), which translates I/O calls to the host computer.

Setup Riscv-GNU-TOOLCHAIN and SPIKE for the Vector Extension, Syed Hassan Ul Haq, [https://medium.com/@ulhaqhassan1/setup-riscv-gnu-toolchain-and-spike-for-the-vector-extnesion-a10ede8b1857](https://medium.com/@ulhaqhassan1/setup-riscv-gnu-toolchain-and-spike-for-the-vector-extnesion-a10ede8b1857)

Setting up RISC-V toolchain and simulator, Mohamed A. Bamakhrama, [https://gist.github.com/mohamed/a6e406e086e6c9cc0ded222d23bcb0a6](https://gist.github.com/mohamed/a6e406e086e6c9cc0ded222d23bcb0a6)

All Aboard, Part 1: The -march, -mabi, and -mtune arguments to RISC-V Compilers, part of a SiFive blog series, [https://www.sifive.com/blog/all-aboard-part-1-compiler-args](https://www.sifive.com/blog/all-aboard-part-1-compiler-args)

RISC-V64 assembly language setup and first steps, Russ Ross, [https://www.youtube.com/watch?v=5g8M85r8Au8](https://www.youtube.com/watch?v=5g8M85r8Au8)

RISC-V64 assembly language, Digital Design and Computer Architecture Chapter 6 Architecture, Sarah L Harris and David Harris, [https://www.youtube.com/playlist?list=PLcbc_WBQVCytATU2xxAqcFkynK8hRflSz](https://www.youtube.com/playlist?list=PLcbc_WBQVCytATU2xxAqcFkynK8hRflSz)

RISC-V Linux syscall table, Juraj Borza, [https://jborza.com/post/2021-05-11-riscv-linux-syscalls/](https://jborza.com/post/2021-05-11-riscv-linux-syscalls/)

RISC-V Assembly Hello World (part 1), LaurieWired, [https://www.youtube.com/watch?v=0IeOaiKszLk](https://www.youtube.com/watch?v=0IeOaiKszLk)

RISC-V Assembly Programmer's Handbook, [https://github.com/riscv-non-isa/riscv-asm-manual](https://github.com/riscv-non-isa/riscv-asm-manual)

RISC-V Assembly Cheat Sheet, [https://projectf.io/posts/riscv-cheat-sheet/](https://projectf.io/posts/riscv-cheat-sheet/)
