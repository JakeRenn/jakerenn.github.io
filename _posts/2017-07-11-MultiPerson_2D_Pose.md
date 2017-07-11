---
layout:   single
title: "Realtime Multi-Person 2D Pose Estimation using Part Affinity Fields"
date: 2017-07-11 15:57:00
categories: Paper
tags: CNN
---

[Download the paper](https://arxiv.org/abs/1611.08050)
{% include toc %}

## Abstract (References form the original)
We present an approach to efficiently detect the 2D pose of multiple people in an image. The approach uses a non-parametric representation, which we refer to as Part Affinity Fields(PAFs), to learn to associate body parts with individuals in the image. The architecture encode the global context, allowing a greedy bottom-up parsing step that maintain high accuracy while achieving realtime performance, irrespective of the number of people in the image. The architecture is designed to jointly learn part locations and their association via two branches of the same sequential predition process. Our method places the first in the in argural COCO 2016 keypoints challenge, and significantly exceeds the previous state-of-the-art result on MPII Multi-Person benchmark, both in accuracy and efficiency.


## 简介
Human 2D pose estimation就是一个找到某个部位的关键点的任务。从一张图片中推断出多个人的姿势是十分具有挑战性的。首先，每张图片中包含的人的数量、位置、大小都不知道。第二，人与人之间的交互会导致十分在空间上推断骨骼点位置更加复杂。第三，运行时间复杂度往往和图片中人的数量成正比，使得多人骨骼点预测得实时性更难以实现。
一个常用的方法是使用人物的检测器，并且对每个检测出的任务单独进行单人的骨骼点预测。这个也会出现严重的问题：如果人物检测器检测失败，比如人们靠得很近，就会使得后面的骨骼预测结果很差。此外，这种从上到下方法的运行时间和图片中人的数量成正比关系。对于每个人都需要进行一次pose estimation inference，而对于多个人，则计算量就成相应倍速增加。相反的，从下到上的方法更具有吸引力，因为这样在早起就能提供更稳定的特征并分离运行复杂度和图片中人数量的关系。但是，bottom-up方法并没有直接从其他骨骼点或者人上使用全局的上下文信息。实际上，之前的bottom-up方法并没有获得在效率上的提升，因为最终的分析需要很复杂的全局推理。
这篇论文中，作者提出一个有效的多人骨骼点检测方法，并且可以达到最高水准的准确率。作者提出bottom-up方式，这是通过Part Affinity Fields(PAFs)实现的，这是一系列2D向量来对图片上的位置和方向编码。我们证明了同时推断bottom-up检测并关联的方法有效地对全局上下文信息编码，并可以通过贪心算法来达到高质量的结果。
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiPerson_2D_Pose/post-MultiPerson_2D_Pose1.png){: .align-center}

## 方法
下图解释了本文论方法的中体过程
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiPerson_2D_Pose/post-MultiPerson_2D_Pose2.png){: .align-center}
首先，以大小为$$w x h$$的彩色图片作为输入(a)，并输出每个人相应的骨骼点(e)。首先，一个前馈网络同时预测一系列2D骨骼点的概率图(b)，并且一系列的2D向量来表示骨骼与骨骼之间的联系性(c)。$$S = (S_1, S_2, ..., S_J)$$有J个概率图，每个都对应相应的部位。$$L = (L_1, L_2, ..., LC) $$表示C个向量域，每个对应相应个肢体limb。最后，概率图和关系域通过贪心推断来分析并输出图中所有人的2D关键点。

### 同时检测和联系
如下图所示，同时检测出概率图和关系域。
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiPerson_2D_Pose/post-MultiPerson_2D_Pose3.png){: .align-center}
这个网络分为两个分支：

* 上分支（橙色），预测概率图
* 下分支（蓝色），预测关系域

每个分子都是一个迭代前馈的过程。图片用卷积神经网络来处理，生成一系列的特征图F，它作为后面多个stage的输入。在第一个stage，网络生成第一系列的detection confidence map $$S^1 = f^1(F)$$，和一系列的part affinity fields $$ L^1 = g^1(F) $$，这里$$f^1$$和$$g^1$$都是CNN在Stage 1的推断结果。在之后的stage中，来自上一时刻的输出特征图S和L 以及 原始特征图F会concatenated连接在一起并生成新的特征图。

$$ S^t = f^t(F, S^{t-1}, L^{t-1}) $$
$$ L^t = g^t(F, S^{t-1}, L^{t-1}) $$

