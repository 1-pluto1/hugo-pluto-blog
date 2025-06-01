---
title: 3D-LLM：Injecting the 3D World into Large Language Models
date: 2024-06-15T11:30:03+00:00
tags:
  - MLLM
  - 3DLLM
categories:
  - MLLM
author: ZhaoYang
showToc: true
TocOpen: true
draft: false
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

https://blog.51cto.com/u_16282361/7841645

#### 3D-LLM的优势：

By taking the 3D representations of scenes as input, LLMs are blessed with twofold advantages: (1) long-term memories about the entire scene can be stored in the holistic 3D representations, instead of episodic partial-view observations. (2) 3D properties such as affordances and spatial relationships can be reasoned from 3D representations, far beyond the scope of language-based or 2D image-based LLMs.

（1）有关整个场景的长期记忆可以存储在整体 3D 表示中，而不是情景式的部分视图观察中。（2）诸如可供性和空间关系等 3D 属性可以从 3D 表示中推理出来，远远超出了基于语言或基于 2D 图像的 LLM 的范围。

#### 第一个困难：

One major challenge of training the proposed 3D-LLMs lies in data acquisition.

数据获取是一大困难。

##### 解决：

 To address this, we propose a set of unique data generation pipelines that could generate large-scale 3D data paired with language

为了解决这个问题，我们提出了一套独特的数据生成流程，可以
生成与语言配对的大规模 3D 数据。

 Specifically, we make use of ChatGPT [33] and devise three efficient prompting procedures for communication between 3D data and language

具体来说，我们利用 ChatGPT [33] 并设计了三种有效的提示程序，用于 3D 数据和语言之间的通信。

#### 第二个困难

The next challenge resides in how to obtain meaningful 3D features that could align with language features for 3D-LLMs. 

下一个挑战在于如何获得有意义的 3D 特征，以与 3D-LLM 的语言特征保持一致。

一种方法是像训练CLIP一样，训练一个文本和3D信息对齐的3D编码器。但是太贵了

另一种方法：

Since our extracted 3D features are mapped to the same feature space as 2D pretrained features, we can seamlessly use 2D VLMs as our backbones and input the 3D features for the efficient training of 3D-LLMs.

由于我们提取的 3D 特征被映射到与 2D 预训练特征相同的特征空间，我们可以无缝地使用 2D VLM 作为我们的主干并输入 3D 特征以高效训练 3D-LLM。



弥补语言和空间的信息差距：

因此，我们开发了一种 3D 定位机制，以弥合语言和空间位置之间的差距。具体来说，我们将 3D 位置嵌入附加到提取的 3D 特征中，以更好地编码空间信息

In addition, we append a series of location tokens to the 3D-LLMs, and localization can be trained via outputting location tokens given the language descriptions of specific objects in the scenes. In this way, 3D-LLMs could better capture 3D spatial information.

此外，我们在 3D-LLM 中附加了一系列位置标记，可以根据场景中特定对象的语言描述输出位置标记来训练定位。这样，3D-LLM 就可以更好地捕捉 3D 空间信息。



#### 所有的贡献点：

- 我们引入了一系列新的基于 3D 的大型语言模型 (3D-LLM)，它们可以将具有特征和语言提示的 3D 点作为输入，并执行各种与 3D 相关的任务。我们专注于超出普通 LLM 或 2D-LLM 范围的任务，例如有关整体场景理解、3D 空间关系、可供性和 3D 规划的任务。
- 我们设计了新颖的数据收集管道，可以生成大规模 3D 语言数据。基于管道，我们收集了一个包含超过 30 万个 3D 语言数据的数据集，这些数据涵盖了多种 3D 相关任务，包括但不限于 3D 字幕、密集字幕、3D 问答、任务分解、3D 基础、3D 辅助对话、导航等。
- 我们使用 3D 特征提取器，从渲染的多视图图像中提取有意义的 3D 特征。我们利用 2D 预训练 VLM 作为主干，以实现高效训练。我们引入了 3D 定位机制来训练 3D-LLM，以更好地捕获 3D 空间信息。
- 在保留的评估数据集 ScanQA 上进行的实验优于最先进的基线。
  特别是，3D LLM 在 ScanQA 上的表现远超基线（例如，BLEU-1 为 9%）。
  在保留的数据集上进行的 3D 字幕、任务组合和 3D 辅助对话实验表明，我们的模型优于 2D VLM。定性研究进一步表明，我们的模型能够处理多种任务
- 我们计划发布我们的 3D-LLM、3D 语言数据集以及数据集的语言对齐 3D 特征，以供未来的研究开发





#### 三维数据生成方法

1) ##### boxes-demonstration-instruction based prompting

We input the axis-aligned bounding boxes (AABB) of both the rooms and the objects in the 3D scenes, providing information about the semantics and spatial locations of the scene. We then provide specific instructions to the GPT model to generate diverse data

2) ##### ChatCaptioner based prompting.

   采用了类似于该论文的技术：

   [52] D. Zhu, J. Chen, K. Haydarov, X. Shen, W. Zhang, and M. Elhoseiny. Chatgpt asks, blip-2 answers: Automatic questioning towards enriched visual descriptions, 2023.

   为了收集 3D 相关数据，我们将不同视角的图像输入到 BLIP-2，并指示 ChatGPT 提出问题并收集不同区域的信息，以形成整个场景的全局 3D 描述。

3) ##### Revision based prompting

.它可用于将一种类型的 3D 数据传输到另一种类型。





#### 训练3DLLM

##### 第一步：构建有意义的 3D 特征，以与语言特征保持一致。

从头开始预训练这样的特征学习器非常困难，因为在数量和多样性方面，没有 3D 语言资产可以与互联网规模的图像语言对相媲美

相反，已经提出了许多从二维多视角图像中提取三维特征的方法[26, 20, 16, 23]。受这些工作的启发，我们通过从几个不同的视角渲染三维场景来提取三维点的特征，并从渲染后的图像特征中构建三维特征。

我们首先按照 [26] 为渲染图像提取像素对齐的密集特征。然后，我们利用三种方法从渲染图像特征中构建 3D 特征。这些方法是针对不同类型的 3D 数据而设计的。

• Direct Reconstruction. We directly reconstruct point cloud from rgbd images rendered from the 3D data using ground-truth camera matrixes. The features are directly mapped to the reconstructed 3D points. This method is suitable for rendered rgbd data with perfect camera poses and intrinsics.

 • Feature Fusion. Similar to [26], we fuse 2D features into 3D maps using gradslam [28]. Different from dense mapping methods, the features are fused in addition to depths and colors. This method is suitable for 3D data with noisy depth map renderings, or noisy camera poses and intrinsics.

 • Neural Field. We utilize [20], which constructs 3D compact representation using neural voxel field [43]. Specifically, each voxel in the field has a feature in addition to density and color. Then we align 3D features in the rays and 2D features in the pixels using MSE loss. This method is for 3D data with RGB renderings but no depth data, and noisy camera poses and intrinsics. In this way, we are able to obtain the < N, Dv >-dim 3D features of each 3D scene, where N is the number of points in the point cloud, and Dv is the feature dimension

##### 第二步：训练3D-LLMs

TODO