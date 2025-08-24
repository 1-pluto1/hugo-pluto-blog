---
title: Qwen-Image Technical Report
date: 2022-09-15T11:30:03+00:00
tags:
  - 深度学习
  - LLM
  - MLLM
  - NLP
categories:
  - AI
  - MLLM
  - Research
author: ZhaoYang
showToc: true
TocOpen: true
draft: true
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

### Abstract

我们推出了Qwen-Image，它是Qwen系列中的一款图像生成基础模型，在复杂文本渲染和精准图像编辑方面取得了显著进展。

为应对复杂文本渲染的挑战，我们设计了一套全面的数据处理流程，涵盖了大规模数据收集、筛选、标注、合成与平衡。此外，我们采用了一种渐进式训练策略，该策略从非文本到文本的渲染开始，由简至繁地处理文本输入，并逐步扩展至段落级别的描述。这种课程学习（curriculum learning）方法极大地增强了模型原生的文本渲染能力。因此，Qwen-Image不仅在英语等字母语言上表现出色，在更具挑战性的中文等象形文字（logographic languages）上也取得了卓越的进展。

为了提升图像编辑的一致性，我们引入了一种改进的多任务训练范式。该范式不仅包含了传统的文本到图像（T2I）和文本-图像到图像（TI2I）任务，还加入了图像到图像（I2I）的重建任务，从而有效地对齐了Qwen2.5-VL和MMDiT之间的潜在表征（latent representations）。此外，我们将原始图像分别输入到Qwen2.5-VL和VAE编码器中，以分别获取语义表征和重建表征。这种双重编码机制使得编辑模块能够在保持语义一致性与维持视觉保真度之间达到平衡。

我们在多个公开基准测试上对Qwen-Image进行了全面评估，包括用于通用图像生成的GenEval、DPG和OneIG-Bench，以及用于图像编辑的GEdit、ImgEdit和GSO。Qwen-Image取得了业界顶尖（state-of-the-art）的性能，展现了其在图像生成和编辑两方面的强大能力。此外，在LongText-Bench、ChineseWord和CVTG-2K上的测试结果表明，Qwen-Image在文本渲染方面表现卓越——尤其是在中文文本生成上——以显著优势超越了现有的顶尖模型。这凸显了Qwen-Image作为一个领先图像生成模型的独特定位：它既具备广泛的通用能力，又拥有卓越的文本渲染精度。
