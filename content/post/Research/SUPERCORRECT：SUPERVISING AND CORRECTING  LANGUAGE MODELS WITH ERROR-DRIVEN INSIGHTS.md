---
title: SUPERCORRECT：SUPERVISING AND CORRECTING  LANGUAGE MODELS WITH ERROR-DRIVEN INSIGHTS
date: 2025-03-02T11:30:03+00:00
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



单位： Peking University, National University of Singapore, UC Berkeley, Stanford University

代码：https://github.com/YangLing0818/SuperCorrect-llm

基座模型： Qwen2.5-Math-7B, Meta-Llama3.1-8B, DeepSeek-Math-7B

原文地址：https://arxiv.org/pdf/2410.09008

显卡：8 * NVIDIA A100-PCIE-40GB GPU

**总结**
我们将我们的贡献总结如下：(i) 我们提出了一种新颖的两阶段微调方法SUPERCORRECT，用于提高大型语言模型（LLMs）的推理准确性和自我纠正能力。(ii) 我们提出了基于层次化思维的微调方法，使小型LLMs能够生成更准确和细粒度的推理思维。(iii) 我们提出了跨模型协作的DPO（Distributed Policy Optimization），创新性地利用最先进的LLMs来定位和纠正较小学生LLMs在推理过程中的特定错误思维，从而提升其自我纠正能力并突破其思维瓶颈。(iv) 我们构建了两个高质量的数据集，并开发了三个强大的推理LLMs：SUPERCORRECT-Qwen/DeepSeek/Llama-7B，在MATH数据集上达到了70.2%的准确率，在GSM8K数据集上达到了89.5%的准确率，在所有7B模型中创造了新的SOTA（State-of-the-Art）性能。

![](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/20250601112609350.png)

### abstract

大型语言模型（LLMs）如GPT-4、PaLM和LLaMA在各种推理任务中展示了显著的改进。然而，较小的模型如Llama-3-8B和DeepSeekMath-Base在复杂的数学推理方面仍然存在困难，因为它们无法有效地识别和纠正推理错误。最近的基于反思的方法旨在通过启用自我反思和自我纠正来解决这些问题，但它们仍然面临在独立检测推理步骤中的错误方面的挑战。为了克服这些限制，我们提出了SUPERCORRECT，一个新颖的两阶段框架，该框架使用一个大型教师模型来监督和纠正较小学生模型的推理和反思过程。在第一阶段，我们从教师模型中提取层次化的高级和详细思维模板，以指导学生模型引出更细粒度的推理思维。在第二阶段，我们引入了跨模型协作直接偏好优化（DPO），通过在训练过程中跟随教师的纠正轨迹来增强学生模型的自我纠正能力。这种跨模型DPO方法教导学生模型有效地定位和解决错误思维，通过从教师模型获得的错误驱动洞察力，打破其思维瓶颈，并获取新技能和知识以应对具有挑战性的问题。广泛的实验一致证明了我们相对于先前方法的优越性。值得注意的是，我们的SUPERCORRECT-7B模型在MATH/GSM8K基准测试中显著超越了强大的DeepSeekMath7B和Qwen2.5-Math-7B，分别提高了7.8%/5.3%和15.1%/6.3%，在所有7B模型中实现了新的SOTA性能。

### INTRODUCTION

大型语言模型（LLMs）（Brown等，2020；Anil等，2023；Achiam等，2023；Du等，2022；Jiang等，2024），如GPT-4（Achiam等，2023）、PaLM（Anil等，2023）和LLaMA（Touvron等，2023a；b），在各种推理任务中展示了显著的改进。然而，尽管这些模型在大规模数学数据集上使用多种技术进行了预训练，较小的模型如Llama-3-8B（Dubey等，2024）和DeepSeekMath-Base（Shao等，2024）在复杂的数学推理任务中仍然表现不佳。

现有的研究旨在通过各种方法提升LLMs的数学性能。我们将这些方法分为两类：传统的微调优化和基于反思的优化。传统的微调方法主要探索训练技术，如监督微调（SFT）（Roziere等，2023；Shao等，2024；Dubey等，2024），以及LLM对齐策略，如基于人类反馈的强化学习（RLHF）（Achiam等，2023；Ouyang等，2022；Bai等，2022a；b）和替代方法，如直接偏好优化（DPO）（Rafailov等，2024）。尽管这些方法在广泛的语言任务中展示了显著的进展，但它们的优化目标仅集中在直接答案或简单的推理逻辑上。因此，它们在定位推理过程中的错误和修正语言模型的错误推理逻辑方面存在困难。

