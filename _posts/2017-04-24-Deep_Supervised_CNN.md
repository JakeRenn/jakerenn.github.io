---
layout:   single
title: "U-net"
date: 2017-04-24 12:38:00
categories: Paper
tags: CNN
---

文章来源 [Deeply-Supervised CNN for Prostate Segmentation](https://arxiv.org/abs/1703.07523)

### 概要
核磁共振图不清晰的边缘会导致诊断困难。为了克服这个问题，这篇Paper提出一个深度监督的卷积神经网络，使用它来更精确得对核磁共振图进行分割。对于其他网络，深度神经网络中深层的**特征图(feature map)**往往比原输入图片小，也就是分辨率低。为了更精确地分割图片，输出原图大小的特征图是十分有必要的。

由于深度神经网络有很强的阶级(hierarchical)特征表达能力，这篇论文就用运用深度学习来有效检测边缘特征，效果显著。

## 模型
该网络主要分三个阶段(stage)，

1. 压缩图片，不断提取图片特征和减低特征图的分辨率，也就是convolution和max pooling的过程
2. 拓展图片，从中间小的特征图不断上取样并反卷积，也就是upsampling和deconvolution
3. 为了更好地帮助网络学习更加精确，使用残差信息（residual information）优化训练过程，也就是使用深度监督来监督模型的训练

### U-net
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-U_net.png)
U-net分为两部分

* 左边，再分为4个stage，每个stage由两个3x3卷积层和一个ReLU组成，之后接一个2x2 max pooling
* 右边，同样4个stage，每个stage包含两个操作，一个是上取样upsampling，另外一个是减少channels数和进行反卷积操作deconvolution。

### 深度监督CNN
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-U_net2.png)
**网络结构**： 正如VGG所证明，特征表达的深度对于分类的准确度很有帮助。然而更深的网络往往带来两个瓶颈

* 更深的网络代表更多的参数，也就倾向于过拟合(overfitting)，尤其是在训练样本不充足的情况下。
* 另外一个是更深的网络需要更强大的计算资源。

为了解决以上问题，每个stage的两次3x3卷积后接1x1卷积，它有两个优点：

* 减少参数，降低计算资源的要求
* 增加了网络的深度

### 结果展示
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-U_net3.png)
结果的优劣一目了然
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-U_net4.png)
从左到右分别分原图，标签，和深度监督CNN的各层特征


## 总结
模型架构总结：

* 左边4个stage，称为压缩路径。在第一个stage的特征通道数(feature channels) 为64，之后每个stage后都加倍特征通道数，第四个stage通道数为512。在每个stage里面，使用两个3x3卷积层，一个1x1卷积层和一个2x2 max pooling。
* 同样，右边4个stage，称为扩张路径。上取样特征，减半特征通道数和把卷积换成反卷积，其他操作一样。

思想总结：

* 运用了resNet的思想，把误差信息从高层传递到低层，加速和优化训练过程
* 最终图像和原图大小一致，能够做到高分辨率的特征提取
* 一开始，有一个downsampling和convolution的过程，得到一个更小但更稳定更加抽象的中间特征图，为后面的upsampling和deconvolution提供很好的特征。
