---
title: "TensorFlow, Apple's MLX and our micrograd"
layout: post
---

An artificial neural network (ANN) is usually a function of input <math><mi>X</mi></math> and some parameters <math><mi>b</mi></math>,

<math display="block">
<mi>f</mi><mo>(</mo><mi>X</mi><mo>,</mo><mi>b</mi><mo>)</mo><mtext>.</mtext>
</math>

Given <math><mi>X</mi></math> we also observe <math><mi>Y</mi></math> as output of the function or mechanism <math><mi>f</mi></math>. The training of the ANN would involve adjusting <math><mi>b</mi></math> such that

<math display="block">
<mi>f</mi><mo>(</mo><mi>X</mi><mo>,</mo><mi>b</mi><mo>)</mo><mo>=</mo><mi>Y</mi>
</math>

for any <math><mi>X</mi></math>, <math><mi>Y</mi></math> pair, or as close as possible by some measure.

We would compute the mathematical derivates (gradients)

<math display="block">
<mfrac>
<mrow><mo>&part;</mo><mi>f</mi></mrow>
<mrow><mo>&part;</mo><mi>b</mi></mrow>
</mfrac>
</math>

and modify <math><mi>b</mi></math> against the direction of
<math>
<mfrac>
<mrow><mo>&part;</mo><mi>f</mi></mrow>
<mrow><mo>&part;</mo><mi>b</mi></mrow>
</mfrac>
</math>.


