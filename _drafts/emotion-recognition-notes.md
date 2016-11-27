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

+ [1-norm Support Vector Machine](https://www.evernote.com/l/ApjEtnJA98NGaZ3WmZMfpnfMkbbVbPrN3ys)
  给出了1-norm和2-norm支持向量机的定义公式

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

+ [Emotion Recognition Based on Physiological Changes in Music Listening](https://www.evernote.com/l/Aph3Wcxnf4NEebBWITcY3_z4cY7tpcGxVNA)
  + 详细列出了ECG影响emotion的各个feature以及计算方法
  + 详细描述了ECG等信号的特征
  + 详细讲解了2D emotion 模型
  + 根据这篇文章，只需找到合适的数据集即可开始实现算法

+ [https://www.evernote.com/l/ApipVmoHFx9DZ5jyjE4zrvKxQ69Y1fg_FVc](https://www.evernote.com/l/ApipVmoHFx9DZ5jyjE4zrvKxQ69Y1fg_FVc)
  + 这篇摘要提到了MIT和德国一所大学的emotion dataset, 已经找到MIT的数据集，德国的还没有
  + MIT的数据集没有数据格式说明，无法使用
  + 德国大学的数据集能查找到的是RECOLA Database，但它不是任意下载的，需要申请

+ [Wearable and Automotive Systems for Affect Recognition from Physiology](https://www.evernote.com/l/Aphh-USaadZIy6kAs2ncuONbgQNVGnQdznA)
  + 详细介绍了BVP,EKG,Skin,EMG,Respiration的测量方法以及关于该指标的相关工作，很详细
  + 详细讲解了EKG的频谱特征
  + 总结了所有的情感模型理论 
  + 对历史研究成果讲解的很透彻
  + 实验使用的是8种情感模型
  + 3.3给出了feature计算的具体公式和每种信号计算的featrue种类
  + 3.5给出了详细的分类算法,再次提到了Fisher Projection
  + 5.5.1详细介绍HRV,并提到了一个计算库http://ecg.mit.edu
  + 这篇文章给出的HRV featrue计算比较可行

+ [Emotion Detection and Recognition using HRV Features Derived from Photoplethysmogram Signals](https://www.evernote.com/l/Apgy2wSWnJNKa735I591_RmHoCBWZ6NcMVA)
  + 该文章使用PPG信号作为物理信号源
  + 对PPG信号的处理主要是HRV features,包含时域和频域两个部分
  + 使用SVM进行情感识别
  + 给出了系统架构图
  + 给出了各个预处理过程的输出图形
  + 指出不同频域带宽的比值预示着不同的情绪
  + 给出了提取的featrue描述，但是没有具体计算公式
  + 没有给出每个featrue是基于那种数据计算的，有没有进行光滑和normalize处理

+ [Wiki:Heart rate variability](https://en.wikipedia.org/wiki/Heart_rate_variability)
  + 指出featrue中的NN是用来替代RR,表示是normal的heart beat
  + 指出在紧张时HRV的高频活动会降低
  + 指出HRV是和呼吸相关的
  + 给出了HRV的详细定义

+ [iOS用手机摄像头检测心率(PPG)](https://www.evernote.com/l/Apg4fKkZKQxNa5wdwgJOIjpjn9dIvTCOkpY)
  + 完整描述了使用PPG检测心率的架构
  + 使用带通滤波器进行预处理
  + 使用基音检测算法检测波峰
  + 提出了一些降低采样数据量的方法
  + 给出了PPG的计算方法

+ [Paper:Heart rate variability](https://www.evernote.com/l/ApiDigYqYWNOY49kVkHgNNFO57xqeVCvml0)
  + 指出时域分析优先选择SDNN,HRV Triangular index,RMSSD.

+ [A Real-Time QRS Detection Algorithm](https://www.evernote.com/l/ApgHrY9YrNRFiLFvqtf9w0p-lso4h7qjbNs)
  + 给出了数据预处理的过程，模拟信号滤波器将ECG信号限制在50Hz,以200p/s的采样频率进行离散化，经过带通滤波器(包括高通和低通)过滤噪音，经过另外一个滤波器(貌似是估计导数)，再经过一个squaring process和moving window
  + QRS 的频率在5-15Hz 
  + 给出了高通和低通滤波器的数学公式
