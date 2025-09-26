---
title: "TensorFlow, Apple's MLX and our micrograd"
layout: post
---

## Automatic differentiation (autodiff)
An artificial neural network (ANN) is usually a function of input <math><mi>X</mi></math> and some parameters <math><mi>b</mi></math>,

<math display="block">
<mi>f</mi><mo>(</mo><mi>X</mi><mo>,</mo><mi>b</mi><mo>)</mo><mtext>.</mtext>
</math>

Given <math><mi>X</mi></math> we observe <math><mi>Y</mi></math> as output of the function or mechanism <math><mi>f</mi></math>.

The training of the ANN would involve adjusting <math><mi>b</mi></math> such that <math><mi>f</mi><mo>(</mo><mi>X</mi><mo>,</mo><mi>b</mi><mo>)</mo></math> is as close to <math><mi>Y</mi></math> as possible by some measure, called "loss". For example, below is a loss,

<math display="block">
<mi>l</mi><mo>(</mo><mi>X</mi><mo>,</mo><mi>Y</mi><mo>,</mo><mi>b</mi><mo>)</mo><mo>=</mo><mrow><mo>|</mo><mi>f</mi><mo>(</mo><mi>X</mi><mo>,</mo><mi>b</mi><mo>)</mo><mo>-</mo><mi>Y</mi><mo>|</mo></mrow><mtext>,</mtext>
</math>

where <math><mi>X</mi></math>, <math><mi>Y</mi></math> are given. <math><mi>b</mi></math> can be adjusted to make <math><mi>l</mi></math> smaller.

