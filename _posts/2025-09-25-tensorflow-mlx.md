---
title: "TensorFlow, Apple's MLX and our micrograd"
layout: post
---

## Automatic differentiation (autodiff)
An artificial neural network (ANN) is usually a function of input <math><mi>X</mi></math> and some parameters <math><mi>b</mi></math>,

<math display="block">
<mi>f</mi><mo>(</mo><mi>X</mi><mo>,</mo><mi>b</mi><mo>)</mo><mtext>.</mtext>
</math>

Given <math><mi>X</mi></math> we also observe <math><mi>Y</mi></math> as output of the function or mechanism <math><mi>f</mi></math>. The training of the ANN would involve adjusting <math><mi>b</mi></math> such that

<math display="block">
<mi>f</mi><mo>(</mo><mi>X</mi><mo>,</mo><mi>b</mi><mo>)</mo><mo>=</mo><mi>Y</mi>
</math>

for any <math><mi>X</mi></math>, <math><mi>Y</mi></math> pair, or as close as possible by some measure.

We would compute the mathematical derivates

<math display="block">
<mfrac>
<mrow><mo>&part;</mo><mi>f</mi></mrow>
<mrow><mo>&part;</mo><mi>b</mi></mrow>
</mfrac>
</math>

and move <math><mi>b</mi></math> against the direction of <math><mfrac><mrow><mo>&part;</mo><mi>f</mi></mrow><mrow><mo>&part;</mo><mi>b</mi></mrow></mfrac></math>.

The capability to automatically perform mathematical differentiation (autodiff) of a complex function with respect to its parameters is essential to machine learning libraries: for example Google's TensorFlow, Meta's PyTorch, [JAX](https://jax.dev), the emergent Apple's [MLX](https://mlx-framework.org), and [micrograd](https://github.com/brief-ds/micrograd) developed by us Brief Solutions Ltd.

## micrograd autodiff library
micrograd was started by Andrej Karpathy. Initially [it](https://github.com/karpathy/micrograd) worked only on scalar values (any parameter or the function value is a single number). We extended it to work with vectors, including matrices (2-dimensional) and arbitrary-order tensors. The repository is at

[https://github.com/brief-ds/micrograd](https://github.com/brief-ds/micrograd)

### Philosophy
The **philosophy** of micrograd is:
1. micrograd does the structural manipulation (mathematical differentiation);
2. actual numerical calculation is delegated to a numerical library as NumPy.

### micrograd can be taught to or maintained by high schoolers
The core file [micrograd/engine.py](https://github.com/brief-ds/micrograd/blob/master/micrograd/engine.py) is less than 500 lines, 10,000+ times smaller than full-featured libraries.

Each mathematical operation is defined in 10-20 lines, for example the sum operation in [micrograd/engine.py](https://github.com/brief-ds/micrograd/blob/master/micrograd/engine.py):

```python

    def sum(self, axis=None):
        # map any negative dimension index to non-negative one
        de_neg = lambda x: self.ndim + x if x < 0 else x

        if axis is None:
            _axis = tuple(range(self.ndim))
        elif isinstance(axis, int):
            _axis = de_neg(axis)
        else:
            _axis = tuple(map(de_neg, axis))

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

### Each mathematical operation can be timed with Python's native profiler
To time any code is called "profiling". The other libraries generally would require additional profiling code written. Because micrograd is pure Python, one may time the code with the cProfile module out of box.

```sh
python3 -m cProfile -s tottime <program_using_micrograd> <param> ...
```

We rewrote the model behind [https://tsterm.com](https://tsterm.com) using micrograd. From the cProfile's output, one could readily see what costs most time was the tensordot operation (tensor multiplication), followed by differentiation of the element-wise multiplication operation.

```
   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
     3258    2.749    0.001    2.814    0.001 numeric.py:1002(tensordot)
     ...
     1440    1.009    0.001    1.266    0.001 engine.py:91(_backward)
```
