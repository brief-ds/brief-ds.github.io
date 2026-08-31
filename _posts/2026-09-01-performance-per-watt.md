---
title: Performance per watt
layout: post
---

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

```
| GPU |  Architecture |   FP32 |   TDP |  GFLOPS/W  |
| --- | ------------- | ------ | ----- | ---------- |
|A100 SXM  |   Ampere |  19.5 TFLOPS |  400 W |  48.8 |
|H100 SXM  |  Hopper | 67 TFLOPS |  700 W |  95.7  |
|B200 SXM  |  Blackwell |  ~90 TFLOPS | 1,000 W | 90 |
```
