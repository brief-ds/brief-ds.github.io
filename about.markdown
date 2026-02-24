---
layout: page
title: Our interests
permalink: /about/
---

## lean hardware
RISC-V is a family of [open instruction sets](/2025/08/11/first-riscv64-program.html) for microprocessors. The [V extension](/2025/10/31/rv64-v.html) is for vector operations, underpinning signal processing, machine learning and AI. There are [open designs of implementation](https://github.com/pulp-platform) for the V extension that run at about one watt.

## lean software
[micrograd](https://github.com/brief-ds/micrograd) is a 500-line machine learning library. It is [easy](/2025/09/25/tensorflow-mlx.html) to learn and innovate with, yet competitive in performance. It paves the way for [simplifying attention-based models](/2026/02/10/roadmap-att.html).

## generic causal prediction
We developed a [generic approach](https://patents.google.com/patent/US11893069B2/en) to identify causal predictors first for a prediction target, then make verifiable probabilistic prediction. One embodiment is Time Series Terminal [https://tsterm.com](https://tsterm.com). The engine behind can be configured with data from any domain: agriculture, climate, etc.

## references
Introduction to RISC-V, [/2025/08/11/first-riscv64-program.html](/2025/08/11/first-riscv64-program.html)

The V extension for vector operations, [/2025/10/31/rv64-v.html](/2025/10/31/rv64-v.html)

TensorFlow, Apple's MLX and our micrograd, [/2025/09/25/tensorflow-mlx.html](/2025/09/25/tensorflow-mlx.html)

A roadmap to a simple attention mechanism, [/2026/02/10/roadmap-att.html](/2026/02/10/roadmap-att.html)

Hannart, A., J. Pearl, F. E. L. Otto, P. Naveau, and M. Ghil, Causal counterfactual theory for the attribution of weather and climate-related events, Bulletin of the American Meteorological Society, 97(1):99-110, 2016.

Koenker, R. (2005). Quantile Regression (Econometric Society Monographs). Cambridge: Cambridge University Press. [doi:10.1017/CBO9780511754098](https://www.cambridge.org/core/books/quantile-regression/C18AE7BCF3EC43C16937390D44A328B1)

Komunjer, Ivana, Quasi-maximum likelihood estimation for conditional quantiles, Journal of Econometrics, Volume 128, Issue 1, September 2005, Pages 137-164.

Diebold, F.X., Gunther, T. and Tay, A., Evaluating Density Forecasts, 1998.

Platform, method, and system for a search engine of time series data, [United States patent US11893069B2](https://patents.google.com/patent/US11893069B2/en).

Time Series Terminal, [https://tsterm.com](https://tsterm.com)
