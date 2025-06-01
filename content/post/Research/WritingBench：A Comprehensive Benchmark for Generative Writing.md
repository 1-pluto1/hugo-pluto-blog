---
title: "WritingBench: A Comprehensive Benchmark for Generative Writing"
date: 2025-03-27T11:30:03+00:00
tags:
  - LLM
  - NLP
  - Benchmark
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

近年来，大型语言模型（LLMs）的进展显著提升了文本生成能力，然而评估其在生成性写作中的表现仍然是一个挑战。现有的基准测试主要集中于通用文本生成或有限的写作任务，未能捕捉到跨领域高质量写作内容的多样化需求。为了填补这一空白，我们提出了WritingBench，这是一个全面的基准测试，旨在评估LLMs在6个核心写作领域和100个子领域中的表现，涵盖创意、说服性、信息性和技术性写作。我们进一步提出了一种查询依赖的评估框架，使LLMs能够动态生成实例特定的评估标准。该框架辅以一个经过微调的批评模型，用于基于标准的评分，支持在风格、格式和长度方面的评估。该框架的有效性通过其数据整理能力得到进一步证明，使得7B参数模型能够接近最先进的（SOTA）性能。我们开源了该基准测试，以及评估工具和模块化框架组件，以推动LLMs在写作领域的发展。

  

### Introduction

  

近年来，LLMs（DeepSeek-AI等，2025；Hurst等，2024）在文本生成领域取得了革命性进展，展示了从生成创意内容（Mirowski等，2023；Lei等，2024）到辅助教育（Wang等，2024a；Li等，2024b）以及增强专业工作流程（Shao等，2024；Li等，2024a）等多样化应用中的卓越表现。然而，生成性写作需要高度的连贯性、创造力、逻辑推理和风格精确性，这对现有的评估方法提出了独特的挑战，目前的方法未能充分应对。

  

当前生成性写作的评估基准存在两个主要局限性：

1. **任务设计的范围有限且缺乏多样性**；
    
2. **对复杂写作任务的评估指标不足**。
    

首先，缺乏覆盖广泛写作任务的专门基准。大多数现有的写作导向基准局限于单一领域，例如小说（Karpinska等，2024；Gómez-Rodríguez和Williams，2023），且其任务设计往往过于简化——通常依赖于单句查询（Bai等，2024）或少量指令模板（Paech，2023；Que等，2024）。此外，许多基准使用同质化的输入材料（Que等，2024；Karpinska等，2024），限制了其适应现实写作场景中复杂和定制化需求的能力。因此，它们未能捕捉到实际写作任务的多样性和复杂性（见图1）。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjAyMTJkZTYyNzllMzY2MDg1NjMyZjU2NTk2YmNiMzdfQUMzM3JVV2RkVjRUREF3b0NvSUFJb1BFQmxCd05HemxfVG9rZW46UFJCWmJkUHo3bzRnaVF4ME0yNmNjbGJwblRnXzE3NDg3NTk0MTc6MTc0ODc2MzAxN19WNA)

  

为了解决这些挑战，我们提出了**WritingBench**，这是一个用于评估通用写作的综合基准和鲁棒框架。我们的方法始于基于现实写作需求的精心设计的二级领域分类。我们开发了一个四阶段的查询构建流程（如图2所示），其中LLM首先生成并多样化写作查询，随后进行人工驱动的材料收集和优化。这一过程确保了多样化的写作任务，涵盖广泛的领域、不同的需求以及异质材料的整合。为了实现更细致的评估，我们提出了一个依赖查询的评估框架，该框架利用LLM动态生成五个实例特定的标准，并通过微调的批评模型进行评分。最后，我们整合该框架以筛选写作特定数据，并训练一个小规模模型以验证其识别高质量写作样本的能力。

### Related Work

#### Writing Benchmarks

