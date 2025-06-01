---
title: "FollowBench: A Multi-level Fine-grained Constraints Following Benchmark for Large Language Models"
date: 2025-03-27T11:30:03+00:00
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

指令遵循能力对于大语言模型（LLMs）处理各类实际应用至关重要。现有基准测试主要聚焦于评估纯响应质量，而非判断响应是否遵循指令中陈述的约束条件。为填补这一研究空白，本文提出FollowBench——一个面向LLMs的多层次细粒度约束遵循基准测试。FollowBench全面涵盖了五种不同类型的细粒度约束（即内容、情境、风格、格式和示例）。为实现对不同难度下约束遵循的精确评估，我们引入了一种多层次机制，在每提升一个级别时向初始指令逐步添加单个约束。为评估LLMs输出是否满足每个独立约束，我们提出利用约束演化路径提示强LLMs，以处理具有挑战性的开放式指令。通过在FollowBench上评估13个闭源和开源的主流LLMs，我们揭示了LLMs在指令遵循方面的不足，并指出了未来工作的潜在方向。相关数据和代码已公开于https://github.com/YJiangcm/FollowBench。

### Introduction

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MGQyMTkyZTY5NWMwZGRiMDQ0N2E2NmMyYWFhYmNlZmNfdVBYaDRMZnNHSFhXWVZZNm1iN2dUME8xMlpzZE5LcUVfVG9rZW46TEtkWGJKd3Ztb05rS0V4a3FLSmM2OGZNbmRnXzE3NDg3NTczMzI6MTc0ODc2MDkzMl9WNA)

大型语言模型（LLMs）（Brown 等，2020；OpenAI，2022）在互联网规模语料库上进行预训练后，已展现出生成流畅且逼真文本的能力。然而，现实场景中的人类指令不仅要求模型生成的文本具有高度的自然性，还需遵循特定的约束条件（Yang 等，2023）。例如，模型可能需要推荐十本专门用中文撰写的书籍（图1），或者被期望生成具有特定语气的回应。

当前评估模型指令遵循能力的主流范式依赖于人工标注者或强对齐大语言模型，从帮助性、相关性、准确性、深度性、创造性和细节度等维度评判响应质量（Wang等，2023a；Li等，2023；Zheng等，2023；Xu等，2023）。然而现有研究仍存在双重局限：其一，忽视了指令内部细粒度约束这一评估指令遵循能力的核心客观标准。尽管已有基准测试针对语义限制（Chen等，2022）和复杂格式（Tang等，2023）等单一约束类型展开严格考察，但缺乏跨多约束类别的系统性分析；其二，鲜有基准测试考虑通过约束数量调控指令难度，这导致难以精确量化大语言模型的指令遵循程度。为此，我们的研究问题是：如何系统且精确地评估大语言模型的指令遵循能力？

本文构建了FollowBench，一个多层次细粒度约束遵循基准测试。FollowBench全面涵盖了现实场景中的五种不同类型约束，即内容（即对响应内容的明确限制）、情境（即问题中添加的特定情境/背景信息）、风格（即响应风格要求）、格式（即响应格式要求）以及示例（即示例模式识别与遵循）。为精确评估大语言模型遵循指令的难度程度，如图1所示，我们提出了一种新颖的多层次机制，该机制在每提升一个级别时向初始指令逐步添加单个约束。该多层次机制使我们能够精确定位大语言模型无法遵循指令的难度级别，从而更精确地估计大语言模型指令遵循能力的上限。总体而言，FollowBench由来自50多个自然语言处理任务的820条精心设计的指令组成，包括封闭式和开放式问题。为评估目的，我们提出了一种混合评估方法，包含基于规则和基于模型的解决方案。给定大语言模型的输出，这两种解决方案均判断输出是否满足指令中的每个约束。基于规则的解决方案侧重于封闭式指令，而基于模型的解决方案则应用于开放式指令。对于基于模型的解决方案，我们不仅将当前指令和响应作为输入，还在输入提示中额外提供指令的演化过程，以帮助大语言模型评估者更好地理解每个独立约束。数据构建和评估过程均经过人工验证。

