---
layout:   single
title: "Center Loss"
date: 2017-07-04 22:58:00
categories: Paper
tags: Loss
---

在图像分类中，常用的方法是使用softmax来得到seperable的特征，这种方法在测试集的类别属于训练集类别的子集的情况比较适用。然而，在人脸识别任务中，不可能把每个人的脸都作为一个类别来训练模型，这需要巨大的人力去收集如此庞大的数据。因此，更可行的方法是对比特征的差别。显然，特征仅仅是seperable是不够的，而且需要discriminative，也就是类间差距大，类内差距小。要实现这种训练方式，就需要用到本文所提出的center loss。
[A Discriminative Feature Learning Approach for Deep Face Recognition](http://ydwen.github.io/papers/WenECCV16.pdf)
{% include toc %}

## Abstract (References form the original)
Convolutional neural network (CNNs) have been widely used in computer vision community, significantly improving the state-of-the-art. In most of the available CNNs, the softmax loss function is used as supervision signal to train the deep model. In order to enhance the discriminative power of the deeply learned features, this paper proposes a new supervision signal, called center loss, for face recognition task. Specifically, the center loss simultaneously learns a center for deep feature of each class and penalize the distance between the deep features and their corresponding class centers. More importantly, we prove that the proposed center loss function is trainable and easy to optimize in the CNNs. With the joint supervision of softmax loss and center loss, we can train a robust CNNs to obtain the deep features with the two key learning objectives, inter-class dispension and intra-class compactness as much as possible, which are very essential to face recognition. It is encouraging to see that our CNNs (with such joint supervision) achieve the state-of-the-art accuracy on several important face recognition benchmarks, Labeled Faces in the Wild(LFW), YouTube Face(YTF), and MegaFace Challenge. Especially, our new approach achieve the best results on MegaFace(the largest public domain face benchmark) under the protocol of small training set(containing under 500000 images and under 20000 persons), significantly improving the previous results and setting new state-of-the-art for both face recognition and face verification tasks.

## 简介
卷积神经网络在很多视觉任务上都能取得很好的效果，它能够把输入映射到一个十分抽象的高级特征上，而这个特征容易被计算机所理解。
在普遍的物体，场景，动作识别上，测试集的类别往往是训练集类别的子集，这被称为close-set识别。所以，我们只需要用softmax把特征分类就可以了。
然而，在人脸识别的任务上，所需的特征不仅仅是能够被分类，而且不同类别特征之前的差别也需要有较大的差别。因为不可能把每个人的脸都作为一个单独类别来训练模型，而是需要对比特征之间的差别才是人脸识别可取的方式。
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-CenterLoss/post-CenterLoss1.png){: .align-center}
正如上图所示，discriminative特征比seperable特征更容易分类。显然，只用Softmax来训练网络值得得到seperable特征，如果需要得到discriminative特征，则可以使用这篇论文所提出的center loss来训练模型。
在这之前，就已经有人提出了contrast loss和triplet loss来训练得到discriminative特征，但是随着训练样本数量的增长，这两种loss所需要的pairs或者triplets呈阶乘式增长，这就导致十分缓慢的收敛速度。
因此，作者提出center loss，它能够有效地对于每一个类别学习一个中心。在训练过程中，同时减少类内差别和更新中心的位置。结合softmax(用于分开不同类别的特征)，能够得到类间差别大，类内差别小的高级抽象特征。

## Softmax
Softmax是使用指数函数并归一化，得到一个特征差别被拉大并且归一化得输出。具体公式如下
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-CenterLoss/post-CenterLoss2.png){: .align-center}
使用Softmax训练，得到以下的特征
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-CenterLoss/post-CenterLoss3.png){: .align-center}
a是属于训练集的，b是属于测试集。不同类用不同颜色表示。显然，各个类别之间是分开的(seperable)，但是并不是很集中。

## CenterLoss
对于$$y_i$$类有中心特征$$c_{yi}$$，对于同一类的特征，我们用欧式距离来作为损失函数来监督模型的训练。具体为以下公式
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-CenterLoss/post-CenterLoss4.png){: .align-center}
同时，在学习过程中，每一类自己的中心特征也会跟着学习而更新。具体公式如下
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-CenterLoss/post-CenterLoss5.png){: .align-center}
delta表示示性函数indicator function

![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-CenterLoss/post-CenterLoss6.png){: .align-center}
通过以上联合损失函数来监督模型训练

具体的实现算法
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-CenterLoss/post-CenterLoss7.png){: .align-center}
感觉都比较容易懂，- -。

显然，只使用softmax来训练，并没有办法降低intra-class variations类内差异。使用center loss之后，类内差异显著降低。并且，center loss 和contrastive loss、triplet loss对比，省去了构建pairs和triplets的庞大的数据扩展，和他们比其他更加简单有效。

对于alpha和lambda的设置，可以参看一下实验结果
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-CenterLoss/post-CenterLoss8.png){: .align-center}
最后看一下使用Center Loss之后的特征分布式怎样的，同一颜色表示同一类，显然类内分布更加紧密了。
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-CenterLoss/post-CenterLoss9.png){: .align-center}

## 总结
经本人测试，center loss还是比较有效的。而且contrastive loss 和triplet loss需要构建的组合类型会呈阶乘式增长。当然，也可以说不一定要穷尽所有构建训练样本的方式，只需要满足条件去小部分出来训练就足够了，不过根据前辈指点，triplet loss收敛还是需要比较长时间的。

center loss简单易用，也十分便于实现，我们提高模型性能的好伙伴。





