---
title: UNIFORK EXPLORING MODALITY ALIGNMENT FOR  UNIFIED MULTIMODAL UNDERSTANDING AND GENERATION
date: 2022-09-15T11:30:03+00:00
tags:
  - MLLM
  - LLM
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

在多模态人工智能领域，统一的图像理解与生成已成为一种前景广阔的新范式。尽管近期取得了一些进展，但这类统一模型的最佳架构设计仍是一个开放性挑战。

在这项工作中，我们首先分析了用于理解和生成的任务专属专家模型以及当前统一模型的模态对齐行为。我们的分析揭示了一个关键的观察结果：**理解任务**受益于随网络深度逐渐增强的模态对齐，这有助于构建语义信息以实现更好的理解；相反，**生成任务**遵循一种不同的趋势——模态对齐在浅层网络中增强，但在深层网络中减弱，以恢复空间细节。

这些存在差异的对齐模式在完全共享的 Transformer 主干网络中造成了根本性的冲突，其中统一的表示流通常会导致两种任务之间的性能折衷。

受此发现启发，我们引入了 **UniFork**，这是一种新颖的 **Y 形架构**，它共享浅层网络以进行跨任务表示学习，同时在深层网络中采用任务专属的分支以避免任务间的干扰。这种设计有效地平衡了共享学习和任务专业化。通过广泛的消融实验，我们证明了 Unifork 的性能持续优于传统的完全共享 Transformer 架构，并且其性能达到或超过了任务专属模型。

我们的代码已在 https://github.com/tliby/UniFork 开源。



### INTRODUCTION

