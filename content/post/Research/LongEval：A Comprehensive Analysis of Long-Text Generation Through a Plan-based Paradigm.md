---
title: "LongEval: A Comprehensive Analysis of Long-Text Generation Through a Plan-based Paradigm"
date: 2025-03-25T11:30:03+00:00
tags:
  - LLM
  - NLP
  - Benchmark
  - Long-Context
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


### Abstract

大型语言模型（LLMs）在各类自然语言处理任务中已取得显著成就，但其长文本生成能力仍缺乏深入理解和系统评估。本研究表明，现有LLMs在长文本生成中普遍面临文本长度要求与信息密度的双重挑战，且性能随文本增长呈衰减趋势。为量化定位此类性能衰减现象并为模型开发提供新见解，我们提出LongEval基准测试——该评估体系通过直接生成与规划生成双范式（受认知与语言学写作模型启发）系统评估长文本生成能力。综合实验揭示了若干重要发现：虽然模型规模与生成能力存在相关性，但经过长文本专项训练的小规模模型（如LongWriter）可达到相当性能水平。所有代码与数据集均已发布于https://github.com/Wusiwei0410/LongEval。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=NzgwOThmMWIzMTNlYzUyMTQwNzY2ZmY4OWY0ZjhjODFfUFpZTFdFbEtqalc0NWxnSjJtZGd0b053Rks4WXdVMThfVG9rZW46T1VWOGJ5N2R1b05hMUp4d01US2NIVVlHbktnXzE3NDg3NTg3OTI6MTc0ODc2MjM5Ml9WNA)

### Introduction

大型语言模型（LLMs）在自然语言处理（NLP）领域引发了革命性变革，在对话生成（Abdullin等，2024）、故事创作（Zhao等，2023）、开放式文本生成（Zhou等，2024）以及复杂推理任务（Zhang等，2023；Wu等，2024）等广泛生成任务中取得了显著成就。尽管LLMs在现实世界应用中的部署日益增多，但其处理长文档生成的能力仍未被充分探索，尽管这一能力具有重要意义。

尽管近期已有研究致力于提升长文本生成能力（Bai等，2024；Que等，2024）和长上下文理解能力（Xu等，2023；Li等，2023a，2024a），长文本生成能力的系统评估在当前研究中存在显著空白。

现有多数基准测试仅聚焦于长上下文检索与理解任务（Bai等，2024；Zhang等，2024b；Pham等，2024a；Quan等，2024；Tang等，2024；An等，2024）。近期并行研究HelloBench（Que等，2024）尝试通过从现有任务（如开放式问答）中选取样本来评估长文本生成能力，但这些任务本身并不固有需要长文本生成能力。

为全面探究大型语言模型（LLMs）的长文本生成能力，我们首先收集了一组内容丰富且篇幅较长的文档，并选取当前主流LLMs，要求其根据给定摘要直接生成完整文档。如图1所示，文档信息量与文本长度呈正相关，这一发现凸显了长文本生成能力的必要性。此外研究发现，当前主流大语言模型（参数量1B至70B）在信息密度与文本长度两个维度上仍与理想参照存在显著差距。我们尝试通过指定生成长度来验证模型能否输出符合要求的长文本，但未能成功。如图2所示，随着目标长度的增加，模型遵循指定长度的能力呈现下降趋势，当文本超过1000词时，这种性能衰减现象尤为显著。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=ODZiOWNhNzZhNTE2MzhkNjA2ODg4ZWZjY2JhMDE3ZmZfeDlxbGhpQ3FkSzlwcDdCYzVvZDF5OEh5TFUyQWZqSmdfVG9rZW46SWpiMGJhNGJ4b0I2VGt4bnVTN2M2dU00bjBnXzE3NDg3NTg3OTI6MTc0ODc2MjM5Ml9WNA)

受认知写作理论启发——该理论认为有效写作产生于通过计划、转化和审阅来"烹煮长期记忆中的知识"的过程（Flower与Hayes，1981）——我们推测当前大语言模型的生成范式可能与人类长文档写作实践存在偏差：相比基于计划的写作，大语言模型在单次长文本生成中往往难以保持内容连贯性与深度见解。具体而言，规划阶段对构建逻辑论证和结构化思维具有关键奠基作用（Scardamalia与Bereiter，1987），但现有研究大多忽视了文本生成的这一维度。

