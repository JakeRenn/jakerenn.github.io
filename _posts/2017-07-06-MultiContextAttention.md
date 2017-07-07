---
layout:   single
title: "Multi-Context Attention for Human Pose Estimation"
date: 2017-07-04 22:58:00
categories: Paper
tags: CNN
---

[Download the paper](http://ydwen.github.io/papers/WenECCV16.pdf)
{% include toc %}

## Abstract (References form the original)
In this paper, we propose to incorporate convolutional neural network with a multi-context attention mechanism into an end-to-end framework for human pose estimation. We adopt stacked hourglass networks to generate attention maps from features at multiple resolutions with various semantics. The Conditional Random Field (CRF) is utilized to model the correlations among neighboring regions in the attention map. We further combine the holistic attention model, which focus on the global consistency of the full human body, and the body part attention model, which focuses on the detailed description for different body parts. Hence our model has the ability to focus on different granularity from local salient regions to global semantic consistent spaces. Additionally, we design novel Hourglass Residual Units(HRUs) to increase the receptive field of the network. These units are extensions of residual units with a side branch incorporating filters with larger receptive fields, hence features with various scales are learned and combined within the HRUs. The effectiveness of the proposed multi-context attention mechanism and the hourglass residual units is evaluated on two widely used human pose estimation benchmarks. Our approach outperforms all existing methods on both benchmarks over all the body parts.

## 简介
感觉Human pose estimation（人体姿态估计）有点拗口，这个实际上就是骨骼点的识别，接下来我们也这么称呼这类task。
因为由于肢体的变化的多样性、自身遮挡、衣服颜色变化等原因，骨骼点识别是个challenging task。虽然卷积神经网络在这个任务上能够取得不错的效果，但是在背景与人体肢体十分相像或者人体自遮挡的时候，要预测准骨骼点的位置还是十分困难的。
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiContextAttention/post-MultiContextAttention1.png){: .align-center}
如上图所示，第一行的是输入图片，整体的attention map，还有每个部分的attention map。第二行的是预测的骨骼点heatmap。第三行可视化了预测得骨骼点，a显然是错的，b在attention的具体指引下得到更加稳定的输出，c则进一步使用part attention map来进一步优化骨骼点的输出。

根据文献显示，结合不同的范围信息可以提升模型的性能。直观上，更大的attention map可以捕获整体的目标信息，而更小的attention map可以专注在局部的目标信息。之前的文献中，他们往往使用多个bounding box或者多种大小的image crop，这显然就缺乏灵活性。因此，这篇论文使用了multi-context attention map来捕捉全局和局部的目标信息，有效增强模型性能。此外，本文还使用了stacked hourglass 模型结构，这种结构会把特征图pooling（池化）到十分小的中间特征图，再upsampling（上采样）到原特征图大小，同时与之前的特征图相结合。多分辨率的特征图有效整合了全局信息和局部信息。

## 模型
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiContextAttention/post-MultiContextAttention2.png){: .align-center}
模型概观如上图所示。hourglass整合了multi resolution的信息，同时生成的attention map重新赋予特征图不同的权重。

#### Baseline Network
采用了8-stack 的hourglass网络，输入图片为256 x 256，输出特征图为64 x 64，通道数为K个，对应K个人体骨骼点，使用均方差Mean Squared Error来作为损失函数。

#### Nested Hourglass Networks
把hourglass中的残差模块增加一个分支来整合多分辨率的信息，也就是本文所提出的Hourglass Residual Units(HRUs)，所得网络就是nested hourglass network。如下图所示。
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiContextAttention/post-MultiContextAttention3.png){: .align-center}

#### Multi-Resolution Attention
在每个hourglass里面，多个分辨率的attention map生成于不同大小的特征。然后attention map被结合与feature map来得到更优的输出。
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiContextAttention/post-MultiContextAttention4.png){: .align-center}
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiContextAttention/post-MultiContextAttention5.png){: .align-center}

#### Multi-Semantics Attention
不同的stack专注于不同的语义：低级的stack专注于输入的局部，而高级的stack能够整合整体的表达。

#### Hierarchical Attention Mechanism
在1 - 4 stack中，我们使用两个全局attention map来整合全局信息。在5-8 stack中，我们设计了阶级性由粗到精的attention策略来放大局部信息。

## Nested Hourglass Networks
如上图所示，Nested Hourglass Networks由HRUs模块组成，而HRUs模块是残差模块的进阶版。首先残差模块有两个分支，一个是identity mapping branch，另外一个是residual branch，具体实现和模型在网络上都有很多资料。而HURs增加了一个分支，Max-pooliing-2x2 + Conv-3x3 + Conv-3x3 + Deconv-2x2。显然这个分支中间处理的特征边长是另外两个分支的一半，有更高级、全局、稳定的特征表达，此外，这个分支的视野域为10，而residual branch的视野域只有3，这个分支可以捕捉更全局的特征。

## Attention Mechanism
### 传统的attention
对于原文比较枯燥的公式，我这里用十分宏观的方式来介绍一下传统的attention，希望了解细节的同学可以看看原文。对于特征图进行一个线性变换和一次非线性变换，然后在特征图[H, W]上做softmax变换，得到attention map。最后用attention map和原图点乘得到输出。

### Our Multi-Context Attention Model
