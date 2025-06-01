---
title: SciQAG：A Framework for Auto-Generated Science Question Answering Dataset with Fine-grained Evaluation
date: 2025-03-12T11:30:03+00:00
tags:
  - LLM
  - NLP
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


地址：https://arxiv.org/abs/2405.09939

代码：https://github.com/MasterAI-EAM/SciQAG/

机构：GreenDynamics Pty. Ltd

  

### abstract

使用问答（QA）对来训练和评估大型语言模型（LLM）引起了相当大的关注。然而，很少有可用的 QA 数据集是基于科学文献中的知识。在这里，我们通过提出自动生成科学问题答案（SciQAG）来弥补这一差距，这是一个自动生成和评估来自已发表的科学文献的科学问答对的框架。我们对开源 LLM 进行微调，从全文科学论文中生成960000个科学 QA 对，并提出一个五维指标来评估生成的 QA 对的质量。我们通过基于 LLM 的评估表明，生成的 QA 对在五个维度上始终达到 2.5 分（满分 3 分）的平均分，这表明我们的框架可以大规模地将论文中的关键知识提炼成高质量的 QA 对。我们公开数据集、模型和评估代码。

### 研究问题

本文研究的核心问题是: 如何基于科学文献自动生成高质量的问答对。本文研究问题的特点和现有方法面临的挑战主要体现在以下几个方面:

- 大多数现有的问答数据集都是基于通用领域文本或者维基百科生成的,缺乏专门针对科学文献的数据集。科学文献通常包含大量领域特定的专有名词、化学方程式、数学公式等,为问答对的生成带来了额外的挑战。
    
- 大部分现有的自动问答生成方法需要预先给定答案,然后根据答案生成相应的问题。但是对于科学文献来说,很难预先确定每一个潜在的答案,这使得这类方法在科技文献场景下的应用受到限制。
    
- 传统的基于规则或模板的问答生成方法缺乏灵活性,难以应对科学文献内容的多样性和复杂性。需要一种能够理解上下文、抓取关键信息的生成模型。
    

需要一种高效的评估方法,能够全面衡量生成问答对的质量,包括相关性、独立性、完整性、准确性和合理性等多个维度。

  

针对这些挑战,本文提出了一种基于大型语言模型的"SciQAG"框架:

SciQAG的核心思想是利用大型语言模型(LLM)强大的理解和生成能力,结合人工设计的提示,从科学论文中提取关键知识并生成问答对。它包含三个主要步骤:

- Seed QA生成: 使用GPT-4辅助专家设计合适的提示,从少量论文中生成种子问答数据,用于后续微调LLM。
    
- 科学QA生成: 基于种子数据,微调一个开源的LLM模型,作为生成器对大量论文进行问答对生成。
    
- QA评估: 使用另一个LLM模型对生成的问答对进行多维度评估,保证质量。
    

  

整个过程中,SciQAG灵活引入了多个选择性微调步骤,例如使用评估过滤器对生成器进行迭代改进等。它设计了一个新颖的五维评估指标RACAR,从相关性、独立性、完整性、准确性和合理性五个方面对生成的问答对进行综合评估。

  

  

### 研究方法

论文提出了一种名为SciQAG的框架,用于从科学文献中自动生成高质量的问答(QA)对。该框架由三个主要步骤组成:种子QA生成、科学QA生成和QA评估。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MzM0ZDE0YTgzMWYwYzI2YjExNmNkMGFlZDE2OTEyOWVfQmgyWXNGSmhjYTNiMWw4OUFXQkVhemxLV0txMmMwSVZfVG9rZW46TGdaYWJrRWFFbzdpU0t4Ukx4d2NONHVybm1iXzE3NDg3NTkwMTg6MTc0ODc2MjYxOF9WNA)

#### 种子QA生成

为了生成初始的高质量QA数据作为语言模型微调的基础,论文采用了 GPT-4 大型语言模型辅助的方式。具体过程如下:

- 论文收集并选取了123篇来自材料科学、化学、物理和能源等领域的文献作为种子论文集。
    
- 邀请了相关领域的专家,经过多次迭代设计了一个提示语句。该提示语句的目的是指导 GPT-4 从论文内容中提取关键知识点,生成独立于原文上下文的QA对。
    
- 将提示语句和种子论文的内容一并输入到 GPT-4 中,获得由 GPT-4 生成的初始QA数据集。
    

