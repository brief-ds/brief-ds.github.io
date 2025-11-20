---
layout: page
title: About
permalink: /about/
---

We develop lean software and hardware to enable intelligent computing at one watt.

## Our interests
### generic causal prediction
We developed a [generic approach](https://patents.google.com/patent/US11893069B2/en) to identify causal predictors first for a prediction target, then make probabilistic prediction based on the extracted set of them. One embodiment is Time Series Terminal [https://tsterm.com](https://tsterm.com). The engine behind can be configured with data from any domain: agriculture, climate, etc.

### lean software
[micrograd](https://github.com/brief-ds/micrograd) is a 20 kilobytes automatic differentiation library, found at the core of any machine learning framework. It is [transparent and easy](/2025/09/25/tensorflow-mlx.html) for kids to learn, for researchers to experiment with, and yet competitive in performance. It paves the way for simplifying attention-based models.

### lean hardware
RISC-V is a family of [open instruction sets](/2025/08/11/first-riscv64-program.html) for microprocessors. The [V extension](/2025/10/31/rv64-v.html) groups instructions of vector operations, backbone of AI calculations. There are [open designs of hardware](https://github.com/pulp-platform) for RISC-V that run at about one watt.

## References
Hannart, A., J. Pearl, F. E. L. Otto, P. Naveau, and M. Ghil, Causal counterfactual theory for the attribution of weather and climate-related events, Bulletin of the American Meteorological Society, 97(1):99-110, 2016.

Koenker, R. (2005). Quantile Regression (Econometric Society Monographs). Cambridge: Cambridge University Press. [doi:10.1017/CBO9780511754098](https://www.cambridge.org/core/books/quantile-regression/C18AE7BCF3EC43C16937390D44A328B1)

Komunjer, Ivana, Quasi-maximum likelihood estimation for conditional quantiles, Journal of Econometrics, Volume 128, Issue 1, September 2005, Pages 137-164.

Platform, method, and system for a search engine of time series data, [United States patent US11893069B2](https://patents.google.com/patent/US11893069B2/en).

Time Series Terminal, [https://tsterm.com](https://tsterm.com)

TensorFlow, Apple's MLX and our micrograd, [/2025/09/25/tensorflow-mlx.html](/2025/09/25/tensorflow-mlx.html)

Introduction to RISC-V, [/2025/08/11/first-riscv64-program.html](/2025/08/11/first-riscv64-program.html)

The V extension for vector operations, [/2025/10/31/rv64-v.html](/2025/10/31/rv64-v.html)