最近的基于反思的方法试图解决微调方法的不足，并利用预先设计的提示或通用规则来指导语言模型在推理过程中进行自我反思和自我纠正（Shinn等，2024；Kim等，2024）。一些方法（Li等，2023；2024c）进一步利用大语言模型（LLMs）合成基于规则的数据集，以增强其在训练阶段的自我纠正能力。然而，正如Tyen等（2024）所提到的，LLMs仍然难以独立识别其推理步骤中的错误。如果没有准确的错误识别，自我纠正将变得更加困难。在复杂的数学推理中，即使提供了错误的位置，LLMs仍然常常受到其先前推理背景的偏见或误导。因此，语言模型在单一LLM内澄清推理错误的原因仍然具有挑战性。

为了解决这些局限性，我们提出了一种新颖的两阶段框架，即SUPERCORRECT，利用大型教师模型的思维来监督和纠正较小学生模型的推理和反思过程。如图1所示，在第一阶段，我们从教师LLM中提取层次化的思维模板，以指导学生模型生成更细粒度的推理思维。该模板包含一个高级思维，为类似问题提供总结性和概括性的解决方案，以及一个详细解决方案，提供关键推理步骤的详细解释。与之前的思维格式（如CoT（Wei等，2022）和BoT（Yang等，2024b））相比，我们的层次化思维模板为后续的错误纠正提供了更深层次和更具信息量的推理见解。在第二阶段，我们提出了跨模型协作的DPO（Direct Preference Optimization）来优化学生模型，并通过在训练过程中遵循教师的跨模型纠正轨迹来增强其自我纠正能力。具体来说，我们不仅模拟正确答案或首选的推理过程，还指导教师LLM识别并纠正学生思维中的错误部分。然后，这种跨模型纠正轨迹被用来指导学生模型进行更好的自我纠正，使其能够避免并纠正特定的错误。我们跨模型DPO方法的关键见解是使学生语言模型能够突破其思维的瓶颈，并从教师的纠正轨迹中获得新的错误驱动的见解和知识。

此外，我们构建了一个高质量的精调数据集，配备了包含10万个样本的分层思维模板，以及一个用于思维级校正优化的成对偏好数据集，包含1万个样本。该数据集由以下部分组成：1）一个数学问题，2）按照我们预先设计的格式编写的先前推理步骤，3）由教师大语言模型基于真实解生成的带有选定分析和校正指导的步骤，4）由学生大语言模型在没有访问真实解的情况下生成的带有拒绝分析和校正指导的步骤。

我们将我们的贡献总结如下：(i) 我们提出了一种新颖的两阶段微调方法SUPERCORRECT，用于提高大型语言模型（LLMs）的推理准确性和自我纠正能力。(ii) 我们提出了基于层次化思维的微调方法，使小型LLMs能够生成更准确和细粒度的推理思维。(iii) 我们提出了跨模型协作的DPO（Distributed Policy Optimization），创新性地利用最先进的LLMs来定位和纠正较小学生LLMs在推理过程中的特定错误思维，从而提升其自我纠正能力并突破其思维瓶颈。(iv) 我们构建了两个高质量的数据集，并开发了三个强大的推理LLMs：SUPERCORRECT-Qwen/DeepSeek/Llama-7B，在MATH数据集上达到了70.2%的准确率，在GSM8K数据集上达到了89.5%的准确率，在所有7B模型中创造了新的SOTA（State-of-the-Art）性能。

### RELATED WORK

**基于人类反馈的强化学习用于大型语言模型**

为了提高大型语言模型（LLMs）的性能和可靠性，引入了基于人类反馈的强化学习（RLHF）方法，如Christiano等人（2017）和Ouyang等人（2022）的研究，用于LLM的对齐。这种方法对数据集的要求更高，因为它需要成对的标注数据来训练奖励模型，从而反映人类偏好。然后使用强化学习训练策略模型，以最大化估计的奖励。尽管这种方法被证明是有效的，但由于其对奖励模型质量的依赖，这一过程复杂且计算密集。为了简化这一过程，Rafailov等人（2024）提出了直接偏好优化（DPO），它直接使用成对数据进行优化。通过将偏好损失定义为策略的函数，DPO可以使用简单的训练技术优化策略，避免了强化学习的复杂性。然而，由于优化单元的设计，当前方法在数学推理方面仅显示出有限的改进。Lai等人（2024）的工作通过将每个中间推理步骤视为基本单元，建立了更细粒度的奖励单元。然而，它们未能明确错误原因并提供纠正错误的明确指导。在本文中，我们特别设计了一种基于跨模型师生协作思维的奖励机制，将每个纠正步骤作为基本优化单元。

