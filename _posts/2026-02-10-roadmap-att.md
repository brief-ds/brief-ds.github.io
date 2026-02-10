---
title: A roadmap to a simple attention mechanism
layout: post
---

## Background
The attention mechanism in neural network models underlies the phenomenal success of AI. The commonly implemented attention mechanism today is computationally costly. Given <math><mi>n</mi></math> tokens, and one value's size <math><mi>d</mi></math>, the total computation cost is

<math display="block">
<mi>O</mi><mo>(</mo><mi>n</mi><mi>n</mi><mi>d</mi><mo>)</mo>
</math>

to transform these tokens' values taking account of attention, quadratic in terms of <math><mi>n</mi></math>.

ChatGPT: "why the current transformer deep learning model is computationally costly? could you give mathematical notions and equations to illustrate?"

## Aim of this project
We ask the question whether a model with a less costly attention mechanism, can behave as intelligently. Particularly, we will take a vanilla recurrent neural network, and impose an attention vector on each state vector.

The [Turing machine](https://en.wikipedia.org/wiki/Turing_machine) has a "head" that during its execution, at any point of time, is positioned over a cell on the memory tape. This "head" can be assimilated to attention. A recurrent neural net can [simulate any Turing machine](https://www.sciencedirect.com/science/article/pii/S0022000085710136).

## Methods
In a recurrent neural net, the state vector is assimilated to the memory tape of a Turing machine. The state vector undergoes change while proceeding in time. Denote the state vector at any time by `X`, a row vector of size <math><mi>m</mi></math>. Rather than multiplying the entire `X` by a matrix, we will attend over and transform only certain elements in `X` for the next state vector:

```
X + X[args] M
```

The transformation matrix `M` will be with fewer number of rows. If the number of `args` is capped, the above operation costs <math><mi>O</mi><mo>(</mo><mi>m</mi><mi><mo>)</mo></math>. Over <math><mi>n</mi></math> tokens, it is

<math display="block">
<mi>O</mi><mo>(</mo><mi>n</mi><mi>m</mi><mo>)</mo><mtext>,</mtext>
</math>

linear in terms of <math><mi>n</mi></math>.

In most machine learning libraries in Python, `X[args]` copies into a new array the selected elements. Backpropogation (calculating of mathematical derivatives) will be with respect to this new array but not the original `X`.

We have rewritten the autodifferentiation part of any machine learning library into a 500-line Python lib [micrograd](https://www.brief-ds.com/2025/09/25/tensorflow-mlx.html) which opens up the interface of any op(erator) for the forward and backward propogation for implementation. For example, attending over selected elements is

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
We will collect enough data to see if a simpler model can behave as intelligently as today's popular chatbots.
