---
title: Software Performance Engineering for  Foundation Model-Powered Software (FMware)
date: 2022-09-15T11:30:03+00:00
tags: 
categories: 
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

**摘要**——基础模型（FMs）如大语言模型（LLMs）的兴起，正在彻底改变软件开发 1。尽管出现了令人印象深刻的原型，但将FMware（基础模型驱动的软件）转变为可用于生产的产品需要跨越多个领域的复杂工程 2。一个关键但被忽视的方面是性能工程，其旨在确保FMware满足诸如吞吐量和延迟等性能目标，以避免用户不满和经济损失 3。通常，对性能的考量往往是事后才进行的，这导致了部署后高昂的优化工作 4。FMware对计算资源的高需求凸显了高效利用硬件的必要性 5。持续的性能工程对于防止性能下降至关重要 6。本文强调了软件性能工程（SPE）在FMware中的重要性，并识别出四个关键挑战：认知架构设计、通信协议、调优与优化以及部署 7。这些挑战基于文献调研和开发内部FMware系统的经验 8。我们为软件工程社区讨论了相关问题、当前实践和创新路径 9。

**索引术语**——基础模型、大语言模型、FMware、软件性能工程 10

### INTRODUCTION