现有的评估基准在领域覆盖范围和任务粒度方面存在显著局限性。例如，EQ-Bench使用模板化查询涵盖故事相关任务（Paech, 2023），而LongBench-Write在120个查询中引入了长度限制（Bai等, 2024）；然而，它们都缺乏层次化的领域分类和多维度需求规范（例如风格和格式）。此外，当前大多数基准依赖于固定的指令模板、短上下文或主要来自单一来源的材料（Que等, 2024；Karpinska等, 2024；Marco等, 2024），这使得它们无法满足现实世界数据需求的复杂性。相比之下，我们提出的基准通过引入1,239个自由形式的查询填补了这些空白，这些查询分布在6个主要领域和100个子领域中，并具有对风格、格式和长度的潜在控制，同时输入范围从几十字到数千字不等。

#### Evaluation Methods

使用大型语言模型（LLMs）作为评估工具已成为评估生成响应质量的普遍方法。通常，研究人员会预先定义一组适用于所有测试实例的固定评估维度（Gómez-Rodríguez和Williams，2023；Marco等，2024）。例如，SuperCLUE（Xu等，2023）采用了三个维度，而LongBench-Write（Bai等，2024）则采用了六个维度（如相关性和准确性）。HelloBench（Que等，2024）引入了任务特定的维度，但这些维度在给定任务的所有查询中保持一致。尽管LLM作为评判者的方法增强了可扩展性，但静态评估维度往往无法适应写作风格和规范的多样性。为了解决这一局限性，最近的研究（Liang等，2024）训练了一个模型，以动态生成针对单个查询的评估维度。然而，这些维度仍然局限于一个小的预定义集合。相比之下，我们的依赖查询的评估框架利用LLMs生成多样化的实例特定标准，并通过微调专门的批评模型来执行评估。

#### Writing-Enhanced Models

尽管现有的LLM展示了卓越的写作能力，研究人员仍致力于进一步提升其整体写作水平。最近的模型，如Weaver（Wang等，2024b）和Suri（Pham等，2024），在特定领域表现出显著优势。例如，Weaver受益于超过2000亿参数的预训练，支持四个不同的写作领域，而LongWriter（Bai等，2024）则专注于长度限制任务。然而，这些模型在跨领域场景和多约束任务中表现出显著的性能下降。在本研究中，我们利用评估框架的有效性，引入了基于高质量数据训练的写作增强模型，在各种任务中实现了高性能。

### WritingBench

在本节中，我们将主要介绍WritingBench的构建过程，以及基于查询的评估框架及其配套的批评模型，用于标准感知评估。此外，我们训练了一个写作增强模型，以展示我们框架在数据筛选能力方面的有效性。

#### Benchmark Construction

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=OTFmYjhiOTJjZjJhNjU2Y2ZkMzFhMDg2Mjg4ODJmMTBfMXk2OFhpM1pUcHpnR2xlem5IaEpqc1h1ZW5Gd2hhYlNfVG9rZW46UmxRVGJwdk81b2JxdW14U3htS2NENlVSbmplXzE3NDg3NTk0MTc6MTc0ODc2MzAxN19WNA)

WritingBench 的构建遵循一个精细的流程（见图2），该流程结合了模型生成的查询优化和人工标注，确保了多样性和现实相关性。以下是两阶段的构建过程说明。

##### Model-Augmented Query Generation

该阶段利用大型语言模型（LLMs）生成初始的写作查询集，并通过系统性指导进行丰富和多样化，同时根据需要提供材料建议。

**Phase 1: Initial Query Generation**

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MWRjYjJlMjk5YTYzOTJmODUzNDM2MDk4NjIxYjg0MGRfeXpxWnVyM1k1ZzV1NEFKelVOekJvclJQc2pLODJZZ1JfVG9rZW46TzdGMGJjckFRb2pVSEp4cEkyYWM2VnVxbkRnXzE3NDg3NTk0MTc6MTc0ODc2MzAxN19WNA)

  

我们首先构建了一个基于现实写作场景的两级领域池，包括6个主要领域和100个次级子领域（详细描述见图3和附录A）。所选领域旨在捕捉传统和新兴的用户对AI辅助写作的需求，涵盖学术与工程、金融与商业、政治与法律、文学与艺术、教育、广告与营销等类别。利用领域和子领域标签，我们提示两个不同的LLM（ChatGPT-4o-latest和Claude3.5-Sonnet）生成初始的写作查询，以模拟现实中的用户请求，从而最大化多样性（见附录B.1）。