**自我纠正/反思推理**

自我纠正在提升大语言模型（LLM）输出的风格和质量方面显示出潜力。先前的研究（Li et al., 2023; Shinn et al., 2024; Madaan et al., 2024; Saunders et al., 2022; Miao et al., 2023; Chen et al., 2023a）聚焦于自我纠正的概念，即让LLM纠正其自身的输出。然而，正如Huang等人（2023）所提到的，尽管自我纠正在提升模型输出的风格和质量方面可能有效，但在推理任务中，LLM在没有外部反馈的情况下难以识别和修正错误。例如，Reflexion（Shinn et al., 2024）和RCI（Kim et al., 2024）都使用真实正确性作为信号来停止自我纠正循环。此外，一些尝试自我纠正逻辑或推理错误的努力有时会将正确答案变为错误答案，导致整体表现更差（Huang et al., 2023）。虽然先前的研究通常将自我纠正描述为在特定LLM内部进行的过程，但我们的方法利用大规模LLM来明确识别错误并从错误中获取纠正见解。通过这种跨模型奖励，我们可以通过微调和基于纠正的偏好优化来修正小型LLM在推理任务中暴露的弱点。

**数学推理中的思维扩展**

数学推理中的思维扩展主要关注预设计的推理结构或模板，利用提示技术来增强大语言模型（LLMs）的数学推理能力。链式思维（Chain-of-Thought, CoT）提示（Wei等，2022）及其变体（Kojima等，2022；Press等，2023；Arora等，2022），如最少到最多（Least-to-Most, Zhou等，2022）、分解提示（Decomposed Prompting, Khot等，2022）和自动CoT（Auto-CoT, Zhang等，2022），提示LLMs将复杂问题分解为更简单的子任务，并系统地解决它们，最后总结出最终答案。像树状思维（Tree-of-Thought, Yao等，2024）和图状思维（Graph-of-Thought, Besta等，2024）这样的创新进一步复杂化了这一领域，通过探索动态、非线性的推理路径来扩展LLMs的启发式能力（Chen等，2023b；Ning等，2023）。其他方法如PoT（Chen等，2022）、PAL（Gao等，2023b）和Gou等（2023）尝试利用外部工具（如代码）来避免LLMs在数学推理过程中的幻觉。然而，这些方法存在资源需求增加、时间复杂性更高、依赖手动提示设计，并且通常针对特定任务类型的问题。最近的BoT（Yang等，2024b）提出了一种任务无关的范式，通过元缓冲区基于累积的思维模板高效解决问题。然而，这是一个无需训练的框架，可能无法本质上提升LLMs的推理能力。为了进一步提升LLMs的内部推理能力，Quiet-STaR（Zelikman等，2024）使用基于RLHF的自我教学，通过LLMs自我生成的思维来改进普通任务和简单数学问题的推理。对于超出学生能力的更复杂问题，这种“先思考后推理”的模式可能效果不佳。在本文中，我们利用一种新的跨模型范式，使LLMs能够从外部模型反馈中提升推理和自我纠正能力，从而打破LLMs原始思维的瓶颈，拓宽模型解决更广泛问题的能力。

### PRELIMINARY


**基于人类反馈的强化学习**

基于人类反馈的强化学习（Reinforcement Learning from Human Feedback, RLHF）（Christiano et al., 2017）是一种有效的方法，用于增强大语言模型（LLMs）的鲁棒性、事实性和安全性（Ouyang et al., 2022）。RLHF包含三个训练阶段：1）监督微调（SFT）；2）奖励模型训练；3）策略模型微调。

**监督微调阶段**：RLHF通常从对预训练语言模型进行监督微调开始，使用高质量数据针对下游任务（如对话、摘要等）进行训练，以获得模型πsft。

**奖励模型训练阶段**：对于任何文本，奖励模型会为最后一个标记分配一个标量奖励值，奖励值越大，样本质量越高。根据Stiennon等人（2020）的研究，训练奖励模型通常使用由同一输入生成的两个响应对比组成的数据集。每对偏好样本和非偏好样本的建模损失为：

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MjEyZWQ0N2JiMDQzMDIyODI4NTJjOTRiNmJhOTRiMzlfaEhYendadmowRjZJSWVlQjR6UEdWWHlmcW5jcmQ0emRfVG9rZW46V0I3WWJXNHZMb1N4WjZ4Y1RFUWM1MzFYblRiXzE3NDg3NDc5NTU6MTc0ODc1MTU1NV9WNA)

其中σ为Sigmoid函数，r表示参数为ψ的奖励模型，r(x, y)是输入提示x和响应y的预测奖励标量值。然而，由于训练流程复杂，这种方法通常被认为较为复杂。

