---
layout: post
title: Emotion App
excerpt: ""
---

* Toc
{:toc}

# 技术要点

## Samples
The sample rate is 20p/s, emotion signal length is between 2000 to 5000 point 

## Segmention

## Data Preprocess

### Smooth
Using Hanning window which length is 500 to make  convolution with emotion signal

$$
\omega(n)=0.5-0.5cos(\frac{2\pi{n}}{M-1}) \quad 0 \le n \le  M-1
$$

### Normalized

$$
\Gamma=\frac{g-min(g)}{max(g)-min(g)}
$$

## Extract Features

### Features Defination
  + Mean for raw signal

    $$
    \mu_X=\frac{1}{N}\sideset{}{_{n=1}^N}\sum{X_n}
    $$

  + Mean for normalized signal 

    $$
    \tilde{\mu}_X=\frac{1}{N}\sideset{}{_{n=1}^N}\sum{\tilde{X}_n}
    $$

  + Variance for raw signal

    $$
    \sigma_X^2=(\frac{1}{N-1}\sideset{}{_{n=1}^N}\sum{( X_n-\mu_X )^2})
    $$

  + Variance for normalized signal

    $$
    \tilde{\sigma}_X^2=(\frac{1}{N-1}\sideset{}{_{n=1}^N}\sum{( \tilde{X_n}-\tilde{\mu}_X )^2})
    $$

  + First forward difference mean for smoothed signal

    $$
    \delta_x=\mu(x_{n+1}+x_n)=\frac{1}{N-1}(x_N-x_1)
    $$

  + First forward difference mean for normalized smoothed signal

    $$
    \tilde{\delta}_x=\frac{1}{N-1}(\tilde{x}_N-\tilde{x}_1)
    $$

### Feature for each signal
  + EMG
    + Mean for raw signal
      Reflect jaw cleaching, frowning and smiling
  + GSR
    + Mean for normalized signal
    + First forward difference mean for normalized smoothed signal
  + BVP(Blood Volume Pulse) Heart Rate
    + Mean for smoothed signal(no normalized) 
    + First forward difference mean for smoothed signal(no normalized)  
  + Respiration
    + Time domain
      + Mean for normalized signal
      + Variance for normalized signal
    + Frequency domain


## Select Features

## SVM

# 实现思路
+ 由于很难找到能够使用的数据集，决定先实现数据处理算法，计算出feature向量，再根据向量间的距离来判断情绪是否有起伏。
+ 从目前查到的资料看，使用BVP(Blood Volume Pulse)信号进行情感预测是比较可行的，该信号可通过手机摄像头获取，且有iOS代码库可供参考：https://github.com/chroman/HeartBeats
+ 剩下需要考虑的问题是该计算哪些feature，以及如何判断情绪起伏，目前能想到的方案是:
  + 按固定时间间隔获取BVP信号
  + 对BVP信号进行预处理并计算feature向量
  + 计算当前时间feature向量与前一次的向量距离，如果距离大于某个阈值则认为情绪发生变化
  + 如果距离小于阈值则将两次向量进行融合，最简单的方法是进行光滑处理
    $$
    X_c=\alpha X_c + (1-\alpha) X_t
    $$