为解决上述局限性，本研究提出LongEval评估框架——一个通过支持直接生成与规划生成双路径来系统评估大语言模型长文本生成能力的基准体系。该框架包含两大创新：1）双重评估范式，同时考察零样本直接生成与更符合人类写作习惯的规划式结构化生成；2）聚焦内容质量、结构连贯性与信息密度的自动化评估指标体系。

基于科学文本与科普文章普遍遵循规范写作结构的特点，我们选取arXiv论文、技术博客和维基百科条目三类需生成超2000词长文本的领域构建评估基准。相较于同类研究HelloBench（Que等，2024，含300个通用任务样本）和LongWriter（Bai等，2024，含120个合成评估样本），本研究采集了166份源自真实长文本生成领域的人类撰写样本。我们设计了基于开源大模型Qwen2.5-72BInstruct的数据处理流程，对符合许可协议的跨领域文档进行结构化处理：先将各章节提炼为包含4-5句核心阐述的规划纲要，再经人工校验确保内容完整性。

在基于规划路径的评估中，模型需根据内容规划纲要逐章节生成全文，同时保持与已生成章节的语义连贯性。该方法系统评估大语言模型长文本生成能力的同时，保持了各章节直接生成的范式一致性。我们设计了八个维度的评估指标：i) 通用文档级指标（验证模型指令遵循与内容合理性）：内容遵循度(Cont-fol)、冗余度(Red)、长度符合度(Len)和一致性(Con)；ii) arXiv科研论文领域的专项指标：引言(Intro)、相关工作(RW)、方法(ME)和实验分析(EA)章节质量评估。

### Related Work

长文本生成研究

近期长文本生成研究主要聚焦于模型性能提升（Pham等，2024a；Zhang等，2024b；Bai等，2024；Quan等，2024；Tang等，2024）。主流方法包括构建针对长文本生成的大规模指令跟随数据集，并采用多种优化策略增强大语言模型能力。除直接模型训练外，基于规划的生成方法逐渐兴起。LongWriter（Bai等，2024）证明通过GPT-4o结构化规划生成的合成数据集可有效提升模型长文本生产能力；Wang等（2024）提出分章节生成综述论文的框架，Lu等（2024）则采用类似策略生成完整科研论文。这些研究表明结构化生成方法能改善长文本输出的连贯性与可控性。

长上下文理解

长文本生成的核心挑战在于确保模型有效理解并利用长上下文。该领域研究通过扩展输入长度与强化上下文学习能力（Chen等，2023；Jiang等，2023；Li等，2023b；Jin等，2024；Zhang等，2024a；Ding等，2024），主要面向阅读理解等任务——例如LongICLBench（Li等，2024a）、∞BENCH（Zhang等，2024d）和LonGLE（Li等，2023a）等基准测试中，模型需从长输入中提取关键信息。然而既有研究多聚焦于检索或摘要任务，对生成连贯且上下文一致的长文本这一挑战关注不足。

长文本评估

长文本评估仍是开放难题。HelloBench（Que等，2024）通过选取通用任务的长文本样本并采用直接生成法进行评估。现有评估框架多依赖基于大语言模型的评分，但其鲁棒性存疑。Zhang等（2024c）另辟蹊径提出专用于长文本评估的奖励模型。数据集建设方面，Suri（Pham等，2024b）采用规划生成与回译技术（Li等，2024b；Köksal等，2024）构建教学文本数据集，但侧重创意写作而非学术内容；Köksal等（2024）则基于维基百科和CommonCrawl构建长文本数据集，强调直接生成而非结构化规划。这些研究凸显了需要兼顾规划生成与直接生成方法的高质量数据集与评估指标，特别是在需要结构化长文本输出的领域。

### The LongEval Benchmark

