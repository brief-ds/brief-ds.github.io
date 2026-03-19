---
title: A roadmap to a simple attention mechanism
layout: post
---

## Background
The attention mechanism in neural network models underlies the phenomenal success of AI. The commonly implemented attention mechanism today is computationally costly. Given <math><mi>n</mi></math> tokens, and one value's size <math><mi>d</mi></math>, the total computation cost is

<math display="block">
<mi>O</mi><mo>(</mo><mi>n</mi><mi>n</mi><mi>d</mi><mo>)</mo>
</math>

to transform these tokens' values with "attention", quadratic in terms of <math><mi>n</mi></math>.

ChatGPT: "why the current transformer deep learning model is computationally costly? could you give mathematical notions and equations to illustrate?"

## Aim of this project
We ask the question whether a model with a less costly attention mechanism, can behave as intelligently. Particularly, we will take a vanilla recurrent neural network, and impose an attention vector on each state vector.

The [Turing machine](https://en.wikipedia.org/wiki/Turing_machine) has a "head" that during its execution, at any point of time, is positioned over a cell on the memory tape. This "head" can be assimilated to attention. A recurrent neural net can [simulate any Turing machine](https://www.sciencedirect.com/science/article/pii/S0022000085710136).

## Method
In a recurrent neural net, the state vector is assimilated to the memory tape of a Turing machine. The state vector undergoes change while proceeding in time. Denote the state vector at any time by `X`, a row vector of size <math><mi>m</mi></math>. Rather than multiplying the entire `X` by a matrix, we will attend over and transform only certain elements in `X`,

```
X + X[args] @ M[args]
```

before a non-linear transform on the result for the next state vector, where `@` is the multiplication in linear algebra.

If the number of `args` is capped, the above operation costs <math><mi>O</mi><mo>(</mo><mi>m</mi><mo>)</mo></math>. Over <math><mi>n</mi></math> tokens, it is

<math display="block">
<mi>O</mi><mo>(</mo><mi>n</mi><mi>m</mi><mo>)</mo><mtext>,</mtext>
</math>

linear in terms of <math><mi>n</mi></math>.

If each row of `M` is sparse, `X[args] @ M[args]` will be a sparse vector. `X` will only have to be sparsely incremented for the next state vector. The total compute will be further less. Note in [human brain](https://en.wikipedia.org/wiki/Neuron#Connectivity), averagely each neuron is connected with very few others: less than <math><msup><mn>10</mn><mn>-5</mn></msup></math> of all.

We will then do extensive training following training detail provided in

* nanochat  [https://github.com/karpathy/nanochat/](https://github.com/karpathy/nanochat/)
* xLSTM     [https://arxiv.org/abs/2510.02228](https://arxiv.org/abs/2510.02228)

for various AI tasks and report results.

### the library that does autodifferentiation
PyTorch allows indexing into a vector `X[args]` and handles the autodifferentiation well, but its install size is about 1 gigabyte today, on top of which it mandates installing several gigabytes of CUDA libraries.

We will use a tensor-capable [micrograd](https://github.com/brief-ds/micrograd) for autodifferentiation, about 500 lines in Python. Its only dependency is NumPy.

### batched training
If one input instance at one time is in one row, the attention on it, the `args` can be different than that on a separate input instance. How to handle multiple training instances?

One way may be: note for one instance `X` and its attention indices `args`,

```python
X[args] @ M[args]
```

is either a zero-row (if the `args` is empty) or one-row matrix, so we can still go training instance by instance, compute above for the current instance, and vertically stack the results into a matrix. The new matrix will be of fewer or the same number of rows than the precedent matrix of training instances.

While a human is conscious, typically there is only one object being attended over at one time. When asleep, the human may recall and process many instances in parallel. But if something during the day left some strong impression, it is possible after one step of attention, all the other unimportant instances received nil attention, and the matrix of training instances becomes a single row.

### how is the attention determined?
If it is simply the indices of the top k values, no model is needed, otherwise a model has to be specified. For example, there can be a vector for inhibition levels `B`, one for stimulus levels `X`,

```python
args = (X - B).topk(k)
B = f(X[args], B[args])
X = g(X[args], B[args])
```

The functions `f` and `g` are the model to determine the `B` and `X` at the next step, and need be trained. The output at the current time will be in relation to both `X[args]` and `B[args]`. The compute of each operation here is in the order of the state size, if the size of `args` is capped.

## Expectation of outcome
We will collect enough data to see if a simpler model can behave as intelligently as today's popular chatbots.

## References
TODO