**强化学习微调阶段**：在强化学习阶段，学习到的奖励函数用于为语言模型提供反馈。根据先前的研究（Tutor; Jaques et al., 2020），优化目标被表述为：

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=NWRlOGFhNzgyZWNkZjc0ZDVlYjBiZjliYjk5NGZlMmRfSFdSZ3NCUHdmOTRyNlJZQ3dJUHMxZldvYkg4M045elRfVG9rZW46R1QzcGJ1eHZGb2V5V2F4ZzNZcGNQOTh3bkJnXzE3NDg3NDc5NTU6MTc0ODc1MTU1NV9WNA)

其中β是控制与基础参考策略πref（即初始SFT模型πsft）偏离程度的参数。在实践中，语言模型策略πθ也初始化为πsft。由于语言生成的离散性，该目标不可微分，通常通过强化学习进行优化。标准方法（Ziegler et al., 2019; Bai et al., 2022a; Ouyang et al., 2022）是构建如公式（1）所示的奖励函数，并使用PPO（Schulman et al., 2017）进行最大化。

  
Direct Preference Optimization (DPO) 是一种与传统RLHF方法竞争的替代方案，由Rafailov等人于2024年提出，旨在直接利用成对偏好来优化策略模型，并具有等效的优化目标。具体来说，给定一个输入提示x和一个偏好数据对(y+, y−)，DPO的目标是最大化优选输出y+的概率，并最小化不理想输出y−的概率。优化目标公式为：

$$L_{DPO}(\theta) = -\mathbb{E}_{(x,y+,y−)\sim D}\left[\log \sigma\left(\beta \log \frac{\pi_\theta(y+|x)}{\pi_{\text{ref}}(y+|x)} - \beta \log \frac{\pi_\theta(y−|x)}{\pi_{\text{ref}}(y−|x)}\right)\right], $$

其中，$$D$$是成对偏好数据集，$$σ$$是sigmoid函数，$${\pi_\theta(·|x)}$$是要优化的策略模型，$${\pi_{ref}(·|x)}$$是在训练期间保持不变的参考模型，超参数β控制与参考模型的距离。

### METHOD

#### SUPERVISED FINE-TUNING WITH HIERARCHICAL THOUGHT TEMPLATE

构建从教师大语言模型（LLMs）中提取的分层思维模板。传统的用于训练LLMs的指令-响应数据集（Ouyang等，2022年）主要关注响应的正确性，导致LLMs仅仅模拟提供的解决方案和答案，而忽略了中间推理思维的重要性。最近的工作如BoT（Yang等，2024b）利用高级推理指南（思维模板）使LLMs能够以无需训练的方式有效解决类似问题。然而，对于复杂且多样的数学推理任务，我们发现仅使用高级思维模板是不够的，特别是对于小型LLMs。为了使小型LLMs能够处理复杂的推理任务，我们特别设计了一种从大型教师LLMs中提取的分层思维模板，用于转移到小型学生LLMs。这种新的分层思维模板包括高级思维和详细解决方案。前者为类似问题提供了总结和概括的解决方案，而后者则提供了关键推理步骤的详细解释。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MzE0YmVkMDVmOTI1Mjc2N2RjYzk4NmFiNDBhNDEwMzVfSnVaZTNVRENnVGpTYlV1QXB5REljYjZzdTNJQ0ZDUzFfVG9rZW46TUlRU2I1VGd4b1QyQ014NUsxU2NjT3BUblBHXzE3NDg3NDgwOTE6MTc0ODc1MTY5MV9WNA)

  

基于思维的有监督微调 在整理好我们的基于思维的数据集Dsft后，我们的优化目标是使学生大语言模型π能够通过分层思维进行推理，并对每个问题解决过程有更全面的理解，这可以表示为：

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MzA5NmNjNTJmNmM3MTNkM2U5Yjc4MmQ3MzY5Y2FmYzRfa3djdW9uMm9CbWpVT21Cb1RzdG9zc2d4RkVjSlJHMXVfVG9rZW46SkNzOGJPWHhxb1F3MDB4blZKa2NRM2ZUbmpiXzE3NDg3NDgwOTE6MTc0ODc1MTY5MV9WNA)

从基础的学生大语言模型π开始，$$L_{sft}$$最大化在给定提示$$P_{stu}$$和输入问题x的情况下响应$$(T_{tea}, s_{tea})$$的可能性，其中$$P_{stu}$$表示预定义的提示$$P_{tea}$$。在微调过程之后，我们通过从最先进的推理大语言模型中学习分层思维，极大地增强了基础学生大语言模型的推理能力，并使学生大语言模型能够生成类似的分层思维以及最终答案。然后，我们获得了微调后的学生大语言模型πref，可用于第4.2节中的跨模型协作dpo。