为填补长文档生成评估领域的空白，本文提出LongEval基准测试——一个基于统一框架构建的长文本生成评估体系，并引入多维综合评价系统。如表1所示，相较于同类研究，LongEval在数据采集、生成范式、领域特异性指标和层级化度量等维度均建立了差异化评估体系。本节首先阐述长文本生成范式的统一理论框架，继而说明据此设计的评估系统。

#### Long Text Generation Paradigms

认知写作理论强调规划在人类写作中的核心作用（Flower与Hayes，1981），基于规划的范式已有效应用于大语言模型训练所需的长文本合成数据生成（Bai等，2024）。因此，分段生成超长文本成为当前主流范式（Wang等，2024；Bai等，2024）。基于此，本文采用直接生成与规划生成两种方法进行长文本生成评估。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=YTEzMWQwMTc3MDhiNDg5Y2I2N2NiZmM2YWUyNGU5YjZfWnV4dXo0Z00xUnByY2NiQkh6TGN2TFhMbTQ5R3l0SnVfVG9rZW46WFdZYmJDMm95b3UzUXV4NkxQSWNzQW81bjVkXzE3NDg3NTg3OTI6MTc0ODc2MjM5Ml9WNA)

直接生成

尽管直接生成方法适用于多数自然语言处理任务，但如图2所示，当前大多数大语言模型无法直接生成超过1000词的长文本。本研究同时评估了大语言模型的端到端长文本生成能力：通过向模型输入章节内容规划p、目标文本长度l及其他辅助写作材料（如实验结果exp、参考文献ref等）进行直接生成实验。

规划生成

基于规划的方法因性能优于直接生成（Bai等，2024；Lu等，2024）被广泛应用于长文本生成。实验重点分析了大语言模型的长度遵循能力，并通过跨领域内容生成深入探究模型局限。如图1所示，我们以人类撰写文本为基线，定量分析了文本长度与信息含量的关系。基于图2的实证结果，我们认为当前大语言模型尚无法满足高信息量文本的生成需求。为此设计了一种统一的规划生成方法：通过分章节生成确保输出符合长度要求——对每个样本输入章节规划p与长度要求l，使模型分节生成完整文章。针对特定领域需求进行定制化处理（如arXiv论文领域额外输入实验结果生成分析章节，维基百科条目则输入参考文献确保内容真实性）。该方法详见附录B。

#### Evaluation System and Prompts

先前研究主要聚焦于大语言模型的长上下文理解能力（Xu等，2023；Li等，2024a；Jin等，2024；Zhang等，2024d）。这些任务大多类似于具有标准答案的阅读理解任务（例如根据长上下文回答"杰克多少岁"等问题）。虽然HelloBench（Que等，2024）已评估过大语言模型的长文本生成能力，但其评估指标未考虑超长文本生成特性（如超长文本生成中的指令遵循能力）。本研究创新性地在文档级和章节级双重维度上评估长文生成质量。

##### Domain-Agnostic Document-level Metrics

  

  

内容遵循度（Cont-fol）评分

长文本生成的输入包含完整文章写作纲要（即4.2节生成的内容规划）。模型生成文本是否遵循纲要要求是评估生成质量的关键指标。如附录A图4所示，我们设计专用提示模板，将模型生成的各章节文本与对应提示共同输入，以评估模型在长文本生成中的指令遵循能力。

长度符合度（Len）评分

各章节长度评分计算公式如下：

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=N2VlNWI2NDA4YjI2YjUyZjk3YjNlMzJkNDgyM2Q5MDFfY0M2T2d6VTJ3TG94M01LWnlBOWM5cjBrMkREOVNuWnNfVG9rZW46UkxDSmI2amtlb0lpZ094SkFGRmMyTjU0bmNjXzE3NDg3NTg3OTI6MTc0ODc2MjM5Ml9WNA)

其中lgen生成代表生成文本长度，lreq要求代表提示中的长度要求。章节级指标最终得分为所有章节得分的算术平均值。

冗余度（Red）评分

长文本生成时，大语言模型易将各章节视为独立单元，导致跨章节内容重复。如图4所示，我们专门设计提示模板来评估模型生成内容是否存在冗余成分。

一致性（Con）评分

长文本写作需确保章节与段落间的衔接。针对模型生成文本，如附录A图4所示，我们通过设计提示模板评估其内容一致性。

