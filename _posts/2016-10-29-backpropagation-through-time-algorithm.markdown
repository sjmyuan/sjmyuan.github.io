---
title: "Backpropagation Through Time Algorithm"
excerpt: ""
date: "2016-10-29 22:18:03 +0800"
---

Let's say:

+ the  level $$r$$ is recurrent level, their output value will feedback into themselves.
+ $$e$$ denote the loss between network output and real value
+ $$U_i^k$$ denote the $$i$$-th node total input value in level $$k$$
+ $$X_i^k$$ denote the $$i$$-th node output value in level $$k$$
+ $$W_{i,j}^k$$ denote the weight between $$i$$-th node in level $$k-1$$ and $$j$$-th node in level $$k$$
+ $$V_{i,j}$$ denote the weight between $$i$$-th node in level $$r$$ and $$j$$-th node in level $$r$$
+ $$\Delta W_{i,j}^k$$ denote the increment of $$W_{i,j}^k$$ in each iteration
+ $$\Delta V_{i,j}$$ denote the increment of $$V_{i,j}$$ in each iteration
+ $$f_i^k$$ denote the $$i$$-th node activation function in level $$k$$
+ $$\partial{f_i^k}$$ denote the derivation of  $$i$$-th node activation function in level $$k$$
+ $$\alpha$$ denote the learning rate.
+ $$T$$ is the total time

The value flow:

$$
\left \{
  \begin{align*}
   X^{r-1}(t) & \xrightarrow{W^r}\\
   X^{r}(t-1) & \xrightarrow{V}
  \end{align*}
\right \}
  \xrightarrow{+} U^r(t) \xrightarrow{f^r} X^{r}(t)
$$

The error flow:

$$
e \longrightarrow X^r(t) \xrightarrow{f^r} U^r(t) \xrightarrow{}
\left \{
  \begin{align*}
   \xrightarrow{X^{r-1}(t)} & W^r \\
   \xrightarrow{X^{r}(t-1) } & V 
  \end{align*}
\right \}
$$

Derivation process:

$$
\begin{align*}
U_i^r(t)
 &=\sideset{}{_l}\sum{W_{l,i}^r \times X_l^{r-1}(t)}+ \sideset{}{_m}\sum{V_{m,i} \times X_m^{r}(t-1)}\\
 \\
\frac{\partial{e}}{\partial{X_i^r(t)}} 
  &=\sideset{}{_l}\sum{  \frac{\partial{e}}{\partial{U_l^{r+1}(t)}} \times 
  \frac{\partial{U_l^{r+1}(t)}}{\partial{X_i^r(t)}}} + \sideset{}{_m}\sum{ 
  \frac{\partial{e}}{\partial{U_m^{r}(t+1)}} \times 
  \frac{\partial{U_m^{r}(t+1)}}{\partial{X_i^r(t)}}} \\
  &=\sideset{}{_l}\sum{\frac{\partial{e}}{\partial{U_l^{r+1}(t)}} \times 
  \frac{\partial{ (\sideset{}{_{l_1}} \sum{W_{l_1,l}^{r+1} \times X_{l_1}^r(t)})}}{\partial{X_i^r(t)}}} \\
  &+ \sideset{}{_m}\sum{   \frac{\partial{e}}{\partial{U_m^{r}(t+1)}} \times \frac{\partial{(\sideset{}{_{m_1}}\sum{W_{m_1,i}^r \times X_{m_1}^{r-1}(t+1)}+  \sideset{}{_{m_2}}\sum{V_{m_2,i} \times X_{m_2}^{r}(t)})}} {\partial{X_i^r(t)}}} \\
    &=\sideset{}{_l}\sum{ \frac{\partial{e}}{\partial{U_l^{r+1}(t)}} \times W_{i,l}^{r+1}}+
  \sideset{}{_m}\sum{ \frac{\partial{e}}{\partial{U_m^{r}(t+1)}} \times V_{i,m}} \\
  \\
  \frac{\partial{e}}{\partial{U_i^r(t)}} &= \frac{\partial{e}}{\partial{X_i^r(t)}} \times \frac{\partial{X_i^r(t)}}{\partial{U_i^r(t)}}\\
  &= \frac{\partial{e}}{\partial{X_i^r(t)}} \times \partial{f_i^r} \\
  &= (\sideset{}{_l}\sum{ \frac{\partial{e}}{\partial{U_l^{r+1}(t)}} \times W_{i,l}^{r+1}}+
  \sideset{}{_m}\sum{ \frac{\partial{e}}{\partial{U_m^{r}(t+1)}} \times V_{i,m}}) \times \partial{f_i^r} \\
  \\
  \frac{\partial{e}}{\partial{W_{i,j}^r}}(t) 
  &= \frac{\partial{e}}{\partial{U_j^{r}}} \times \frac{\partial{U_j^r}}{\partial{W_{i,j}^r}} \\
  &=\frac{\partial{e}}{\partial{U_j^{r}}} \times \frac{\partial{(\sideset{}{_l}\sum{W_{l,j}^r \times X_l^{r-1}(t)}+ \sideset{}{_m}\sum{V_{m,j} \times X_m^{r}(t-1)}})}{\partial{W_{i,j}^r}} \\
  &= \frac{\partial{e}}{\partial{U_j^{r}}} \times X_i^{r-1} \\
  \\
  \frac{\partial{e}}{\partial{V_{i,j}}}(t) 
  &= \frac{\partial{e}}{\partial{U_j^{r}}} \times \frac{\partial{U_j^r}}{\partial{V_{i,j}}} \\
  &=\frac{\partial{e}}{\partial{U_j^{r}}} \times \frac{\partial{(\sideset{}{_l}\sum{W_{l,j}^r \times X_l^{r-1}(t)}+ \sideset{}{_m}\sum{V_{m,j} \times X_m^{r}(t-1)}})}{\partial{V_{i,j}}} \\
  &= \frac{\partial{e}}{\partial{U_j^{r}}} \times X_i^{r}(t-1) 
 \end{align*}
$$

Alogorithm:

$$
\begin{align*}
\Delta W_{i,j}^k &=-  \alpha  \times   \sideset{}{_{t=1}^{t=T}} \sum{\frac{\partial{e}}{\partial{W_{i,j}^k}}(t)}  \\
W_{i,j}^k(T)&=W_{i,j}^k(T-1)+\Delta W_{i,j}^k \\
\Delta V_{i,j} &=-  \alpha  \times   \sideset{}{_{t=1}^{t=T}} \sum{\frac{\partial{e}}{\partial{V_{i,j}}}(t)}  \\
V_{i,j}(T)&=W_{i,j}(T-1)+\Delta V_{i,j}
\end{align*}
$$

