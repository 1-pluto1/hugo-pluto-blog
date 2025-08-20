---
title: Detecting and Reducing the Factual Hallucinations of Large Language Models with Metamorphic Testing
date: 2022-09-15T11:30:03+00:00
tags:
  - 深度学习
  - AI4SE
  - LLM
  - SoftwareEngineering
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

问答（QA）是大型语言模型（LLMs）的一项基本任务，它要求LLM能用自然语言自动回答人类提出的问题。然而，众所周知，LLM在处理问答任务时会扭曲事实并做出非事实性陈述（即**幻觉**），这可能会影响LLM在现实生活场景中的部署。在这项工作中，我们提出了 **DrHall**，一个使用**蜕变测试（MT）\*来检测和减少黑盒LLM事实性幻觉的框架。我们认为，产生幻觉的答案是不稳定**的。因此，当LLM产生幻觉时，如果我们使用蜕变测试让LLM通过不同的执行路径重新执行同一任务，它们更有可能产生不同的答案，这激发了DrHall的设计。DrHall的有效性在三个数据集上进行了实证评估，包括一个自建的自然语言问题数据集 **FactHalluQA**，以及两个编程问题数据集：**Refactory** 和 **LeetCode**。评估结果证实，DrHall能够持续优于当前最先进的基线方法，在**幻觉检测**方面获得了超过 **0.856** 的平均**F1分数**。在**幻觉纠正**方面，DrHall同样能优于最先进的基线方法，平均**幻觉纠正率**超过 **53%**。我们希望我们的工作能够增强LLM的**可靠性**，并为LLM**幻觉缓解**的研究提供新的见解。

### Introduction

