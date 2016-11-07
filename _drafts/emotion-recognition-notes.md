---
layout: post
title: Emotion Recognition Notes
excerpt: ""
---
# Literature
+ [Emotion Detection Using Physiological Signals](https://www.evernote.com/l/AphI0EO9GGFNRZxLe7pID7Pd-6dLHpfiIjE)
该论文描述了一个完整的情感分析系统，该系统使用EEG作为数据输入，主要包括预处理、特征提取、情感分类等部分。文中提到
  + 情感主要分为fear,anger,sadness,happiness,suprise和disgust种
  + 输入数据主要有：
    + 2D/3D face image
    + Audio Recording
    + Body movement
    + Physiological signal

+ [Using Neural Network to Recognize Human Emotions from Heart Rate Variability and Skin Resistance](https://www.evernote.com/l/ApglkTm-eDhI1LI_uHuGsdrFwLXdaTOuaRw) 
  该论文虽然说了使用神经网络对情感进行分类，但没有详细讲解网络构造，没有太多参考意义

+ [Recurrent Neural Networks for Emotion Recognition in Video](https://www.evernote.com/shard/s664/sh/40c0e1a3-d6ab-4cbc-94d8-34ccd0bd676e/d6162ef7c4c9295f4f416f1ec98819d9) 
  该论文主要介绍使用RNN对录像进行情感分析，不能在心跳分析上进行借鉴

+ [ECG Signal Feature Selection for Emotion Recognition](https://www.evernote.com/l/ApjeZ8VCVNdJW5Dl35cJ8w3WGIji8x3JIao)
  + 讲解了ECG的坐标轴意义以及特征
  + 提到了一个德国大学提供的数据集，该数据集已经根据情感进行分类，可用于算法测试
  + 提到了多种feature计算方法
  + 主要对joy和pleasure进行了验证
  + 从feature筛选，feature计算，到情感分类，算法讲解的都还比较清楚
  + 重点参考文章！！！！！！！

+ [Emotion Recognition using Wireless Signals](https://www.evernote.com/l/Apj3lsx5Cm9C6oJv9Wobk-owxo4139O4XfI)
  + 麻省理工的论文，他们使用无线电波测量人们的心跳和呼吸信号，然后对其进行情感分析
  + 该文章使用2D emotion model进行情感分类
  + 这篇文章使用无线信号，而我们打算使用Watch测量信号，两者与ECG都有较大差别，可以参考该文章的系统设计方案进行实施
  + 重点参考文章！！！！！！！