在实验中，我们提出三项指标评估13个主流闭源与开源大语言模型在FollowBench上的指令遵循能力，主要发现如下：（1）所有测试模型性能均随指令难度级别（约束数量增加）显著下降；（2）尽管闭源模型（如GPT4与GPT-3.5）平均仅能连续满足约三个约束，其表现仍显著优于所有开源模型；（3）特定约束类型（如情境约束与示例约束）对模型的挑战性尤为突出；（4）超越知识与推理等传统能力维度，指令遵循能力可为全面评估大语言模型水平提供新视角。

### Related Work

#### Instruction-Following Language Models

先前研究发现，通过使用由语言指令及其预期结果组成的标注“指令”数据进行微调，大语言模型（LLMs）能够有效遵循一般语言指令（Weller等，2020；Sanh等，2021；Mishra等，2022；Jiang等，2024）。为了增强大语言模型对现实场景中用户复杂多样意图的理解，诸如ChatGPT（OpenAI，2022）和GPT-4（OpenAI，2023）等模型在广泛的人工编写指令和任务类别上实施了指令微调。近期研究（Zheng等，2023；Xu等，2023；Jiang等，2023）已转向自动生成高质量数据，以提升大语言模型的指令遵循能力，从而应对人工标注劳动密集型带来的挑战。

#### Evaluation for Instruction Following

现有研究在评估大语言模型（LLMs）对特定任务的指令遵循能力方面已取得多项进展。Tang等人（2023）聚焦于评估LLMs生成复杂结构化表格数据的能力，涵盖文本、HTML和Latex格式。他们首先从现有自然语言处理基准和网站中收集表格数据，随后基于这些数据构建指导性指令。Chen等人（2022）则评估LLMs是否能够遵循特定知识密集型生成指令，其方法为先提供示例列表（如英国体育明星名单），随后设置与示例相矛盾的约束条件（如不得提及任何运动员）。这些基准测试仅能展示LLMs在特定类型指令下的遵循能力。相比之下，FollowBench全面涵盖了包含五种不同类型细粒度约束的多难度层级指令，能够为现有LLMs提供全面且精确的指令遵循能力评估。关于LLM评估的更多细节，可参考近期综述文献（Chang等人，2023；Wang等人，2023b）。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=N2NmNDU3ZWZiOWY4N2IwMGRhNGJkY2RkNzExZmQzNDNfTlNlb2JRRWtFek84blk1Tk4wRElKeW9kWm1SY21UZjNfVG9rZW46VW51aGJQeGtvb0Z6NG94aXZKQ2M4TjllbjRmXzE3NDg3NTczMzI6MTc0ODc2MDkzMl9WNA)

### FollowBench

如表1所示，FollowBench包含五种不同的细粒度约束类别：内容、情境、风格、格式和示例。每个类别均由来自不同自然语言处理任务的指令组成。与以往基准测试不同，我们引入了一种多层次机制，该机制在初始指令基础上逐步添加约束（见图2示例），生成包含1至5个约束的指令集。在本文后续部分中，我们使用“第n级”表示包含n个约束的指令。值得注意的是，约束添加方式针对每个任务在其相应约束类别内进行了精心设计。该多层次机制使我们能够精确定位大语言模型无法遵循指令的难度级别，从而更精确地估计大语言模型指令遵循能力的上限。总结而言，我们将在§3.1节介绍FollowBench的数据构建过程，包括细粒度约束和多层次机制。在§3.2节中，我们提出了一种与多层次机制无缝集成的评估协议，包含三项指标。

#### Data Construction

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MTU0MjQwYzdjM2FhNjE0YzVhMDZhNzBiMGJmOGUzODJfOE5Belc2QTByNlBmSG1WSTNXWERUNng0aTQ4d1MzYVlfVG9rZW46SWFodWJpYTcyb09DWHR4cUdmdWNTM1NYblZnXzE3NDg3NTczMzI6MTc0ODc2MDkzMl9WNA)

内容约束

