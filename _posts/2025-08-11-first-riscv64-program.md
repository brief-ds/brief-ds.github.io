---
title: First RISC-V64 program
layout: post
---

RISC-V64, as ARM64, is another simple [reduced instruction set computer](https://en.wikipedia.org/wiki/Reduced_instruction_set_computer) architecture. RISC-V is not owned by a corporate but a [consortium](https://riscv.org/). It is standardised into base sets and extensions, e.g, "I" for the base integer instruction set, "V" for the extension for vector operations. Any base set or extension may undergo revisions, but it tends to be overall stable. Manufacturers can combine a base set with any extensions for a chip, for example "E" only for microcontrollers, but "IFD" for integer, single-width floating point and double-width floating point arithmetic.

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
