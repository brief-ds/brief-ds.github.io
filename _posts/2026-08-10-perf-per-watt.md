---
title: Performance per watt
layout: post
---

## Introduction
Today's laptop may have 6 CPU cores and consume 20 watts, while a supercomputer may have tens of thousands of cores and consume many kilo watts. We will see that the performance per watt is a meaningful benchmark across the spectrum of computing systems.

It is usually quoted as GFLOP/s/watt, for giga floating-point operations per second per watt, or GOP/s/watt, for giga operations per second per watt. 1 second x 1 watt = 1 Joule, a unit of energy. So GFLOP/s/watt = GFLOP/Joule, which measures the amount of compute work per unit of energy.

## CPU with vector extension and GPUs
Traditionally-designed CPU carries out arithmetic or logical compute one per clock cycle. Afterwards techniques such as speculative [branch prediction](https://en.wikipedia.org/wiki/Branch_predictor) were invented, which would accelerate marginally, whilst introducing security risk such as [Spectre Attack](https://en.wikipedia.org/wiki/Spectre_(security_vulnerability)).

The CPU vector extensions or GPUs can carry out multiple arithmetic operations per clock cycle, increasing the compute throughput by orders. The CPU state would be more deterministic, increasing introspectability and security.

A GPU is traditionally designed with many cores, which can each be programmed separately. The recent Nvidia [Tensor Cores](https://www.nvidia.com/en-us/data-center/tensor-cores/) are specialised to accelerate [matrix-matrix multiplication](https://developer.nvidia.com/blog/programming-tensor-cores-cuda-9/). For example, for 32-bit multiplication, one may first cast each 32-bit operand to 16-bit, do 16x16 multiplication to still obtain a 32-bit result. The throughput would double compared to 32x32 multiplication.

## Compute-bound vs Memory-bound
If after loading <math><mi>n</mi></math> elements of a vector, the number of arithmetic operations will be the same, the program is [memory-bound](https://en.wikipedia.org/wiki/Memory-bound_function). But if the number of arithmetic operations will be higher by some orders, the program is [compute-bound](https://en.wikipedia.org/wiki/CPU-bound).

## Compute-bound Performance per Watt

## Memory-bound Performance per Watt

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

Nvidia A100 Tensor Core GPU, [https://www.nvidia.com/en-eu/data-center/a100/](https://www.nvidia.com/en-eu/data-center/a100/)

Nvidia H100 Tensor Core Datasheet, [https://resources.nvidia.com/en-us-hopper-architecture/nvidia-tensor-core-gpu-datasheet](https://resources.nvidia.com/en-us-hopper-architecture/nvidia-tensor-core-gpu-datasheet)

Nvidia Blackwell Ultra Datasheet, [https://resources.nvidia.com/en-us-blackwell-architecture/blackwell-ultra-datasheet](https://resources.nvidia.com/en-us-blackwell-architecture/blackwell-ultra-datasheet)

Nvidia Grace CPU Superchip, [https://www.nvidia.com/en-us/data-center/grace-cpu-superchip/](https://www.nvidia.com/en-us/data-center/grace-cpu-superchip/)

Nvidia Grace performance tuning guide, [https://docs.nvidia.com/dccpu/grace-perf-tuning-guide/index.html](https://docs.nvidia.com/dccpu/grace-perf-tuning-guide/index.html)

NVIDIA Grace CPU Superchip Architecture In Depth, [https://developer.nvidia.com/blog/nvidia-grace-cpu-superchip-architecture-in-depth/](https://developer.nvidia.com/blog/nvidia-grace-cpu-superchip-architecture-in-depth/)

NVIDIA Grace Hopper Superchip Architecture In-Depth, [https://developer.nvidia.com/blog/nvidia-grace-hopper-superchip-architecture-in-depth/](https://developer.nvidia.com/blog/nvidia-grace-hopper-superchip-architecture-in-depth/)

AMD ROCm Core SDK, [https://rocm.docs.amd.com](https://rocm.docs.amd.com)

AMD ROCm blog: Understanding Typically Attained FLOPs, [https://rocm.blogs.amd.com/artificial-intelligence/taf-blog/README.html](https://rocm.blogs.amd.com/artificial-intelligence/taf-blog/README.html)

RiVEC benchmark suite on vector microarchitecture, [https://github.com/RALC88/riscv-vectorized-benchmark-suite](https://github.com/RALC88/riscv-vectorized-benchmark-suite)

CS107 guide to x86-64, [https://web.stanford.edu/class/archive/cs/cs107/cs107.1202/guide/x86-64.html](https://web.stanford.edu/class/archive/cs/cs107/cs107.1202/guide/x86-64.html)