内容约束指对响应内容的深度或范围施加明确限定条件的约束类型。如图2所示案例，其对检索对象设置了特定标准。确保大语言模型（LLMs）遵循内容约束已成为受控文本生成领域的关键挑战（Zhang等，2022），这要求模型既能理解特定规范，又能使响应适配预设条件（Chen等，2022）。为此，我们从以下任务中采集数据：(1) 复杂信息抽取——旨在从给定文本中检索特定对象的详细信息；(2) 语言约束文本生成——需在遵循指定约束的前提下生成流畅的切题内容；(3) 开放式问答——源自开源平台等真实场景以防止数据泄露风险。随后，我们通过每次向原始指令添加一项内容约束构建多层次指令。约束引入方式因任务而异（详见附录A.1）：针对复杂信息抽取任务，逐步缩小待提取信息的范围；对于语言约束文本生成，从WordNet（Miller，1992）和Wikidata（Vrandečić与Krötzsch，2014）引入额外限制；在开放式问答任务中，采用GPT-4等先进大语言模型基于给定指令生成新增约束的新指令。虽然大语言模型输出仅作为参考，我们仍人工筛选最具相关性与挑战性的合成指令以确保数据质量。

情境约束

情境约束指通过施加特定情境或背景信息，从而隐式引导响应答案适切性的约束类型。如图2所示案例，在寻求定制化建议时必须阐明具体情境。另一典型案例是要求大语言模型（LLMs）在特定环境下模拟不同角色的角色扮演任务（Shanahan等，2023；Wang等，2023c），此类约束能为用户提供更细腻的交互体验。情境约束迫使LLMs超越简单的事实检索或表层信息整合，需要具备对情境的精细理解能力、动态适应能力以及复杂推理能力（Yao等，2022；Liu等，2023）。除现实生活类问题外，我们还纳入包含数学应用题、时空推理和代码生成的复杂情境推理任务。这些任务均要求模型在给定情境中解析并解决问题，完全符合情境约束的定义。我们首先从上述来源采集初始指令，随后通过逐步增补情境信息的方式人工构建多层次指令（详见附录A.2）。

风格约束

风格约束旨在控制输出的风格变化，以实现特定的风格目标，如语调、情感、正式度和同理心等（Tsai等，2021），如图2所示。风格约束对大语言模型（LLMs）的挑战在于其对语言细微差别的复杂理解与适应能力，确保输出在上下文中的恰当性与风格一致性（Smith等，2020；Cheng与Li，2022）。我们从开放式问答数据集和在线平台中收集初始指令，随后利用LLMs的上下文学习能力，构建具有多层次风格约束的指令。提示模板如图8所示。最终，由人类专家对LLMs生成的输出进行审查与优化。

格式约束

格式约束指对生成内容的结构、语言或输出形式进行规约的限制类型。如图2所示案例，其不仅限定回答字数，还要求以表格形式呈现响应内容。此类约束要求大语言模型（LLMs）具备对语言结构的深度理解能力，使其能根据多样化且通常复杂的规范灵活调整输出（Zhao等，2023）。最新研究表明，即使是性能最优异的LLMs，在生成表格、JSON、HTML或LaTeX等复杂结构化输出时仍存在困难（Tang等，2023）。为涵盖多样化格式约束，我们首先从文本转表格生成和开放式问答等广泛领域收集初始指令，随后利用高性能LLMs逐步添加从字数限制、层级结构到特殊语言特征及输出媒介等维度的格式约束（提示模板见图9）。最终由人类专家对合成指令进行严格校验与优化。

示例约束

大语言模型（LLMs）展现了令人惊叹的少样本学习能力（Brown等，2020），使其能够通过识别提示中提供的少量示例快速适应新查询。然而，少样本学习的鲁棒性，即在引入“噪声”示例后LLMs是否仍能遵循正确模式，尚未得到充分探索。因此，我们提出了一种新的约束类别——示例约束，以评估LLMs的示例模式识别和遵循能力。我们基于PromptSource（Bach等，2022）自动构建了具有多层次示例约束的指令，其中第n级指令在输入中包含n-1个噪声示例。详细信息见附录A.3。

混合约束