论文采用 GPT-4 的主要原因是其强大的理解和生成能力,能够基于提示语有效地从论文中提取知识。同时,GPT-4 对于领域术语、公式等内容也有较好的处理能力。

#### 科学QA生成

基于种子QA数据集,论文微调了一个开源的大型语言模型 Vicuna,将其用作生成器对更多论文生成QA对。具体步骤包括:

- 将种子QA数据转化为符合Vicuna输入格式的"instruction"形式数据。
    
- 使用改良的Longformer注意力机制对Vicuna模型进行微调,以支持较长的文本输入。
    
- 在微调过程中采用了数据增广的策略,使用微调后的中间模型生成部分新数据,与原始种子数据混合作为最终微调数据。
    
- 将微调完成的Vicuna模型应用于论文数据集,为96,000篇论文自动生成了960,000个QA对。
    

论文选择使用Vicuna是因为其开源、能处理较长序列、性能较好。同时通过微调的方式,使模型专门针对生成科学QA对进行了定制。

  

#### QA评估

为了评估生成的QA对的质量,论文设计了一个五维评估指标RACAR:

- Relevance - 评估问题与原论文内容的相关性
    
- Agnosticism - 评估问题是否独立于原论文的特定内容(如图表)
    
- Completeness - 评估答案是否全面完整地回答了问题
    
- Accuracy - 评估答案在事实层面上的准确性
    
- Reasonableness - 评估答案在逻辑层面上的合理性
    

具体评估过程是:使用GPT-4根据设计好的提示,对每个QA对在上述五个维度上给出1-3分的评价,并生成文字说明。

除了RACAR,论文还从以下几个方面对QA质量进行了评估:

- 问题的多样性 - 通过计算问题之间的语义相似度,确保生成的问题不是彼此的重复。
    
- 答案的覆盖率 - 计算答案所引用的句子在原文的分布,确保答案覆盖了文章的各个方面。
    
- 数值的来源 - 统计答案中数值出现在原文的比例,避免"虚构"数字。
    

通过自动化的评估方式,论文对96万个生成的QA对进行了质量把控。此外,人工专家评估的结果也表明,RACAR的评价与人工评价高度一致。

为了提高质量,论文还提出了一种迭代改进的方法:利用高分QA对对生成器进行进一步微调,使其产生更加优质的结果。

总的来说,SciQAG框架通过组合GPT-4辅助数据构建、语言模型微调和自动化评估等技术,实现了从科学文献中高效、自动化地提取知识并生成高质量独立QA对的目标。

  

### 实验

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=YjJiOTQ5ZWQzMjQzODJmMmY2N2FhZThmYjI5NzgwMjNfQ2F2RVFnV3c0cHpoZDhPeGFWVVpTWHkyWE9od3VlOXNfVG9rZW46VGxWS2JVR3BIbzRLZWt4TjdxWWNZcjFlbjNiXzE3NDg3NTkwMTg6MTc0ODc2MjYxOF9WNA)

  

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=NGUwYjgzNGRjMDZlYjQwZGU1NGE5MGRkN2Q2ODJmZjhfZmE4c1lycVM0ZXZqd2N6d3R0QkRLZEk4emNheDY5U2RfVG9rZW46T2ZmeGJTMkdpb3lDbEd4R2ZQWmNOSGhJbkdjXzE3NDg3NTkwMTg6MTc0ODc2MjYxOF9WNA)

  

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=YjMzYmFkOTA2MzkwYzdlMzkwMzFiYTdmNDBhMmQ5NzNfQlFwUjFDVjdmYklPQXVXSEhDNXhHYXI0STdxMUl5a2dfVG9rZW46VjlUSmJITFdnbzBwa2l4bTIyVWNyMWlIbkFPXzE3NDg3NTkwMTg6MTc0ODc2MjYxOF9WNA)

### 总结后记

本论文针对大规模科学问答数据集构建问题,提出了一种名为SciQAG的自动生成框架。该框架利用种子问答引导LLMs生成更多问答数据,并设计了一个五维度的RACAR指标用于自动评估生成的问答质量。实验结果表明,SciQAG生成的96万个科学问答对在RACAR指标上平均得分达到2.5分(满分3分),证明了该框架能大规模生成高质量的科学领域问答数据。


论文翻译：https://dppemvhuzp.feishu.cn/docx/BYR6d11x4of5pgxyolxcV0monVz?from=from_copylink