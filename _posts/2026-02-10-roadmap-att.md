---
title: Roadmap to a simple attention mechanism
layout: post
---

## Background
The attention mechanism in neural network models underlies the phenomenal success of AI. The common attention mechanism today is computationally costly. Given <math><mn>n</mn></math> tokens,

## Aim of this project
We ask the question whether a model with a less costly attention mechanism, can behave as intelligently. For example, we will take a vanilla recurrent neural network, and impose an attention vector on each state vector.

The [Turing machine](https://en.wikipedia.org/wiki/Turing_machine) has a "head" that, at any point of time, is positioned over a cell on the memory tape. This "head" is assimilated to attention. A recurrent neural net can [simulate any Turing machine](https://www.sciencedirect.com/science/article/pii/S0022000085710136).

## Methods
As to a recurrent neural net, denote the state vector at a certain step by `X`, a row vector. Rather than multiplying the entire `X` by a matrix for the next state vector, we attend over and transform only certain elements in `X` for the next state vector:

```
X + X[args] M
```

The transformation matrix `M` will be with fewer number of rows, to reduce the compute.

In most machine learning libraries in Python, `X[args]` assembles into a new array a copy of relevant elements. Backpropogation (calculating of mathematical derivatives) will be with respect to this new array but not the original `X`.

We have rewritten the backpropogation into a 500-line Python lib [micrograd](https://www.brief-ds.com/2025/09/25/tensorflow-mlx.html) which opens up the forward, backward backpropogation implementations for any op(erator). For example, attending is

[https://github.com/brief-ds/micrograd/commit/61db262bbb2409974dd2615113dc443ec072e1f4](https://github.com/brief-ds/micrograd/commit/61db262bbb2409974dd2615113dc443ec072e1f4):

```python
    def attend(self, args):
        out = Value(self.data[args], (self,), 'attend')

        def _forward(**kwds):
            # make a copy of attended data
            out.data = self.data[args]
        out._forward = _forward

        def _backward():
            # mathematical derivatives got propogated into
            # selected coordinates of the original vector
            self.grad[args] += out.grad
        out._backward = _backward

        return out

```

We will then do extensive training following training detail provided in

* nanochat  [https://github.com/karpathy/nanochat/](https://github.com/karpathy/nanochat/)
* xLSTM     [https://arxiv.org/abs/2510.02228](https://arxiv.org/abs/2510.02228)

for various AI tasks and report results.

## Expectation of outcome