针对上述五种约束类别，我们通过顺序添加同类约束构建了多层次指令。然而，现实应用场景中的指令往往需要同时实施多种约束类型。因此，我们将混合约束定义为不同约束类别的组合。例如在文本编辑任务中，既可能需要增补内容（内容约束），又需调整输出格式（格式约束）。此外，我们还筛选了三类天然适合构建混合约束的任务：摘要生成、机器翻译和故事生成（详见附录A.4）。多层次混合约束指令通过以下方式构建：

1. **格式约束**：规定答案生成的结构（如表格/JSON）
    
2. **内容约束**：要求生成文本必须包含/排除特定关键词
    

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MWJiYjRkYThhYmQzZTUxY2QyMzczOTA4NDc5NGYzZDFfWVdDUUgyd3hQaUF1NXd1WXloN2dUdXVCUWxOSFhxQURfVG9rZW46Q0N6SWI1SGlVb2t5TFR4akFUU2MwalpUbkRjXzE3NDg3NTczMzI6MTc0ODc2MDkzMl9WNA)

数据质量控制

为确保FollowBench的数据质量，我们采用双层级验证机制对每条指令进行校验。两名标注员独立评估：(1) 指令与其指定约束类别的适配性；(2) 指令中新增约束的有效性。若出现评估分歧，则由第三名标注员介入进行详细复核以确保结论一致性。

我们对FollowBench的全面性与多样性进行分析，该数据集共包含820条指令。为保持数据多样性，我们严格控制任意两条初始指令间的ROUGE-L相似度分数低于0.7。图3展示了FollowBench指令的动词-名词结构分布，其中内环为使用频次最高的20个动词，外环则呈现这些动词对应的前4个直接名词宾语。

#### Evaluation Protocol

鉴于FollowBench中近半数指令为开放性问题且无参考答案，设计基于规则的评估程序极具挑战性。为解决该问题，受（Gilardi等，2023；Huang等，2023）启发，我们提出采用高性能大语言模型（LLMs）作为评估者的模型化方案。现有研究通过提示LLMs综合考量实用性、相关性和细节程度等多维指标（Li等，2023；Zheng等，2023）来判定响应质量。为客观准确地评估模型遵循约束的能力，我们设计了如图4所示的多层次感知提示模板。

我们采用指令演化过程呈现的创新方法，而非简单要求大语言模型（LLMs）判断约束满足情况。通过分阶段展示指令的迭代过程，引导LLMs精准识别每一层级新增的约束条件。这种渐进式暴露约束演变路径的策略，能够提升模型对细粒度约束的辨识能力，§5.1的消融实验证实了该方法的有效性。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=NGM0NzhhZmIzMTVkN2NlODg5NjNkMDVkM2JmNjkyNDVfRThubkFPSm95Rm8yNVVqaHZnWnRLOEN6TFFCcGdnOUFfVG9rZW46UG1mbGJ6YXRKb1h1U3N4WlVDMmNRcVlFbmRmXzE3NDg3NTczMzI6MTc0ODc2MDkzMl9WNA)

  

此外，我们提出了三个新颖的指标来评估大语言模型（LLMs）的指令遵循能力。对于包含n个约束（第n级）的指令，我们使用基于规则的程序或LLM评判器（参见表1）来判别模型的响应是否满足指令中的每个约束。在第n级，给定一组m条指令，我们定义硬性满足率（HSR）和软性满足率（SSR）如下：

$$\text{HSR} = \frac{1}{m} \sum_{i=1}^{m} \prod_{j=1}^{n} s_{j}^{i}$$

$$\text{SSR} = \frac{1}{mn} \sum_{i=1}^{m} \sum_{j=1}^{n} s_{j}^{i}$$

其中， $$s_{j}^{i} = $$ 表示第i条指令的第j个约束被满足，否则 $$s_{j}^{i} = $$ 。HSR衡量的是所有约束在单条指令中完全满足的平均比率，而SSR计算的是所有指令中单个约束的平均满足率。如§3所述，我们通过逐步向初始指令添加五个约束来构建FollowBench，这使得我们能够精确定位LLMs在哪个难度级别上无法遵循指令。因此，我们提出了一个称为**连续满足级别（CSL）**的指标，用于估计模型从第1级开始能够连续满足的级别数量：

