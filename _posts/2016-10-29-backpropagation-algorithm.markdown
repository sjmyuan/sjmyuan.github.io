---
title: "Backpropagation Algorithm"
excerpt: ""
mathjax: true
about: "Neural Network"
date: "2016-10-29 20:56:49 +0800"
---
Let's say:

+ $$e$$ denote the loss between network output and real value
+ $$U_i^k$$ denote the $$i$$-th node total input value in level $$k$$
+ $$X_i^k$$ denote the $$i$$-th node output value in level $$k$$
+ $$W_{i,j}^k$$ denote the weight between $$i$$-th node in level $$k-1$$ and $$j$$-th node in level $$k$$
+ $$\Delta W_{i,j}^k$$ denote the increment of $$W_{i,j}^k$$ in each iteration
+ $$f_i^k$$ denote the $$i$$-th node activation function in level $$k$$
+ $$\partial{f_i^k}$$ denote the derivation of  $$i$$-th node activation function in level $$k$$
+ $$\alpha$$ denote the learning rate.

The value flow:

$$
X^{k-1} \xrightarrow{W^k} U^k \xrightarrow{f^k} X^{k}
$$

The error flow:

$$
e \longrightarrow X^k \xrightarrow{f^k} U^k \xrightarrow{X^{k-1}} W^k
$$

Derivation process:

$$
\begin{align*}
\frac{\partial{e}}{\partial{X_i^k}} &=\sideset{}{_l}\sum{ \frac{\partial{e}}{\partial{U_l^{k+1}}} \times \frac{\partial{U_l^{k+1}}}{\partial{X_i^k}}} \\
&=\sideset{}{_l}\sum{ \frac{\partial{e}}{\partial{U_l^{k+1}}} \times \frac{\partial{(\sideset{}{_m} \sum{W_{m,l}^{k+1} \times X_m^k}})}{\partial{X_i^k}}} \\
&=\sideset{}{_l}\sum{ \frac{\partial{e}}{\partial{U_l^{k+1}}} \times W_{i,l}^{k+1}} \\
\frac{\partial{e}}{\partial{U_i^k}} &= \frac{\partial{e}}{\partial{X_i^k}} \times \frac{\partial{X_i^k}}{\partial{U_i^k}}\\
&= \frac{\partial{e}}{\partial{X_i^k}} \times \partial{f_i^k}\\
&= （\sideset{}{_l}\sum{ \frac{\partial{e}}{\partial{U_l^{k+1}}} \times W_{i,l}^{k+1}} ） \times \partial{f_i^k}\\
\frac{\partial{e}}{\partial{W_{i,j}^k}} &= \frac{\partial{e}}{\partial{U_j^{k}}} \times \frac{\partial{U_j^k}}{\partial{W_{i,j}^k}} \\
&=\frac{\partial{e}}{\partial{U_j^{k}}} \times \frac{\partial{\sideset{}{_l} \sum{(X_l^{k-1} \times W_{l,j}^k)}}}{\partial{W_{i,j}^k}} \\
&= \frac{\partial{e}}{\partial{U_j^{k}}} \times X_i^{k-1} 
\end{align*}
$$

Alogorithm:

$$
\begin{align*}
\Delta W_{i,j}^k &=-  \alpha  \times \frac{\partial{e}}{\partial{W_{i,j}^k}} \\
W_{i,j}^k(t)&=W_{i,j}^k(t-1)+\Delta W_{i,j}^k(t-1)
\end{align*}
$$
