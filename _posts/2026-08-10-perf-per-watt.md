---
title: Performance per watt
layout: post
---

## Introduction
Today's laptop may have 6 CPU cores and consume 20 watts, while a supercomputer may have tens of thousands of cores and consume many kilo watts. Across the spectrum of computing systems, we will see that the performance per watt is a meaningful benchmark.

It is usually quoted as GFLOP/s/watt, for giga (<math><msup><mn>10</mn><mn>9</mn></msup></math>) floating-point operations per second per watt. In this article it is **64-bit** floating-point operation when we quote GFLOP/s/watt.

As

<math display="block">
<mn>1</mn><mtext> second</mtext><mo>×</mo><mn>1</mn><mtext> watt</mtext><mo>=</mo><mn>1</mn><mtext> Joule,</mtext>
</math>

a unit of energy,

<math display="block">
<mn>1</mn><mtext> GFLOP/s/watt</mtext><mo>=</mo><mn>1</mn><mtext> GFLOP/Joule,</mtext>
</math>

which measures the amount of work per unit of energy.

The performance per watt should be measured over the _entire_ computing system, including CPUs, memory, networking, maybe even including the cooling system when it was involved.

## CPU with vector extension and GPUs
Traditionally-designed CPU carries out one arithmetic or logical operation per clock cycle. Techniques were invented to accelerate the execution marginally such as speculative [branch prediction](https://en.wikipedia.org/wiki/Branch_predictor), while introducing security risk such as [Spectre Attack](https://en.wikipedia.org/wiki/Spectre_(security_vulnerability)).

In contrast, CPUs with vector textension or GPUs can carry out multiple arithmetic operations per clock cycle, increasing the compute throughput by orders. The CPU state would be more deterministic, increasing introspectability and security.

A GPU is traditionally designed with many cores, which can each be programmed separately. The recent Nvidia [Tensor Cores](https://www.nvidia.com/en-us/data-center/tensor-cores/) are specialised to accelerate [matrix-matrix multiplication](https://developer.nvidia.com/blog/programming-tensor-cores-cuda-9/). For example, for 64-bit multiplication, the Tensor Core can first cast each 64-bit operand to 32-bit, do 32x32-bit multiplication to still obtain a 64-bit result. The throughput would double compared to 64x64-bit multiplication.

## Compute-bound vs Memory-bound tasks
If after loading <math><mi>n</mi></math> elements of a vector, the number of arithmetic operations will be the same, the program is [memory-bound](https://en.wikipedia.org/wiki/Memory-bound_function). But if the number of arithmetic operations will be higher by some orders, the program is [compute-bound](https://en.wikipedia.org/wiki/CPU-bound).

Today, _the_ most well-known compute-bound task is matrix-matrix multiplication, as to multiply two squares of matrix of size <math><msup><mi>N</mi><mn>2</mn></msup></math> takes <math><msup><mi>N</mi><mn>3</mn></msup></math> order of compute.

A wide range of algorithms are memory-bound, for example, partial derivative equation solution, sequence alignment, etc.

## Compute-bound performance per watt
As data is loaded once yet used many times for calculation, the energy cost of memory operation is ignorable in the limit, the whole-system performance per watt is bounded by that of the CPU or GPU alone.

The [TOP500 supercomputer list](https://www.top500.org) publishes a Green benchmark for the (compute-bound) performance per watt. The official source code does compute in 64-bit. However Nvidia systems would run Nvidia's closed source code for the benchmark, which may or may not have reduced the precision of operands in flight. For Nvidia products, we deduce the performance per watt from the products' datasheets instead for 64x64-bit multiplication.

|  Package or System  | Arch. |  Watt  |  GFLOP/S/Watt  |
| --------- | ------------ | ------ | -------------------- |
| LineShine | CPU |  42,200,000 |   52   |
| A100 SXM  | GPU |  400  |   24        |
| H100 SXM  | GPU |  700  |   48        |
| Vitruvius+ | CPU | 0.45 |  47 |
| AraXL 16-lane | CPU |  1.1 |  39 |

LineShine is currently the No. 1 supercomputer by total compute throughput. It is a CPU-only system using [ARM instructions](https://support.arm.com/architectures/a-profile-architecture) with no GPU, which shows the potential of CPU with vector extension. The next two are Nvidia GPUs. The last two are research products by the Barcelona Supercomputing Centre and ETH Zurich respectively, using [RISC-V instructions](https://riscv.org/specifications/ratified/).

Most above hover around 50 GFLOP/s/watt. What is the theoretical upper limit on the compute-bound performance per watt? If a 64-bit floating point unit consumes 10 pJ/FLOP, then the theoretical upper limit is

<math display="block">
<mtable>
<mtr>
<mtd></mtd>
<mtd><mfrac><mn>1</mn><mrow><mn>10</mn><mo>×</mo><msup><mn>10</mn><mn>-12</mn></msup><mtext> Joule/FLOP</mtext></mrow></mfrac></mtd>
</mtr>
<mtr>
<mtd><mo>=</mo></mtd>
<mtd><msup><mn>10</mn><mn>11</mn></msup><mtext> FLOP/Joule</mtext></mtd>
</mtr>
<mtr>
<mtd><mo>=</mo></mtd>
<mtd><mn>100</mn><mtext> GFLOP/Joule,</mtext></mtd>
</mtr>
</mtable>
</math>

supposing the remaining circuit in the system consumes minimal power.

## Memory-bound performance per watt

```
              Energy efficiency
                    ↓

HBM          ███████████████████████████  ~500 GB/s/W
AXI          ████████                    ~50–150
LPDDR        █████                       ~40–100
NVLink       ████                        ~30–60
GDDR         ███                         ~20–60
Optical      ██                          ~5–30
```

## References
TOP500 supercomputer list, [https://www.top500.org/](https://www.top500.org/)

Frequently asked questions on the Linpack benchmark and TOP500, [https://www.netlib.org/utk/people/JackDongarra/faq-linpack.html](https://www.netlib.org/utk/people/JackDongarra/faq-linpack.html)

Linpack source code, [https://github.com/icl-utk-edu/hpl/](https://github.com/icl-utk-edu/hpl/)

HPCG (High Performance Conjugate Gradient) benchmark, [https://www.hpcg-benchmark.org/](https://www.hpcg-benchmark.org/)

HPCG benchmark source code, [https://github.com/hpcg-benchmark/hpcg](https://github.com/hpcg-benchmark/hpcg)

Programming Tensor Codes in CUDA 9, [https://developer.nvidia.com/blog/programming-tensor-cores-cuda-9/](https://developer.nvidia.com/blog/programming-tensor-cores-cuda-9/)

LineShine supercomputer, [https://en.wikipedia.org/wiki/LineShine](https://en.wikipedia.org/wiki/LineShine)

LineShine system, [https://www.top500.org/system/180490/](https://www.top500.org/system/180490/)

Nvidia A100 GPU, [https://www.nvidia.com/en-eu/data-center/a100/](https://www.nvidia.com/en-eu/data-center/a100/)

Nvidia H100 GPU, [https://www.nvidia.com/en-us/data-center/h100/](https://www.nvidia.com/en-us/data-center/h100/)

Nvidia Grace CPU Superchip, [https://www.nvidia.com/en-us/data-center/grace-cpu-superchip/](https://www.nvidia.com/en-us/data-center/grace-cpu-superchip/)

NVIDIA Grace CPU Superchip Architecture In Depth, [https://developer.nvidia.com/blog/nvidia-grace-cpu-superchip-architecture-in-depth/](https://developer.nvidia.com/blog/nvidia-grace-cpu-superchip-architecture-in-depth/)

NVIDIA Grace Hopper Superchip Architecture In-Depth, [https://developer.nvidia.com/blog/nvidia-grace-hopper-superchip-architecture-in-depth/](https://developer.nvidia.com/blog/nvidia-grace-hopper-superchip-architecture-in-depth/)

Nvidia Grace performance tuning guide, [https://docs.nvidia.com/dccpu/grace-perf-tuning-guide/index.html](https://docs.nvidia.com/dccpu/grace-perf-tuning-guide/index.html)

AMD ROCm Core SDK, [https://rocm.docs.amd.com](https://rocm.docs.amd.com)

AMD ROCm blog: Understanding Typically Attained FLOPs, [https://rocm.blogs.amd.com/artificial-intelligence/taf-blog/README.html](https://rocm.blogs.amd.com/artificial-intelligence/taf-blog/README.html)

Vitruvius+: an efficient RISC-V decoupled vector coprocessor, [https://dl.acm.org/doi/full/10.1145/3575861](https://dl.acm.org/doi/full/10.1145/3575861)

AraXL: a physically scalable, ultra-wide RISC-V vector processor, [https://arxiv.org/abs/2501.10301](https://arxiv.org/abs/2501.10301)

RiVEC benchmark suite on vector microarchitecture, [https://github.com/RALC88/riscv-vectorized-benchmark-suite](https://github.com/RALC88/riscv-vectorized-benchmark-suite)

CPU vector extension and GPU benchmarking, [https://github.com/jli05/vbench](https://github.com/jli05/vbench)

CS107 guide to x86-64, [https://web.stanford.edu/class/archive/cs/cs107/cs107.1202/guide/x86-64.html](https://web.stanford.edu/class/archive/cs/cs107/cs107.1202/guide/x86-64.html)