$$\text{CSL} = \frac{1}{g} \sum_{i=1}^{g} \arg\max_{l} \left( l \times \prod_{n=1}^{l} S_{n}^{i} \right)$$

其中，g是初始指令的组数， $$S_{i}^{n} = 1$$ 表示第i条指令在第n级的所有约束均被满足，否则 $$S_{i}^{n} = 0$$ 。

  

### Experiments

本节首先在§4.1中介绍实验设置，随后从两个关键维度展示主要实验结果：§4.2中的难度级别和§4.3中的约束类别。

#### Experimental Setup

我们评估了13个主流大语言模型，包括GPT-4-Preview-1106（OpenAI，2023）、GPT-3.5-Turbo-1106（OpenAI，2022）、千问系列Qwen-Chat 72B/14B/7B（Bai等，2023）、LLaMA2-Chat 70B/13B/7B（Touvron等，2023）、WizardLM-13B-V1.2（Xu等，2023）、Vicuna 13B/7B-V1.5（Zheng等，2023）、百川2-Chat 7B（Baichuan，2023）以及ChatGLM3-6B（Du等，2022）。实验采用OpenAI官方接口调用GPT系列模型，其余开源模型均从其官方代码库获取。推理过程中，温度参数设为0以确保输出确定性，最大生成长度设置为2048 tokens，其余参数保持默认值。为评估大语言模型的多语言指令遵循能力，我们还在附录D中构建了中文版测试集FollowBench-zh。

#### Level-categorized Results

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MGZiZmZlZmY0ZjJhMDJmNzQ0OWE3NzYwNGE4MjA4YzZfNUVydDJvYkI1bm45YW9UV0YzeVA3cU1hYmpLbG1TNGhfVG9rZW46SFlLZmJhYXZRb3pTOXN4ZmFGd2N1RFVSbkViXzE3NDg3NTczMzI6MTc0ODc2MDkzMl9WNA)

表2全面比较了不同模型在L1至L5五个难度级别上的表现，各约束类别的详细结果见附录B。总体而言，随着难度级别从L1升至L5，几乎所有模型的性能都呈现下降趋势，这与更高难度级别所增加的复杂性或严格性要求相吻合。此外，参数量更大的模型普遍表现优于较小规模的模型。

然而值得注意的是，规模扩展法则对LLaMA2-Chat-70B模型的适用性相对有限。如附录B所示，虽然该70B参数版本在情境约束类任务上确实优于13B版本，但在格式约束和混合约束两个类别中却表现出相对劣势。

更重要的是，闭源模型（即GPT-4与GPT-3.5）与开源模型之间存在显著性能差距。就连续满足级别（CSL）而言，可推知GPT-4和GPT-3.5的指令遵循上限约为初始指令附加3个约束条件（第3级），而开源模型普遍上限约为2个约束条件（第2级）。这一显著差异印证了专有模型更优的指令遵循能力，可能源于更优质的数据或强化学习人类反馈（RLHF）等优化策略（Ouyang等，2022）。值得注意的是，即使最先进的模型目前也仅能稳定遵循约三个约束条件的指令，这表明该领域仍有巨大改进空间。

#### Constraint-categorized Results

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=Y2FjNjA5ZGFiMDhiNjlhZGQ0ODExOGE2ODUwZTIxNGJfZHNZTlo4SXNqOTFpQU9EbXVaM0NzaEVQZmswekRiQzBfVG9rZW46SWJTSmJnWVg4b3haYW14MTZ5dWNhMUFabmtkXzE3NDg3NTczMzI6MTc0ODc2MDkzMl9WNA)