#### CROSS-MODEL COLLABORATIVE DPO

Boosting DPO with Thought Correction 虽然DPO在某些领域（如聊天、风格等）表现出色，但其优化目标在处理复杂的数学推理任务时效果不佳。Lai等人（2024）指出，问题在于解决复杂数学问题时，错误往往出现在最具挑战性的步骤（如复杂的计算、巧妙的变换）。这可能导致训练过程中错误的优化，因为即使前面的步骤是正确的，也会被拒绝。此外，单个大型语言模型（LLM）很难检测和纠正自身的错误（Tyen等人，2024）。这类似于学生难以从自己的错误解答中获得洞察。错误的根源在于有缺陷的推理，仅仅模仿正确的解决方案而不解决思维层面的错误是低效的。为了解决这个问题，我们精心设计了新颖且细粒度的优化目标，优先考虑思维层面的纠正，而非传统的实例层面的偏好。具体而言，我们首先准确定位错误步骤，然后使用该错误步骤的纠正轨迹作为优化单元。这种方法优先选择来自教师大语言模型（πtea）的跨模型纠正轨迹，而非来自学生大语言模型（πref）的自我纠正轨迹，从而增强学生大语言模型的错误检测和自我纠正能力。

  

为了实现思维层面的纠正，我们需要收集一个包含细粒度自我和交叉纠正轨迹配对数据的数据集。具体来说，我们利用微调后的学生大语言模型$${\pi_{ref}}$$在我们采样的测试数据集$$D_{test} = \{x_{test}, \hat{y}_{test}, \hat{s}_{test}\}$$上进行基于思维的推理，并得到测试结果$$\pi_{\text{sft}}(x_{\text{test}}) = \{x_{\text{test}}, s_{\text{test}}, T_{\text{test}}, y_{\text{test}} \mid x_{\text{test}} \in D_{\text{test}}\}.$$。通过过滤掉满足$$ y_{\text{test}} \neq y_{\hat{t}\text{est}} $$的错误问题-解决对，最终得到错误数据集：$$D_{\text{err}} = \{x_{\text{test}}, \hat{y}_{\text{test}}, \hat{s}_{\text{test}}, s_{\text{err}}, T_{\text{err}}, y_{\text{err}} \mid x_{\text{test}} \in D_{\text{test}}\}.$$，其中serr是错误解决方案，$$T_{\text{err}}$$是相应的错误思维，$$y_{\text{err}}$$是从$$s_{\text{err}}$$中提取的错误答案。

  

  

