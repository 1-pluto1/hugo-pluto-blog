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

得益于自然语言处理（NLP）技术的快速发展，像苹果的Siri [Apple Inc. 2023] 和百度的小度 [Baidu Inc. 2023] 这样基于深度神经网络的虚拟助手近年来日益普及，促进了更自然的人机交互。在这些交互中，**问答（QA）任务**扮演着至关重要的角色 [Clark et al. 2019; Hirschman and Gaizauskas 2001; Xiong et al. 2016]。因此，机器能够理解自然语言，并以类似于人类的自然方式回应人类提出的查询，这一点至关重要 [Botana et al. 2023]。

大型语言模型（LLMs）的出现为虚拟助手的发展带来了新的可能性 [Gondala et al. 2021; Piñeiro-Martín et al. 2023]。然而，LLM引发的**幻觉（hallucinations）**问题令人担忧 [Amodei et al. 2016; Hendrycks et al. 2021]。具体来说，LLM可能对给定的查询提供错误的回答，这会损害用户满意度，甚至引发法律问题 [Devaraj et al. 2022; Kaye 2023; Tam et al. 2022]。因此，及时且自动地检测LLM在问答任务中的幻觉至关重要。

幻觉通常分为两大类：**事实性幻觉（factuality hallucinations）和忠实性幻觉（faithfulness hallucinations）**。事实性幻觉指的是生成内容与可验证的现实世界事实之间存在差异。相比之下，忠实性幻觉指的是生成内容与输入上下文之间存在差异 [Huang et al. 2025]。以往的研究大多集中于识别忠实性幻觉 [Huang et al. 2023; Miao et al. 2023; Shen et al. 2022]。然而在实践中，大量的问答任务并不包含上下文，这使得事实性幻觉比忠实性幻觉更频繁地发生 [Chen and Shu 2024; Wang et al. 2023, 2024; Zhao et al. 2023]。

现有用于事实性幻觉检测的方法仅取得了有限的成功。许多工作利用模型的**置信度分数（confidence score）来检测事实性幻觉 [Luo et al. 2023; Varshney et al. 2023]。对于不提供置信度分数的黑盒LLM** [OpenAI Inc. 2023]，一些工作提出根据其可观察行为来推断LLM的置信度分数 [Kadavath et al. 2022; Manakul et al. 2023; Xiong et al. 2024]。例如，SelfCheckGPT通过获取LLM的多种回答并比较它们的一致性来推断其置信度分数 [Manakul et al. 2023]。然而，LLM置信度分数的可靠性与其性能相关 [Orgad et al. 2024; Zhang et al. 2023]。因此，当LLM产生幻觉时，其置信度分数也可能不可靠，这使得基于置信度分数来检测幻觉变得不合适 [Huang et al. 2025]。

在这项工作中，我们专注于LLM在处理问答任务时的**事实性幻觉**，并提出了 **DrHall**，一个使用**蜕变测试（Metamorphic Testing, MT）**[Chen et al. 2020, 2018] 来检测和减少黑盒LLM事实性幻觉的框架。我们认为，产生幻觉的答案是**不稳定**的 [Huang et al. 2025; Sclar et al. 2023]。因此，当LLM产生幻觉时，如果我们使用蜕变测试让LLM通过不同的执行路径重新执行同一任务，它们更有可能产生不同的答案。为了引导出不同的执行路径，我们精心设计了**蜕变关系（Metamorphic Relations, MRs）**，这些关系考虑了LLM问答能力背后的机制，包括**问题理解、知识回忆和知识推理** [Tan et al. 2023; Wei et al. 2022; Zheng et al. 2023]。我们最终得到了六个基本MR和三个复合MR。表1展示了我们的MRs。通过研究原始回答与蜕变测试下的后续回答之间的一致性，我们可以检测出黑盒LLM的事实性幻觉。我们进一步提出，通过将我们的MR与**多路径投票（multi-path voting）**相结合来纠正黑盒LLM的事实性幻觉。

我们使用三个问答数据集来评估DrHall的有效性。对于自然语言问答任务，为避免基准泄露对评估结果的影响并刻画更真实的应用场景 [Zhou et al. 2023b]，我们利用维基百科的知识构建了一个新数据集 **FactHalluQA**。此外，我们还考虑了编程问答任务，并评估了DrHall处理**代码幻觉（code hallucinations）的能力。代码幻觉指的是LLM生成的代码在语法上正确，甚至在语义上看似合理，但实际上无法按预期工作 [Tian et al. 2024]。具体来说，我们关注两类编程问题：程序修复和代码生成**。图1展示了一个编程问题示例以及LLM生成的代码幻觉。

