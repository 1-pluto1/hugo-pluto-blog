---
title: "Sky-T1: Train your own O1 preview model within $450"
date: 2025-02-16T11:30:03+00:00
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
原博客地址：https://novasky-ai.github.io/posts/sky-t1/

代码：https://github.com/NovaSky-AI/SkyThought

费用： less than $450

组织： the NovaSky team at UC Berkeley

基座模型： Qwen2.5-32B-Instruct

  

  

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=YzlkMWU0ZDY5ZmEwMTU4MmY5M2JkNDQxMzIwZTkyOWZfYzZERkpTOXBzakFCUkxqN20wVTc2d25Rd0lVOUVIbGhfVG9rZW46VVJ6VWJSZkJVb3ptWUV4WTdsaGNlZU1tbkplXzE3NDg3NTkwODg6MTc0ODc2MjY4OF9WNA)

  

##### 模型训练：

**训练了3个epoch，学习率1e-5，batch_size : 96，在八卡H100上使用 DeepSpeed Zero-3 offload 训练了19小时，训练框架为Llama-Factory。**

The model is trained with 3 epochs, learning rate 1e-5 and batch size 96. The model training finishes in 19 hours on 8 H100 with DeepSpeed Zero-3 offload (~ $450 according to Lambda Cloud pricing). We use Llama-Factory to perform training.

##### 模型测评

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MTljZWMxNWQxMjkwZTk5YTg0ODY3YmNhMmRhOGRiZWVfRGZLdFdRT2xPdXN4Y2U2cHVXd0hGT054Ym9JOXJ5VHJfVG9rZW46RE80WWI2ZXhmb0JZUVB4MVZLZ2NhdUdNbm9mXzE3NDg3NTkwODg6MTc0ODc2MjY4OF9WNA)