给定每个错误的解决方案都明确地表示为一系列推理步骤 $$s_{err} = s_1, s_2, \ldots, s_$$ ，我们继续验证每个推理步骤的正确性，直到找到第一个错误并记录其步骤编号 $$$$ 。在这里，我们利用当前强大的模型（例如，gpt-4o, o1-mini）在数学推理中作为经验丰富的教师模型 $$\pi_{tea$$ 。为了获得相应的错误步骤和原因分析，我们设计了一个提示 $$P_$$ 来指导 $$\pi_{tea$$ 在提供的推理步骤中搜索逻辑缺陷和错误。在搜索 $$s_{err$$ 并评估每个推理步骤后，我们可以定位每个错误步骤，并用错误原因分析 $$a_$$ 和修正指导 $$c_$$ 对每个错误步骤进行注释。因此，我们可以获得一个成对自我和交叉修正的注释数据集： $$D_{corr} = \{(x, \{s_i\}_{i=0}^{k-1}, (a^+_k, c^+_k), (a^-_k, c^-_k)) | x \in D_{err}\$$ ，其中 $$$$ 表示第一个错误步骤。这里 $$(a^+_k, c^+_k$$ 被选为教师模型的修正步骤和分析， $$(a^-_k, c^-_k$$ 被选为学生模型的拒绝修正步骤和原因分析，使用与教师相同的修正提示。为了进一步确保我们数据集的质量，我们还提出了一个检查器 LLM 来进行迭代评估，通过将其与输入问题和真实解决方案进行比较来验证修正轨迹的准确性。如果检测到问题，有问题的部分将被发送回教师 LLM 进行修订。这个迭代检查过程将持续到没有错误为止，最多允许三次迭代。在我们的实现中，我们在 HSFT 数据集的整理过程和成对自我和修正数据集中都应用了检查器 LLM。更多细节请参阅第 5.5 节，我们还在第 5.5.2 节中对数据集质量进行了详细分析。


如表1所示，我们的方法在所有7B模型中实现了新的SOTA性能，在MATH基准测试中显著超越了强大的DeepSeekMath-7B 7.8%和Qwen2.5-Math-7B 15.1%。这一有希望的结果证明了我们在处理复杂推理任务方面的优越性和有效性。值得注意的是，我们可以在GSM8K和MATH数据集上取得比更大规模的模型（如Llama370B-Instruct，Touvron等，2023a）更好的结果，并且使用我们最好的模型SUPERCORRECT-Qwen-7B可以达到与GPT-4o和GPT-4o-mini相当的准确率。我们将这种推理准确率的提升归因于两个方面：1）第一个HSFT阶段，它为学生LLM配备了更深层次和更细粒度的推理过程。与传统的CoT推理过程相比，它帮助学生LLM更仔细地思考，从而提高了推理的一致性，并减少了在学生LLM已经掌握的问题上的幻觉问题。2）第二个跨模型DPO阶段，它利用教师LLM的错误驱动洞察力，帮助学生LLM打破思维的瓶颈，从而使其能够处理那些学生LLM在获取技能和知识时以前无法解决的问题。我们还在附录B中展示了一些来自不同数据集的层次推理的详细示例，请查看它们以全面了解我们的SUPERCORRECT。

![img](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=Y2JmNjIxNjVhOTRjY2RiMDc0NzFkOWZhODQwMWNlMjVfR2RiSjRzZXBXU1BEbHZrOGhRRlNSbjVzZWFwRkVGT2dfVG9rZW46SU83R2IwV3g5b2FVbWN4eFk4UWNjVk1nbnRwXzE3NDg3NDgxMTg6MTc0ODc1MTcxOF9WNA)

  

  

在这里，我们还展示了我们的SUPERCORRECT在自我纠正能力方面的改进，如图3所示。在初始推理阶段之后，我们让所有的大型语言模型（LLMs）验证推理过程，并检测每个推理步骤中的逻辑缺陷和错误，并尝试纠正它们。由于自我纠正的结果，我们的SUPERCORRECT进一步将准确率提高了5∼6%，而其他LLMs在提高准确率方面无效，甚至有些LLMs还降低了原有的准确率。这是因为我们的跨模型DPO通过学习教师的纠正轨迹，帮助LLMs准确定位每个步骤中的错误和逻辑缺陷，并使用细粒度的分析和纠正来帮助LLMs更好地纠正它们。在跨模型DPO过程之后，LLMs不仅能够在其能力范围内一致地解决问题，而且还能够通过从教师LLMs获得的错误驱动洞察力解决更广泛的问题。我们在表6中提供了更多关于跨模型DPO如何使学生模型和教师模型更接近的定量分析。我们还提供了来自不同数据集的一些自我纠正示例，更多详情请查看附录C。

  

**消融研究** 我们对我们的SUPERCORRECT进行了消融研究，并将结果展示在表2中。正如我们所看到的，传统的SFT（监督微调）的改进有限，与我们的HSFT（分层监督微调）相比，准确率落后了5%。基于我们的HSFT模型，我们进一步应用了一些自校正方法，如Reflexion（Shinn等人，2024），与我们的Cross-DPO（交叉模型直接偏好优化）进行比较。从结果中可以看出，我们的方法再次胜出，与Reflexion相比，准确率领先了7%。这些令人鼓舞的结果证明了我们的HSFT和交叉模型DPO的有效性。为了更好地理解我们有效的分层思维推理，我们在第5.3节的表3中提供了一个说明性示例。CoT（链式思维）提示方法显示出对“空集”的误解，因为它未能考虑到512个集合已经包含了空集。通过我们基于分层思维的推理（在附录A中表示为HT），我们可以看到模型意识到512个集合包含了空集。然而，它未能正确回忆起问题要求将空集包含在最终答案中，这是由于幻觉问题导致的。最终，我们的HSFT大语言模型能够正确解决这个问题，准确理解空集并避免幻觉问题。


SupperCorrect 打破思维瓶颈

MATH 数据集中的问题涵盖了七个广泛的主题，包括代数、计数与概率、中级代数、数论、几何、初等代数和预微积分。在我们的实验中，我们观察到每个主题的准确性差异较大。对于大多数大型语言模型（LLMs）来说，它们在代数和初等代数上表现较好，但在其他主题上，准确性往往有所下降，这可能是因为它们在这些主题上存在思维瓶颈。如图4所示，我们的SUPPERCORRECT在所有主题上都提高了推理性能。值得注意的是，对于那些原本对LLMs来说较难的主题，SUPPERCORRECT 表现出了更显著的改进，相比于模型已经掌握的主题。这是因为我们在跨模型DPO阶段利用了错误驱动的洞察力，打破了LLMs原有的思维瓶颈，从而启发它们使用新的技巧和方法来解决以前无法解决的问题。结果进一步证明，我们的SUPPERCORRECT 能够帮助打破原有的思维瓶颈，显著提高LLMs的推理能力，并缩小不同主题之间的性能差距。更多详细的推理和自我纠正结果可以在附录B和附录C中找到。

  

  

SuperCorrect在推理稳定性方面表现更佳。MATH数据集的测试集包含5000个问题，分为5个不同的难度级别。为了进一步评估我们方法的推理稳定性，我们从MATH测试数据集中额外抽取了300个难度级别为5（最难）的问题。我们通过重复实验256次进行定量分析，并计算了准确率的均值和方差，如图5所示。我们可以观察到，与基础模型相比，我们的SUPERCORRECT方法有助于实现更高的准确率均值。此外，我们的SUPERCORRECT方法显著降低了多次推理的准确率分布的方差。这些现象表明，我们的SUPERCORRECT方法能够有效提高困难推理问题的

  

  

  

#### DETAILED QUALITATIVE ANALYSIS

详细的定性分析

在本节中，我们对三种不同方法（包括CoT提示、我们第一阶段HSFT模型和我们的SUPERCORRECT）在易错推理步骤和推理结果方面进行了详细的比较。

  

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MGVmMzRlZDBjMTQ0YzFhYzhjZmUzN2U5YjQyMzViZDBfWFBSR1hJcjZZNG0xYTNCY3BTOGtJdVFEV0d0TG1ic0VfVG9rZW46V1JJbmJIcFBpb3pEb2d4dlpnemNidzA0bmpmXzE3NDg3NDgxNDQ6MTc0ODc1MTc0NF9WNA)

  

  

#### COMPARISON BETWEEN STEP-DPO AND CROSS-MODEL DPO

我们进行了Step-DPO与我们的Cross-model DPO之间的定性分析。我们选择Qwen2.5-Math-Instruct作为基础模型，并在该模型上应用Step-DPO以比较结果。需要注意的是，Step-DPO使用了CoT风格的提示，为了公平比较，我们为每个模型选择了最合适的提示方法。如表4所示，基于之前未解决的问题，Step-DPO能够定位错误的推理步骤并进行修正（例如，进一步识别另一个7的倍数），但它难以完全纠正这些错误。与Step-DPO相比，我们的方法不仅能够定位错误步骤，还能进行准确的自我修正，从而解决之前无法解决的问题。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=ZGIyYThjZDYxMDY2NmM2YjgwY2Y1MjEwMDg2YzMzMzlfRmZ4c1ZHc0pVblFNVllOMmdQb3VvQlUyenlWbnJBZ2dfVG9rZW46QlBQUGI3Wm11b09LMjd4Zml2UWN4TXl1bmFjXzE3NDg3NDgxNDQ6MTc0ODc1MTc0NF9WNA)

  

#### **QUALITY EVALUATION FOR TEACHER LLM GENERATED CONTENT**

5.5.1 检查者LLM的评估

我们讨论了检查者LLM的有效性，它进一步确保了教师LLM生成内容的质量。如表5所示，我们比较了三种不同教师LLM在三个数据集上生成的修正轨迹的正确性。与直接生成相比，应用检查者LLM显著提高了最终修正轨迹的质量。值得注意的是，对于已经能够生成高质量输出的具备先进能力的LLM，检查者LLM仍然显示出明显的改进。这些结果表明，检查者LLM显著提高了修正轨迹的准确性，尤其是在初始性能较低的数据集上

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=NmM2NDYyZWEyZTkzZWNiZjBhNmE0ZTk2NWRmOTQ4OTBfZ3NIMHNFRk85elhSMGFpM2RjUG9CdnpSYW5WZWp4clJfVG9rZW46SkFSS2JwdlFSb1RGZWZ4M2hkT2M5eUdrbkNiXzE3NDg3NDgxNDQ6MTc0ODc1MTc0NF9WNA)

  

  

5.5.2 直接生成质量分析

根据表5中的结果，在没有使用Inspector LLM的情况下，实验结果表明我们直接生成的修正轨迹已经具有较高的质量。我们将此归因于以下设计方法：

- **1. 利用前沿教师LLM**：为了确保教师LLM生成内容的质量，我们采用了最先进的LLM，特别是o1-mini作为教师LLM。这些模型能够识别逻辑缺陷和错误，并生成高质量的分析和修正，定量结果也证实了这一点。
    
- **2. 基于真实上下文的修正轨迹**：为了确保教师LLM生成的修正轨迹的准确性，如附录A所示，生成分析（ai）和修正（ci）的提示基于输入问题及其真实解决方案。这种方法将修正轨迹与真实解决方案作为上下文相结合，从而确保生成内容的准确性。
    

#### MORE ABLATION STUDIES

  

关于跨模型DPO的进一步分析，我们首先从数据集中抽取了500个错误解决方案，并使用o1-mini对数据集进行修正追踪，以此作为衡量模型对齐的基准。我们在HSFT阶段后对三个不同模型进行了实验，结果如表6所示。为了评估跨模型DPO的有效性，我们引入了两个指标：(1) 定位正确性：表示模型是否能够正确找到错误步骤。(2) 修正准确性：表示模型是否能够准确修正错误步骤。我们利用o1-preview作为评判工具，将跨模型DPO后各模型生成的修正追踪与基准进行比较。结果表明，跨模型DPO在所有模型上均显示出显著改进，证明了其有效性。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MTNjOTQyMzUwMzZjNTNhZDI1N2RhNDQ3NTY3YTkyNjdfbVlTWlZzcm4xOFAxZlczZmplb1MwQ0F4em84TjVJdnFfVG9rZW46UG1Ib2JSdUl3b0NmeFN4VjNEMWNpVFUxblZnXzE3NDg3NDgxNDQ6MTc0ODc1MTc0NF9WNA)

  

如表7所示，消融研究结果表明，我们的SuperCorrect方法能够泛化到不同的LLM架构，并在HSFT阶段和跨模型DPO阶段均表现出更优的性能，进一步验证了其有效性。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=YTRkOTgzMWQ5YjRhZjQxZmZjOTA1NTJiMWRiNDlkYTdfcjJQTTB2dExJeGFsQWZwblhQT0FOU1NSTmc2SWg4WHVfVG9rZW46UHBpemJudE8zb0psaGt4RjFlU2NjS294bktiXzE3NDg3NDgxNDQ6MTc0ODc1MTc0NF9WNA)

  

**关于提示风格的消融研究**

为了进一步评估我们精心设计的层次化思维模板的有效性，我们进行了定量实验，以展示提示风格和我们的层次化提示设计的影响。我们使用了五种提示风格：1) 思维链（CoT）；2) 思维链 + 层次化提示（无泛化步骤）；3) 思维链 + 层次化提示（含泛化步骤）；4) 我们的层次化提示（非XML格式）；5) 我们的层次化提示（XML格式）。此外，我们基于相同的10万道数学问题，为前四种提示风格构建了四个数据集。随后，我们在相同的训练设置下，使用这些数据集训练了Qwen2.5-Math-Instruct、Llama3.1-8B-Instruct和DeepSeek-Math-7B模型，并在数学数据集上评估了准确性。如表8所示，实验结果表明，与使用思维链作为基线相比，层次化推理显著提高了模型的准确性。此外，改变提示风格（例如转换为XML格式）对最终准确性的影响较小，进一步证明了我们层次化推理设计的有效性。尽管添加泛化步骤有助于模型更好地总结任务并提升其性能，但我们的实验结果表明，在HSFT阶段，性能提升的主要贡献来自于我们设计的层次化推理风格。

  

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MDAwMjhhYzIyMzA2Yjk0ZjZjNGRmNTQ5ZTNmZWExYWRfN1pSOHkzV3luUlJQQVBqbXRtTHhmNVJnZnZPcnFSalNfVG9rZW46WThCbWJoOHpNb1Q5R2x4b3dnNWMwTldzblBoXzE3NDg3NDgxNDQ6MTc0ODc1MTc0NF9WNA)

  

  

  

  

### CONCLUSION

在这篇论文中，我们提出了SUPERCORRECT，一个新颖的两阶段框架，显著改进了语言模型的推理和反思过程。在SUPERCORRECT中，我们提出了基于层次思维的微调方法，使大语言模型（LLMs）能够生成更细粒度的推理思维，并引入了跨模型协作的DPO（Direct Preference Optimization）方法，通过跟随教师的修正轨迹来增强学生LLMs的自我修正能力。大量的实验一致证明了我们的方法相较于之前方法的优越性，在MATH和GSM8K基准测试中，分别超越了强大的DeepSeekMath-7B模型5.3%∼7.8%和Qwen2.5-Math-7B模型6.3%∼15.1%。在未来的工作中，我们将把这个新框架推广到更大的模型和更复杂的数据集上。