**Phase 2: Query Diversification**

为了提高查询的多样性和实际适用性，同时考虑到现实世界的需求，我们设计了一套查询多样化策略（见附录B.2），灵感来源于Xu等人（2024）的研究，包括三个核心要求和三个辅助维度：

- **风格调整**（例如，“使用友好且简单的语气，便于儿童理解。”）
    
- **格式规范**（例如，“遵循IEEE会议模板”）
    
- **长度限制**（例如，“生成一份500字的执行摘要”）
    
- **个性化**（例如，“从一位拥有二十年教育经验的教师的角度出发。”）
    
- **内容具体性**（例如，“详细说明2023年第三季度的财务指标”）
    
- **表达方式**（例如，将查询表达修改得更简短。）
    

前三个类别代表了典型的写作能力，我们将通过专门的评估进行测试。在优化查询的同时，LLM还会提供材料需求（例如，要求提供财务报告作为市场分析查询的输入）（见附录B.3）。这种方法最终生成了丰富的查询，并附带了相应的推荐参考资料。

##### Human-in-the-Loop Refinement

该阶段结合了人类专业知识，以验证模型生成的查询并补充材料需求，从而确保其与现实应用的一致性，并避免有害内容。

**Phase 1: Material Collection**

在此阶段，我们聘请了30名经过培训的标注员，每小时报酬为18美元，并通过领域知识测试证明了其专业能力。他们的任务是根据LLM生成的材料需求，收集必要的开源材料（例如，公开的财务报表或法律模板）。为了最大限度地减少因解析不同格式文档而产生的错误，标注员会仔细提取并验证最相关的文本片段。

**Phase 2: Expert Screening & Optimization**

接下来，五名具有LLM经验或行业背景的专家进行数据筛选。专家们执行了一个精细的两阶段过滤过程：（1）查询适配：模糊或不切实际的查询会被修改，以更好地与提供的材料和实际场景保持一致（例如，调整法律意见查询以引用所提供法规中的具体条款）；（2）材料修剪：从收集的材料中删除冗余或不相关的内容，确保写作任务提供的上下文保持聚焦和相关性，同时保证信息不包含有害内容。

最后，我们构建了WritingBench，这是一个包含1,239个查询的基准测试集，这些查询按照领域和任务类型进行了分类。与表1中总结的现有写作基准相比，WritingBench在实例数量、领域多样性、需求覆盖范围以及输入长度变化性方面表现出显著优势。WritingBench的详细统计分布如表2所示。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=ZGQ5ZTUzNjA5NGVkZGUxOGIyMzVlMzk3NGNmOTllNzNfbE41VVBEY2tEcFdnZVpyVlhUU3kxb21zenRCYUcweXZfVG9rZW46TVU2YWJ2Q1Zwb1dPWmh4ZGNSMWNJRW95blNiXzE3NDg3NTk0MTc6MTc0ODc2MzAxN19WNA)

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=NmQ4N2YyZTJiOWQ2OTE0YzNiZmE4ODBmY2VhOGFkNWJfVUMwSHIxNWJDRlZVWHZWb3dkajhHT2NWOFRMOThmbXZfVG9rZW46WGVmN2JzQ0x6b3hOVjN4YWVLTWNHZHhWbkViXzE3NDg3NTk0MTc6MTc0ODc2MzAxN19WNA)

#### Query-Dependent Evaluation Framework

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjZhYzg5YzZjN2MzZjcxZmQyZjRhYmNhZTlhMTdiMTZfM2YxeGJnTVY4Z1dxd2tuZHRHR3hZeU5XZkgyQVprZzZfVG9rZW46RTlxN2JYWUVBb0RjSDJ4VG85RmNNQlpMbjVlXzE3NDg3NTk0MTc6MTc0ODc2MzAxN19WNA)

