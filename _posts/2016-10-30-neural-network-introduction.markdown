---
layout: post
title: "Neural Network Introduction"
excerpt: ""
date: "2016-10-30 17:31:25 +0800"
---
# Contents
{:.no_toc}

* Toc
{:toc}

##  Evolution

+ Neural

![enter image description here](http://img.blog.csdn.net/20141213201613758?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvenp3dQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

+ Artificial neural

![enter image description here](http://www.funnyai.com/AI/Book/DigtalNN/images/4.2.ht13.gif)

+ MLP

![enter image description here](http://n.sinaimg.cn/tech/transform/20160223/tpiF-fxprucs6391728.png)

+ RNN

![enter image description here](http://img.ptcms.csdn.net/article/201501/29/54c985da2f578.jpg)
![enter image description here](http://img.blog.csdn.net/20150725153333816)
![enter image description here](http://www.wildml.com/wp-content/uploads/2015/09/bidirectional-rnn.png)
![enter image description here](http://www.wildml.com/wp-content/uploads/2015/09/Screen-Shot-2015-09-16-at-2.21.51-PM.png)

+ CNN

![enter image description here](http://www.36dsj.com/wp-content/uploads/2015/03/511-600x224.jpg)
![enter image description here](http://www.36dsj.com/wp-content/uploads/2015/03/710-600x247.jpg)
![enter image description here](http://www.36dsj.com/wp-content/uploads/2015/03/6.gif)
![enter image description here](http://www.36dsj.com/wp-content/uploads/2015/03/122-600x240.png)

+ AlphaGo

![enter image description here](http://img.blog.csdn.net/20160130153948867)

## History

+ **1943**

  心理学家Warren McCulloch和数理逻辑学家Walter Pitts建立了神经网络和数学模型，称为MP模型。他们通过MP模型提出了神经元的形式化数学描述和网络结构方法，证明了单个神经元能执行逻辑功能，从而开创了人工神经网络研究的时代

+ **1949**

  心理学家 Donald Hebb通过对大脑神经细胞的人类学习行为和条件反射的观察和研究，发表了《行为自组织》一书，提出了神经元学习的一般规则。Hebb学习规则指出神经网络的学习过程最终是发生在神经元之间的突触部位，突触的联结强度随着突触前后神经元的活动而变化，变化的量与两个神经元的活性之和成正比，这一思想至今仍为许多算法所采用，并在最近的生理解剖学中得到了证实

+ **1954**

  Farley and Wesley A. Clark 在麻省理工学院对第一次用计算机对Hebb的神经网络模型进行了模拟。

+ **1958**

  Frank Rosenblatt提出了感知器的概念，它是一个基于两层网络的模式识别算法，只用到了简单的加减操作。

+ **1959**

  斯坦福大学的Bernard Widrow和Marcian Hoff 开发出了ADALINE和MADALINE模型，该模型用于二进制码流的识别，当它通过电话线读取到一串二进制码流后，可以预测出下一个bit的值。MADALINE是第一个用于解决实际问题的神经网络模型，用于消除电话线中的回声，它和空中交通管理系统一样古老，并且仍在商用。

+ **1969**

  Marvin Minsky 和Seymour Papert仔细分析了以感知器为代表的神经网络系统的功能及局限后，出版了《Perceptron》一书，指出感知器不能解决异或等线性不可分问题且计算机的处理能力无法满足大型神经网络模型的运算。他们的论点极大地影响了神经网络的研究，加之当时串行计算机和人工智能所取得的成就，掩盖了发展新型计算机和人工智能新途径的必要性和迫切性，使人工神经网络的研究处于低潮。在此期间，一些人工神经网络的研究者仍然致力于这一研究，提出了适应谐振理论（ART网）、自组织映射、认知机网络，同时进行了神经网络数学理论的研究

+ **1982**

  美国加州工学院物理学家J.J.Hopfield提出了Hopfield神经网格模型，引入了“计算能量”概念，给出了网络稳定性判断

+ **1984**

  J.J.Hopfield又提出了连续时间Hopfield神经网络模型，为神经计算机的研究做了开拓性的工作，开创了神经网络用于联想记忆和优化计算的新途径，有力地推动了神经网络的研究

+ **1985**

  有学者提出了波耳兹曼模型，在学习中采用统计热力学模拟退火技术，保证整个系统趋于全局稳定点

+ **1986**

  进行认知微观结构地研究，提出了并行分布处理的理论

  **Rumelhart, Hinton, Williams发展了BP算法。Rumelhart和McClelland出版了《Parallel distribution processing: explorations in the microstructures of cognition》**

+ **1988**

  Linsker对感知机网络提出了新的自组织理论，并在Shanon信息论的基础上形成了最大互信息理论，从而点燃了基于NN的信息应用理论的光芒
  Broomhead和Lowe用径向基函数(Radial basis function, RBF)提出分层网络的设计方法，从而将NN的设计与数值分析和线性适应滤波相挂钩

+ **90年代**

  Vapnik等提出了支持向量机(Support vector machines, SVM)和VC(Vapnik-Chervonenkis)维数的概念。人工神经网络的研究受到了各个发达国家的重视，美国国会通过决议将1990年1月5日开始的十年定为“脑的十年”，国际研究组织号召它的成员国将“脑的十年”变为全球行为。在日本的“真实世界计算（RWC）”项目中，人工智能的研究成了一个重要的组成部分

## Classify

 + 多层网络BP算法
 + Hopfield网络模型
 + 自适应共振理论
 + 自组织特征映射理论
 
##   Advantage

 + 可以充分逼近任意复杂的非线性关系；

 ![enter image description here](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQM6Ki7qqm2qBXZ_WrhfisZg0ksgZzSwpG__kEh119B8i4CxhJK-Q)
 ![enter image description here](https://encrypted-tbn2.gstatic.com/images?q=tbn:ANd9GcSn4XVh9RzRB2dlBRCu6hByDJZMEYYfN2bRN-bczTQ_VlVm7FV0Jg) 

 + 所有定量或定性的信息都等势分布贮存于网络内的各神经元，故有很强的鲁棒性和容错性；
 + 采用并行分布处理方法，使得快速进行大量运算成为可能；
 + 可学习和自适应不知道或不确定的系统；
 + 能够同时处理定量、定性知识
 
##   Research Direction
+ 理论研究
 + 利用神经生理与认知科学研究人类思维以及智能机理
 + 利用神经基础理论的研究成果，用数理方法探索功能更加完善、性能更加优越的神经网络模型，深入研究网络算法和性能， 如：稳定性、收敛性、容错性、鲁棒性等；开发新的网络数理理论，如：神经网络动力学、非线性神经场等

+ 应用研究
 + 神经网络的软件模拟和硬件实现的研究
 + 神经网络在各个领域中应用的研究 
    1.  模式识别
    2.  信号处理
    3.  知识工程
    4. 专家系统
    5. 优化组合
    6. 机器人控制

##   Research Tools

+ Keras
+ Caffe
+ Theano
+ Torch
+ Pylearn2
+ TensorFlow
+ DeepLearning4j

##  Typical Steps

```flow
st=>start: Start
e=>end
PD=>operation: Prepare Data
CM=>operation: Config Model
BM=>operation: Build Model
Train=>operation: Train Model
Test=>operation: Test Model
Pre=>operation: Predict

st->PD->CM->BM->Train->Test->Pre->e
```

## Attention

1. Network topology
2. Matrix initial values
3. Gradient
4. Learning Rate