实验结果表明，DrHall能够持续优于当前最先进的基线方法，在幻觉检测方面获得了超过 **0.856** 的平均**F1分数**。在幻觉纠正方面，DrHall同样能优于最先进的基线方法，平均**幻觉纠正率**超过 **53%**。我们的贡献可以总结如下：

- 我们提出了 **DrHall**，一个使用蜕变测试来检测和减少黑盒LLM事实性幻觉的框架。我们为DrHall精心设计了六个基本MR和三个复合MR。基于这些MR，DrHall可以通过一致性评估和多路径投票分别检测和纠正黑盒LLM的事实性幻觉。
- 我们构建了一个新的自然语言问答数据集 **FactHalluQA**，以避免基准泄露对评估结果的影响。FactHalluQA包含超过八百个英文问题，涵盖七个不同学科。
- 我们进行了广泛的实验来验证我们框架的有效性。除了自然语言问答任务，我们还考虑了编程问答任务，这是现有事实性幻觉缓解方法尚未探索的领域。实验结果表明，我们的框架能够优于最先进的基线方法，并实现出色的幻觉检测和纠正性能。

本文的其余部分组织如下：第2节介绍本文的背景和相关工作。第3节描述我们的技术细节。第4节介绍实验设置。第5节报告并分析实验结果。第6节总结本文。

### Background and Related Work

##### 事实性幻觉的检测 (Detection of Factuality Hallucinations)

尽管深度学习模型表现出色，但众所周知它们存在可靠性 [Deng et al. 2023; Wu et al. 2024] 和隐私问题 [Huang et al. 2024; Wu et al. 2023]。LLM一个独特的可靠性问题是它们有时会产生幻觉。因此，检测LLM的幻觉对于确保生成内容的可靠性和可信度至关重要。

为了有效检测LLM的**事实性幻觉**，一个直观的策略是将模型生成的内容与可靠的知识源进行比较 [Chen et al. 2024; Chern et al. 2023; Guo et al. 2022; Min et al. 2023]。例如，Chern等人 [Chern et al. 2023] 提出了一个统一的框架，该框架允许LLM通过使用一系列外部工具收集证据来检测事实错误。然而，这种方法需要访问外部数据库，并可能产生高昂的推理成本。

当前一种常见的方法是**不确定性估计（uncertainty estimation）**，该方法认为LLM的幻觉源于模型的不确定性 [Luo et al. 2023; Varshney et al. 2023]。Varshney等人 [Varshney et al. 2023] 通过考虑关键概念中的最小词元（token）概率来确定模型对这些概念的不确定性。然而，当只能通过API调用访问LLM时，输出的词元级概率分布通常是不可用的。由于这一限制，一些研究人员提出从模型的可观察行为中推断其**置信度分数** [Kadavath et al. 2022; Manakul et al. 2023; Xiong et al. 2024]。例如，Manakul等人 [Manakul et al. 2023] 通过对同一提示词进行多次采样以获得LLM的多个回答，并评估这些事实性陈述之间的一致性来检测幻觉。

##### 蜕变测试 (Metamorphic Testing)

**蜕变测试（Metamorphic Testing, MT）是一种黑盒测试方法。它首先基于待测软件的领域知识建立蜕变关系（Metamorphic Relations, MRs）**。然后，它利用蜕变关系生成新的测试用例，并通过验证蜕变关系是否得以维持来判断测试是否通过。蜕变测试因其能够缓解软件测试过程中的**“测试预言机”（test oracle）问题**，而被广泛用于测试不同任务的深度学习（DL）模型 [Liu et al. 2014]。自动驾驶和神经机器翻译模型是吸引了许多基于MT的测试方法的两种典型深度学习模型 [Luu et al. 2022; Raunak et al. 2022; Wang and Su 2020]。MT也被应用于问答模型。然而，现有方法要么测试准确性不足，要么侧重于验证问答模型的**忠实性（faithfulness）** [Chen et al. 2021; Shen et al. 2022]。

##### 思维链与自洽性 (Chain of Thought and Self-Consistency)

**思维链（Chain of Thought, CoT）**是一种通过引入中间推理步骤来实现复杂推理能力的技术 [Wei et al. 2022]。研究表明，该技术可以显著提升模型在数学、常识和推理任务上的性能。原因是CoT能让模型将问题分解为更易于处理的子问题，从而提高其理解和解决问题的能力。大量研究致力于使用CoT的概念来增强LLM的性能 [Kojima et al. 2022; Liu et al. 2022; Wang et al. 2022]。其中最先进的方法之一是自洽性（Self-Consistency） [Wang et al. 2022]，它要求模型采用不同的推理路径，然后选择最一致的答案。

### Approach
