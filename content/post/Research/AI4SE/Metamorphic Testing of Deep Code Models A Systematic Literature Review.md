---
title: "Metamorphic Testing of Deep Code Models: A Systematic Literature Review"
date: 2022-09-15T11:30:03+00:00
tags:
  - AI4SE
  - 深度学习
  - LLM
  - Survey
categories:
  - AI
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

专为代码智能设计的大型语言模型和深度学习模型，因其能够执行各种与代码相关的任务，已经给软件工程领域带来了革命性的变革。在代码补全、缺陷检测和代码摘要等任务中，这些模型能够高精度地处理源代码和软件工件；因此，它们有潜力成为现代软件工程实践中不可或缺的一部分。

尽管具备这些能力，但**鲁棒性**（robustness）对于深度代码模型而言，仍然是一项关键的质量属性，因为它们在多变和对抗性的条件下（例如，对变量进行重命名）可能会产生不同的结果。**蜕变测试**（Metamorphic testing）已成为一种广泛用于评估模型鲁棒性的方法，其具体做法是：对输入程序应用**“保义转换”**（semantic-preserving transformations），并分析模型输出的稳定性。

尽管已有研究探索了深度学习模型的测试方法，但这篇系统性文献综述专门聚焦于针对深度代码模型的蜕变测试。通过对45篇主要研究论文的学习，我们分析了其中用于评估鲁棒性的（代码）转换、技术和评估方法。我们的综述总结了当前的研究现状，明确了常被评估的模型、编程任务、数据集、目标语言和评估指标，并强调了推动该领域发展的关键挑战与未来方向。

### INTRODUCTION
