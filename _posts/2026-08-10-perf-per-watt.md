---
title: Performance per watt benchmarks
layout: post
---

## Performance per watt
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

```
Dense FP16/BF16 Tensor Core GFLOPS/W

A100     █                                      780
H100     ██                                   1,413
B200     ███████████████████████████████     18,000
```

| GPU |  Architecture |   FP32 |   TDP |  GFLOPS/W  |
| --- | ------------- | ------ | ----- | ---------- |
|A100 SXM  |   Ampere |  19.5 TFLOPS |  400 W |  48.8 |
|H100 SXM  |  Hopper | 60 TFLOPS |  700 W |  85.7  |
|B200 SXM  |  Blackwell |  ~90 TFLOPS | 1,000 W | 90 |


## References
TOP500 supercomputer list, [https://www.top500.org/](https://www.top500.org/)

Frequently asked questions on the Linpack benchmark and TOP500, [https://www.netlib.org/utk/people/JackDongarra/faq-linpack.html](https://www.netlib.org/utk/people/JackDongarra/faq-linpack.html)

Linpack source code, [https://github.com/icl-utk-edu/hpl/](https://github.com/icl-utk-edu/hpl/)

HPCG (High Performance Conjugate Gradient) benchmark, [https://www.hpcg-benchmark.org/](https://www.hpcg-benchmark.org/)

HPCG benchmark source code, [https://github.com/hpcg-benchmark/hpcg](https://github.com/hpcg-benchmark/hpcg)

Nvidia Tensor Cores explained in simple terms, [https://www.digitalocean.com/community/tutorials/understanding-tensor-cores](https://www.digitalocean.com/community/tutorials/understanding-tensor-cores)

Nvidia A100 Tensor Core GPU, [https://www.nvidia.com/en-eu/data-center/a100/](https://www.nvidia.com/en-eu/data-center/a100/)

Nvidia H100 Tensor Core Datasheet, [https://resources.nvidia.com/en-us-hopper-architecture/nvidia-tensor-core-gpu-datasheet](https://resources.nvidia.com/en-us-hopper-architecture/nvidia-tensor-core-gpu-datasheet)

Nvidia Blackwell Ultra Datasheet, [https://resources.nvidia.com/en-us-blackwell-architecture/blackwell-ultra-datasheet](https://resources.nvidia.com/en-us-blackwell-architecture/blackwell-ultra-datasheet)

Nvidia Grace CPU Superchip, [https://www.nvidia.com/en-us/data-center/grace-cpu-superchip/](https://www.nvidia.com/en-us/data-center/grace-cpu-superchip/)

Nvidia Grace performance tuning guide, [https://docs.nvidia.com/dccpu/grace-perf-tuning-guide/index.html](https://docs.nvidia.com/dccpu/grace-perf-tuning-guide/index.html)

NVIDIA Grace CPU Superchip Architecture In Depth, [https://developer.nvidia.com/blog/nvidia-grace-cpu-superchip-architecture-in-depth/](https://developer.nvidia.com/blog/nvidia-grace-cpu-superchip-architecture-in-depth/)

NVIDIA Grace Hopper Superchip Architecture In-Depth, [https://developer.nvidia.com/blog/nvidia-grace-hopper-superchip-architecture-in-depth/](https://developer.nvidia.com/blog/nvidia-grace-hopper-superchip-architecture-in-depth/)

AMD ROCm Core SDK, [https://rocm.docs.amd.com/en/latest/index.html](https://rocm.docs.amd.com/en/latest/index.html)

RiVEC benchmark suite on vector microarchitecture, [https://github.com/RALC88/riscv-vectorized-benchmark-suite](https://github.com/RALC88/riscv-vectorized-benchmark-suite)

CS107 guide to x86-64, [https://web.stanford.edu/class/archive/cs/cs107/cs107.1202/guide/x86-64.html](https://web.stanford.edu/class/archive/cs/cs107/cs107.1202/guide/x86-64.html)