其中$$f^t$$和$$g^t$$都是CNN在Stage t的输出结果。

![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiPerson_2D_Pose/post-MultiPerson_2D_Pose4.png){: .align-center}
上图表示多个stage不断优化confidence map的过程。

训练网络的过程中，我们使用两个Loss应用在每个stage后面。使用计算欧氏距离作为损失函数。
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiPerson_2D_Pose/post-MultiPerson_2D_Pose5.png){: .align-center}
以上公式中，带星号的是groundtruth。W是binary mask，当标签不存在对应的信息是，W为0。这是防止在训练过程中一直正确的正例预测。在每个stage中间的监督信息解决了梯度消失的问题。最后的总体损失函数是把每个stage的以上两个损失函数全部加起来。

### Confidence Maps for Part Detection
Confidence Maps上的每个像素表达每个部位在对应位置出现的概率。理想来说，如果一个人出现在图中，会出现一个单峰值，如果多个人出现在图中，则会出现多个峰值。每个部位有独立的confidence map表示，如果图中检测到两个人，则在每个confidence map中会出现两个峰值点。生成groundtruth时，使用高斯分布来概率图。要得到每个点的确切位置时，取对应的极大值对应额位置。

### Part Affinity Fields for Part Association
下图是部位联系的策略。
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiPerson_2D_Pose/post-MultiPerson_2D_Pose6.png){: .align-center}
(a)是身体部位的检测结果，(b)是使用中点表达，显然绿线是错误的而黑线才是正确的，(c)使用PAFs所得结果。

给了一系列的身体部位的点，我们需要把它们组装起来得到全身的pose。其中一个方法就是使用重点来表示两个部位之间的limb。结果如上图b所示。当人们拥簇在一起，中点方法很容易会得到错误的联系方式。这样的联系方式有以下缺点：它只表达了位置，没有表达方向，并且它把方向的表达简化为单个点。

为了解决这些限制，作者提出一个新的特征表达方法，较part affinity fields(PAFs)来保存位置和方向信息。PA是一个表达每个limb的二维向量域。二维向量encode了从一个点到另外一个点的方向。
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiPerson_2D_Pose/post-MultiPerson_2D_Pose7.png){: .align-center}
方程如下
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiPerson_2D_Pose/post-MultiPerson_2D_Pose8.png){: .align-center}
也就是说在一定的长和宽范围里面，这里的值都等于v值。

在每个点上，得到的是每个人的PA点在该点的均值。
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiPerson_2D_Pose/post-MultiPerson_2D_Pose9.png){: .align-center}

在测试的时候，我们通过计算PAF的积分来衡量候选点之间的联系。具体来说，对于两个候选的部位候选点，d1和d2，我们取样d1和d2之间的PAF来衡量他们之间联系的置信度：
![](https://raw.githubusercontent.com/JakeRenn/jakerenn.github.io/master/images/post-MultiPerson_2D_Pose/post-MultiPerson_2D_Pose10.png){: .align-center}

### Multi-Person Parsing using PAFs
我们在detection confidence maps使用non-maximum suppression来获得一系列离散的骨骼点位置。而对于每个点之间的联系的可能性是一个NP-hard问题，所以我们采取贪心算法来实现高质量的匹配，并使用以上公式的积分来获得匹配的得分。

首先，我们获得一系列的骨骼点$$D_j = \{d^m_j: for j in \{j .. J\}, m in \{1 .. N_j\}\}$$，$$d^m_j$$是第j个骨骼点的第m个候选点。我们需要选择实际上相连接的一对点。我们使用$$z^{mn}_{j1j2} in \{0, 1\}$$来表示点$$d^m_{j1}$$和$$d^n_{j2}$$是否连接。所以所有连接的可能性为$$Z = \{z^{mn}_{j1j2}: for j1, j2 in \{1 ... \}, m in \{ 1 ... N_{j1}\}. n in \{ 1 ... N_{j2}\}\}

首先我们先考虑一对点j1，j2。首先考虑两点之间连接的可能性，然后选择得分最高的一项。然而对于多人的多个骨骼点检测的情况，骨骼点之间连接的可能性是NP-hard问题。对此，作者做了两个简化，第一个，作者选择最少数量的边来获得骨骼点的生成树，而不是完整的图。第二，作者进一步分离问题成邻近点之间的连接可能性是独立的。