##### Domain-Specific Section-Level Metrics

针对某些格式规范性较强的特定领域，我们为arXiv科研论文和维基百科条目设计了兼顾预设结构的专项评估指标。

**引言与相关工作评分**

基于提供的详细写作纲要及相关文献，如图4（附录A）所示，我们设计提示模板评估arXiv论文引言与相关工作章节。以原文为黄金标准，采用大语言模型评估生成文本与标准答案的相似度。技术博客写作无需包含参考文献，而维基百科条目虽无独立相关工作章节，但需通篇引用确保内容真实性，故将其整体视为相关工作章节进行评估。

**实验分析评分**

在科研论文领域，我们发现当前大语言模型难以判断实验结果的适用场景（例如会在方法章节误用实验结果），且倾向于复述实验要点而缺乏对结果成因的深入分析。为此，如图4（附录A）所示，我们设计评估提示模板对比原文与生成文本的实验分析章节。

**方法描述评分**

大语言模型生成的方法描述常存在表述模糊、缺乏详细设计方案或公式化说明的问题。针对此现象，如图4（附录A）所示，我们专门设计提示模板对比原文与生成模型的方法章节。

### Dataset Curation

在先前的研究中（Que等，2024），构建长文本生成评估数据集的一种方法是从对话延续等现有任务中筛选长文本。这些任务通常不涉及长文本写作需求，因此难以真实评估模型的长文本生成能力。长文本内容在学术论文、技术博客和维基百科条目等多个领域普遍存在。为此，我们选取这三个领域的自然长文本数据构建基准测试，以评估模型在真实长文本场景下的生成能力。

#### Data Collection Pipeline

我们设计了一个自动化流程，用于从无版权限制的网页中收集文档，并根据预定义的规则将其拆分为不同的部分。我们从arxiv.org收集论文，从wikipedia.org收集文章，从HuggingFace收集博客。这些来源均具有允许的版权许可。为了确保基准测试的质量，我们聘请了一位熟悉自然语言处理（NLP）的研究生，对处理后的数据进行人工检查。具体来说，我们会删除不符合预定义格式的样本（例如，没有摘要的论文或缺少介绍的博客）。

#### Content Plan Generation

为支持第3.1节介绍的基于规划的长文本生成方法，我们采用Qwen2.5-72B-Instruct模型生成内容规划方案。具体而言，我们将文档的每个章节输入模型，并通过精心设计的提示词使模型将各章节内容概括为4-5个句子，从而形成该章节的内容规划框架。

##### Human Evaluation of Generated Content Plans

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=ZGM4ZTk4OWVmYWJhZDI2ZWE2M2M5ZWEwMjQ5OTU1M2ZfdVhXb0xJY0w2UnNjaHlRVVl5bDZRVDZYMGN0eUlhcWJfVG9rZW46QmFZMmJuenVBb2NqMTN4b3JzcWNRelU1bmllXzE3NDg3NTg3OTI6MTc0ODc2MjM5Ml9WNA)

  

  

为了评估内容规划方案是否保留了文档的关键信息，我们聘请了一位研究生对每个领域的10%文档进行人工评估。具体而言，如果某个章节的内容规划方案未能捕捉到足够的相关信息，我们将其视为不合格样本。如表3所示，在Wikipedia、Blog和arXiv上，我们的人工评估准确率分别为88.6%、91.4%和86.2%。平均而言，88.7%的人工审核内容规划方案包含了足够的信息，这表明这些内容规划方案保留了足够的信息，使大语言模型能够忠实（重新）生成原始文档的内容。

#### Dataset Characteristics

如表2所示，我们对三个领域中原始样本的平均长度（真实文本长度）与生成的内容规划方案进行了对比分析。学术论文领域的内容规划方案长度居首，维基百科条目次之，博客内容最短。这一分布规律与各领域的固有写作复杂度高度吻合：学术论文要求严谨的论述框架，维基百科侧重科普性阐述，而博客则采用更为随性的行文风格。研究结果表明，不同领域的写作复杂度与其文本长度存在显著相关性。