如图5所示，我们通过不同约束类别评估各模型，以单维度简明展示大语言模型的指令遵循能力。值得注意的是，GPT-4与GPT-3.5在所有约束类别中均超越开源模型，其中在内容约束、情境约束、示例约束和混合约束方面优势尤为显著。此外，多数模型在风格约束条件下均展现出值得肯定的性能表现。在众多模型中，GPT-4、GPT-3.5和LLaMA2 Chat-70B表现最为突出。研究数据显示，风格适应能力是大多数模型的优势领域，这一特征预示着该技术在现实应用场景中具有重要实用价值。然而，"示例约束"和"混合约束"对多数模型构成了挑战。尽管GPT-4在该领域保持领先，但其得分仍显著低于其他类别。以"示例"类别为例，我们通过引入具有不同自然语言模板的"噪声示例"来评估大语言模型（LLMs）的指令跟随能力。观察到的性能下降主要源于LLMs在基于上下文的学习场景中处理此类噪声输入的训练不足。通常，LLMs是在干净统一的数据集上进行微调的，这未能使它们充分具备筛选和忽略无关或误导信息的能力。当面对现实世界数据的复杂性时，这一局限尤为明显。我们的研究结果既揭示了这些约束条件的复杂性，也指明了潜在的改进方向。

### Analysis

本节包含以下内容：通过消融研究验证了提示模板在基于模型评估中的有效性（§5.1）；指令遵循能力与其他大语言模型能力的对比分析（§5.2）；失败一致性的检验（§5.3）；以及多种解码策略的探究（§5.4）。此外，附录C中提供了案例研究以供进一步分析。

#### Ablation Study of Model-based Evaluation

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=OWJmMTcxNzUxNzQzNzRlMjQyYzE4ZTAxM2NiOGJkZWVfYVZublpTbFpkMzkxczNDeTN1cnc3eXVKakhqNlowZXRfVG9rZW46UFpsZ2J0ZjY1b3ZUenp4QUhyWmM0YTVCblVlXzE3NDg3NTczMzI6MTc0ODc2MDkzMl9WNA)

  

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=ZWZiOGQ3NWUxOGZmNTRmNjA5ZjQzNzAwN2YxZmVhMTBfSmVUQlV5WnZ1dkJadllkV1FDdnVUcGRpc1U2UHRsazRfVG9rZW46Qm53Y2JGYk1Yb1BTMzN4Ukw1YWN6NW9ZbmdlXzE3NDg3NTczMzI6MTc0ODc2MDkzMl9WNA)

我们从需要大语言模型评估的案例中随机抽取了100个样本，涵盖五种约束条件、五个不同难度等级和四种不同模型，以确保样本的全面代表性。随后，我们邀请三位专家级人工标注员评估模型响应是否满足每个案例中的所有约束条件，并以多数表决作为最终的人工标注结果。如表3所示，我们的提示模板（图4）与专家人工评估的一致性达到了88%，甚至超过了人工专家之间85%的内部一致性。值得注意的是，当从提示模板中移除多层次约束的演化过程后，一致性下降了9%。这凸显了指令演化细节描述在提升大语言模型辨别精度中的关键作用。作为对比，我们还采用了Vicuna（Zheng等，2023）的提示模板，这是一种评估响应整体质量的标准提示。该模板要求大语言模型为每个响应打分（0-10分），我们将得分高于5.0的响应视为满足指令的所有约束条件。这种方法与人工评估的一致性为67%。这种差异凸显了评估指令遵循能力与评估响应整体质量之间的本质区别。

#### Instruction Following vs. Other Abilities

表4展示了代表性大语言模型（LLMs）在不同能力维度的对比分析，这些能力不仅限于指令遵循（FollowBench），还包括整体响应质量（AlpacaEval）、知识掌握（MMLU）和推理能力（BBH）。通过分析可以得出，我们的FollowBench为全面评估大语言模型提供了新的视角。例如，虽然WizardLM13B-V1.2在整体响应质量上超越GPT-3.5，但其指令遵循能力明显落后；同样地，VicunaV1.5在知识和推理领域优于LLaMA2-Chat，却在指令遵循任务上表现欠佳。

#### Does Failure at Lower Level Necessarily Lead to Failure at Higher Level?