传统的LLM作为评判者的评估方法通常依赖于从一般写作评估惯例中得出的固定评估标准（Shao等，2024；Bai等，2024；Que等，2024）。然而，这种静态标准存在三个关键局限性：（1）领域全面性：固定标准无法适应专业领域，未能解决技术文档或创意写作等领域的独特特征；（2）需求特异性：此类标准缺乏灵活性，无法涵盖与风格、格式或长度控制相关的特定要求；（3）材料依赖性：它们不足以验证响应是否恰当地利用了提供的参考材料，例如整合数据点或保持叙事连续性。为了解决这些挑战，我们提出了一种依赖查询的评估框架，能够动态适应多样化的写作场景。我们的方法包括两个阶段：

**Phase 1: Dynamic Criteria Generation**

如图4所示，给定WritingBench中的一个查询q，通过精心设计的指令提示LLM自动生成一组五个评估标准，Cq = {c1, . . . , c5}（选择五个标准反映了当代许多评估场景中的常见做法（Shao等，2024；Ma等，2024；Marco等，2024）），以确保在标准制定过程中提供结构性指导（见附录B.4）。每个标准由三个部分组成：一个简明的名称总结该标准，一个扩展描述详细说明评估重点，以及详细的评分细则，为相应的评估维度提供细化的质量等级。

**Phase 2: Rubric-based Scoring**

对于每个标准ci ∈ Cq，评估者独立地为响应r在10分制上打分，并提供分数和理由。最终的总分通过所有标准的平均分计算得出。详细的提示见附录B.5。

#### Critic Model

为降低基于大语言模型（LLM）评估的计算开销，我们开发了专用评判模型M，用于实现基于评分细则的评估框架。具体而言，该模型执行映射关系Mc：(q, r, Ci) 7→ [1, 10] × J，其输出包含符合预设评估细则的数值化评分及对应的论证文本J。

我们在一个包含50,000个实例的数据集上对评判模型进行了微调，这些实例通过实验中使用的大语言模型收集而来。该数据集涵盖了多样化的查询、评估标准以及相应的评估结果，确保了评估的鲁棒性。第4.3节中的实验验证了评判模型的一致性。

#### Evaluation-Guided Data Curation for Writing Enhancement

基于查询的评估框架通过两阶段筛选机制，实现了多样化写作任务的系统性数据优化。首先，对初始24K监督微调样本实施实例特异性标准生成；随后，批评模型完成50%样本的筛选，最终产出12K经框架优化的高质量样本。实验采用llama-3.1-8b-instruct和qwen-2.5-7b-instruct模型进行微调，并在两个写作基准测试上开展评估，验证了该框架在识别高质量写作样本方面的有效性。

### Experiment

#### Experiment Settings

本节详细介绍了使用WritingBench框架进行实验时的相关设置，包括评估协议和模型训练配置，具体如下：

**评估协议**：在WritingBench上对模型响应采用统一的生成设置，允许最多16,000个token。我们将温度参数设为0.7，top-k为20，top-p为0.8，以确保生成结果的创造性与稳定性之间的平衡。批评模型采用相同的参数设置，但输入长度限制为2,048个token，以保障评分的稳定性。

**模型训练**：批评模型基于Qwen-2.5-7B-Instruct基础模型进行微调，使用AdamW优化器，学习率为7e-6。训练数据为50K个监督微调样本，在8块A100 GPU上训练3个epoch，批次大小为64，梯度累积步数为8。写作模型分别在Qwen-2.5-7B-Instruct和Llama-3.1-8B-Instruct上进行训练，在32块A100 GPU上训练5个epoch，总批次大小为128，梯度累积步数为4。

#### Comparison between LLMs

