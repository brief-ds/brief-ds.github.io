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
|H100 SXM  |  Hopper | 67 TFLOPS |  700 W |  95.7  |
|B200 SXM  |  Blackwell |  ~90 TFLOPS | 1,000 W | 90 |


## References
Frequently asked questions on the Linpack benchmark and TOP500, [https://www.netlib.org/utk/people/JackDongarra/faq-linpack.html](https://www.netlib.org/utk/people/JackDongarra/faq-linpack.html)

Nvidia Grace performance tuning guide, [https://docs.nvidia.com/dccpu/grace-perf-tuning-guide/index.html](https://docs.nvidia.com/dccpu/grace-perf-tuning-guide/index.html)

RiVEC benchmark suite on vector microarchitecture, [https://github.com/RALC88/riscv-vectorized-benchmark-suite](https://github.com/RALC88/riscv-vectorized-benchmark-suite)

CS107 guide to x86-64, [https://web.stanford.edu/class/archive/cs/cs107/cs107.1202/guide/x86-64.html](https://web.stanford.edu/class/archive/cs/cs107/cs107.1202/guide/x86-64.html)