针对包含五个难度等级的指令集，若某模型响应未能满足第n级（n取值1至4）约束条件，则将其失败一致性定义为该响应在任意高于n的后续等级中同样无法满足约束的概率。结合表2与表5数据可见，指令遵循能力更强的模型可能表现出更低的失败一致性。潜在原因在于：性能更优的模型对指令中约束数量的敏感性较低，因而能更好地适应并满足随难度递增的要求。这种适应性意味着，尽管模型可能在较低难度等级出现失误，但仍能应对更高难度的需求，从而导致失败一致性指标的下降。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=Njg5MmVjNDE2ODQ5MTUyOTY2ZjU2NzMxZWI3YmU3OTRfNWtZUXcyWGVyUWR6eHFPYzRWZE1sbnpLV0d6ejN0d1NfVG9rZW46Qkg4TGJiTTRPb0JJZkJ4UXBZbWMzR05iblhlXzE3NDg3NTczMzI6MTc0ODc2MDkzMl9WNA)

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=NjFhMmI5ZGIxNjA3ZDVhOTFjNTRmNDlmMjVhOGRmZjRfd2dMckloaG91czA3QWtJa1QxRVQ2Mm5jY1FFNG40SElfVG9rZW46Q3BzZGJoOGdab3l6SDl4dUZ5cGNPSzJqbm5iXzE3NDg3NTczMzI6MTc0ODc2MDkzMl9WNA)

#### Does Different Decoding Strategies Affect the Instruction-following Ability?

  

  

在本节中，我们系统研究了不同解码策略（以温度参数τ为代表）对大语言模型（LLM）指令遵循能力的影响。温度参数τ是控制下一个词元采样分布锐度的常用参数：

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=OGQ5NzRjMTk0ZDM4ZTIzNzQ3MzEwNjdjODgwYjMxZDBfdnRsdnNidGtyem5OQnpvRERXMXU5eE9NajdDbEU2a05fVG9rZW46RldMaGJLVXc4b1g5STB4d292UmNBVjVRbmtjXzE3NDg3NTczMzI6MTc0ODc2MDkzMl9WNA)

其中zw表示词w的logit值，V为词汇表。较低的τ值会产生更一致的输出，而较高的τ值则会生成更多样化和创造性的结果。如图6所示，温度参数τ对所有四种模型的指令遵循能力都有显著影响。最佳表现区间似乎位于中间值范围——既能保持足够的变化性以捕捉复杂指令的细微差别，又不至于让模型偏离主题。这种平衡行为确保模型始终保持在预期语境中，生成的输出既紧密贴合给定指令，又在必要时保留适度的创造性发挥空间。

### Conclusion

本文提出FollowBench——一个专为评估大语言模型指令遵循能力而设计的多层次细粒度约束遵循基准。该基准涵盖5个细粒度约束类别和50余项自然语言处理任务，采用创新的多层次机制精准测算指令遵循能力上限。我们进一步提出包含三项指标的评价方案，与多层次机制实现无缝衔接。通过对13个主流大语言模型的系统测试发现：GPT-4与GPT-3.5展现出显著性能优势，而当前大语言模型的指令遵循能力仍存在巨大提升空间。

### Limitations

本研究虽贡献了重要见解，但需认识到若干值得关注的局限性：首先，当前研究仅限于单轮交互，旨在提供受控评估环境。未来研究可扩展至多轮对话（Kwan等，2024），以全面评估大语言模型在动态长对话中的指令遵循能力。其次，实验采用的模型评估框架虽严谨，但依赖提示工程，存在固有缺陷。尽管我们精心筛选了高性能提示模板，进一步优化的可能性仍会影响现有评估指标。最后，我们暂未针对已识别的指令遵循缺陷提出具体解决方案。未来研究可通过基于FollowBench基准的模型微调来增强指令遵循性，该路径有待后续研究系统考察不同交互复杂度下的大语言模型能力。

### Ethics Statement

本文旨在系统且精准地评估大语言模型（LLMs）遵循自然语言指令的能力。然而，必须认识到恶意指令可能诱导模型生成有害或不恰当的输出。因此，在评估LLMs的指令遵循能力时，确保安全且负责任的操作至关重要。在FollowBench中，每条数据都经过细致的人工审查流程，以识别并消除任何潜在的恶意指令或冒犯性内容。这种严谨的方法彰显了我们维护安全且合乎道德的评估框架的承诺。


论文翻译：https://dppemvhuzp.feishu.cn/docx/OJwadgvMVoppYQxEb4VcZVjSnBd?from=from_copylink