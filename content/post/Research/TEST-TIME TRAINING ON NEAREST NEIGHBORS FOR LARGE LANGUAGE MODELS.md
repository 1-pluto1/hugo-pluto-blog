---
title: TEST-TIME TRAINING ON NEAREST NEIGHBORS FOR LARGE LANGUAGE MODELS
date: 2025-03-12T11:30:03+00:00
tags:
  - LLM
  - NLP
  - ReasoningModel
categories:
  - AI
  - Research
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


TEST-TIME TRAINING ON NEAREST NEIGHBORS FOR LARGE LANGUAGE MODELS

ICLR 2024

最近的工作都聚焦于将检索到的数据添加到输入上下文中来增强具有检索能力的LLM，这种方式虽然能取得很好的效果，但是必须在训练和测试时添加检索到的数据。此外由于输入长度随着检索到的数据大小线性增长，Transformer的复杂度和计算成本急速上升。

为了避免这些复杂度，本文提出了：在测试时，使用标准的训练设置，对检索到的数据模型进行微调（对于每一个测试示例，从庞大的数据库中检索其最近的邻居，并在将其应用于测试实例之前对这些邻居进行微调模型）

Note：索引质量和索引大小很重要！！！

## Contributions:

1. 提出了一种基于大规模索引的最近邻测试时间训练系统。该分布式索引可以在标准硬件上以大约1秒的速度将每个最近邻查询服务到大约200w个向量和1TB的数据。
    
2. 在Pile基准测试的22个任务上进行评估，发现拥有20个邻居之后已经实现了大多数的增益（对训练期间没有见过的任务的网络带来的改进可能是显著的，对见过的任务有改进，但是改进的程度比较“温和”）
    

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDBhY2ZiMWIzNzE0ZDZmZWIwMjgzMGE2YWY0OTk5YTNfZmRXb1RLZG1OY2JDQXd1cnF0YlIzYjZWR0xETVdjS0NfVG9rZW46Q0N4dGJNb3NKb0RJaVR4RFhkUGM5OEJ5bnZkXzE3NDg3NTkzNzQ6MTc0ODc2Mjk3NF9WNA)

## 具体做法：

1. 从最近的邻居开始，以递增的距离顺序对每个邻居进行训练。从最远的到最近的顺序反而会导致模型的性能要差的多，一种解释是，最近的邻居提供了最优质的信息，通过在微调的早期阶段利用这些信息，可以更好的引导整个微调过程。
    
2. 作者还尝试了批量微调方法，将所有的最近邻数据放在一个批次中，然后对整个批次反复进行更新，但是实际上却没有带来什么 显著的性能提升。
    

  

## 未来工作：

1. 是否有自适应的、动态的顺序调整方法？
    
2. TTT显著提升了代码生成的任务，未来可以考虑在该方向上的进一步拓展
    
3. 缓解数据源对少数群体的偏差（这块放到cv可能会更好？比如黑人的人脸识别准确率不高）
    
4. 通过测试时叠加来自可信数据源的数据，帮助环节对抗性行为，包括数据投毒攻击。