我们评估了16个大语言模型：ChatGPT-4o-latest（Hurst等，2024）、o1-Preview（Jaech等，2024）、Claude-3.5-Sonnet（Anthropic，2024）、Gemini1.5-Pro（Reid等，2024）、Qwen-Max（Team，2025）、Deepseek-R1（DeepSeek-AI等，2025）、Deepseek-V3（DeepSeek-AI等，2024）、MistralLarge-Instruct（MistralAI，2024）、Qwen-2.5-72BInstruct和Qwen-2.5-7B-Instruct（Yang等，2024）、Llama-3.3-70B-Instruct和Llama-3.1-8BInstruct（Dubey等，2024）、Suri（Pham等，2024）、LongWriter（Bai等，2024），以及我们的写作模型——基于Qwen-2.5-7B和Llama-3.1-8B的微调模型，均在WritingBench上进行测试。每个查询通过5个实例特定的标准进行评估，采用10分制，并通过子领域特定的子类别热图展示详细的变化情况。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=YzY0MGIxYjUzNmI0MTUwMzUwNTc4NWY3NDgzZDRkZTFfY3BVTjFZSEE5b3JMaUxsdVhqQjJ5N01pQmt4T2RGTXNfVG9rZW46WVVRM2JvWTM5b3Y1TVZ4TGExZWNFcGt3bnVoXzE3NDg3NTk0MTc6MTc0ODc2MzAxN19WNA)

**领域得分的关键洞察**：从表3中可以看出，教育领域（D5）在模型中的表现始终优异，其次是学术与工程领域（D1）。文学与艺术领域（D5）得分最低，且表现出显著的差异。具备思维链（CoT）能力的模型，如Deepseek-R1和o1预览版，优于非CoT模型，表明CoT在处理叙事和创造性内容方面具有潜力。图5揭示了在小说续写、招标书和白皮书等小众领域中模型普遍得分较低，这些任务需要更高的知识水平、长文本生成能力以及对上下文一致性的把握，指出了未来改进的方向。

**需求得分的关键洞察**：表3展示了三个需求子集的得分：R列显示总体得分，C列显示特定需求的平均得分。例如，如果某个风格查询包含两个相关标准（经人工标注确认），则C列仅对这些标准的得分进行平均。风格（R1）和格式（R2）维度揭示了模型之间的差异，Deepseek-R1和Qwen-Max通常表现更优。然而，长度要求（R3的C列）仍然具有挑战性，尤其是在特定部分的约束（见附录16中的示例）和长文本生成（见第4.5节）方面。

**WritingBench实验的总体分析**表明：Deepseek-R1在领域和需求维度上均表现领先，展现了其多功能性和强大的语言能力。在创造性内容领域内，模型表现存在显著差异，CoT模型显示出明显的潜力——这一假设通过我们的对比实验得到进一步验证（见附录C.1，其中采用CoT机制的模型在创造性写作任务中优于未采用CoT的模型）。此外，我们观察到，先进模型在三个常见需求的特定得分（C列）上通常高于其总体得分（R列）。这些特定标准之外的标准往往更强调内容方面，如与源材料的整合和写作深度，凸显了提升内容质量的必要性。这一分析为现有模型的能力提供了基准，并指明了未来发展中需要改进的具体领域。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MmE5NzZhOWZkYmIyMGFhODhlOTM5YjAxNTVhNDIxODhfZDdyempmekFYTGNwYzlySlNSQXdVUlRvd1pwTFVRVFJfVG9rZW46VWVmQmJGcnJjb21mUDN4SEl1Q2N4bVB1blliXzE3NDg3NTk0MTc6MTc0ODc2MzAxN19WNA)

#### Human Consistency

为了验证自动化评估与人类判断之间的一致性，我们对涵盖所有100个子领域的300个查询进行了人工评估。五名具有语言学背景且经过专业训练的标注员对不同模型的响应进行成对比较。对于每个查询，从不同模型中随机选择两个响应。标注员根据查询的要求选择更优的响应或声明其等价性。我们将动态的基于查询的标准与两个基线进行比较：静态的全局统一标准和静态的领域特定定制标准（静态标准由领域专家设计），并在两个大语言模型评判者（ChatGPT-4o-latest和Claude-3.5-Sonnet）上进行测试。如表4所示，与静态的全局统一标准或领域特定定制标准相比，我们的动态基于查询的标准在人类一致性方面表现更优。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MTE1ZjI2M2IyODNmNWExNWJiOWQ0YTQ0ZDU1YTBjNDNfNVA3QU1yWTM4VjBWZmU3eDdEZUQ1VDg4b2pFVXo0UHRfVG9rZW46RnJTQ2J6Q0M0b0xiUXJ4NjQ4YmNMNUswblJoXzE3NDg3NTk0MTc6MTc0ODc2MzAxN19WNA)

