---
title: First RISC-V64 program
layout: post
---

RISC-V64, as ARM64, is another simple [reduced instruction set computer](https://en.wikipedia.org/wiki/Reduced_instruction_set_computer) architecture. As CPUs implementing it are not widely available, we will install the development toolchain  on a x86_64 host and write the first RISC-V64 assembly program on this foreign host x86_64.

A toolchain is the collection of assembler, compiler, and linker that translate any assembly, C programs in text to executable binary ones.

# Bare metal development toolchain
Bare metal, means executable files produced by the toolchain will be linked with a minimal set of code that enables the executable file to run alone, with no support of the underlying operating system or any simulator (of RISC-V64 on x86_64) at run time.

Clone the repo at [https://github.com/riscv-collab/riscv-gnu-toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain), follow the instructions and make the default target as under the "Installation (Newlib)" section. There will be more explanation on the parameters `--with-arch` and `--with-abi`, but below should make the toolchain for a RISC-V64 with enough features as a general computer platform (integer, floating point, etc).

```sh
configure --prefix=/opt/rv64gcv --with-arch=rv64gcv --with-abi=lp64d
make -j 2
```

After the make, 18G disk space is taken on a Ubuntu x86_64 system.

If you follow the instruction as under the "Installation (Linux)" section, it will take considerably longer and more disk space to make the Linux/GNU toolchain. Executable files produced by this toolchain will depend on support of the underlying operating system and simulators to run, harder to get right.

## `arch` and `abi` for RISC-V64
The [RISC-V user-level instruction sets](https://riscv.github.io/riscv-isa-manual/snapshot/unprivileged/) are standardised at the [RISC-V International](https://riscv.org). They give each set an acrynom, for example, "I" Extension for the integer instructions, "V" Extension for the vector calculations.

In the previous section, the `--with-arch=rv64gcv` parameter for the `configure` script specifies the toolchain to support the "G", "C", "V" Extensions. The "G" Extension is a shorthand for "I", "M", "A", "F", "D", and two more features. [The RISC-V Instruction Set Manual Volume I: Unprivileged Architecture](https://riscv.github.io/riscv-isa-manual/snapshot/unprivileged/)

| acrynom | instruction set |
| --------- | ------------------- |
|  I      | integer    |
|  M      | integer multiplication/division |
|  A      | atomic instructions |
|  F      | single-precision floating point |
|  D      | double-precision floating point |
|  C      | compressed 16-bit instructions (instead of 32-bit) |
|  V      | vector      |

For the meaning of `abi`, refer to [The -march, -mabi, and -mtune arguments to RISC-V Compilers](https://www.sifive.com/blog/all-aboard-part-1-compiler-args). The whole series is well worth a read.

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