本数据集在每个领域均保留约50个样本，所有原始文本（真实文本）均超过2000词。为评估内容规划方案的效率，我们引入了信息压缩率（ICR）指标，其计算公式为ICR = L真实文本/L输入文本，其中真实文本指原始长文本，输入文本指用作大语言模型输入的摘要式内容规划方案。所有领域的ICR值稳定维持在20%-30%区间，这表明我们的内容规划方案在保留核心内容的同时，能够有效避免向模型泄露过多细节信息。

### Experiments and Result Analysis

#### Baseline

我们采用了一系列开源大语言模型（LLMs），包括Llama3（Llama3.2-1B、Llama3.2-3B、Llama3.3-70B）（AI@Meta, 2024）、Qwen2.5（3B、7B、72B）（Yang等, 2024b,a）以及擅长数学推理的InternLM2.5（Cai等, 2024）。此外，我们还引入了LongWriter，这是一个针对长文本生成进行微调的GLM模型（Bai等, 2024），以及GPT-4o，这是一个在多任务中表现均衡的专有模型。

#### Overall Analysis

表4展示了各模型在arXiv、博客和维基百科任务中的实验结果。Qwen2.5系列模型展现出卓越的长文本生成能力，其中Qwen2.5-72BInstruct在arXiv领域获得了82分的最高综合评分，在博客领域获得了83分的最高综合评分。随后是GPT-4o和LongWriter-8B的表现。我们观察到一种一致的趋势，即同一系列中较大规模的模型始终优于较小模型，这凸显了模型规模在长文本生成中的优势。

  

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MzA3YTBjYTJlN2Q5YTNjNjMyZDk3ODZkOTQ3OGE1OGZfSjh2WWpUNVdsZDQ4MjhleUtpWGprdGEzcUZhcjA0VDNfVG9rZW46VDR6QWJFU1FHb0pobnN4VGZTVGM0dzNQblRoXzE3NDg3NTg3OTI6MTc0ODc2MjM5Ml9WNA)

在各项评估指标中，内容遵循度（Cont-fol）和冗余控制（Red）表现出最显著的性能差异。例如在arXiv领域，Qwen2.5-72B-Instruct的内容遵循度得分为88，而InternLM2.5-7B-Chat等较小模型仅获得68分。维基百科领域中，LongWriter8B达到85分，InternLM2.5-7B-Chat则落后至69分。这些结果表明，指令遵循和冗余控制仍是长文本生成的主要挑战。相比之下，可读性（RW）、引言质量（Intro）和长度控制（Len）的模型间差异较小——arXiv领域各模型的RW评分集中在75-80区间，Len评分普遍维持在92-98区间。然而，方法论阐述（ME）和实验分析（EA）指标呈现较大波动，其中Qwen2.5-72B-Instruct在arXiv领域的ME得分为79，而InternLM2-5.7B-Chat仅为65。这表明：虽然基础写作能力在不同模型间相对稳定，但涉及数据分析与实验方法的高阶任务仍具挑战性。即使提供结构化写作指导（如内容规划方案），模型在高层级推理任务中仍存在局限，需要更先进的分析能力才能取得理想表现。

#### Long Text Generation Under Different Paradigm

如表6所示，我们对比了大语言模型在直接生成和基于规划生成两种设置下的长文本生成能力。值得注意的是，基于规划方法生成的文本整体评分显著高于直接生成方法。此外，我们发现直接生成方法产生的文本不仅相对较短，而且存在较高程度的冗余。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MjU0MzE5OGJhNWFlYTJlYzU0MzBjN2E1Yjk3NWFhNjBfR2hnN25UNFBRYzk1elpOR295QzBHNGJYa25PVzcyeHdfVG9rZW46Vmt6aWJhZE9Yb1RkMjR4d0Q0WmNCQkhRbmZlXzE3NDg3NTg3OTI6MTc0ODc2MjM5Ml9WNA)

这进一步验证了我们设计的基于规划生成方法的有效性，且该方法更适用于长文本生成场景。

#### Effectiveness of LLM-As-A-Judge