We would compute the [mathematical derivatives](https://www.mathsisfun.com/calculus/derivatives-introduction.html)

<math display="block">
<mfrac>
<mrow><mo>&part;</mo><mi>l</mi></mrow>
<mrow><mo>&part;</mo><mi>b</mi></mrow>
</mfrac>
</math>

and move <math><mi>b</mi></math> against the direction of <math><mfrac><mrow><mo>&part;</mo><mi>l</mi></mrow><mrow><mo>&part;</mo><mi>b</mi></mrow></mfrac></math> to make <math><mi>l</mi></math> smaller.

The capability to automatically perform mathematical differentiation (autodiff) of a complex function with respect to its parameters is essential to machine learning libraries: for example Google's TensorFlow, Meta's PyTorch, [JAX](https://jax.dev), the emergent Apple's [MLX](https://mlx-framework.org), and [micrograd](https://github.com/brief-ds/micrograd) developed by us Brief Solutions Ltd.

## micrograd autodiff library
Our repository is at [https://github.com/brief-ds/micrograd](https://github.com/brief-ds/micrograd). The README serves as the self-sufficient documentation.

micrograd was started by Andrej Karpathy. [The initial version](https://github.com/brief-ds/micrograd/tree/scalar), now the code under tag `scalar`, works only on scalar values. We extended it to work with vectors, including matrices (2-dimensional) and arbitrary-dimensional tensors. 

micrograd's core is just one 500-line Python file [micrograd/engine.py](https://github.com/brief-ds/micrograd/blob/master/micrograd/engine.py). The project does not need particular compilation, making, set up. Many code bases are open, but would take a special team of programmers to maintain. micrograd can be easily maintained by one amateur.

|  Library     |  Install size  |
| ------------ |  ------------- |
| micrograd*   |   20 kilobytes |
| MLX          |   23 megabytes |
| PyTorch      |  700 megabytes |
| TensorFlow   | 1,700 megabytes |

NOTE: micrograd solely depends on a numerical library for linear algebra calculation. Today it is NumPy, taking 38 megabytes.

### The philosophy of micrograd
micrograd separates the symbolic differentiation and numerical calculation:
1. micrograd does the differentiation, a manipulation of symbols;
2. actual numerical calculation is delegated to a numerical library as NumPy.

When a machine learning library implements the mathematical functions again, it is possible the result is different across libraries, for example the `arctanh(x)` in TensorFlow vs NumPy, when `x` is close to 1 or -1.

If in the future there is a numerical library more compact and performant, we will switch to that one. The clean division of job in the design enables that.

### micrograd can be taught to high schoolers
The core file [micrograd/engine.py](https://github.com/brief-ds/micrograd/blob/master/micrograd/engine.py) is less than 500 lines, 10,000+ times smaller than full-featured libraries.

Each mathematical operation is defined in 10-20 lines, for example the sum operation in [micrograd/engine.py](https://github.com/brief-ds/micrograd/blob/master/micrograd/engine.py):

```python

    def sum(self, axis=None):
        ...         # 8 lines of pre-processing

        out = Value(_sum(self.data, axis=axis), (self,), 'sum')

        def _forward(**kwds):
            out.data = _sum(self.data, axis=axis)
        out._forward = _forward

        def _backward():
            # expand out.grad to same number of dimensions
            # as self.data, self.grad
            _out_grad = expand_dims(out.grad, _axis)

            # ... expand further to same shape as self.data
            self.grad += broadcast_to(_out_grad, self.shape)
        out._backward = _backward

        return out

```

where
* the `_forward()` function evaluates the sum, and
* the `_backward()` function differentiates the sum with respect to the elements, over which the sum was calculated.

### micrograd can be uniquely inspected with Python's built-in profiler
To time any code is called "profiling". Complex machine learning libraries would require additionally written code to inspect itself. Because micrograd is pure Python, one may time it with the cProfile module built in Python.

```sh
python3 -m cProfile -s tottime <program_using_micrograd>
```

We rewrote the model behind [https://tsterm.com](https://tsterm.com) using micrograd and profiled it.

```
   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
     3258    2.749    0.001    2.814    0.001 numeric.py:1002(tensordot)
     ...
     1440    1.009    0.001    1.266    0.001 engine.py:91(_backward)
     ...
```

cProfile's output clearly ranks each forward or backward function of the mathematical operators by the total time, under the `tottime` column. On one run, the most costly was the tensordot operation (tensor multiplication), followed by the differentiation of the element-wise multiplication.

### micrograd is comparable in performance
micrograd turns out not to lose out in performance. We benchmarked the model behind [https://tsterm.com](https://tsterm.com) written with different libraries. The shorter the run time is the better.

|  Hardware | Operating System |   TensorFlow  | MLX |  micrograd  |
| --------- | ----------- | ------------- | ------ | ----- |
|  x86_64 (AMD EPYC) | Amazon Linux 2 | 10s |   | 12s  |
|  AArch64 (Graviton3) | Ubuntu 24.04 LTS | 13s | 10s | 12s |
|  AArch64 (Graviton4) | Ubuntu 24.04 LTS | 11s | 9s  | 11s |

The model performs quantile regression on 600 megabytes of data in memory. The data type was float32.

MLX is only for [AArch64](https://en.wikipedia.org/wiki/AArch64), ARM's 64-bit architecture, unable to run on other hardware.

We can see on x86, TensorFlow wins; on AArch64, MLX leads, followed by micrograd.

### micrograd is most widely deployable
MLX is only available on AArch64, not available on x86, RISC-V. TensorFlow is available on x86 as well, but only selectively made for a few mainstream Linux distributions.

The deployability for complex machine learning library is usually restricted. micrograd, as it only depends on Python and NumPy, is as portable as both, therefore available on x86, AArch64, RISC-V, etc architectures as well as a great number of operating systems.

### micrograd is easiest to extend
As we saw above, tensordot was the most costly operation. If you have an idea to accelerate a particular kind of tensordot, go into `micrograd/engine.py`, and add a few lines:

```python

    def my_tensordot(self):

        out = ...

        def _forward():
            pass
        out._forward = _forward

        def _backward():
            pass
        out._backward = _backward

        return out

```

likewise to define any new operator. That's a snap!

## Next up
In the profiler's output, we have seen the tensordot (tensor multiplication) is most costly. We will see in the next post if we can win back some runtime!

## References
Introduction to Derivatives, Math is Fun, [https://www.mathsisfun.com/calculus/derivatives-introduction.html](https://www.mathsisfun.com/calculus/derivatives-introduction.html)

Differentiation, BBC Bitsize, [https://www.bbc.co.uk/bitesize/guides/zyj77ty/](https://www.bbc.co.uk/bitesize/guides/zyj77ty/)

Install Apple's MLX machine learning library, [/2025/09/26/install-mlx.html](/2025/09/26/install-mlx.html)

Dive into MLX, Pranay Saha, [https://medium.com/@pranaysaha/dive-into-mlx-performance-flexibility-for-apple-silicon-651d79080c4c](https://medium.com/@pranaysaha/dive-into-mlx-performance-flexibility-for-apple-silicon-651d79080c4c)

How Fast is MLX?, Tristan Bilot, [https://towardsdatascience.com/how-fast-is-mlx-a-comprehensive-benchmark-on-8-apple-silicon-chips-and-4-cuda-gpus-378a0ae356a0/](https://towardsdatascience.com/how-fast-is-mlx-a-comprehensive-benchmark-on-8-apple-silicon-chips-and-4-cuda-gpus-378a0ae356a0/)
