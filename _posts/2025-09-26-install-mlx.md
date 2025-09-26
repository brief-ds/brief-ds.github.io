---
title: "Install Apple's MLX machine learning library"
layout: post
---

[MLX](https://mlx-framework.org) is an Apple's project to build a machine learning library, for Apple Silicon and ARM. [AArch64](https://en.wikipedia.org/wiki/AArch64) architecture is necessary. To install on Ubuntu 24.04 LTS,

Install openblas,

```sh
sudo apt install libopenblas-dev
```

Install Lapack. Not sure which one exactly is needed, I installed them all,

```sh
sudo apt install liblapack-dev liblapack64-dev liblapacke-dev
```

Install nanobind,

```sh
sudo apt install nanobind-dev
```

Then clone MLX and build it,

```sh
git clone https://github.com/ml-explore/mlx.git
cd mlx
python3 -m venv venv
. venv/bin/activate
pip3 install .
```

The install size is 23 megabytes,

```sh
$ du -h -d 1 /home/ubuntu/mlx/venv/lib/python3.12/site-packages
..
23M	/home/ubuntu/mlx/venv/lib/python3.12/site-packages/mlx
..
```

## References
Dive into MLX, Pranay Saha, [https://medium.com/@pranaysaha/dive-into-mlx-performance-flexibility-for-apple-silicon-651d79080c4c](https://medium.com/@pranaysaha/dive-into-mlx-performance-flexibility-for-apple-silicon-651d79080c4c)

How Fast is MLX?, Tristan Bilot, [https://towardsdatascience.com/how-fast-is-mlx-a-comprehensive-benchmark-on-8-apple-silicon-chips-and-4-cuda-gpus-378a0ae356a0/](https://towardsdatascience.com/how-fast-is-mlx-a-comprehensive-benchmark-on-8-apple-silicon-chips-and-4-cuda-gpus-378a0ae356a0/)
