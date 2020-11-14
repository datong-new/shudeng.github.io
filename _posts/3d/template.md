---
layout: post
title: 3DSSD：Point-based 3D Single Stage Object Detector
categories: [计算机视觉，深度学习，3D 目标检测]
description: 
keywords: 3D 目标检测 点云
---

## 引言
**主要问题：**
- 在 point-based 的方法中，FP 层（Feature Propagation）耗费了将近一半的推断时间。但是，FP 层又是不能丢弃的。
- 现有 SA 层（Set Abstraction）的采样方法是 FPS （Furthest Point Sampling），主要标准是 3D 空间位置的欧式距离（D-FPS）。有些目标表面的点很少，因此这种采样会丢失其很多的信息。
- 如果没有上采样操作，仅仅基于降采样后的点进行目标预测，检测的性能会下降很多。因此在 FP 层中必须进行上采样。**为了加快推断的速度，本文提出，基于特征距离的采样方法，叫做 F-FPS。** 它可以为不同的目标采样更好的点。文中最终的采样方法是 F-FPS 和 D-FPS 的融合。
- 在 SA 层之后，本文提出一个 box 预测网络，其包含一个 CG 层（candidate generation）。
- 实验表明，本文的方法超过所有 voxel-based 的单阶段方法，并且与两阶段的方法性能差不多，但是更快的推断速度。

## 方法介绍
### Fusion Sampling
Point-based 的方法主要包含两个步骤：1）SA 层（Set Abstraction），将数据进行下采样，以提高运算效率和增大感受域。FP 层（Feature Propagation）将特征传递给下采样过程中丢弃的点，使得所有点的特征都能得到更新。2）基于得到的特征，一个 refinement 模块对 RPN 提出的 proposals 进行修复，以得到更准确的结果。

SA 层对提取点的特征是不可或缺的，FP 层和 refinement 模块限制了模型的性能，可以被进一步改进。

SA 层使用 D-FPS 对点云数据进行下采样。如果没有 FP 层，box 预测网络必须基于这些下采样后的点进行预测。但是，D-FPS 方法只考虑了点直接的空间距离，下采样后的点很大比例是背景点，而对于表面点较少的目标，丢失了很多的信息。下采样后点的个数是固定的， $N_m$，对于较远（小）的目标，他们表面的点会变得更少，甚至没有。这种问题在某些复杂的数据集比如 nuScenes 上更加严重。

![](../../images/3d/2020-11-14-3dssd-01.PNG)

为了量化，文中定义一个指标，召回率 —— $\frac{\#下采样后的目标数量}{\#下采样前的目标数量}$. 上图是 nuScenes 的下采样召回率情况。使用 D-FPS 的方法，当下采样的点很少是，大概只有一半的召回率。为了缓解召回率低的问题，现在的方法都是使用 FP 层去上采样，召回下采样过程中被丢弃的点，但是这种方法十分耗时。