为验证大语言模型作为评估者（LLM-as-a-judge）在本文指标上的有效性，我们在arXiv任务中设计了随机替换测试：将模型生成内容中p%的章节随机替换为其他模型生成的文本片段，检验评估模型能否识别质量退化并在评分中体现。实验采用Qwen-2.5-72B的生成结果，替换比例p从0.1至0.9递增。如表5所示，在指令遵循度（Cont-fol）指标上，随着随机替换比例上升，模型评分呈现断崖式下跌（从88%降至52%）。其他评估特定章节质量的指标（RW、Intro、EA、ME）也整体呈现随替换比例增加而下降的趋势，证明评分模型能有效识别内容质量变化并反映内容规划要求。至于文本长度（Len）和冗余度（Red）评分，由于二者评估的是文本自身写作特征（而非内容与指令的相关性），其分数在p值增长时未出现显著波动，体现了指标的稳健性。

此外，我们框架中引入GPT-4o作为对比评估模型（表7），虽然GPT-4o与Qwen2.5-72B在部分指标上存在评分差异，但不同模型间的评分分布趋势保持一致，证实Qwen2.5-72B在本框架下同样具备有效评估大语言模型长文本生成能力的功能。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=YTFhYmVlZTZjNjJjZGIwMjE0ZjU1OGY5NmQyZmY0MWFfQmhMUVRUUFFaUnZOeGo4VEV3MzN2T2g2ZklOanU2RzNfVG9rZW46TmlEOGJubUo5b0FXQXd4Mmh2aGNPZlN0blJkXzE3NDg3NTg3OTI6MTc0ODc2MjM5Ml9WNA)

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=OWNmMzFiNmYzYWFkNTU5YmE5OTViNTQzNWQ3YzczOTVfeU9jcDY4a3RKYXJxSkEzd1JMN0d3dUk0ZXZ4Zlh5MkZfVG9rZW46R1pkS2JPR28xb0ZqT1h4NnFUUGN2ZHpYbk1kXzE3NDg3NTg3OTI6MTc0ODc2MjM5Ml9WNA)

#### The Length Following Ability of LLMs

为评估大语言模型生成指定长度文本的能力，我们直接要求模型生成特定长度的文本，并比较目标长度与实际长度的差异（即Len指标）。如图2所示，我们的实验要求大语言模型生成100至32,000词不同长度的文本。当需求长度（len_req）低于400词时，大多数模型Len得分可达1；但随着len_req增加，所有模型的Len得分均急剧下降。当len_req超过4,000词时，多数模型得分低于0.4，表明当前大语言模型难以精确控制生成长文本的篇幅。值得注意的是，Qwen2.5和Llama3表现优于其他模型，且更大规模的模型展现出更强的长度控制能力。

### Conclusion

现有长文本评估方法存在两大局限：一是忽视了长文本生成范式的特殊性，二是缺乏高质量样本（例如需要人类撰写的长文本生成任务样本，如论文写作）。本研究设计了LongEval基准测试，收集了156个涵盖三大领域的长文本样本，这些领域均需要大语言模型具备长文本写作能力。通过对主流大语言模型的实验验证，我们证明了基于规划的长文本生成方法显著优于直接生成方法。此外，尽管大语言模型展现出相对较好的内容遵循能力，但在高层级推理写作任务（如实验分析撰写与方法设计）中仍存在明显不足。

### Limitations

尽管实验结果具有显著性，但由于资源和时间限制，我们仅在arXiv领域下测试了这些模型在直接生成设置中的性能，以与基于规划范式进行对比。在未来的研究中，应考虑采用相同的数据处理流程扩展基准测试，以实现更稳健的评估。

### Ethics Statement

本研究采用公开数据源构建数据集，确保不涉及隐私问题或违规行为。所有数据均未收集个人身份信息，且严格遵循法律与伦理标准获取。在数据标注阶段，我们聘请了三位具有自然语言处理研究经验的研究生参与标注工作，按每小时约13美元的标准支付报酬（显著高于当地平均工资），并在标注人员对流程存在疑虑时开展建设性讨论。


论文翻译：https://dppemvhuzp.feishu.cn/docx/JQ0edIfKBoadmqxQ625cuUQnnPc?from=from_copylink