尽管进行了定制化处理，领域特定标准的表现仍然欠佳，因为我们的查询涉及高度多样化的任务和不同的来源。这些发现证实，与传统的静态方法相比，基于上下文敏感的查询依赖评估能更好地捕捉现实世界中的写作复杂性。此外，批评模型达到了83%的一致性，验证了其实际可行性。

#### Ablation of Data Curation for Writing-Enhanced Models

基于第3.2节所述的查询依赖评估框架，我们对涵盖多个领域的初始24K监督微调数据集进行了实例特定标准生成，其中包括包含补充材料的长输入查询。随后，我们的批评模型应用查询依赖评估将该数据集筛选为高质量的12K子集。我们使用Llama-3.1-8B-Instruct和Qwen-2.5-7B-Instruct模型进行了微调实验。如图5所示，基于筛选数据训练的这两个模型相较于其先前版本均表现出显著的性能提升。值得注意的是，它们优于基于完整数据集训练的模型，甚至接近了最先进模型的能力。在另一个基准测试LongBench-Write（遵循Bai等，2024中的质量评估设置）上的进一步验证也证实了这些发现，我们的模型超越了基线性能。这些结果验证了我们的查询依赖评估策略的鲁棒性以及批评模型在筛选高质量写作样本方面的实用性。这一综合方法使得较小模型能够在多样化的写作任务中与较大模型竞争甚至超越。

#### Ablation of Length

我们对大语言模型在不同输入输出长度下的性能表现进行了评估，通过排除样本量少于5个的区间确保统计有效性。如附录6所示，实验表明大多数前沿模型在不同输入长度下均能保持稳定性能，这得益于其先进的长期上下文理解能力。然而附录7的分析显示，多数模型在响应生成长度上存在固有局限，通常输出被限制在约3000个token以内。虽然在此范围内输出质量相对稳定，但较小模型会出现更明显的性能下降，表现为输出内容重复。值得注意的是，只有Qwen-Max和LongWriter这两个针对长文本生成进行架构优化的模型，能够有效支持更长的响应输出。这一发现凸显了增强长文本输出能力和优化生成长度在写作任务中的重要性。

### Conclusion

本文提出WritingBench——一个用于评估大语言模型跨领域生成式写作能力的综合性基准测试。该基准包含1,239个测试查询，涵盖6大核心领域和100个子领域，提供风格、格式和长度要求三个维度的评估体系。我们提出的基于查询依赖的评估框架通过批评模型实现了高度的人类评判一致性。实验证明，基于精选数据训练的轻量化模型可达到与前沿模型相媲美的性能表现。通过公开WritingBench及其相关资源，我们旨在推动大语言模型写作能力的深入研究与技术发展。

### Limitations

本研究存在三个主要局限性值得关注：首先，写作模型与批评模型均采用传统监督微调方法训练，未系统探索增强优化策略。尽管我们证明了思维链机制在创意领域的部分有效性，但与数学推理任务中的成熟应用相比，其潜力仍有待挖掘。其次，当前评估框架对复杂多维长度要求的处理精度不足，包括时序约束和章节特定字数限制。这一局限凸显了需要开发融合学习指标与结构化规则评估的增强评分方法，以更精准规范输出规格。第三，组合性任务的可靠成对偏好标注仍存在固有挑战。尽管实施严格标注协议，评估者在评判两个优质响应时仍难免引入主观偏差，尤其在叙事偏好和语境解读方面。虽然共识构建流程降低了部分变异度，但完全契合多样化用户偏好在理论上仍不可实现。

