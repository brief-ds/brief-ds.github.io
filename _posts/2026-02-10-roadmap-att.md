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
We ask the question: Can selective sparse recurrent computation provide competitive predictive performance with significantly improved efficiency for long-context or streaming tasks, especially on low-power hardware targets?

Particularly, we will take a vanilla recurrent neural network, and impose an attention vector on each state vector.

The [Turing machine](https://en.wikipedia.org/wiki/Turing_machine) has a "head" that during its execution, at any point of time, is positioned over a cell on the memory tape. This "head" can be assimilated to attention. A recurrent neural net can [simulate any Turing machine](https://www.sciencedirect.com/science/article/pii/S0022000085710136).

As one reads on, one will see a specification of the model is a net of the [McCulloch-Pitts Neurons](https://jli05.github.io/2024/04/05/Walter-Pitts-bibliography.html).

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
Note that once the `args` are determined no matter how, usually the mathematical derivative of `args` with respect to any variable is zero. Backpropogation stops at `args`. This fact may allow us to be daring and write flexible functions to chain (relate) coordinates over steps.

PyTorch allows indexing into a vector `X[args]` and handles the autodifferentiation well, but its install size is about 1 gigabyte, on top of which it mandates installing several gigabytes of CUDA libraries today.

We will use a tensor-capable [micrograd](https://github.com/brief-ds/micrograd) for autodifferentiation, about 500 lines in Python. Its only dependency is NumPy. The `att` branch implements the proposed mechanism: to attend over `X` is `X.attend(args)`.

### batched training
If one input instance at one time is in one row, the attention on it can be different than that on a separate input instance. How to handle multiple training instances?

One way may be: note for one instance `X` and its attention indices `args`,

```python
X[args] @ M[args]
```

is either a zero-row (if the `args` is empty) or one-row matrix, so we can still go training instance by instance, compute above for the current instance, and vertically stack the results into a matrix. The new matrix will be of fewer or the same number of rows than the precedent matrix of training instances.

While a human is conscious, typically there is only one object being attended over at one time. When asleep, the human may recall and process many instances in parallel. But if something during the day left some strong impression, it is possible after one step of attention, all the other unimportant instances received nil attention, and the matrix of training instances becomes a single row.

### how is the attention determined?
If it is simply the indices of the top k values, no model is needed, otherwise a model has to be specified. For example, we extend the model by allowing a vector for inhibition levels `B`, one for stimulus levels `X`,

```python
args = (X - B).topk(k)
B = f(X[args], B[args])
X = g(X[args], B[args])
```

The functions `f` and `g` will determine the `B` and `X` at the next step, and need be trained. This completes the speficication for the evolution of the state.

The model can make an explicit output at the current step in relation to both `X[args]` and `B[args]`. The compute of each operation here is in the order of the state size, if the size of `args` is capped.

### information in the stimulus vector `X`
It can fall into several cases,

* at some coordinates, external information from senses: vision, hearing, etc. These coordinates are fixed, as the fixed addressess for input/output ports on computer architecture
* at some coordinates, the results of internal processing, such as logical reasoning or dreaming
* the remaining can be called the long-term memory, with information infrequently updated

### a toy example
We did a [few variations](/2026/03/16/gpt2.html) on the Andrej Karpathy's GPT-2 model `microgpt.py`. The last variation was a recurrent net that sparsely fires neurons. It would run about 5 times faster than the vectorised Transformer model, yet the optimised loss was in the same ball park.

|  version  |  description  | number of lines (the fewer the simpler) | run time (the lower the better) | optimised loss (the lower the better) |
| --------- | -------------- | ----------- | ---------- | ---------- |
| scalar    |  the original version, updating model coefficient one by one   |      |  160s       |   2.66     |
| vector    |  uses vector extension on the hardware architecture, updating a whole array of model coefficients at once | 164      |   7s        |   2.63     |
| rnn       |  changes the model to vanilla recurrent net, and uses vector extension |   118    |  1.2s    |   2.35     |
| rnn_att   |  as the `rnn` version but at each step, fire neurons sparsely rather than all |  120     |  1.3s       |   2.30     |

## Expectation of outcome
We will collect enough data to see if selective sparse recurrent computation can provide competitive predictive performance with significantly improved efficiency for long-context or streaming tasks, especially on low-power hardware targets.

## Related work
State Space Models models the evoluation of the state with linear models. At each step, the scale of compute is fixed. Over all steps, the total compute is only linear in number of the tokens. Mamba and DeltaNet are examples among others, which all differ in how to scale the current state vector, and how to compute the new incremental information.

| model    | state update equation  |
| -------- | -------------- |
| Mamba    | <math><msub><mi>h</mi><mi>t</mi></msub><mo>=</mo><msub><mi>a</mi><mi>t</mi></msub><mo>⊙</mo><msub><mi>h</mi><mrow><mi>t</mi><mo>-</mo><mn>1</mn></mrow></msub><mo>+</mo><msub><mi>B</mi><mi>t</mi></msub><msub><mi>x</mi><mi>t</mi></msub></math>  |
| DeltaNet | <math><msub><mi>h</mi><mi>t</mi></msub><mo>=</mo><msub><mi>h</mi><mrow><mi>t</mi><mo>-</mo><mn>1</mn></mrow></msub><mo>+</mo><msub><mi>g</mi><mi>t</mi></msub><mo>⊙</mo><mo>(</mo><mrow><msub><mi>v</mi><mi>t</mi></msub><mo>-</mo><msub><mi>h</mi><mrow><mi>t</mi><mo>-</mo><mn>1</mn></mrow></msub></mrow><mo>)</mo></math>  |

where <math><mo>⊙</mo></math> is element-wise multiplication, and <math><msub><mi>h</mi><mi>t</mi></msub></math> is the state at step <math><mi>t</mi></math>.

## References
* Turing, A. M. (1936). "On Computable Numbers, with an Application to the Entscheidungsproblem." Proceedings of the London Mathematical Society. 2. 42 (published 1937): 230–265. doi:10.1112/plms/s2-42.1.230.
* Siegelmann H.T. Sontag E.D. (1995). "On the Computational Power of Neural Nets". Journal of Computer and System Sciences. 50 (1): 132-150. https://doi.org/10.1006/jcss.1995.1013
* McCulloch, W S.; Pitts, W (1943-12-01). "A logical calculus of the ideas immanent in nervous activity". The Bulletin of Mathematical Biophysics. 5 (4): 115–133. doi:10.1007/BF02478259. ISSN 1522-9602.
* Walter Pitts' bibliography. https://home.csulb.edu/~cwallis/artificialn/walter_pitts.html https://jli05.github.io/2024/04/05/Walter-Pitts-bibliography.html
* Brief Solutions Ltd (2026). micrograd, `att` branch. A tiny autograd engine. https://github.com/brief-ds/micrograd/tree/att
* Lee, J (2026). Four experiments on GPT-2. https://www.brief-ds.com/2026/03/16/gpt2.html
* Gu, A, Dao, T (2024). Mamba: Linear-Time Sequence Modeling with Selective State Spaces. https://arxiv.org/abs/2312.00752
* Yang, S, et al (2025). Parallelizing Linear Transformers with the Delta Rule over Sequence Length. https://arxiv.org/abs/2406.06484
